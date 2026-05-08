# IRCTC Product Design Sprint — Part B: Feature Specifications

**Last Updated**: May 8, 2026
**Design Sprint Phase**: Discovery → Solutions → Prioritisation

---

## Table of Contents
1. [Feature Spec 1: Tatkal Virtual Queue System](#feature-spec-1-tatkal-virtual-queue-system)
2. [Feature Spec 2: Persistent Search Filters](#feature-spec-2-persistent-search-filters)
3. [Feature Spec 3: Seat Selection State Management](#feature-spec-3-seat-selection-state-management)
4. [Feature Spec 4: PNR Status Page Optimization](#feature-spec-4-pnr-status-page-optimization)
5. [Feature Spec 5: Modal Dialog Redesign](#feature-spec-5-modal-dialog-redesign)
6. [Feature Spec 6: Mobile Responsiveness Overhaul](#feature-spec-6-mobile-responsiveness-overhaul)

---

## Feature Spec 1: Tatkal Virtual Queue System

### Problem Statement
IRCTC receives 20–40 lakh concurrent requests at 10:00 AM when Tatkal quota opens, causing server overload, session crashes, and 40–60% booking failure rate. Users experience generic 500 errors with no feedback about queue position, wait time, or whether their request is being processed. This directly results in loss of ₹10+ crores daily in ticket sales and massive user frustration. **[Linked to Part A: Problem 1]**

### Proposed Solution
Replace the direct-hit booking approach with a **virtual waiting room**. When users visit the Tatkal section between 9:55–10:05 AM, they are automatically assigned a queue position. At 10:00 AM, the queue begins processing at a controlled rate (e.g., 500 users per minute). Each user sees their live queue position, estimated wait time, and a countdown timer. When their turn arrives, they get a **90-second booking window** to select seats and confirm. If the 90 seconds expires unused, the slot passes to the next user in queue.

**User Flow — Proposed:**
```
1. User navigates to Tatkal booking at 9:58 AM
   ↓
2. System shows: "Tatkal opens in 02:15 — You are in queue. Position: 4,281"
   ↓
3. At 10:00 AM queue begins processing
   ↓
4. UI shows live: "Position 4,281 → 3,100 → 1,205 → 89 → YOUR TURN"
   ↓
5. User's turn arrives → seat map loads (pre-populated from saved preferences)
   ↓
6. 90-second countdown starts; user selects seats and confirms booking
   ↓
7. Payment gateway opens → confirmation shown
   ↓
8. If 90s expires without action → slot released, user offered next available slot or refund
```

### Technical Implementation

**Backend Components:**
- **Redis-backed queue** using sorted sets
  - O(log N) insertion and polling performance
  - Atomic queue operations (no race conditions)
  - TTL on queue entries (auto-expire stale positions after 15 mins)
  
- **WebSocket server** (Socket.io or native WebSocket)
  - Real-time position updates pushed to each client
  - Handles 50K+ concurrent connections
  - Graceful fallback to polling (5s interval) for users with connectivity issues
  
- **Queue controller service**
  - Dequeues N users per second based on server capacity (configurable)
  - Manages 90-second booking window timers
  - Handles slot expiration and re-queueing logic
  - Tracks booking success/failure for each slot
  
- **Session token binding**
  - Each queue position tied to unique session token
  - Prevents double-booking and token reuse

**Frontend Components:**
- **Queue position counter** component
  - Real-time animated position updates
  - Estimated wait time (minutes:seconds)
  - Progress bar showing position progress
  - Mobile-optimized layout (full screen on mobile)
  
- **Background data pre-fetching**
  - While in queue, pre-fetch seat availability and train details
  - Reduce load time when slot arrives
  
- **90-second booking timer**
  - Visual urgency indicator (color changes: green → yellow → red)
  - Audio alert at 30s and 10s remaining
  - Auto-save draft booking if timer expires
  
- **Fallback flow**
  - If WebSocket disconnects, switch to polling
  - If queue system fails, fall back to direct booking (with rate limiting)

**API Changes:**
```
POST /api/v1/tatkal/queue/join
Request:
{
  "route": { "from": "DEL", "to": "BOM" },
  "class": "SL",
  "date": "2026-05-10"
}
Response:
{
  "queueId": "q_xyz123",
  "position": 4281,
  "estimatedWaitSeconds": 515,
  "sessionToken": "token_abc456"
}

GET /api/v1/tatkal/queue/status/:queueId
Response:
{
  "position": 1205,
  "estimatedWaitSeconds": 125,
  "status": "WAITING" | "YOUR_TURN" | "EXPIRED"
}

POST /api/v1/tatkal/booking/confirm
Request:
{
  "queueId": "q_xyz123",
  "sessionToken": "token_abc456",
  "seatSelections": [...],
  "passengerDetails": [...]
}
Response:
{
  "bookingId": "BK_123456",
  "confirmationNumber": "1234567890",
  "status": "CONFIRMED"
}

WebSocket Events:
- Emit: "tatkal:position:update" → { position, estimatedWait }
- Emit: "tatkal:slot:ready" → { slotStartTime, durationSeconds }
- Emit: "tatkal:slot:expired" → { message, nextAvailableSlot }
```

### Success Metrics
- **Tatkal booking completion rate**: Increase from 40% → 70% during peak hours
- **Server error rate at 10:00 AM**: Reduce from 35–40% → less than 5%
- **User satisfaction**: Tatkal-related complaints reduce by 80% within 30 days
- **Queue position accuracy**: 99.9% accuracy in queue position display
- **Slot fulfillment rate**: 85%+ of users complete booking within their 90s window
- **Revenue impact**: Recover ₹8–10 crores daily in lost ticket sales

### Edge Cases and Constraints

**Railway Backend Constraints:**
- Railway seat reservation API must still be called at final booking step (unavoidable latency)
- Seat blocking timeout on Railway backend (typically 5 mins) — queue slot must complete before timeout
- Railway API rate limits must not be exceeded during slot processing

**Technical Constraints:**
- Redis failover: If Redis fails, fall back to in-memory queue (loses persistence but maintains service)
- WebSocket limits: Browser limits typically 6 concurrent connections per domain
- Mobile connectivity: 2G/3G users must use polling (higher latency acceptable for fairness)

**User Scenarios:**
1. **User closes browser mid-queue**: Session token expires after 15 mins; queue position released
2. **User's 90s window expires**: Slot released, system offers option to rejoin queue or accept next available slot
3. **Internet drops during 90s booking window**: Session preserved for 30s; user can resume if they return
4. **Multiple queue joins by same user**: Session token ensures only one position per user per Tatkal window
5. **Queue system failure**: Fallback to direct booking with aggressive rate limiting to prevent cascade overload

---

## Feature Spec 2: Persistent Search Filters

### Problem Statement
Train search filters (Class, Departure Time, Availability, etc.) fail to apply correctly ~30–40% of the time, and when users navigate back to search, previously selected filters are lost. On mobile, filter application failure reaches 50%+. This forces users to manually scroll through full result lists, increasing search time by 2–3x and leading to 20% higher booking abandonment. **[Linked to Part A: Problem 2]**

### Proposed Solution
Implement **filter state persistence** using three layers: (1) URL query parameters, (2) session storage, and (3) server-side session. Filters are applied **synchronously** on the frontend with immediate visual feedback (loading state → filtered results). Selected filters are persisted to the URL so back-button navigation preserves filter state. Filter state is also cached server-side for 24 hours to support session resumption.

**User Flow — Proposed:**
```
1. User searches: "DEL → BOM, 10/05/2026"
   ↓
2. Results load showing 15 trains (unfiltered)
   ↓
3. User clicks filter: "Available seats only"
   ↓
4. URL updates to: ?filters=availability:available
5. Results update instantly (no page reload)
   ↓
6. User clicks second filter: "Sleeper class"
   ↓
7. URL updates to: ?filters=availability:available,class:SL
8. Results narrow to 4 Sleeper trains with available seats
   ↓
9. User clicks back button
   ↓
10. URL preserved; filters reapply automatically
11. Same 4 filtered results shown
```

### Technical Implementation

**Frontend State Management:**
- Filter state stored in **Redux/Vuex** (in-memory)
- State synced to **URL query parameters** (persistent across browser sessions)
- State also persisted to **sessionStorage** as backup

**Backend Session:**
- When filters applied, POST to `/api/v1/search/session` with filter selections
- Server stores filter state in session cache (Redis) with 24-hour TTL
- User can resume search from 24h earlier with same filters applied

**Filter Application Logic:**
- Frontend **debounces** filter changes (300ms wait before re-applying)
- On filter change, **immediately** show loading state in results
- Fetch filtered results via API: `GET /api/v1/trains?from=DEL&to=BOM&date=2026-05-10&filters=class:SL,availability:available`
- Results return within 500ms; results update with fade-in animation

**API Changes:**
```
GET /api/v1/trains/search
Query parameters:
- from, to, date (required)
- filters (optional): "class:SL,availability:available,departure:morning"
- limit, offset (pagination)

Response:
{
  "trains": [...],
  "appliedFilters": {
    "class": ["SL"],
    "availability": ["available"],
    "departure": ["morning"]
  },
  "availableFilters": {
    "class": ["SL", "3AC", "2AC", "1AC"],
    "availability": ["available", "waitlist"],
    "departure": ["morning", "afternoon", "evening", "night"]
  },
  "resultCount": 4
}

POST /api/v1/search/session
Request:
{
  "from": "DEL",
  "to": "BOM",
  "date": "2026-05-10",
  "filters": { "class": "SL", "availability": "available" }
}
Response:
{
  "sessionId": "sess_123",
  "expiresAt": "2026-05-09T10:00:00Z"
}
```

**Frontend Filter Components:**
```
- FilterPanel component (re-renders only when filters change)
- FilterCheckbox component (debounced onChange)
- ResultsList component (listens to filter state; re-renders filtered trains)
- FilterBadges component (shows active filters; allows quick removal)
```

### Success Metrics
- **Filter reliability**: 95%+ filter application success rate (vs. current 60–70%)
- **Filter persistence**: 99%+ filter state preserved on back-button navigation
- **Search time reduction**: Average search-to-booking time reduced from 4–5 mins → 2 mins
- **Mobile filter success**: Increase from 50% → 90%
- **Booking completion**: Increase by 15% due to easier filter-based selection
- **Support tickets**: Reduce filter-related complaints by 75%

### Edge Cases and Constraints

**Browser/Device Constraints:**
- URL length limits: Modern browsers support up to 2KB URLs; filter params estimated at <500 bytes
- sessionStorage quota: 5–10MB per domain (typical filter state < 1KB)
- Private browsing mode: sessionStorage not available; fall back to URL params only

**Filter Combination Edge Cases:**
1. **Mutually exclusive filters**: "Available" + "Waitlist" should show combined results (not error)
2. **No results after filtering**: Show message "No trains match your filters. Try removing some filters."
3. **Filter timeout**: If filter API takes >3s, show cached previous results with "results may be outdated" warning
4. **Concurrent filter changes**: User rapidly clicks 3 filters in quick succession — debounce ensures only last combination is applied

**Server-Side Constraints:**
- Filter state stored in Redis with 24h TTL; after expiry, filters cleared (user must re-apply)
- Large result sets (1000+ trains): Implement server-side pagination with cursor-based offset

---

## Feature Spec 3: Seat Selection State Management

### Problem Statement
During booking, when users select seats on the seat map and proceed to passenger details, the selected seats do **not match** the original selection ~40–50% of the time. On mobile, this rises to 65–75%. Users must navigate back to re-select, losing critical booking time. This leads to 20–30% booking abandonment on mobile and forces users to competitor platforms with more reliable booking flows. **[Linked to Part A: Problem 3]**

### Proposed Solution
Implement **server-side seat locking** with client-side state synchronization. When a user selects seats, the selection is (1) stored in memory on the client, (2) immediately persisted to the backend with a lock token, and (3) verified on every page transition. Session state is fully reconstructed from the server during navigation, ensuring no data loss on back-button or page reload.

**User Flow — Proposed:**
```
1. User selects seat 32-Lower and 33-Upper on seat map
   ↓
2. Both seats highlight locally; API POST sent to /api/v1/booking/seat/lock
   ↓
3. Backend verifies seats available, locks them, returns lockToken: "tk_xyz"
   ↓
4. Client stores: { selectedSeats: [32L, 33U], lockToken: "tk_xyz" }
   ↓
5. User clicks "Proceed" → page transition happens
   ↓
6. New page loads; client retrieves seat state from server using lockToken
   ↓
7. Page displays: "Seats confirmed: 32-Lower, 33-Upper"
   ↓
8. User fills passenger details and confirms booking
```

### Technical Implementation

**Backend Seat Locking:**
- In-memory cache (or Redis) stores locked seats with TTL (10 minutes)
- Lock structure: `{ bookingId, sessionId, lockedSeats: [32L, 33U], lockToken, expiresAt }`
- Lock prevents other users from booking same seats while this user is in booking flow
- Lock released automatically after 10 mins or on booking completion/cancellation

**Frontend State Persistence:**
- Seat selections stored in local Redux/Vuex store
- On component mount, verify seats with backend using lockToken
- If server state differs from client, show warning: "Your seat selection changed. [Show new seats] [Re-select]"
- URL state preserved: `/booking/seats?lockToken=tk_xyz` enables browser history navigation

**Mobile-Specific Handling:**
- Coach carousel/swipe events do **not** trigger state resets
- Add event handlers: `@touchmove` debounced to prevent accidental deselections
- Show selected seats in persistent footer (even when scrolling carousel)

**API Changes:**
```
POST /api/v1/booking/seat/lock
Request:
{
  "trainId": "12622",
  "coachId": "S1",
  "seatNumbers": [32, 33],
  "classCode": "SL",
  "journeyDate": "2026-05-10"
}
Response:
{
  "lockToken": "tk_xyz123",
  "lockExpiresAt": "2026-05-08T10:35:00Z",
  "lockedSeats": [
    { "number": 32, "position": "Lower", "fare": 1250 },
    { "number": 33, "position": "Upper", "fare": 1250 }
  ]
}

GET /api/v1/booking/seat/status/:lockToken
Response:
{
  "lockedSeats": [32, 33],
  "status": "LOCKED" | "EXPIRED" | "RELEASED",
  "expiresIn": 245  // seconds
}

POST /api/v1/booking/confirm
Request:
{
  "lockToken": "tk_xyz123",
  "passengerDetails": [...],
  "paymentMethod": "UPI"
}
Response:
{
  "bookingId": "BK_123456",
  "confirmationNumber": "1234567890"
}
```

### Success Metrics
- **Seat selection persistence**: 99%+ accuracy (vs. current 50–60%)
- **Mobile booking completion**: Increase from 35% → 65%
- **Seat re-selection rate**: Reduce from 30% → <2%
- **Booking abandonment (mobile)**: Reduce from 25% → 8%
- **Lock collision (overbooking)**: <0.1% (near-zero double-booking incidents)

### Edge Cases and Constraints

**Lock Management Challenges:**
1. **Lock expires mid-booking**: User takes >10 mins to fill passenger details. Lock expires → seat released to another user. Solution: Show countdown timer "Lock expires in 3 mins" + option to extend lock for another 5 mins
2. **User selects seat, changes mind, selects different seat**: Old lock must be released, new lock acquired. Solution: Atomic swap operation (release old, acquire new in single transaction)
3. **Seat unavailable**: Another user's lock acquired seat during user's page transition. Solution: Show "This seat was just booked. [Show alternatives] [Back to map]"
4. **Network failure during lock**: User selects seat but POST fails silently. Solution: Retry with exponential backoff; if retry fails after 3 attempts, show "Unable to lock seat. Try again?"

**Railway Backend Constraints:**
- Railway API doesn't support seat locking natively; IRCTC must implement own lock mechanism
- Lock duration limited by Railway API seat reservation timeout (typically 5 mins); IRCTC lock (10 mins) must account for Railway's timeout

---

## Feature Spec 4: PNR Status Page Optimization

### Problem Statement
PNR (Passenger Name Record) status page times out ~25–35% of the time with no progress feedback. Successful loads take 45+ seconds. Users have no way to know if their query is processing, queued, or failed. This creates anxiety and drives support ticket volume. During peak hours (8–10 AM, 5–9 PM), timeout rate reaches 60%+. **[Linked to Part A: Problem 4]**

### Proposed Solution
Optimize backend PNR queries using **indexed database lookups**, **query result caching**, and **asynchronous processing**. For slow queries, implement **progressive loading** with partial results. Add a **queue system** to manage concurrent PNR checks without server overload. Introduce **push notifications** so users can check back later instead of waiting on the page.

**User Flow — Proposed:**
```
1. User enters PNR number: "1234567890"
   ↓
2. System checks cache for recent PNR data (expires after 5 mins)
   ↓
3a. (If cache hit) Results show instantly
   ↓
3b. (If cache miss) Queue PNR check request; show: "Checking... expected wait: ~30s"
   ↓
4. Once query completes (within 5-10s), results load progressively:
   - Passenger name, train, class (loaded first)
   - Seat assignment and status (loaded next)
   - Coach position, boarding/arrival times (loaded last)
   ↓
5. User sees complete PNR details or TDR status
   ↓
6. Option: "Notify me of status changes" → receive push notification when WL confirms
```

### Technical Implementation

**Backend Optimization:**
- **Database indexing**: PNR number as primary index; query execution time reduced from 2s → 200ms
- **Query result caching**: Redis cache with 5-minute TTL for PNR data
- **Query optimization**: Replace full table scans with indexed lookups
  
- **Asynchronous PNR processor**:
  - Queue incoming PNR requests in RabbitMQ/Kafka
  - Process queue at 500 requests/second (configurable)
  - Return results to client via WebSocket or long-polling
  
- **Progressive result streaming**:
  - Return partial results (passenger info) within 1s
  - Stream remaining details (seats, status) as they become available

**Frontend Experience:**
- **Progressive loading skeleton**: Show placeholder content while waiting
- **Queue position transparency**: "You're #234 in queue. Expected wait: 45 seconds"
- **Auto-retry with backoff**: If request fails, auto-retry with 2s, 4s, 8s delays
- **Offline support**: Cache last known PNR status; show "Last checked 2 hours ago" if offline

**API Changes:**
```
POST /api/v1/pnr/check
Request:
{
  "pnrNumber": "1234567890",
  "prioritize": false  // optional: pay for priority processing
}
Response (streaming):
{
  "queuePosition": 234,
  "estimatedWaitSeconds": 45,
  "status": "QUEUED" | "PROCESSING" | "COMPLETE"
}

# Once COMPLETE, separate result:
GET /api/v1/pnr/:pnrNumber
Response:
{
  "pnrNumber": "1234567890",
  "passengers": [
    { "name": "John Doe", "age": 35, "gender": "M", "status": "CONFIRMED" }
  ],
  "train": { "number": "12622", "name": "Tamil Nadu Express" },
  "from": "DEL", "to": "BOM", "date": "2026-05-10",
  "class": "SL",
  "seats": ["32-Lower", "33-Upper"],
  "boardingAt": "DEL", "boardingTime": "22:55",
  "arrivalAt": "BOM", "arrivalTime": "10:35 +1"
}

WebSocket event: "pnr:status:update"
```

### Success Metrics
- **Query response time**: Reduce from 45s → <5s (for cache hits) or <15s (for misses)
- **Timeout rate**: Reduce from 25–35% → <2%
- **Peak hour performance**: Maintain <10s response time even at 8–10 AM
- **User satisfaction**: PNR-related support tickets reduce by 60%
- **Cache hit rate**: Achieve 70%+ for repeat PNR checks

### Edge Cases and Constraints

**Data Freshness:**
1. **PNR updated after cached**: Railway backend updated seat status, but IRCTC cache shows old data. Solution: 5-minute TTL acceptable for most use cases; offer "Refresh now" button for urgent updates
2. **PNR not found**: User enters invalid PNR or booking not yet in Railway system. Solution: Show "PNR not found. Check number and try again in 2 minutes."

**Queue Management:**
- If queue length exceeds 5000, show "System is busy. Check back in 10 minutes." + offer async notification option
- Priority processing: Premium/paid PNR checks bypass queue (optional feature for revenue)

---

## Feature Spec 5: Modal Dialog Redesign

### Problem Statement
Login modals, promotional popups, and alerts frequently **block form interactions** by overlaying forms with no clear dismiss mechanism. Form data is lost when modals are closed. Close buttons are hard to find or click on mobile. Users cannot navigate modals with keyboard. These blocking modals appear in 30–40% of user sessions, preventing bookings in ~20% of affected sessions. **[Linked to Part A: Problem 5]**

### Proposed Solution
Redesign modals using **accessibility-first principles** (WCAG AA compliant). Implement a **modal stack manager** to prevent multiple overlays. Add **auto-close on ESC key** and **click outside to dismiss**. Modals should **never overlay critical form elements**; instead, modals should appear in a designated safe zone (top of page or centered but not over inputs). Preserve form state even if modal is dismissed.

**Modal Redesign Principles:**
```
BEFORE (Broken):
- Modal appears centered, overlays entire form
- Close button tiny (10x10px) in top-right corner
- No ESC key handling
- Clicking outside modal does nothing
- Form data lost when modal dismissed

AFTER (Fixed):
- Modal slides in from top or appears in fixed banner zone
- Close button prominent (40x40px), left/right accessible
- ESC key dismisses modal
- Click outside modal dismisses it
- Form state preserved in memory even after modal closes
- Form fields remain interactive through semi-transparent overlay
```

### Technical Implementation

**Modal Stack Manager (Redux/Vuex):**
```javascript
modals: [
  { id: "login", priority: 10, component: LoginModal, state: {...} },
  { id: "promotion", priority: 5, component: PromoBanner, state: {...} }
]
// Only highest-priority modal displayed; lower ones queued
```

**Modal Accessibility (ARIA):**
```html
<div role="dialog" aria-modal="true" aria-labelledby="modal-title">
  <h1 id="modal-title">Welcome to IRCTC</h1>
  <p>Create an account to save preferences.</p>
  
  <button aria-label="Close dialog">✕</button>
</div>
```

**Keyboard Navigation:**
- ESC key closes topmost modal
- TAB key cycles through modal controls only (no focus escape)
- ENTER key activates default button (e.g., "Sign In")

**Form State Preservation:**
- When modal opens, save form state to Redux store
- When modal closes, restore form state (user's typed inputs still there)
- Add event listener on modal dismiss: `localStorage.setItem('formDraft', currentFormState)`

**API Changes:**
```
POST /api/v1/modal/dismiss
Request:
{
  "modalId": "login",
  "reason": "user_clicked_outside" | "pressed_escape" | "clicked_close"
}
// Helps track user behavior; no critical logic depends on this
```

### Success Metrics
- **Modal-blocked interactions**: Reduce from 20% to <2%
- **Form abandonment due to modals**: Reduce from 10% to <1%
- **Modal dismissal time**: Reduce from 5–10 seconds to <1 second
- **Accessibility compliance**: Achieve WCAG AA rating (tested with NVDA/JAWS screen readers)
- **Support tickets**: Reduce modal-related complaints by 85%

### Edge Cases and Constraints

**Multi-Modal Scenarios:**
1. **User logs in via modal, promotional modal appears on top**: Stack manager prevents this; promotion queued until login completes
2. **Modal within modal**: Avoid nesting; use modal stack manager instead
3. **Mobile: modal takes full screen but form needs interaction**: Use drawer pattern (slide-in from bottom) instead of centered modal

---

## Feature Spec 6: Mobile Responsiveness Overhaul

### Problem Statement
On mobile (375px and below), IRCTC search form and results are unresponsive. Input fields are too small to tap, keyboard covers form elements, dropdown options overflow off-screen, and filters don't display. 100% of mobile users encounter responsiveness issues; 30–40% face critical issues preventing booking completion. Mobile represents 65% of traffic, yet has 50–60% lower conversion than desktop due to poor UX. **[Linked to Part A: Problem 6]**

### Proposed Solution
Implement a **mobile-first responsive redesign**. Rebuild form layout for 320px minimum width with proper spacing and tap targets. Use **bottom-sheet drawers** for filters instead of sidebars. Implement **keyboard handling** so form elements stay visible when keyboard opens. Optimize **touch interactions** (larger tap targets, reduced double-taps). Test extensively on real devices (not just browser DevTools).

**Layout Changes by Screen Size:**
```
Desktop (1024px+):
- Form and filters side-by-side
- Sidebar filter panel (25% width)
- Full train details in each result card

Tablet (768px–1024px):
- Form full-width top
- Collapsible filter panel below form
- Compact train details

Mobile (375px–767px):
- Form full-width, optimized spacing
- Filters in bottom-sheet drawer (swipe up to open)
- Minimal train card (essential info only)
- CTA buttons full-width
```

### Technical Implementation

**Mobile-First CSS:**
```css
/* Start with mobile layout */
.search-form {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.form-input {
  padding: 16px;  /* Min 44px touch target */
  font-size: 16px;  /* Prevents zoom-on-focus on iOS */
  width: 100%;
}

/* Tablet breakpoint */
@media (min-width: 768px) {
  .search-form {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 1.5rem;
  }
}

/* Desktop breakpoint */
@media (min-width: 1024px) {
  .container {
    display: flex;
    gap: 2rem;
  }
  .sidebar { width: 300px; }
  .results { flex: 1; }
}
```

**Keyboard Handling (JavaScript):**
```javascript
// Prevent keyboard from covering input fields
window.addEventListener('keyboardshow', (e) => {
  const keyboardHeight = e.keyboardHeight;
  const activeInput = document.activeElement;
  const inputRect = activeInput.getBoundingClientRect();
  
  if (inputRect.bottom > window.innerHeight - keyboardHeight) {
    activeInput.scrollIntoView({ behavior: 'smooth', block: 'center' });
  }
});

// Use viewport-fit=cover to account for notches on modern phones
// <meta name="viewport" content="...viewport-fit=cover">
```

**Filter Drawer Component:**
```html
<!-- Mobile drawer (bottom-sheet) -->
<div class="filter-drawer" data-state="closed">
  <div class="drawer-handle">⋯</div>
  <h2>Filters</h2>
  <div class="filter-options">
    <label><input type="checkbox"> Available Only</label>
    <label><input type="checkbox"> Sleeper Class</label>
    ...
  </div>
  <button class="apply-filters">Apply Filters</button>
</div>

<style>
.filter-drawer {
  position: fixed;
  bottom: 0;
  width: 100%;
  max-height: 80vh;
  background: white;
  border-radius: 16px 16px 0 0;
  transform: translateY(100%);
  transition: transform 0.3s ease;
  z-index: 10;
}

.filter-drawer[data-state="open"] {
  transform: translateY(0);
}
</style>
```

**Touch-Optimized Interactions:**
- Button/link tap target: minimum 44x44 points (48x48 recommended)
- Spacing between clickables: minimum 8px
- Double-tap zoom: Disable for interactive elements (prevent 300ms delay)
  ```html
  <meta name="viewport" content="...user-scalable=no">
  <!-- Or use: touch-action: manipulation on specific elements -->
  ```

**API Changes:** None (UI-only changes)

### Success Metrics
- **Mobile form completion time**: Reduce from 3–4 minutes to <90 seconds
- **Mobile booking completion rate**: Increase from 35% to 65%
- **Mobile conversion parity with desktop**: Narrow gap from 50% → 85%
- **Mobile user satisfaction**: Increase app store rating from 3.2 to 4.5+ stars
- **Mobile bounce rate**: Reduce from 45% to <15%
- **Accessibility compliance**: Pass WCAG mobile accessibility audit

### Edge Cases and Constraints

**Device-Specific Challenges:**
1. **iPhone notch (Dynamic Island)**: Use `viewport-fit=cover` + safe area insets for header/footer positioning
2. **Android back button**: Ensure drawer/modal closes on hardware back button (not app back)
3. **Viewport zoom**: User zoomed to 150%+ — ensure form still usable
4. **Slow 2G/3G**: Images lazy-load; prioritize critical form HTML
5. **Portrait vs landscape**: Ensure same usability in both orientations

**Testing Requirements:**
- Real device testing: iPhone 12/13/14/15 (various sizes), Samsung A-series, older phones
- Browser testing: Chrome, Safari, Firefox mobile
- Network throttling: Test on 2G/3G/4G LTE via DevTools
- Accessibility: Screen reader testing (TalkBack on Android, VoiceOver on iOS)

---

## Summary of Feature Specs

| Spec | Problem | Impact | Effort | Priority |
|------|---------|--------|--------|----------|
| 1. Tatkal Queue | Tatkal crash 40-60% failure | **Critical** | **6-8 weeks** | **P0** |
| 2. Filter Persistence | Filters fail 30-40% | **High** | **2-3 weeks** | **P0** |
| 3. Seat State | Seats reset 40-50% | **High** | **3-4 weeks** | **P0** |
| 4. PNR Optimization | PNR timeouts 25-35% | **High** | **2-3 weeks** | **P1** |
| 5. Modal Redesign | Modals block 20% sessions | **Medium** | **1-2 weeks** | **P1** |
| 6. Mobile Responsive | Mobile conversion 50% lower | **High** | **4-5 weeks** | **P1** |

---

**Next Section**: Move to [WIREFRAMES.md](./WIREFRAMES.md) for UI mockups and [MATRIX.md](./MATRIX.md) for prioritisation.

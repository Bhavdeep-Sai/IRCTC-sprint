# IRCTC Wireframes — Part B: UI/UX Solutions

**Design Level**: Mid-fidelity wireframes (Figma-ready)
**Focus**: Before/After comparison for each UI problem

---

## Wireframe 1: Tatkal Virtual Queue System

### Problem Reference
**Part A Problem 1**: Tatkal Booking Crashes at 10:00 AM — 40-60% failure rate, no user feedback

### Screen: Tatkal Queue Waiting Room (Mobile 375px)

#### BEFORE (Current Broken Flow)
```
┌─────────────────────────────────┐
│  IRCTC  [back]      [menu]      │
├─────────────────────────────────┤
│                                 │
│         [ Loading... ]          │
│                                 │
│     (Page hangs for 15 mins)    │
│     No message, no progress     │
│                                 │
├─────────────────────────────────┤
│  (Eventually shows 500 error)    │
│  "Internal Server Error"         │
│  [ Try Again ]                  │
│                                 │
└─────────────────────────────────┘
```

#### AFTER (Proposed Queue System)
```
┌─────────────────────────────────┐
│  IRCTC  [back]      [menu]      │
├─────────────────────────────────┤
│                                 │
│   🕐 TATKAL OPENS IN             │
│   ┌──────────────────────┐       │
│   │    04 : 23 : 11     │ ←[Live countdown]
│   └──────────────────────┘       │
│                                 │
│   📍 YOUR QUEUE POSITION         │
│   ┌──────────────────────┐       │
│   │      #4,281          │       │
│   │ Est. wait: ~9 mins   │       │
│   │                      │       │
│   │ ███████░░░░░░░░░░░░ │ ←[Progress bar]
│   └──────────────────────┘       │
│                                 │
│   ✅ Train: 12622 Tamil Nadu     │
│   ✅ Class: Sleeper | Pax: 2     │
│   ✅ Passengers: Saved           │
│                                 │
│   [📝 Change Details]            │
│                                 │
│ ℹ️  When your turn arrives, you   │
│    get 90s to complete booking.  │
│                                 │
└─────────────────────────────────┘
```

**Key Changes:**
- Live countdown timer showing time until Tatkal opens
- Queue position prominently displayed with estimated wait
- Progress bar showing movement through queue
- Visual confirmation of selected journey details
- Tips about the 90-second booking window
- All text remains visible; no loading hangs

---

### Screen: Tatkal Booking Window (When Position Arrives) — Mobile 375px

```
┌─────────────────────────────────┐
│  IRCTC  [back]      [menu]      │
├─────────────────────────────────┤
│                                 │
│   🎯 YOUR TURN IS HERE!          │ ←[Highlight in green]
│   ┌──────────────────────┐       │
│   │      ⏱ 90 : 00      │ ←[Red if <30s]
│   └──────────────────────┘       │
│                                 │
│   💺 SELECT YOUR SEATS           │
│   [Seat map shown - pre-loaded] │
│                                 │
│   ┌──────┬──────┐                │
│   │  ⬜  │  ⬜  │ Coach S1        │
│   │ 32-L │ 33-L │ (Lower berths) │
│   │ ⬜ ⬜ │ ⬜ ⬜ │                │
│   │ ⬜ ⬜ │ ⬜ ⬜ │                │
│   │ ⬜ ⬜ │ ⬜ ⬜ │                │
│   └──────┴──────┘                │
│                                 │
│   Selected: 32-Lower, 33-Upper  │ ←[Live confirmation]
│                                 │
│   [Proceed to Payment ✓]         │
│                                 │
│   ⚠️  Hurry! Timer counting down. │
│                                 │
└─────────────────────────────────┘
```

**Key Changes:**
- Large countdown timer (90 seconds) with visual urgency
- Pre-loaded seat map (no delay waiting)
- Selected seats confirmed in real-time
- Single CTA button (proceed to payment)
- Urgency messaging

---

### Desktop/Tablet Comparison (1024px+)

```
┌────────────────────────────────────────────────┐
│ IRCTC Logo        Search    Bookings   Profile │
├────────────────────────────────────────────────┤
│                                                │
│  ┌──────────────────────────────────────────┐  │
│  │  🕐 TATKAL OPENS IN                       │  │
│  │     04 : 23 : 11        [Queue status] ✅ │  │
│  │                                           │  │
│  │  📍 Your Queue Position: #4,281           │  │
│  │     Est. wait: ~9 minutes                 │  │
│  │     ██████████░░░░░░░░░░░░░░░░░░░░░░░░░░│  │
│  │                                           │  │
│  │  Train: 12622 Tamil Nadu Express          │  │
│  │  Class: Sleeper | Passengers: 2          │  │
│  │  ✏️ [Edit Details]                        │  │
│  │                                           │  │
│  │  When your turn arrives, you'll see a    │  │
│  │  live countdown and seat map. You have   │  │
│  │  90 seconds to book.                      │  │
│  └──────────────────────────────────────────┘  │
│                                                │
│  [Additional info: Common issues, FAQ]         │
│                                                │
└────────────────────────────────────────────────┘
```

---

## Wireframe 2: Persistent Search Filters

### Problem Reference
**Part A Problem 2**: Search Filters Do Not Work Reliably — 30-40% failure rate, filters lost on navigation

### Screen: Train Search Results with Filters (Mobile 375px)

#### BEFORE (Filters Broken)
```
┌─────────────────────────────────┐
│ DEL  >  BOM  |  May 10  |  1     │ ←[Limited space]
├─────────────────────────────────┤
│ 🔍 [Search]                     │
├─────────────────────────────────┤
│ FILTERS:                        │
│  ☐ Available Only              │ ←[Partially visible]
│  ☐ Sleeper                     │ ←[Checkbox misaligned]
│  ☐ Morning                     │
│  [Filter panel hidden/broken]   │
├─────────────────────────────────┤
│ Train 1: 12622 (40 seats) ....│
│ Train 2: 12624 (Waitlist)    │ ←[Filters ignored]
│ Train 3: 12626 (40 seats) ....│ ←[Wrong results]
│ [See more results]             │
│                                 │
│ [Clicking filters changes URL   │
│  but results don't update]      │
│                                 │
│ [Back button clears filters]    │
│                                 │
└─────────────────────────────────┘
```

#### AFTER (Filters Working)
```
┌─────────────────────────────────┐
│ ◀ DEL → BOM | May 10, 2026      │
├─────────────────────────────────┤
│ 🔍 [Search]                     │
├─────────────────────────────────┤
│ FILTERS: [≡ Show/Hide]          │ ←[Filter drawer trigger]
│                                 │
│ Active filters:                 │
│ [✕ Available] [✕ Sleeper]     │ ←[Easy removal]
│                                 │
│ 📊 Results (4)                  │ ←[Accurate count]
├─────────────────────────────────┤
│ 🚂 12622 Tamil Nadu Express      │
│ ├─ Depart: 22:55  Arrive: 10:35 │
│ ├─ SL: ✅ 40 seats              │ ←[Only sleeper shown]
│ ├─ Price: ₹1,250                │
│ └─ [Book]                       │
│                                 │
│ 🚂 12625 Karnataka Express       │
│ ├─ Depart: 23:45  Arrive: 11:50 │
│ ├─ SL: ✅ 35 seats              │
│ ├─ Price: ₹1,250                │
│ └─ [Book]                       │
│                                 │
│ [Load more results ...]         │
│                                 │
└─────────────────────────────────┘
```

**Filter Drawer (Overlay):**
```
┌─────────────────────────────────┐
│           FILTERS               │
│ ⋯ (drag handle)                 │ ←[Swipe to close]
├─────────────────────────────────┤
│ ☑ Available Only               │ ←[Checked]
│ ☐ Waitlist OK                  │
│                                 │
│ DEPARTURE TIME:                 │
│ ☐ Early Morning (00-06)         │
│ ☑ Morning (06-12)              │ ←[Checked]
│ ☐ Afternoon (12-18)             │
│ ☐ Evening (18-24)               │
│                                 │
│ CLASS:                          │
│ ☑ Sleeper                       │ ←[Checked]
│ ☐ 3AC                           │
│ ☐ 2AC                           │
│ ☐ 1AC                           │
│                                 │
│ [Apply Filters]                 │
│ [Clear All]                     │
│                                 │
└─────────────────────────────────┘
```

**Key Changes:**
- Filters shown in accessible drawer (swipe up on mobile)
- Active filter badges visible above results
- Results update in real-time as filters applied
- Easy filter removal (click X on badges)
- URL updates with filter params: `?filters=available,class:SL,departure:morning`
- Back button preserves filters

---

### Desktop View (1024px+)

```
┌────────────────────────────────────────────────┐
│ IRCTC Logo        Search    Bookings   Profile │
├────────────────────────────────────────────────┤
│                                                │
│  ┌─────────┬─────────────────────────────────┐ │
│  │ FILTERS │  Results (4 trains found)       │ │
│  │         │                                 │ │
│  │ AVAIL-  │  Active filters:                │ │
│  │ ABILITY │  ☑ Available  ☑ Sleeper  ☑ AM │ │
│  │ ☑ Avail │                                 │ │
│  │ ☐ Wait  │  🚂 12622 Tamil Nadu Express    │ │
│  │         │  22:55 → 10:35 | SL: 40 seats  │ │
│  │ DEPART  │  ₹1,250 | [Book]                │ │
│  │ ☑ 06-12 │                                 │ │
│  │ ☐ 12-18 │  🚂 12625 Karnataka Express    │ │
│  │ ☐ 18-24 │  23:45 → 11:50 | SL: 35 seats  │ │
│  │         │  ₹1,250 | [Book]                │ │
│  │ CLASS   │                                 │ │
│  │ ☑ SL    │  🚂 12633 Kerala Express       │ │
│  │ ☐ 3AC   │  [More results...]             │ │
│  │ ☐ 2AC   │                                 │ │
│  │         │  [Load more ↓]                  │ │
│  └─────────┴─────────────────────────────────┘ │
│                                                │
└────────────────────────────────────────────────┘
```

---

## Wireframe 3: Seat Selection State Management

### Problem Reference
**Part A Problem 3**: Seat Selection Resets During Booking — 40-50% reset rate, 65-75% on mobile

### Screen: Seat Map → Passenger Details (Mobile 375px)

#### BEFORE (Seat Reset Bug)
```
┌─────────────────────────────────┐
│ Booking Step 1: Select Seats    │
├─────────────────────────────────┤
│ Coach S1 - Sleeper              │
│ ┌──────────────────────┐         │
│ │ ⬜ ⬜ ⬜ ⬜ ⬜ ⬜ ⬜│         │
│ │ ⬜ ⬜ 🟢 ⬜ 🟢 ⬜ ⬜│ ←[User selected]
│ │ ⬜ ⬜ ⬜ ⬜ ⬜ ⬜ ⬜│  32-L & 33-U
│ │ ⬜ ⬜ ⬜ ⬜ ⬜ ⬜ ⬜│         │
│ └──────────────────────┘         │
│ Selected: 32-Lower, 33-Upper     │
│ Total: ₹2,500                   │
│                                 │
│ [⟵ Back]  [Proceed ✓]          │
│                                 │
└─────────────────────────────────┘
       ↓ User clicks Proceed
       
┌─────────────────────────────────┐
│ Booking Step 2: Passenger Info   │
├─────────────────────────────────┤
│ Passenger 1:                    │
│ Name: [____________]             │
│ Age: [____]                      │
│ Gender: [Select]                │
│                                 │
│ Passenger 2:                    │
│ Name: [____________]             │
│ Age: [____]                      │
│ Gender: [Select]                │
│                                 │
│ Your Seats:                     │
│ ❌ WRONG SEATS SHOWN!           │ ←[BUG]
│ 15-Lower, 16-Upper (Random!)   │
│ Total: ₹2,500                   │
│                                 │
│ [⟵ Back]  [Confirm ✓]          │
│                                 │
│ Issue: User must go back and    │
│ re-select seats again!          │
│                                 │
└─────────────────────────────────┘
```

#### AFTER (Seat State Persisted)
```
┌─────────────────────────────────┐
│ Booking Step 1: Select Seats    │
├─────────────────────────────────┤
│ Coach S1 - Sleeper              │
│ ┌──────────────────────┐         │
│ │ ⬜ ⬜ ⬜ ⬜ ⬜ ⬜ ⬜│         │
│ │ ⬜ ⬜ 🟢 ⬜ 🟢 ⬜ ⬜│ ←[User selected]
│ │ ⬜ ⬜ ⬜ ⬜ ⬜ ⬜ ⬜│  32-L & 33-U
│ │ ⬜ ⬜ ⬜ ⬜ ⬜ ⬜ ⬜│         │
│ └──────────────────────┘         │
│ Selected: 32-Lower, 33-Upper     │
│ Total: ₹2,500                   │
│                                 │
│ 🔒 Seat Lock Token: tk_xyz [✓]  │ ←[Server-side lock created]
│                                 │
│ [⟵ Back]  [Proceed ✓]          │
│                                 │
└─────────────────────────────────┘
       ↓ User clicks Proceed
       
┌─────────────────────────────────┐
│ Booking Step 2: Passenger Info   │
├─────────────────────────────────┤
│ Passenger 1:                    │
│ Name: [____________]             │
│ Age: [____]                      │
│ Gender: [Select]                │
│                                 │
│ Passenger 2:                    │
│ Name: [____________]             │
│ Age: [____]                      │
│ Gender: [Select]                │
│                                 │
│ ✅ YOUR SEATS (CONFIRMED):      │ ←[FIXED]
│ 🔒 32-Lower, 33-Upper           │
│ Total: ₹2,500                   │
│ (Lock expires in: 8m 42s)        │ ←[Timer shown]
│                                 │
│ [⟵ Back]  [Confirm ✓]          │
│                                 │
│ Seats are server-locked and     │
│ reserved for this booking!      │
│                                 │
└─────────────────────────────────┘
```

**Key Changes:**
- Seat lock token created on backend when seats selected
- Lock persists across page navigation
- Seat confirmation shown on passenger details screen
- Lock timer displayed (user knows timeout)
- Back button no longer clears selections
- If seats accidentally freed, show error with recovery option

---

## Wireframe 4: PNR Status Page Optimization

### Problem Reference
**Part A Problem 4**: PNR Status Page Timeout — 25-35% timeout rate, 45+ second loads

### Screen: PNR Check Loading Experience (Mobile 375px)

#### BEFORE (Hangs and Times Out)
```
┌─────────────────────────────────┐
│ IRCTC  [back]      [menu]      │
├─────────────────────────────────┤
│ CHECK PNR STATUS                │
├─────────────────────────────────┤
│ Enter PNR: [______________]     │
│ [Check Status]                  │
│                                 │
│ (After clicking, page waits...) │
│                                 │
│         ⏳ Loading...             │
│   (Spinner for 30+ seconds)     │
│                                 │
│  (User gets no feedback)        │
│  (Browser shows "Not responding")
│                                 │
│ (After 45 seconds, error:)      │
│ ❌ Request Timeout               │
│ "Unable to fetch PNR details"   │
│ [Try Again]                     │
│                                 │
│ (User must start over)          │
│                                 │
└─────────────────────────────────┘
```

#### AFTER (Progressive Loading with Queue Info)
```
┌─────────────────────────────────┐
│ IRCTC  [back]      [menu]      │
├─────────────────────────────────┤
│ CHECK PNR STATUS                │
├─────────────────────────────────┤
│ Enter PNR: [1234567890]         │
│ [Check Status]                  │
│                                 │
│ (After clicking, shows:)        │
│                                 │
│ 🔄 Checking your PNR...          │
│                                 │
│ Queue Position: #234            │ ←[Live feedback]
│ Estimated Wait: ~30 seconds     │
│ ███████░░░░░░░░░░░░  Progress   │
│                                 │
│ 💡 Tip: You'll see results      │
│ within 30 seconds. Already      │
│ checked? [Notify Me] instead    │
│                                 │
│ (After 5-10 seconds:)           │
│                                 │
│ ✅ PNR Found (Loading details...)│ ←[Partial results]
│                                 │
│ 👤 John Doe, Age 35             │ ←[Streamed in]
│ 🚂 Train 12622 | Sleeper        │
│ 📍 DEL → BOM on May 10          │
│                                 │
│ [Seats and status loading...]   │
│                                 │
│ (After another 2-3 seconds:)    │
│                                 │
│ 🎫 BOOKING CONFIRMED            │
│ 🏷️  Seats: 32-Lower, 33-Upper   │
│ ✅ Status: CONFIRMED             │
│ ⏰ Boarding: 22:55 at DEL       │
│ 🚪 Coach: S1 | Berth: LB-32    │
│                                 │
│ [⬇️ Download Receipt]             │
│                                 │
└─────────────────────────────────┘
```

**Key Changes:**
- Queue position shown with ETA (transparent load balancing)
- Partial results load progressively (passenger info → train info → seat/status)
- Loading states clearly communicated
- Option to be notified later instead of waiting
- No timeouts; if server slow, user can leave and get notification

---

## Wireframe 5: Modal Dialog Redesign

### Problem Reference
**Part A Problem 5**: Modal Dialogs Block Form Interactions — Modals block 20% of sessions

### Screen: Login Modal Redesign (Mobile 375px)

#### BEFORE (Blocking Modal)
```
┌─────────────────────────────────┐
│ IRCTC  [back]      [menu]      │
├─────────────────────────────────┤
│ ┌──────────────────────────────┐ │
│ │ ┌─ x ─────────────────────┐  │ │ ←[Tiny close button]
│ │ │ Welcome! Sign In         │  │ │
│ │ │ ───────────────────────  │  │ │
│ │ │ Email: [_____________]   │  │ │ ←[Modal overlays form]
│ │ │ Password: [___________]  │  │ │
│ │ │ [Create Account]         │  │ │
│ │ │ [Sign In]                │  │ │
│ │ └──────────────────────────┘  │ │
│ │                                 │ │
│ │ Form behind is BLOCKED         │ │
│ │ (Faded, unclickable)           │ │
│ │                                 │ │
│ └──────────────────────────────┘ │
│                                 │
│ Issue:                          │
│ - Close X too small (10x10px)   │
│ - Form behind is grayed out     │
│ - Can't click outside modal     │
│ - ESC key doesn't work          │
│ - Form data lost if closed      │
│                                 │
└─────────────────────────────────┘
```

#### AFTER (Non-Blocking Modal)
```
┌─────────────────────────────────┐
│ IRCTC  [back]      [menu]      │
├─────────────────────────────────┤
│                                 │
│ ──── ⊗ Welcome to IRCTC ────┐  │ ←[Close button prominent]
│ Already have an account?      │
│ [Sign In] | [Create Account]  │ ←[Not modal; banner style]
│ └────────────────────────────┘  │
│                                 │
│ Form REMAINS FULLY VISIBLE:     │ ←[Not blocked]
│                                 │
│ 🔍 Train Search                 │
│ From: [DEL____________] ✓       │ ←[Can still interact]
│ To: [____________]              │ ←[Can still interact]
│ Date: [May 10]                 │ ←[Can still interact]
│ [Search Trains]                │
│                                 │
│ (User can click on form         │
│  while banner visible)          │
│                                 │
│ (Clicking X or clicking        │
│  outside banner dismisses it)   │
│                                 │
│ (Form data preserved)           │
│                                 │
└─────────────────────────────────┘
```

**Key Changes:**
- Modal converted to top banner instead of centered overlay
- Close button large and accessible (40x40px)
- Form remains fully visible and interactive
- Click outside to dismiss
- ESC key dismisses
- Form state preserved in memory

---

## Wireframe 6: Mobile Responsiveness Overhaul

### Problem Reference
**Part A Problem 6**: Mobile Responsiveness Issues — 100% of mobile users affected

### Screen: Search Form Responsiveness Comparison

#### BEFORE (Broken Mobile Layout)
```
Mobile (375px width):           Keyboard open issue:
┌─────────────────┐             ┌─────────────────┐
│ [Search Form]   │             │ [Form partially │
│                 │             │  visible]       │
│ From: [tiny]    │             │                 │
│ [Too small]     │             │ From: [❌❌❌]  │
│                 │             │                 │
│ To: [tiny]      │             │ To: [keyboard_] │
│ [Too small]     │             │ [========]      │
│                 │             │ [keyboard]      │
│ Date: [tiny    │             │ [keyboard]      │
│ calendar]       │             │ [keyboard]      │
│ [Overflows]     │             │                 │
│                 │             │ (Can't see Date │
│ Class:    ▼     │             │  field)         │
│ [Options]       │             │                 │
│ [outside     --→             │ (Must scroll up  │
│  screen]        │             │  in modal)      │
│                 │             │                 │
│ [Search Trains] │             │ (Filter drawer  │
│ [Search] -------→             │  not accessible │
│                 │             │  while typing)  │
└─────────────────┘             └─────────────────┘

Issues:
❌ Input fields too small (< 44px touch target)
❌ Text tiny (8-10px)
❌ Dropdown options hidden off-screen
❌ Keyboard covers form elements
❌ Single column layout not optimized
❌ Filter sidebar takes 50% of mobile width
```

#### AFTER (Mobile Optimized)
```
Mobile (375px width):           Keyboard open:
┌─────────────────┐             ┌─────────────────┐
│ IRCTC │ 🔍│👤  │             │ ✅ From: [___]  │
├─────────────────┤             │ ✅ To: [____]   │
│ FROM STATION:   │             │ ✅ Date: May 10 │
│ ┌─────────────┐ │             │ ✅ Class: SL    │
│ │ [Delhi___]  │ │ ←[44px+ tall]            
│ │ [Tap to clear] │             │ ┌─────────────┐│
│ └─────────────┘ │             │ │[keyboard_] ││
│                 │             │ │[========]  ││
│ TO STATION:     │             │ │[keyboard]  ││
│ ┌─────────────┐ │             │ └─────────────┘│
│ │ [Mumbai___] │ │ ←[16px font]│ (Form scroll) │
│ │ [Tap to clear] │             │ (Keyboard    │
│ └─────────────┘ │             │  doesn't     │
│                 │             │  block)      │
│ DATE & CLASS:   │             │              │
│ ┌─────────────┐ │             │ [Proceed ✓] │
│ │ May 10 │ SL▼│ │ ←[Touch-opt]             │
│ │ [Easy tap]  │ │             └─────────────┘
│ └─────────────┘ │
│                 │
│ ┌─────────────┐ │
│ │ SEARCH     │ │ ←[Full-width CTA]
│ │ TRAINS     │ │ ←[44px+ tall button]
│ └─────────────┘ │
│                 │
│ [≡ Filters ▼]   │ ←[Drawer trigger]
│                 │
└─────────────────┘

✅ Features:
✅ 44x44px minimum touch targets
✅ 16px font (readable, no zoom needed)
✅ Full-width inputs and buttons
✅ Keyboard doesn't cover form
✅ Filter drawer at bottom (easily accessible)
```

**Filter Drawer (Swipe Up):**
```
┌─────────────────┐
│ SEARCH RESULTS  │
│                 │
│ 📍 12622        │
│ 📍 12625        │
│ 📍 12633        │
│                 │
│ ⋯ (Drawer drag) │ ←[Swipe to expand]
├─────────────────┤
│   FILTERS       │
├─────────────────┤
│ ☑ Available     │ ←[44px checkbox]
│ ☐ Waitlist OK   │
│                 │
│ DEPARTURE TIME  │
│ ☐ Early (00-06) │ ←[Touch-friendly spacing]
│ ☑ Morning       │
│ ☐ Afternoon     │
│ ☐ Evening       │
│                 │
│ CLASS           │
│ ☑ Sleeper       │ ←[Clear, legible]
│ ☐ 3AC           │
│ ☐ 2AC           │
│                 │
│ [Apply Filters] │ ←[Full-width CTA]
│ [Clear All]     │
│                 │
└─────────────────┘
```

**Tablet/Desktop Progression:**

```
Tablet (768px):                 Desktop (1024px+):
┌──────────────┐                ┌──────────────────────┐
│ Form (Full)  │                │ Sidebar   │ Results  │
├──────────────┤                │ (Filters  │ (Train   │
│ Results (FW) │                │ visible)  │ cards)   │
├──────────────┤                │           │          │
│ Filters      │                │ Compact   │ Expanded │
│ (Below/above)│                │ layout)   │ details) │
└──────────────┘                └──────────────────────┘
```

**Key Changes:**
- Mobile-first responsive design (320px+ support)
- Minimum 44x44px touch targets
- 16px minimum font size (no zoom needed)
- Full-width buttons and inputs
- Drawer pattern for filters (not sidebar)
- Proper keyboard handling (form scrolls; doesn't get covered)
- Tested on real devices (not just DevTools)

---

## Wireframe Annotations Summary

| Wireframe | Device | Key Interaction | State |
|-----------|--------|-----------------|-------|
| 1. Tatkal Queue | Mobile/Desktop | Real-time countdown, queue position, 90s slot | Active Session |
| 2. Filters | Mobile/Desktop | Filter selection, result update, drawer | Applied Session |
| 3. Seat Selection | Mobile → Desktop | Seat lock token, persistent state, back nav | Booking Flow |
| 4. PNR Status | Mobile/Desktop | Queue position, progressive loading, notifications | Status Check |
| 5. Modal Redesign | Mobile/Desktop | Banner-style, form interactive, ESC/outside dismiss | Overlay |
| 6. Mobile Responsive | Mobile focus | Touch targets, keyboard handling, drawer filters | Form Input |

---

**Next Section**: Continue with AI feature proposal in [AI-FEATURE.md](./AI-FEATURE.md)

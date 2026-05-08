# IRCTC 2×2 Impact vs Effort Prioritisation Matrix — Part B

**Purpose**: Rank 6 proposed solutions by impact and effort to determine execution sequence and resource allocation

---

## Matrix Overview

```
                    ↑ IMPACT (Severity × User Count × Revenue)
                    │
         ┌──────────┼──────────┐
    HIGH │          │          │
    IMPA │  MAJOR   │ QUICK    │
    CT   │ PROJECTS │  WINS    │
         │          │          │
         ├──────────┼──────────┤ ← Effort
    LOW  │ TIME     │ FILL-INS │
    IMPA │ SINKS    │          │
    CT   │          │          │
         └──────────┼──────────┘
              LOW      HIGH
           EFFORT    EFFORT
```

---

## Individual Feature Scoring

Each feature scored on 7 dimensions to place on matrix. Scoring: 1 (very low) — 5 (very high)

### Feature 1: Tatkal Virtual Queue System

**Dimension Scoring**:

| Dimension | Score | Notes |
|-----------|-------|-------|
| **Users affected** | 5 | 50K+ daily attempts; 20-40 lakh concurrent at 10 AM |
| **Core booking flow impact** | 5 | CRITICAL PATH: Directly prevents bookings; not peripheral |
| **Severity of consequence** | 5 | Revenue loss ₹10+ crores daily; 100% predictability (daily crash) |
| **Frontend effort** | 3 | Real-time UI components; Socket.io integration; moderate complexity |
| **Backend/infra effort** | 5 | Redis queue, WebSocket server, queue controller, session tokens; extensive |
| **Railway API dependency** | 4 | Must still call Railway API at final step; creates integration complexity |
| **Risk of breaking existing** | 4 | Queue system is new layer; fallback required; moderate risk if not tested |

**Impact Total (Rows 1-3)**: 5+5+5 = **15/15** ✅ MAXIMUM
**Effort Total (Rows 4-7)**: 3+5+4+4 = **16/20** 🔴 HIGH EFFORT

**Matrix Placement**: **MAJOR PROJECT** (High Impact, High Effort)

---

### Feature 2: Persistent Search Filters

**Dimension Scoring**:

| Dimension | Score | Notes |
|-----------|-------|-------|
| **Users affected** | 4 | ~2-5 million daily searches; 100% of search users impacted by this |
| **Core booking flow impact** | 4 | Not CRITICAL PATH but heavily affects search experience (UX, not blocking) |
| **Severity of consequence** | 3 | Users manually scroll; frustrating; not revenue-blocking like Tatkal |
| **Frontend effort** | 3 | URL params, Redux state sync, filter components; standard React patterns |
| **Backend/infra effort** | 2 | Session storage, server-side filter caching; Redis TTL management |
| **Railway API dependency** | 1 | Independent of Railway backend; IRCTC controls fully |
| **Risk of breaking existing** | 2 | URL params backward compatible; safe to deploy; low regression risk |

**Impact Total (Rows 1-3)**: 4+4+3 = **11/15** ✅ HIGH
**Effort Total (Rows 4-7)**: 3+2+1+2 = **8/20** 🟢 LOW EFFORT

**Matrix Placement**: **QUICK WIN** (High Impact, Low Effort)

---

### Feature 3: Seat Selection State Management

**Dimension Scoring**:

| Dimension | Score | Notes |
|-----------|-------|-------|
| **Users affected** | 5 | 5-10 million daily bookings; 40-50% experience issue; mobile 65-75% |
| **Core booking flow impact** | 5 | CRITICAL PATH: Users re-select seats, lose time, abandon bookings |
| **Severity of consequence** | 5 | Mobile bookings 20-30% abandoned; direct revenue loss; high frustration |
| **Frontend effort** | 3 | Seat component redesign; state sync logic; mobile carousel handling |
| **Backend/infra effort** | 4 | Seat locking mechanism, Redis TTL, lock token management, Railway API calls |
| **Railway API dependency** | 3 | Seat lock timeout must align with Railway's 5-min reservation timeout |
| **Risk of breaking existing** | 3 | Seat locking adds complexity; must handle lock expiry gracefully; moderate risk |

**Impact Total (Rows 1-3)**: 5+5+5 = **15/15** ✅ MAXIMUM
**Effort Total (Rows 4-7)**: 3+4+3+3 = **13/20** 🟡 MODERATE-HIGH EFFORT

**Matrix Placement**: **MAJOR PROJECT** (High Impact, High Effort)

---

### Feature 4: PNR Status Page Optimization

**Dimension Scoring**:

| Dimension | Score | Notes |
|-----------|-------|-------|
| **Users affected** | 4 | 3-5 million daily checks; post-booking confirmation seekers |
| **Core booking flow impact** | 2 | NOT in booking flow; peripheral (post-booking status check) |
| **Severity of consequence** | 3 | Timeout frustrating; generates support tickets; not revenue-blocking |
| **Frontend effort** | 2 | Progressive loading UI, queue visualization; standard components |
| **Backend/infra effort** | 3 | Database indexing, query optimization, queue system, result caching |
| **Railway API dependency** | 2 | PNR data from Railway backend; IRCTC controls only caching layer |
| **Risk of breaking existing** | 2 | Optimization additive; existing PNR query still works if optimization fails |

**Impact Total (Rows 1-3)**: 4+2+3 = **9/15** 🟡 MODERATE
**Effort Total (Rows 4-7)**: 2+3+2+2 = **9/20** 🟢 LOW-MODERATE EFFORT

**Matrix Placement**: **QUICK WIN** or **FILL-IN** (Moderate Impact, Low-Moderate Effort)
→ Placement: **QUICK WIN** (satisfies >70% users, relatively fast)

---

### Feature 5: Modal Dialog Redesign

**Dimension Scoring**:

| Dimension | Score | Notes |
|-----------|-------|-------|
| **Users affected** | 3 | 30-40% of sessions see modals; ~20% blocked = 500K-1M users daily |
| **Core booking flow impact** | 2 | Not CRITICAL PATH; modals are optional UX layer (login/promo) |
| **Severity of consequence** | 2 | Users frustrated but can retry or use mobile app; not revenue loss like Tatkal |
| **Frontend effort** | 2 | Modal stack manager, accessibility fixes, ESC key handling; straightforward |
| **Backend/infra effort** | 1 | No backend changes; pure frontend CSS/JS work |
| **Railway API dependency** | 0 | Zero dependency; independent feature |
| **Risk of breaking existing** | 1 | CSS-only changes; no logic changes; very low risk |

**Impact Total (Rows 1-3)**: 3+2+2 = **7/15** 🟡 MODERATE
**Effort Total (Rows 4-7)**: 2+1+0+1 = **4/20** 🟢 VERY LOW EFFORT

**Matrix Placement**: **FILL-IN** (Moderate Impact, Very Low Effort)
→ Could also be QUICK WIN if impact valued higher

---

### Feature 6: Mobile Responsiveness Overhaul

**Dimension Scoring**:

| Dimension | Score | Notes |
|-----------|-------|-------|
| **Users affected** | 5 | 65-70% of traffic is mobile; 100% of mobile users encounter issues |
| **Core booking flow impact** | 5 | Mobile forms broken; 30-40% encounter blocking issues; critical |
| **Severity of consequence** | 4 | Mobile conversion 50% lower than desktop; massive revenue leak |
| **Frontend effort** | 4 | Responsive redesign, media queries, touch handling, drawer components |
| **Backend/infra effort** | 1 | Zero backend changes; pure frontend responsive CSS/JS |
| **Railway API dependency** | 0 | Zero dependency |
| **Risk of breaking existing** | 2 | Responsive changes could break desktop in edge cases; requires testing |

**Impact Total (Rows 1-3)**: 5+5+4 = **14/15** ✅ VERY HIGH
**Effort Total (Rows 4-7)**: 4+1+0+2 = **7/20** 🟡 MODERATE EFFORT

**Matrix Placement**: **QUICK WIN** (Very High Impact, Moderate Effort)
→ Most valuable feature for volume (65% of users)

---

## Matrix Placement Summary

```
HIGH IMPACT
     │
     │     🏗️ MAJOR PROJECTS    │  🚀 QUICK WINS
     │     (Do next, planned)   │  (Do first, fast)
     │                          │
     │     1. Tatkal Queue ✈️    │  2. Filters 🔍
     │        (15 impact,16eff) │     (11 impact,8 eff)
     │                          │
     │     3. Seats 💺 ⏳        │  6. Mobile 📱 ⚡
     │        (15 impact,13eff) │     (14 impact,7 eff)
     │                          │
     │     4. PNR ⌚ ⏳          │  5. Modals 🖼️
     │        (9 impact,9 eff)  │     (7 impact,4 eff)
     │                          │
     ├──────────────────────────┼─────────────────────
     │                          │
     │  ❌ TIME SINKS           │  🧩 FILL-INS
     │  (Reconsider/deprioritize) (Do if capacity)
     │                          │
LOW IMPACT
      LOW ────────────── EFFORT ────────────── HIGH
```

### Quadrant Breakdown

**🚀 QUICK WINS** (3 features):
1. **Search Filters Persistence** (11 impact / 8 effort)
2. **Mobile Responsiveness** (14 impact / 7 effort)
3. **PNR Status Optimization** (9 impact / 9 effort)

**🏗️ MAJOR PROJECTS** (2 features):
1. **Tatkal Virtual Queue** (15 impact / 16 effort)
2. **Seat Selection State** (15 impact / 13 effort)

**🧩 FILL-INS** (1 feature):
1. **Modal Dialog Redesign** (7 impact / 4 effort)

**❌ TIME SINKS** (0 features): None — all 6 are worth doing

---

## Execution Roadmap (Recommended Sequence)

### Phase 1: Quick Wins (Weeks 1-8)
**Goal**: Build team momentum; deliver visible improvements; minimal risk

**Sprint 1 (Weeks 1-3): Mobile Responsiveness** 🎯
- Rationale: Highest impact (14/15) per effort; 65% of users affected
- Effort: 4-5 weeks estimated; split across 3 sprints
- **Week 1**: Audit responsive design gaps; create mobile wireframes
- **Week 2-3**: Implement CSS media queries, drawer filters, touch handlers
- **Risk**: Low (CSS-only changes)
- **Success Metric**: Mobile booking completion increases 35% → 55%

**Sprint 2 (Weeks 4-6): Search Filter Persistence** 🔍
- Rationale: Second highest impact per effort (11/8); easy to deliver
- Effort: 2-3 weeks
- **Week 4**: Implement URL query params + Redux state
- **Week 5-6**: Server-side session caching, backward compatibility testing
- **Risk**: Very low (additive feature, no breaking changes)
- **Success Metric**: Filter reliability increases 60% → 95%; search time reduces 40%

**Sprint 3 (Weeks 7-8): PNR Optimization** ⌚
- Rationale: Moderate impact (9/15); quick win for support ticket reduction
- Effort: 2-3 weeks
- **Week 7**: Database query optimization, indexing
- **Week 8**: Progressive loading UI, caching layer
- **Risk**: Low (optimization is additive)
- **Success Metric**: PNR response time reduces 45s → <10s

### Phase 2: Major Projects (Weeks 9-18)
**Goal**: Tackle revenue-critical problems; complex engineering; longer runway

**Sprint 4 (Weeks 9-12): Seat Selection State Management** 💺
- Rationale: Critical path (15/15 impact); high effort but blocks mobile revenue
- Effort: 3-4 weeks
- **Week 9**: Seat locking mechanism design, Redis setup
- **Week 10-11**: Frontend state sync, backend seat lock API
- **Week 12**: Mobile carousel testing, edge case handling
- **Risk**: Moderate (new lock system; requires thorough testing)
- **Success Metric**: Seat reset rate drops 50% → <5%; mobile abandonment 25% → 10%
- **Dependency**: May need coordination with Railway API for lock timing

**Sprint 5 (Weeks 13-20): Tatkal Virtual Queue** ✈️
- Rationale: Highest impact (15/15); largest revenue impact (₹10 crores/day)
- Effort: 6-8 weeks
- **Week 13-14**: Queue infrastructure (Redis, WebSocket server, queue controller)
- **Week 15-16**: Backend queue logic, session token binding, Railway integration
- **Week 17-18**: Frontend real-time UI, testing under load, fallback flows
- **Week 19-20**: Canary deployment, monitoring, full rollout
- **Risk**: High (critical system; affects 20-40 lakh users daily at 10 AM)
- **Success Metric**: Tatkal success rate 40% → 70%; error rate 35% → <5%
- **Testing**: MUST load test with 40 lakh concurrent requests before launch

### Phase 3: Fill-Ins (Weeks 19-20, if capacity)
**Goal**: Polish UX with accessibility and accessibility features

**Sprint 6 (Weeks 19-20): Modal Dialog Redesign** 🖼️
- Rationale: Quick quality-of-life improvement; very low effort
- Effort: 1-2 weeks
- **Week 19**: Modal stack manager, accessibility fixes (ARIA), ESC handling
- **Week 20**: Keyboard navigation, form state preservation
- **Risk**: Very low (CSS/JS only)
- **Success Metric**: Modal-blocked interactions drop 20% → <2%
- **Bonus**: Accessibility compliance (WCAG AA)

---

## 3-Sentence Justification per Placement

### Quick Wins

**Search Filter Persistence (High Impact, Low Effort):**
This problem affects all 2-5 million daily search users, making it the highest-volume issue in the set. The fix requires persisting filter state to session storage and URL parameters, then reapplying on page reload — pure frontend work with zero backend complexity or Railway API dependencies. No infrastructure risk; a focused 2-week sprint eliminates 40% of user search frustration entirely.

**Mobile Responsiveness Overhaul (Very High Impact, Moderate Effort):**
Mobile traffic represents 65% of IRCTC users but has 50% lower booking completion than desktop, creating the largest volume-weighted pain point. The redesign requires CSS media queries, touch-optimized controls (44x44px targets), and drawer-pattern filters — well-understood frontend patterns with zero backend work. Completing this single feature could recover ₹1-2 crores daily in mobile bookings with zero infrastructure risk.

**PNR Status Optimization (Moderate Impact, Low Effort):**
The PNR status page times out in 25-35% of checks, generating 10-15% of support tickets monthly with zero revenue impact but high user friction. Optimization requires database indexing and result caching on IRCTC's side — the Railway backend latency is beyond our control, but client-side queuing and progressive loading eliminate the perception of hang. A 2-3 week effort dramatically improves post-booking user experience.

---

### Major Projects

**Seat Selection State Management (High Impact, High Effort):**
Seat resets affect 40-50% of desktop bookings and 65-75% of mobile bookings, directly causing 20-30% booking abandonment on mobile and driving users to competitor platforms. The solution requires server-side seat locking with Redis, client-side state synchronization, and careful Railway API timeout coordination — a 3-4 week engineering effort but directly recovers mobile revenue. The effort is high but justified by impact; this is the second-highest revenue lever after Tatkal optimization.

**Tatkal Virtual Queue System (High Impact, High Effort):**
Tatkal crashes affect 20-40 lakh users daily at 10:00 AM with 40-60% failure rate, directly costing IRCTC ₹10+ crores daily in unprocessed bookings and driving user churn to competitors. The solution requires Redis-backed queue infrastructure, WebSocket real-time updates, and careful coordination with Railway's seat reservation API — a 6-8 week dedicated engineering sprint that is the single highest-leverage feature in the set. The effort is justified by the magnitude of impact; Tatkal is IRCTC's core value proposition.

---

### Fill-Ins

**Modal Dialog Redesign (Moderate Impact, Very Low Effort):**
Modals block form interactions in 20% of user sessions and frustrate users who cannot find close buttons or navigate with keyboards, but the issue is not revenue-blocking like Tatkal or seat resets. Converting modals to top banners and implementing ESC-key dismissal requires only 1-2 weeks of pure frontend CSS/JavaScript work with zero backend or Railway API dependencies. This feature should be completed during downtime between major projects to incrementally improve overall UX and WCAG accessibility compliance.

---

## Risk Mitigation & Dependencies

### Critical Dependencies

```
Dependency Graph:
┌─────────────────────────┐
│ Phase 1: Quick Wins     │ (Independent, can run in parallel)
│ • Mobile (weeks 1-3)    │
│ • Filters (weeks 4-6)   │
│ • PNR (weeks 7-8)       │
└────────────┬────────────┘
             │
             ↓
┌─────────────────────────────────────────────┐
│ Phase 2: Major Projects                    │
│ ┌──────────────────┐  ┌──────────────────┐ │
│ │ Seat Locks       │→ │ Tatkal Queue     │ │ (Seats must be before Tatkal)
│ │ (weeks 9-12)     │  │ (weeks 13-20)    │ │ (Queue depends on stable seats)
│ └──────────────────┘  └──────────────────┘ │
└─────────────────────────────────────────────┘
             │
             ↓
┌─────────────────────────┐
│ Phase 3: Fill-Ins       │ (Can run anytime)
│ • Modals (weeks 19-20)  │
└─────────────────────────┘
```

### Risk Ranking

| Feature | Risk Level | Mitigation |
|---------|-----------|-----------|
| Mobile Responsive | 🟢 Very Low | CSS only; extensive testing on real devices |
| Filter Persistence | 🟢 Very Low | URL params backward compatible; canary deploy 5% first |
| PNR Optimization | 🟢 Low | Index safely added; existing query still works if optimization fails |
| Modal Redesign | 🟢 Very Low | Pure CSS/JS; no business logic changes |
| Seat Locking | 🟡 Moderate | New lock system; must handle lock expiry gracefully; thorough QA required |
| Tatkal Queue | 🔴 High | CRITICAL SYSTEM; 40 lakh concurrent users at peak; MUST load test extensively |

### Pre-Launch Checklist for High-Risk Features

**Tatkal Queue (Weeks 19-20 before launch):**
```
✓ Load testing: Simulate 40 lakh concurrent users
✓ Failover testing: Redis fails → fall back to direct booking
✓ WebSocket testing: 50K+ concurrent connections
✓ Mobile testing: Queue works on 2G/3G with polling fallback
✓ Railway API testing: Queue load doesn't exceed Railway API rate limits
✓ Canary deploy: 5% users first; monitor for 1 week before 100%
✓ Rollback plan: Documented; practiced; runnable in <10 minutes
```

**Seat Locking (Week 12 before production):**
```
✓ Lock expiry testing: User's lock expires mid-booking
✓ Mobile carousel testing: Seat selections don't reset on coach swipe
✓ Back-button testing: Navigation preserves seat state
✓ Railway integration testing: Lock timeout aligns with Railway's 5-min window
✓ Edge case testing: Simultaneous seat selections, lock conflicts
```

---

## Financial Justification

### Expected ROI by Feature

| Feature | Effort | Impact | ROI | Timeline |
|---------|--------|--------|-----|----------|
| **Mobile Responsive** | 3-4w | ₹100-150 cr/year | ✅✅✅ HIGHEST | Weeks 1-3 |
| **Filter Persistence** | 2-3w | ₹50-75 cr/year | ✅✅✅ HIGH | Weeks 4-6 |
| **Seat Locking** | 3-4w | ₹80-120 cr/year | ✅✅✅ HIGH | Weeks 9-12 |
| **Tatkal Queue** | 6-8w | ₹300-500 cr/year | ✅✅✅✅ VERY HIGH | Weeks 13-20 |
| **PNR Optimization** | 2-3w | ₹20-30 cr/year | ✅✅ MEDIUM | Weeks 7-8 |
| **Modal Redesign** | 1-2w | ₹10-15 cr/year | ✅ LOW | Weeks 19-20 |

**Total estimated annual ROI**: ₹560-890 crores
**Total effort**: 17-25 weeks (4-6 months)
**Cost per ₹1 crore recovered**: ~₹6 lakhs per crore

---

## Peer Review Preparation

### Questions to Expect

**On Tatkal Queue:**
- "What if Redis fails at 9:59 AM (one minute before Tatkal opens)?"
  **Answer**: Fallback to direct booking with aggressive rate limiting (500 req/s). No booking blocked; just slower processing. Better than crash.

- "How do we test 40 lakh concurrent users without crashing production?"
  **Answer**: Load test on staging environment with synthetic traffic. CloudFlare or AWS can simulate concurrent users without hitting real Railway backend.

- "Why not just add more servers?"
  **Answer**: Server scaling won't help if root cause is queue management. 40 lakh users hitting same API endpoint simultaneously will still saturate any number of servers. Virtual queue distributes load over 15 minutes.

**On Mobile Responsiveness:**
- "Why prioritize mobile over Tatkal if Tatkal has higher revenue impact?"
  **Answer**: Mobile has equal revenue impact with 1/3 the effort. Quick wins build team momentum; Tatkal requires 6-8 weeks of focused work. Execute mobile first to show progress.

**On Seat Locking:**
- "What if user's lock expires mid-booking and they've already filled passenger details?"
  **Answer**: Show warning "Your seat lock expires in 2 minutes" at 8-minute mark. Offer one-click "Extend lock +5 mins". If expired, show "Seat was released. [Re-select seats] or [Book different seats]".

---

## Summary: Prioritisation Principle

**Execute in this order:**
1. **Mobile Responsive** → Highest impact per effort (14/7)
2. **Filters** → High impact per effort (11/8); build on mobile foundation
3. **PNR** → Moderate impact (9/9); quick support win
4. **Seat Locking** → Critical path (15/13); enables Tatkal
5. **Tatkal Queue** → Highest absolute impact (15/16); complex but highest ROI
6. **Modals** → Polish (7/4); low effort, anytime

**Resource Model**: 
- Phase 1: 1 team (frontend + backend) = 8 weeks
- Phase 2: 2 teams (parallel Seat + Tatkal = 8 weeks)
- Phase 3: As needed = 2 weeks

**Timeline**: 4-6 months for full execution

---

**Next Section**: Move to peer review session documentation

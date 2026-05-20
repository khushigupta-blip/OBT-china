# Mobile Responsive Changes (Galaxy S20 Ultra — 412×915px)

Target viewport: **412×915px** (Samsung Galaxy S20 Ultra). Breakpoint strategy: **mobile-first** with Tailwind `max-md` (&lt;768px) for phone-only UI and **`md:`+** unchanged for desktop.

---

## Task 1 — Mobile header refactor

### Structural change

Replaced the stacked mobile header (duplicate JW + name, Ask Us, Contact, bell, language, profile, menu) with a single row:

| Zone | Content |
|------|---------|
| Left | `#mobile-nav-toggle` (hamburger) → `window.toggleSidebar()` |
| Right | JW profile chip + `data-profile-dropdown` |

### Tailwind mapping

| Element | Classes | Breakpoint |
|---------|---------|------------|
| Mobile header row | `flex justify-between items-center px-4` | `md:hidden` |
| Hamburger | `min-w-[44px] min-h-[44px]` (touch target) | mobile only |
| Profile button | existing rounded-full JW chip | mobile only |
| Desktop header block | `hidden md:flex md:items-center md:gap-4` | **unchanged** |
| Global search | `md:flex-1 md:max-w-sm` | shared; layout unchanged at `md:` |

### Removed on mobile only

- Top-left JW avatar + “John Wick” label
- Static **Ask Us** text button in the header
- Contact Service Team, bell, and language controls from the mobile header row

### Preserved

- **`#fab-ask-us`** — floating bottom-right Ask Us (`fixed z-[35] … md:hidden`); not moved or hidden
- All desktop (`md:`+) header markup and behavior
- `window.toggleSidebar`, `window.toggleProfileDropdown`, `window.toggleAskUsChat` — no changes to filter/search/session logic

---

## Task 2 — Airbnb-style flight filter drawer (mobile only)

### Structural change

1. **Mobile trigger** — `#mobile-flight-filters-open` (`md:hidden`, sticky above results)
2. **Backdrop** — `#flight-filter-drawer-backdrop` (`fixed inset-0 z-[60] bg-black/50 backdrop-blur-sm`, `md:hidden`)
3. **Same filter DOM** — `#online-flight-filter-sidebar` holds all existing controls (single set of IDs/checkboxes/sliders for `applyFlightFilters` / `clearFlightFilters` / `populateFlightAirlineFilter`)

### Why not `hidden md:block` on a duplicate sidebar

Duplicating filter inputs would break `getElementById` and `querySelectorAll('[data-flight-filter]')`. Instead:

| Viewport | Sidebar behavior |
|----------|------------------|
| **&lt; md** | `max-md:fixed max-md:inset-0 max-md:w-full max-md:z-[61]`; off-screen `translate-y-full`; slides to `translate-y-0` when `body.flight-filter-drawer-open` |
| **≥ md** | Original layout: in-flow `mb-4`, `lg:absolute lg:left-0 lg:top-0 lg:bottom-0 lg:w-[260px]`, parent `lg:pl-[276px]` — **unchanged** |

Equivalent to “hide static sidebar on mobile, show on desktop” without cloning controls.

### Drawer chrome (mobile only — `md:hidden`)

| Area | UI |
|------|-----|
| Header | Title “Filters”, **Clear All** → `clearFlightFilters()`, **X** → `closeFlightFilterDrawer()` |
| Body | Existing Stops, Departure Time, Airlines, Price Range, CO₂, Luggage, Meal blocks |
| Footer | Sticky **Apply Filters** → `applyFlightFiltersAndCloseDrawer()` |

Desktop filter header row uses `hidden md:flex` so the in-sidebar title/Clear All row is unchanged on `md:`+.

### Animation

```css
@media (max-width: 767px) {
  #online-flight-filter-sidebar {
    transform: translateY(100%);
    transition: transform 0.3s ease-in-out;
  }
  body.flight-filter-drawer-open #online-flight-filter-sidebar {
    transform: translateY(0);
  }
}
```

Tailwind on the aside: `max-md:translate-y-full max-md:transition-transform max-md:duration-300`, `md:translate-y-0`.

### New UI-only helpers (no changes to filter algorithms)

- `window.openFlightFilterDrawer()`
- `window.closeFlightFilterDrawer()`
- `window.applyFlightFiltersAndCloseDrawer()` — calls existing `applyFlightFilters()` then closes

Resize handler closes the drawer when crossing to `md:`+.

---

## Breakpoint reference

| Token | Width | Usage |
|-------|-------|--------|
| `max-md` | &lt;768px | S20 Ultra, mobile header, filter drawer, FAB |
| `md:` | ≥768px | Desktop header, visible filter sidebar |
| `lg:` | ≥1024px | Absolute filter column + results offset (unchanged) |

---

## Files touched

- `OBT China.html` — header, flight results filter UI, scoped CSS, drawer open/close helpers
- `MOBILE_RESPONSIVE_CHANGES.md` — this document

## Verification checklist (412×915)

- [ ] Header: menu left, JW profile right; no duplicate JW or header Ask Us
- [ ] FAB Ask Us visible bottom-right over scroll content
- [ ] Flight search → results → **Filters** opens full-height sheet
- [ ] Backdrop tap, X, and Apply close drawer; Clear All resets controls
- [ ] ≥768px: header and left filter sidebar match pre-change desktop layout

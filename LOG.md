# Debug log

Your notes. One entry per bug you fixed, using the template below.

This file is read as carefully as your code. A correct fix you cannot
explain counts for little; a bug you could not fix but investigated
honestly still counts for something.

Delete the example before you submit.

## Example — delete this

### CC-99 — "The cart total is wrong"

**Reproduced:** Added 2 dosas at Rs. 60 each. The cart showed
Rs. 119.99999 instead of Rs. 130. Happened every time, on any dish with
a price ending in .50.

**Cause:** The total was being added up with plain floating point and
never rounded, so 0.1 + 0.2 style errors showed up on screen. The
rounding helper existed but this one place was not using it.

**Fix:** Ran the total through the existing rounding helper instead of
adding a new one, so every price on screen goes through the same path.

**Checked:** Cart, checkout and the order screen all show Rs. 130 now.
Prices without decimals still show without a trailing .00.

**Time:** about 40 minutes, most of it working out that the cart and the
order screen round in different places.

## CC-01 — "Search suggestions appear behind page elements and cannot be clicked"

**Reproduced:**Typed a query into the dish search bar to trigger the dropdown list. The suggestion list rendered visually underneath subsequent sibling elements and cards on the page. Only the uppermost edge was clickable, while clicks on the rest of the list failed or registered on underlying page content.

**Cause:**The search suggestions dropdown was trapped in a lower stacking context. Sibling containers below the search bar either appeared later in the DOM order with their own positioning contexts or had higher default z-index values, causing them to paint on top of the suggestions.

**Fix:**Elevated the stacking context of the parent search container by applying explicit relative positioning (position: relative) and increasing its z-index (e.g., z-index: 50 or higher), ensuring the dropdown renders above all adjacent page sections.

**Checked:**Verified the suggestion dropdown opens fully over adjacent cards, headers, and content without clipping.
Confirmed all items in the dropdown list are clickable across their entire hit area.
Verified no unintended overlap or visual clipping occurs on smaller screen sizes.

**Time:**5 mins

## CC-02 — "Can't read anything in dark mode"

**Reproduced:**Switched the interface to dark mode (or loaded the page with a dark theme active). While backgrounds and dividers shifted, the dish names and prices remained dark, rendering them almost invisible against the dark card background.

**Cause:**The typography in .dish-body lacked dynamic color handling that adapts to the active theme, resulting in insufficient contrast against dark backgrounds.

**Fix:**Updated .dish-body in style.css using the native CSS light-dark() function (color: light-dark(#111111, #eeeeee);), automatically serving dark text (#111111) in light mode and high-contrast light text (#eeeeee) in dark mode.

**Checked:**Toggled between light and dark modes to verify dish names and prices remain clearly legible in both views.
Confirmed that the grey divider line and surrounding card styles remain intact.
Verified text does not disappear or wash out when switching themes.

**Time:**5 mins

## CC-03 — "The menu is wider than my phone"

**Reproduced:**Set responsive viewport below 320px down to 240px. While min-width: 0; resolved the issue on standard mobile widths (320px+), sub-260px viewports caused internal row layouts (price and "Add to Cart" button) to collide and push beyond the card boundary.

**Cause:**Fixed rigid child flex rows inside .dish-card exceeded the sub-260px viewport width, preventing nested elements from wrapping vertically.

**Fix:**Retained min-width: 0; on .dish-card to resolve default flex container expansion.

**Checked:**Confirmed layout scales smoothly and without horizontal overflow from 240px up to desktop viewports.
Verified buttons stack cleanly beneath prices without clipping off-screen.

**Time:**7 mins

## CC-04 — "The buttons don't work on my tablet"

**Reproduced:**Tested button interactions on tablet screens and touch viewports. Clicks and taps on the "Add to Cart" and star buttons failed to trigger any action, while hovering/clicking on certain desktop mouse coordinates occasionally worked.

**Cause:**.dish-card::after was styled with position: absolute; bottom: 0; height: 58px; z-index: 3; to create an overlay over the bottom section of the card. Because it lacked pointer-events: none;, it created an invisible physical barrier directly above the buttons that intercepted all click and tap events.

**Fix:**Added pointer-events: none; to .dish-card::after in style.css, allowing all click and touch gestures to pass cleanly through to the underlying action buttons.

**Checked:**Confirmed "Add to Cart" and favorite/star buttons fire properly on touch and click.
Verified visual transitions and blocked state backgrounds on .dish-card::after continue to render without issues.
Confirmed behavior is consistent across desktop, tablet, and mobile views.

**Time:**10 mins

## CC-05 — "The category bar scrolls away on my phone"

**Reproduced:**Tested on mobile screens. Scrolled down through the dish list; the .filters and .cat-tabs bars scrolled out of view along with the rest of the page rather than remaining anchored at the top of the viewport.

**Cause:**html, body had overflow-x: hidden; applied. In CSS, applying overflow: hidden (or overflow-x: hidden) to html or body disables standard viewport scroll context for descendants, causing all position: sticky rules on child elements to fail silently.

**Fix:**Replaced overflow-x: hidden; on html, body with overflow-x: clip;. This cuts off unwanted horizontal overflow without interfering with viewport scrolling contexts, restoring normal position: sticky behavior for the navigation and category bars.

**Checked:**Confirmed the category/filter bar sticks firmly to the top while scrolling through the full dish menu on mobile viewports (< 480px).
Verified no horizontal page scrolling or layout spillage occurs.
Confirmed category pill selection and horizontal tab scrolling continue to operate normally while pinned.

**Time:**10 mins

## CC-06 — "I ordered more than they had"

**Reproduced:**Selected a dish showing 2 items remaining in stock (e.g., Samosa). Incremented the order quantity to 5 and completed checkout. The application allowed the order to go through without throwing a validation error or capping the order quantity.

**Cause:**Absence of a conditional check comparing requested quantity against available inventory (dish.stock < item.qty / newQty > dish.stock). The addition and increment logic executed unconditionally, allowing the cart quantity to surpass actual kitchen availability.

**Fix:**Added a guard condition to verify stock limits before modifying cart state:if (dish.stock < item.qty) {
return errors.push(`Only ${dish.stock} left of ${dish.name}, you asked for ${item.qty}`);
}

**Checked:**Verified the quantity increment button blocks or rejects additions beyond remaining stock (e.g., stops at 2).
Attempted manually inputting a higher quantity; verified it is rejected or clamped to dish.stock.
Verified toast/warning displays properly informing the user of the inventory ceiling.

**Time:**25 mins

## CC-07 — "Cancelling makes it worse"

**Reproduced:**Noted a dish's stock in the menu data (10). Placed an order for 3 of it, and stock dropped to 7, which is correct. Cancelled the order by changing its status to "cancelled". Stock went to 4 instead of back to 10. Repeating place-and-cancel pushed it toward 0 and then negative.

**Cause:**The `releaseStock` function had `stock: dish.stock - line.qty`. It subtracted the cancelled quantity instead of adding it back, so every cancel took more plates off the shelf. The cancel route was correct: it calls `releaseStock` with the order's items and saves the result.

**Fix:**Changed the line to `stock: dish.stock + line.qty`. I changed it in `releaseStock` because that function decides how stock changes on cancel, and the route only calls it. A cancel now undoes exactly what `reserveStock` took when the order was placed.

**Checked:**Reset the data, restarted the server, placed an order for 3 and cancelled it. Stock returned to 10. Repeated place-and-cancel several times and stock stayed at 10 each time. Also checked that an order with several dishes restores every dish on cancel.

**Time:**30 mins

## ## CC-09 — "The menu shows more dishes than it should"

**Reproduced:**
Requested the menu with a small limit (`/api/menu?limit=3`). The response said `limit: 3`, `pages: 13` and `hasMore: true`, but the `dishes` array contained all 37 dishes. The front page was also sending `page=1&limit=5` correctly, so the browser was asking for a small page and the server was still sending everything.

**Cause:**
In the `paginate` function, the page of dishes was cut correctly into `items` with `list.slice(start, start + perPage)`, but the return statement had `items: list`, which sent back the full list instead of the sliced one. The menu route passes `paged.items` to the browser as `dishes`, so every request returned the whole menu. The page numbers (`pages`, `hasMore`, `total`) were calculated correctly, which is why the response looked right but held too many dishes.

**Fix:**
Changed `items: list` to `items` in the return statement of `paginate`, so it returns the sliced page. I fixed it there because `paginate` is the one function that builds a page, and both the route and the frontend were already correct.

**Checked:**
Restarted the server and reloaded the front page. The menu request with `page=1&limit=5` now returns only 5 dishes, and more load as I scroll. Requested `limit=3` pages 1, 2 and 13 and got 3, 3 and 1 dishes with no repeats, `total` 37, and `hasMore` true, true and false. I noticed a `page=2` request arriving before the page loaded. That came from an old tab that was still open, and a fresh tab starts at `page=1`, so it is not a separate bug.

**Time:** 40 mins

## CC-10 — "Sorting by price is backwards"

**Reproduced:**
Requested the menu with `sort=price-asc` and got the most expensive dishes first, and the reverse for `price-desc`. Even after that was fixed, picking "Price: low to high" in the browser still showed the default order. The server log showed `sort=price-asc&page=2`, with no page 1 request being used, and the header said "Showing 20 dishes, first Rs. 66".

**Cause:**
Two problems. (1) In `SORTERS` in the search logic, the comparators for `price-asc` and `price-desc` were swapped. (2) In `menu.js`, changing the sort started a fresh load, but while it ran, the grid was replaced by skeleton cards, the scroll sentinel became visible, and the infinite-scroll observer fired because `state.hasMore` was still true from the old list. It bumped the page to 2 and started an append load, which made the page-1 request stale. The page-2 response was then appended to the old, unsorted list.

**Fix:**
(1) Swapped the comparators back so `price-asc` is `a.price - b.price` and `price-desc` is `b.price - a.price`. (2) In `loadMenu`, a fresh (non-append) load now sets `state.hasMore = false` at the start, so the observer can't fire during it. The next response sets `hasMore` again. I fixed it in `loadMenu` because that is the one place that knows a fresh load has started.

**Checked:**
Restarted the server, hard refreshed, and chose both sort options. The request is now `page=1` and the list starts at 16.50 for low to high and 138.50 for high to low. Scrolling loads further pages in the same order.

**Time:**
About 40 minutes.

## CC-0X — "<the complaint, in short>"

**Reproduced:**

**Cause:**

**Fix:**

**Checked:**

**Time:**

## Could not fix

For anything you investigated but did not solve. Say what you tried and
where you got to. This is worth marks — leaving it blank when you got
stuck is not.

### CC-08: "An old coupon still works"

**What I tried:**searched the codebase for FRESHERS24 and the located the coupon validation function in the checkout services.checked the coupoun database schema to confirm if an expiryDate field exists.

**Where I got to:**found that the validation logic only check if the coupon code exist and is active .

**What I would try next:**add an expiration condition in the validation handler. add a test case for expired coupon to ensure the API return 400 validation error.

## Extra credit

Anything not on the bug log: a problem you found yourself, a test you
wrote, or a fix you are unsure about. Same format, plus one line on how
you noticed it.

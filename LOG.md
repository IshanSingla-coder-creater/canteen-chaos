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

**Checked:**

**Time:**

## CC-0X — "<the complaint, in short>"

**Reproduced:**

**Cause:**

**Fix:**

**Checked:**

**Time:**

## CC-0X — "<the complaint, in short>"

**Reproduced:**

**Cause:**

**Fix:**

**Checked:**

**Time:**

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

### CC-0X — "<the complaint>"

**What I tried:**

**Where I got to:**

**What I would try next:**

## Extra credit

Anything not on the bug log: a problem you found yourself, a test you
wrote, or a fix you are unsure about. Same format, plus one line on how
you noticed it.

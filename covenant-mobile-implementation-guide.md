# Covenant Mobile Fixes — Step-by-Step Implementation Guide

**For:** Michael Burton (and Dominik for the Vercel pieces) · **Date:** October 17, 2026
**Companion to:** `covenant-mobile-audit-and-project-plan.md`

Order below is the recommended build order. Each step has Where / Do / Test / Roll back and a time estimate. Everything in Steps 1 to 7 is done in BigCommerce admin or Script Manager, with no theme file edits and no changes to MiniBC, Searchanise or checkout. Steps 8 onward touch the Vercel apps and the theme.

**Conventions used below**
- "Script Manager" means BigCommerce admin → Storefront → Script Manager → Create a Script. Unless noted: Location **Footer**, Pages **All pages**, Category **Essential**, Type **Script**. Paste the code inside `<script></script>` tags.
- Every script starts with the same guard your existing scripts use, so it stays dormant on checkout and any page without the Cornerstone header.
- Test on a real phone after each step, not only in a desktop browser's device mode. Cloudflare is challenging automated browsers right now (your WAF JA4 rules), so a real phone is the honest test.

---

## Step 0 — Before you change anything (30 min)

1. **Theme backup.** Storefront → Themes → on the active theme's menu choose **Make a copy**. Name it `Cornerstone CHP backup 2026-10-17`.
2. **Script inventory.** Storefront → Script Manager. Screenshot the list. You have roughly 35 scripts; you will add two or three and edit one.
3. **Baseline numbers.** In BigCommerce Analytics → Carts, note today's mobile cart abandonment. In Ahrefs Web Analytics (or GA4) note the 30-day smartphone cart → checkout rate (44% on Oct 17).
4. **Test accounts.** Have one customer login with past orders handy (for Rapid Order testing) and one fresh browser session (for guest flow).

Roll back for any step below: open the script in Script Manager and switch it to **Disabled**. Nothing in Steps 1 to 7 is destructive.

---

## Step 1 — "Continue shopping" and a clear cart page on phones (1.5 h) ★ start here

**Why first:** the cart is the #2 mobile exit page and 56% of mobile cart viewers never reach checkout. The customer's "no way to go back and add stuff" comment is this.

**Where:** Script Manager → Create a Script → name `CHP cart: continue shopping + coupon up top`. Location Footer, Pages All pages (the code only acts on `/cart.php`).

**Do:** paste this script. It (a) remembers the last listing page the shopper was on, (b) adds a full-width "Continue shopping" button above the cart items, (c) moves the coupon line above the totals with a plain-language label, (d) adds a "Reorder a past item" link for signed-in customers, and (e) re-applies itself when the cart redraws after a quantity change.

```html
<script>
(function () {
  if (document.querySelector('link[href*="fusionbot"],script[src*="fusionbot"]') || !document.querySelector('header.header, div.body')) return;

  // (a) remember the last place the shopper was browsing products
  var isListing = document.querySelector('#product-listing-container, .productGrid, .snize-search-results-wrapper, .brand-list, .category-list');
  if (isListing) { try { sessionStorage.setItem('chpLastListing', location.href); } catch (e) {} }

  if (!/^\/cart\.php/.test(location.pathname)) return;

  function fallbackHref() {
    try { var v = sessionStorage.getItem('chpLastListing'); if (v) return v; } catch (e) {}
    if (document.referrer && document.referrer.indexOf(location.host) > -1 && !/cart\.php|checkout|login/.test(document.referrer)) return document.referrer;
    return '/clinic-recommended-products/';
  }

  function apply() {
    var cart = document.querySelector('.cart, [data-cart]');
    if (!cart) return;

    // (b) Continue shopping button above the items
    if (!document.getElementById('chp-continue')) {
      var bar = document.createElement('div');
      bar.id = 'chp-continue';
      bar.innerHTML =
        '<a class="button button--secondary" href="' + fallbackHref() + '" style="width:100%;margin:0 0 10px">&#8592; Continue shopping</a>' +
        '<a class="chp-reorder-link" href="/account.php?action=order_status#rapid-order">Signed in before? Reorder a past item</a>';
      var heading = document.querySelector('.page-heading');
      (heading ? heading.parentNode : cart.parentNode).insertBefore(bar, heading ? heading.nextSibling : cart);
    }

    // (c) coupon row to the top of the totals, with a plain label
    var couponRow = document.querySelector('.cart-totals .cart-total:has([data-coupon-code-add]), .cart-totals .cart-total:has(.coupon-code)');
    var totals = document.querySelector('.cart-totals');
    if (couponRow && totals && totals.firstElementChild !== couponRow) {
      totals.insertBefore(couponRow, totals.firstElementChild);
      var lbl = couponRow.querySelector('.cart-total-label strong');
      if (lbl) lbl.textContent = 'Coupon or clinic code';
      var add = couponRow.querySelector('[data-coupon-code-add]');
      if (add) add.textContent = 'Enter code';
    }
  }

  var css = document.createElement('style');
  css.textContent =
    '#chp-continue{margin:0 0 16px}' +
    '.chp-reorder-link{display:block;text-align:center;font-size:14px;color:#5a7d16;text-decoration:underline;text-underline-offset:3px}' +
    '@media (max-width:800px){.cart-actions .button{width:100%}}';
  document.head.appendChild(css);

  apply();
  // (e) the cart redraws after quantity changes; re-apply
  var target = document.querySelector('[data-cart]') || document.body;
  new MutationObserver(function () { apply(); }).observe(target, { childList: true, subtree: true });
})();
</script>
```

**Test (phone):** add two items, open the cart. You should see "Continue shopping" directly under the "Your Cart" heading, the coupon line first in the totals, and a reorder link. Tap + on a quantity; the button must still be there after the cart redraws. Tap "Continue shopping"; it should return to the category or search you came from.

**If `:has()` is not supported** on an older Android browser the coupon move silently does nothing; the Continue button still works. That is acceptable.

**Roll back:** disable the script.

---

## Step 2 — Rewrite the delivery choice ("Recurring Order: (Required)") (45 min)

**Why:** the option works and defaults to One Time Purchase. The label reads as a demand to subscribe. Copy fix only. Do not rename the shared option in Products → Product Options; MiniBC matches on the option text.

**Where:** Script Manager → Create a Script → name `CHP PDP: delivery wording`. Location Footer, Pages All pages.

**Do:**

```html
<script>
(function () {
  if (document.querySelector('link[href*="fusionbot"],script[src*="fusionbot"]') || !document.querySelector('header.header, div.body')) return;
  function fix() {
    document.querySelectorAll('form[data-cart-item-add] .form-label').forEach(function (label) {
      if (!/recurring order/i.test(label.textContent)) return;
      if (label.dataset.chpDone) return;
      label.dataset.chpDone = '1';
      // keep the <small> node for layout but empty it, then set the visible text
      var small = label.querySelector('small'); if (small) small.textContent = '';
      label.childNodes.forEach(function (n) { if (n.nodeType === 3) n.textContent = ''; });
      label.insertBefore(document.createTextNode('Delivery'), label.firstChild);
      // helper line under the control
      var field = label.closest('.form-field');
      if (field && !field.querySelector('.chp-delivery-help')) {
        var p = document.createElement('p');
        p.className = 'chp-delivery-help';
        p.textContent = 'One-time is the default. Choose auto-refill to have it shipped on a schedule you pick.';
        field.appendChild(p);
      }
      // friendlier option text (display only)
      field && field.querySelectorAll('select option').forEach(function (o) {
        if (/^one time purchase$/i.test(o.textContent.trim())) o.textContent = 'One-time purchase';
        else if (/^every /i.test(o.textContent.trim())) o.textContent = 'Auto-refill ' + o.textContent.trim().toLowerCase();
      });
    });
  }
  var css = document.createElement('style');
  css.textContent = '.chp-delivery-help{font-size:13.5px;color:#5E664F;margin:6px 0 0;text-align:center}';
  document.head.appendChild(css);
  fix();
  new MutationObserver(fix).observe(document.body, { childList: true, subtree: true });
})();
</script>
```

**Check with MiniBC first:** the script changes only what is displayed in the `<option>` text; the submitted value is unchanged. Place one test subscription order after enabling it and confirm it shows up in MiniBC as a subscription. If MiniBC reads the option text from the DOM (unlikely but possible), remove the option-text block and keep only the label and helper changes.

**Test (phone):** product page shows "Delivery" with no "(Required)", the dropdown reads "One-time purchase", helper line beneath. Add to cart still works. Confirm "Cancel any time" is not promised anywhere until Dominik's three yes/no questions are answered.

**Also:** if you want "(Required)" gone from Size too, that string is `common.required` in Storefront → Themes → Advanced → Edit Theme Files → `lang/en.json`. Changing it there removes it everywhere; leave it if you prefer sizes to stay marked.

---

## Step 3 — Make Rapid Order the first thing customers see, and link to it (1.5 h + 1 h Klaviyo)

**Facts from the current script** (the `chp-order-history` Script Manager script): it runs on My Account (which opens on the Orders tab) and appends the "Rapid Order" and "Complete Order History" sections **after** the BigCommerce order list, so on a phone they sit below the account navigation and the orders. Nothing in the header, menu, cart or emails links to them.

**3a. Move Rapid Order to the top (Dominik, 30 min).** In that script, find where the Rapid Order section is added:

```js
host.appendChild(wrap);   // inside renderReorder()
```

change to:

```js
wrap.id = 'rapid-order';
host.insertBefore(wrap, host.firstChild);   // Rapid Order first, above the order list
```

Leave `render()` (Complete Order History) as `appendChild` so it stays below the orders.

**3b. Fix the silent redirect (Dominik, 20 min).** In `addToCart()`:

```js
if (!r.ok) { if (url) { window.location.href = url; return; } throw new Error('cart ' + r.status); }
```

change to:

```js
if (!r.ok) { throw new Error('cart ' + r.status); }
```

and in the `.catch`:

```js
.catch(function (err) {
  btn.textContent = 'Couldn’t add — open product';
  btn.disabled = false;
  btn.onclick = function () { if (url) window.location.href = url; };
  try { fetch(OPTS_API.replace('/api/opts', '/api/log'), { method: 'POST', headers: {'Content-Type':'application/json'}, body: JSON.stringify({ pid: pid, vid: vid, err: String(err) }) }); } catch (e) {}
});
```

(Add a tiny `/api/log` route in the Vercel app, or drop that line and read failures from the browser console for now.)

**3c. Entry points (Michael, 40 min).** New Script Manager script `CHP reorder links`:

```html
<script>
(function () {
  if (document.querySelector('link[href*="fusionbot"],script[src*="fusionbot"]') || !document.querySelector('header.header, div.body')) return;
  var RO = '/account.php?action=order_status#rapid-order';
  // mobile mega menu: add "Reorder" next to Login
  function menu() {
    var list = document.querySelector('.chp-mega__list'); if (!list || document.getElementById('chp-menu-reorder')) return;
    var li = document.createElement('li'); li.className = 'chp-mega__item chp-mega__item--mobile'; li.id = 'chp-menu-reorder';
    li.innerHTML = '<a class="chp-mega__top" href="' + RO + '">Reorder</a>';
    var login = list.querySelector('a[href*="login.php"]'); (login ? login.parentElement : list).insertAdjacentElement('beforebegin', li);
  }
  // order confirmation: reorder next time
  function confirmation() {
    if (!/\/checkout\/order-confirmation/.test(location.pathname) || document.getElementById('chp-conf-reorder')) return;
    var box = document.createElement('div'); box.id = 'chp-conf-reorder';
    box.style.cssText = 'margin:18px 16px;padding:14px 16px;border:1px solid #E1E7D2;border-radius:10px;background:#F5F8EC;font-family:system-ui,sans-serif';
    box.innerHTML = '<b>Next time, reorder in one tap.</b><br>Sign in and open <a href="' + RO + '">Rapid Order</a> to see everything you have bought before.';
    document.body.insertBefore(box, document.body.firstChild);
  }
  menu(); confirmation();
  new MutationObserver(menu).observe(document.body, { childList: true, subtree: true });
})();
</script>
```

For the confirmation page, Script Manager must have **Pages: All pages** and Location Footer; the Cornerstone guard will skip it, so this script intentionally does not use the header guard for the confirmation branch. If the box does not appear on the confirmation page, use Settings → Checkout → Order confirmation page HTML instead and paste the box markup there.

**3d. Klaviyo (Michael, 1 h).** In the "Loyal Re-Order Accelerator" flow and the post-purchase email in "First-Time Customer Flow", add one button: text "Reorder in one tap", link `https://covenanthealthproducts.com/account.php?action=order_status#rapid-order`. Keep the Holistic Health Partners / Carolyn Mulally BS, RN, HN-BC consultation line in the footer of those emails as they have it now.

**Test:** on a phone, sign in → My Account. Rapid Order is the first block. Tap Add to Cart on an item; the cart count changes, no redirect. Open the menu; "Reorder" sits above "Login". Place a test order with a 100%-off coupon or cancel it afterward; the confirmation page shows the box.

---

## Step 4 — Sticky Add to Cart bar on product pages (1 h)

**Where:** Script Manager → `CHP PDP: sticky add to cart`.

**Do:**

```html
<script>
(function () {
  if (document.querySelector('link[href*="fusionbot"],script[src*="fusionbot"]') || !document.querySelector('header.header, div.body')) return;
  var form = document.querySelector('form[data-cart-item-add]'); if (!form) return;
  var real = form.querySelector('#form-action-addToCart, [type=submit]'); if (!real || document.querySelector('.chp-restricted')) return;
  var price = (document.querySelector('[data-product-price-without-tax], .price--withoutTax') || {}).textContent || '';
  var bar = document.createElement('div'); bar.id = 'chp-sticky-atc';
  bar.innerHTML = '<span class="p">' + price.trim() + '</span><button type="button">Add to Cart</button>';
  document.body.appendChild(bar);
  bar.querySelector('button').addEventListener('click', function () { real.scrollIntoView({ block: 'center' }); real.click(); });
  var css = document.createElement('style');
  css.textContent = '#chp-sticky-atc{display:none;position:fixed;left:0;right:0;bottom:0;z-index:950;background:#fff;border-top:1px solid #E1E7D2;padding:10px max(16px,env(safe-area-inset-right)) calc(10px + env(safe-area-inset-bottom)) max(16px,env(safe-area-inset-left));align-items:center;gap:12px}' +
    '#chp-sticky-atc .p{font-weight:700;font-size:16px}#chp-sticky-atc button{flex:1;padding:14px;border:0;border-radius:10px;background:#444;color:#fff;font-weight:700;font-size:16px}' +
    '@media (max-width:800px){#chp-sticky-atc.show{display:flex}}';
  document.head.appendChild(css);
  new IntersectionObserver(function (e) { bar.classList.toggle('show', !e[0].isIntersecting); }).observe(real);
})();
</script>
```

**Conflicts to settle in the same step:** the "Unlock 20% Off" bar (Klaviyo or SaveYourCart, check which app renders it) and the "Need help?" bubble both live at the bottom of the screen. Set the 20% bar to show once per session and not on product or cart pages, and move the help bubble up with `bottom: 90px` on screens 800px and narrower so it does not cover Add to Wish List or the sticky bar.

**Test:** scroll a long product page; the bar appears once the real button leaves the screen and adds the item when tapped.

---

## Step 5 — Visible card buttons on phones (30 min)

**Where:** Storefront → Themes → Customize (Theme Editor) → Products. Turn on the setting that shows the card button ("Show Add to Cart button" or similar wording in your Cornerstone version). Then Script Manager → `CHP cards: visible on mobile`:

```html
<script>
(function () {
  var css = document.createElement('style');
  css.textContent = '@media (max-width:800px){.card-figcaption{display:block!important;opacity:1!important;position:static;margin-top:8px}.card-figcaption-body{display:block}.card-figcaption .button{width:100%;margin:6px 0}.card .quickview{display:none}}';
  document.head.appendChild(css);
})();
</script>
```

Because every product has an option, the button will read **"Choose Options"** and open the product page. That is still better than an invisible button: the shopper sees a tappable call to action and knows the next step. Hide Quick view on phones (done above) since its modal is poor on small screens.

---

## Step 6 — Search results page layout shift (45 min)

**Where:** Searchanise admin (the search app) → Design / Results page settings, and one CSS snippet.

**Do:** set results to 2 columns on mobile and enable "Add to Cart" buttons in results if the plan supports it. Add CSS via Script Manager: `#snize-search-results-wrapper{min-height:70vh}` and reserve the skeleton height `#snize_skeleton_results{min-height:70vh}`. This stops the page jumping from empty to full (CLS 1.4 today).

---

## Step 7 — Trim the mobile menu and nav clutter (30 min)

1. Storefront → Web Pages: for "New Service Page", "You're Booked — Consultation Confirmed", "Home" (the duplicate), "Brand Zone" if not meant for the menu, untick **Show in navigation menu**. They are hidden by CSS today but exposed if the menu script ever fails.
2. In the mega menu script, the Brands panel loads 31 images. Add `loading="lazy"` to those `<img>` tags in the template string.

---

## Step 8 — Measure for two weeks before Phase 2

Compare against the Step 0 baseline: mobile cart → checkout, login views per visitor, chatbot inquiries tagged reorder / coupon / add-to-cart. If cart → checkout has moved from 44% toward 55%, proceed to Phase 2 (script consolidation, image dimensions, blog template) as laid out in the plan. If not, pull Clarity or Hotjar recordings of the cart page first.

---

## Later phases (pointers only; detail is in the project plan)

- **Script consolidation (Dominik):** move the 30+ inline scripts into one versioned file on Vercel with page-type routing; target under 20 KB inline per page and TBT under 300 ms.
- **Image dimensions and reserved space:** add width/height to product, card and logo images in `templates/components/products/card.html`, `product-view.html`, and the blog template; add `min-height` for injected blocks.
- **Brand logos:** move off `clear-path-art.lovable.app` to BigCommerce Image Manager.
- **Order history backfill:** import pre-January-2026 orders into the chp-order-history API.
- **Header redesign:** sticky header with account icon and cart drawer; one search form.

---

## Quick checklist for your work plan

| # | Step | Owner | Est. | Depends on |
|---|---|---|---|---|
| 0 | Backup theme, inventory scripts, baseline numbers | Michael | 0.5 h | — |
| 1 | Cart: Continue shopping, coupon up top, reorder link | Michael | 1.5 h | 0 |
| 2 | Delivery wording on product pages | Michael | 0.75 h | 0 |
| 3a | Rapid Order to top of My Account | Dominik | 0.5 h | 0 |
| 3b | Fix silent redirect in reorder widget | Dominik | 0.3 h | 0 |
| 3c | Reorder links: menu, confirmation page | Michael | 0.7 h | 3a |
| 3d | Reorder button in Klaviyo flows | Michael | 1 h | 3a |
| 4 | Sticky Add to Cart bar; tame 20% bar and help bubble | Michael | 1 h | 2 |
| 5 | Visible card buttons on phones | Michael | 0.5 h | 0 |
| 6 | Search results layout shift | Michael | 0.75 h | 0 |
| 7 | Menu and nav clutter | Michael | 0.5 h | 0 |
| 7a | Guard the hidden Recurring Order select in the v4.6 widget (reset to One Time Purchase when blank) | Dominik | 0.3 h | 0 |
| 7b | Fix the false "backorder" box from BeCloudify Stock Notify on in-stock variants (Magnesium glycinate 180/360 CT) | Michael | 1 h | 0 |
| 8 | Two-week measurement | Michael | 0.5 h | 1–7 |

Total for Steps 0 to 8: about 9 hours of hands-on work.

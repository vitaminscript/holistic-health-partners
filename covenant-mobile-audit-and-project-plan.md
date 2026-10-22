# Covenant Health Products — Mobile Site Audit & Fix Plan

**Prepared for:** Michael Burton · **Date:** October 17, 2026 · **Scope:** covenanthealthproducts.com on phones

> Step-by-step build instructions for Phase 1 are in `covenant-mobile-implementation-guide.md`.

> How this was done: this session's container cannot reach the live storefront, so I did not tap through the site on a phone. Everything below comes from (1) the Ahrefs Site Audit crawl of the store from Oct 9–16 (full HTML of the homepage, a garlic product page, a category page and a search results page), (2) Ahrefs Lighthouse/CrUX mobile scores for the top 40 traffic pages, (3) Ahrefs Web Analytics filtered to smartphones for the last 30 days, (4) Google Search Console by device for the last 90 days, (5) ~200 customer chatbot inquiries forwarded to your inbox in the last year, and (6) the Klaviyo flow list. Items marked **Verify on device** need a 10‑minute phone check before you build.

---

## 1. Executive summary

Mobile is the majority of your organic traffic and the weakest part of the store. The customer complaint you forwarded is not a one-off. Every sentence of it maps to a specific, fixable cause in the current theme setup, and the same themes show up repeatedly in the chatbot inbox.

| Metric | Value | Source |
|---|---|---|
| Share of organic search clicks from mobile (90 days) | 65% (21,856 mobile vs 10,917 desktop) | GSC via Ahrefs |
| Smartphone visitors (30 days) | 17,865 (59% iOS, 41% Android) | Ahrefs Web Analytics |
| Mobile cart viewers → reached checkout | 1,235 → 549 (44%) | Ahrefs Web Analytics |
| Mobile checkout → order confirmation | 549 → 373 (68%) | Ahrefs Web Analytics |
| Mobile cart viewers who completed an order | 30% | derived |
| Login page views per mobile visitor | 2.5 (814 views / 323 visitors) | Ahrefs Web Analytics |
| Search results views per mobile searcher | 4.3 (3,230 views / 751 visitors) | Ahrefs Web Analytics |
| Lighthouse mobile score, top product pages | 31–69 | Ahrefs Site Audit |
| Lighthouse mobile score, blog pages | 12–48 (CLS 0.83–1.42) | Ahrefs Site Audit |
| Inline JavaScript shipped on every page | 37 scripts, ~175 KB, plus jQuery and 9 third-party apps | crawled HTML |
| New customers who reorder within 90 days | ~12% (flat for a year) | Dominik, Sept 29 email |

The single biggest leak is **cart → checkout**: more than half of mobile shoppers who open the cart never reach checkout. The cart page is the #2 mobile exit page (699 exits in 30 days). The second leak is **finding and reordering**: the reorder tool exists but is reachable only by signing in, opening My Account, tapping the Orders tab and scrolling. The third is **adding to cart from listings**: every product carries an option, so product cards show "Choose Options" and send shoppers to the product page, where the add button sits under several blocks of content and the delivery choice is labeled "(Required)". The add itself works; the wording and the trip are the problem.

Note on data quality: Ahrefs shows 322,608 desktop visitors with a 95.6% bounce rate in the same 30 days. That is bot traffic, not people. Mobile numbers are the trustworthy ones.

---

## 2. The complaint, decoded line by line

> "You need to have a button I tried to just hit my last order and it wouldn't let me like re-order your last order that was impossible to find"

**Cause:** The "Rapid Order" and "Complete Order History" widgets (your chp-order-history Vercel app) are injected by a Script Manager script on My Account, which opens on the Orders tab. Both sections are appended **after** the BigCommerce order list (`host.appendChild(wrap)`), so on a phone the path is: menu → Login → My Account → scroll past the account navigation and the order list. Nothing in the header, the mobile mega menu (which offers only "Login"), the cart page, the order confirmation page, or the post-purchase emails links to it. Orders placed before January 2026 (the previous store) are not shown at all, which is why customers write "None of my previous orders show up online" (Apr 28) and "I'm only seeing April of this year" (Jul 9).

**Same complaint from others:** "Why can't I find my past orders so I can just use those to reorder? I broke my back so everything is hard" (May 27).

> "every time I try to add the garlic it would take me to the product description. It wouldn't add it to my cart."

**Cause 1 — the option setup works, the wording doesn't.** Every product carries the MiniBC "Recurring Order" option, pre-selected to "One Time Purchase", and products with sizes pre-select the first size. Add to Cart works on a phone when tapped (confirmed on device by Michael on Oct 17). What goes wrong is the label: "Recurring Order: (Required)" reads as "you must set up a recurring order", and shoppers back out or write in. Because every product has an option, product cards on category, brand and search pages show **"Choose Options"** (a link to the product page) instead of **"Add to Cart"**, and in stock Cornerstone those card buttons are hover-only and hidden on small screens, so on a phone a tap anywhere on a card opens the product page. That is the trip the customer described. **Verify on device:** confirm the card button is invisible on a phone.

**Cause 2 — Rapid Order silently redirects on failure.** In the reorder widget's `addToCart()`, when the Storefront Cart API returns an error the code does `if (url) { window.location.href = url; return; }`. A failed add (expired cart, option mismatch, out-of-stock variant) sends the customer to the product description page with no message. That is exactly the behavior described.

**Cause 3 — the add button is buried and the page is busy.** On a phone the product page order is: image gallery and thumbnails → title, brand, price, review link → SKU / availability / shipping / "Learn About Shop and Save" → free-consultation card and "Talk to an Expert" (moved under the images by script) → Recurring Order select → quantity → Add to Cart. The button is several screens down with no sticky bar. The page ships 37 inline scripts (~175 KB) plus jQuery, Easy Upsell, SaveYourCart, MiniBC, BeCloudify Stock Notify, Klaviyo, Searchanise, Seona and your help widget. Lighthouse mobile Total Blocking Time is 600–3,000 ms on product pages; taps during that window are dropped or feel dead. The MiniBC script also binds to the add-to-cart click. Restricted brands show an "Available to clinic members" box with no add button at all ("I would like to purchase a bottle of RegeneVasc, but do not see a button to click to place the order", Apr 27).

**Same complaint from others:** "I cannot add SleepTides 30c by Biotics Research to the cart" (Jun 9), "How do I add this product to my cart and complete purchase" (Apr 21).

> "when I was ready to order, I think I had placed my order at the bottom, but you never had a modify your order but like to go back and add stuff"

**Cause:** The cart page offers no obvious "Continue shopping" / "Add more items" action on mobile, the cart icon in the header has no slide-out drawer, and the order confirmation page has no "Add to this order" or "Reorder" path. The customer's only visible options after adding one item were the bottom-of-page checkout controls. The coupon field is also hidden at the bottom of the cart, which drives a steady stream of "where do I put my 20% code" messages (Apr 6, Apr 13, May 18, Jun 21).

---

## 3. Full mobile audit by area

Severity: 🔴 Critical (blocks or loses orders) · 🟡 Moderate (friction, measurable) · 🟢 Minor

### 3.1 Header and navigation
| Finding | Severity | Evidence |
|---|---|---|
| Mobile menu has no My Account / Orders / Reorder entry; the only account item is "Login" (mobile-only item in the custom mega menu). | 🔴 | chp-mega script template |
| Two navigation systems load on every page: Cornerstone's `navPages` (hidden by CSS) with 218 brand links, plus the injected custom mega menu with 52 brand links and 31 images. Mobile downloads both. | 🟡 | crawled HTML |
| Two search inputs are present (theme quick search + injected `.chp-search` form placed after the header on ≤1024px). Possible duplicate search bars on phones. | 🟡 **Verify on device** | chp-mega script |
| Header is not sticky; cart and search scroll away on long product pages. | 🟡 | theme CSS |
| Cornerstone "Pages" menu still lists internal pages: "New Service Page", "You're Booked — Consultation Confirmed", "Home", "Brand Zone". Hidden by CSS today, but exposed if the custom menu script fails. | 🟢 | crawled HTML |
| Top bar (phone, shipping, Join our Clinic) is injected by script, so it pops in after first paint and shifts the page. | 🟢 | chp-mega script |

### 3.2 Homepage
| Finding | Severity | Evidence |
|---|---|---|
| CrUX (real users) rates homepage CLS as SLOW; all 58 images lack width/height attributes. | 🟡 | Ahrefs Site Audit |
| Brand logos are hot-linked from `clear-path-art.lovable.app`, a Lovable preview host. If that project is paused or deleted the "Featured brands" section breaks. | 🟡 | crawled HTML |
| Mobile bounce rate 65% on the homepage; it is also the #1 exit page. | 🟡 | Ahrefs Web Analytics |
| Lighthouse mobile 68 with TBT 607 ms; Makeswift CSS + Vimeo embed + redesign CSS injected by script. | 🟢 | Ahrefs Site Audit |

### 3.3 Category, brand and search listings
| Finding | Severity | Evidence |
|---|---|---|
| Every product card shows "Choose Options" (because of the required Recurring Order option); no direct add-to-cart anywhere in listings. | 🔴 | category HTML |
| Card buttons (Quick view / Choose Options) are hover-only and hidden on small screens in stock Cornerstone. | 🔴 **Verify on device** | category HTML + Cornerstone defaults |
| Search results page (Searchanise) has CLS 1.43–1.47 and Lighthouse 19–24; it is the most-viewed mobile page (3,230 views) and users search 4.3 times per visit, a sign they are not finding products. | 🔴 | Ahrefs Site Audit + Web Analytics |
| Anti-Inflammation category: CLS 0.46, Lighthouse 42. Faceted filters sidebar renders above/below the grid on phones. | 🟡 | Ahrefs Site Audit |
| `/product-discontinued/` had 118 mobile visitors and an 82% exit rate; discontinued products redirect there with no alternatives shown. | 🟡 | Ahrefs Web Analytics |

### 3.4 Product page
| Finding | Severity | Evidence |
|---|---|---|
| Add to Cart sits below gallery, info list, consultation card and the Recurring Order select; no sticky add-to-cart bar. | 🔴 | product HTML |
| "Recurring Order: (Required)" reads as a demand to subscribe even though "One Time Purchase" is pre-selected and the add works. Copy problem, not a code problem. | 🔴 | product HTML, device check |
| TBT 600–3,000 ms on product pages (Tru Niagen 2,254 ms; Magnesium Glycinate 2,212 ms; Ultra Binder 2,989 ms). | 🔴 | Ahrefs Site Audit |
| ~12 product pages in the top 40 have CLS 0.7–1.4 (Cayenne 0.91, Wobenzym N 0.88, Charcoal Binder 1.34, Garlic 5000 1.38). All product images lack width/height; late-injected blocks (consultation card, free-shipping line, notices, review form) shift content. | 🔴 | Ahrefs Site Audit |
| The v4.6 buy widget hides the real "Recurring Order" select (`display:none`) but leaves it `required`. If a browser restores it to the blank "Choose Options" entry, Chrome refuses to submit the form with no visible message (reproduced Oct 22 in a stale Chrome profile; Incognito worked). Add a guard that resets it to "One Time Purchase" when blank. | 🟡 | v4.6 script, device check Oct 22 |
| BeCloudify stock-notify widget briefly showed "This item is on backorder" with an email box under Add to Cart on Magnesium (glycinate) 180 CT and 360 CT in one automated desktop run, while BigCommerce reported both sizes in stock. Michael could not reproduce it in Chrome on Oct 22. Treat as a timing quirk to watch, not a confirmed defect. | 🟢 **Verify on device** | automated test Oct 22 |
| Nine third-party apps load on every product page: Easy Upsell, SaveYourCart, MiniBC, BeCloudify Stock Notify, Klaviyo, Searchanise, Seona (usestyle.ai), DropInBlog recent posts, chp-help-widget. Each adds scripts and can intercept the add-to-cart click. | 🟡 | product HTML |
| The review form renders inline on the page (hidden until toggled), adding DOM weight. | 🟢 | product HTML |
| Related products carousel shows 2 per slide on mobile with the same "Choose Options" problem. | 🟢 | product HTML |

### 3.5 Cart and checkout
| Finding | Severity | Evidence |
|---|---|---|
| 56% of mobile cart viewers never reach checkout; cart is the #2 exit page (699 exits). | 🔴 | Ahrefs Web Analytics |
| No visible "Continue shopping" / "Add more items" on mobile cart; no header cart drawer. | 🔴 **Verify on device** | theme defaults |
| Coupon field is at the bottom of the cart; the clinic 20% is applied manually ("we will hand price it"), so members see no discount line and write in. Six coupon-related inquiries in the sample. | 🟡 | inbox |
| Checkout completes for 68% who reach it. That is acceptable for BigCommerce Optimized One-Page Checkout; Apple Pay / PayPal are present. | 🟢 | Ahrefs Web Analytics |
| Order confirmation page has no Reorder / Add-to-order / Create account prompt. | 🟡 | theme defaults |

### 3.6 Account and login
| Finding | Severity | Evidence |
|---|---|---|
| 2.5 login page views per visitor on mobile: repeated attempts. Inbox: "I can't log in" (Jun 11), "don't see how to set up account" (Jun 10), "I want to create an account and register my email" (Jun 9). | 🔴 | Ahrefs Web Analytics + inbox |
| Rapid Order is appended below the order list on My Account, so phone users scroll past navigation and orders to find it. | 🔴 | order-history script |
| Pre-2026 orders missing from history. | 🟡 | inbox + script copy |
| "The edit feature is not letting me update billing address and exp date" (Jul 6). Saved payment methods editing fails on mobile. | 🟡 **Verify on device** | inbox |

### 3.7 Blog (DropInBlog)
| Finding | Severity | Evidence |
|---|---|---|
| Five blog posts are in the top 40 traffic pages; all score 12–48 on mobile with CLS 0.83–1.42 and LCP up to 6.1 s. Blog traffic has 87–93% mobile exit rates. | 🟡 | Ahrefs Site Audit + Web Analytics |

### 3.8 Performance and code hygiene (cross-cutting)
| Finding | Severity | Evidence |
|---|---|---|
| 35–37 inline Script Manager scripts (~160–175 KB) on every page. Many are page-specific (order history, consultation landing page CSS, Professional Formulas notice) but ship everywhere and only exit at runtime. | 🔴 | crawled HTML |
| jQuery 3.5.1 loaded from code.jquery.com alongside the Stencil bundle. | 🟡 | crawled HTML |
| Zero images with width/height across all crawled templates. | 🔴 | crawled HTML |
| 1,876 pages link to redirecting URLs; 207 redirects; 18 4XX pages; 80 images missing alt text; 230 pages with incomplete Open Graph tags. | 🟢 | Ahrefs Site Audit |

### 3.9 Two more items from the Oct 17 phone screenshots
| Finding | Severity | Evidence |
|---|---|---|
| The "Need help?" chat bubble overlaps the Add to Wish List button and sits beside Add to Cart in the thumb zone. | 🟡 | screenshots |
| The "Unlock 20% Off" bar pins to the bottom of the screen on every page and competes with the chat bubble and Add to Cart; show it once per session or hide it on product pages. | 🟡 | screenshots |
| Product page has two large promo cards (Free Product Advice, Blog Post) between the gallery and the buy controls, pushing Add to Cart about two screens down. | 🟡 | screenshots |

### 3.10 What works well
- Checkout itself converts at a healthy 68% once reached.
- Product-option defaults are set correctly: One Time Purchase and the first size are pre-selected, so a single tap on Add to Cart works.
- The Rapid Order concept is right: "Everything you've ordered before — one click to order it again" is exactly what the complaining customer wanted. It just needs to be findable.
- Klaviyo lifecycle flows are in place (Abandoned Checkout, Browse Abandonment, First-Time Customer, RFM segments 1–10), so reorder and recovery messaging has somewhere to live.
- Brand visual language (green #7FA923, Hanken Grotesk) is consistent across the custom widgets.

---

## 4. Project plan

Owner column assumes Michael (BigCommerce admin, Script Manager, theme files) and Dominik (Vercel apps, scripts). Hours are estimates for a developer familiar with Stencil.

### Phase 0 — Reproduce and instrument (Week 1, ~6 hours)
| # | Task | Owner | Effort | Done when |
|---|---|---|---|---|
| 0.1 | Run the device test script in §5 on an iPhone (Safari) and an Android (Chrome). Record a screen video of each scenario. | Michael | 2 h | Each "Verify on device" row above is confirmed or cleared. |
| 0.2 | Add a mobile funnel report in BigCommerce Analytics or GA4 (product view → add to cart → cart → checkout → order) segmented by device. | Michael | 2 h | Baseline numbers saved alongside the Ahrefs figures above. |
| 0.3 | Add Microsoft Clarity (free) with the script loaded `defer` on storefront pages only, to watch real mobile sessions on cart and product pages for two weeks. Remove after Phase 3 if not needed. | Dominik | 1 h | Recordings available. |
| 0.4 | Export the last 12 months of chatbot inquiries to a sheet and tag by theme (reorder, add-to-cart, login, coupon, order history, shipping). This is the complaint baseline. | Michael | 1 h | Counts per theme recorded. |

### Phase 1 — Stop the bleeding: reorder, add-to-cart, cart (Weeks 1–2, ~24 hours)
| # | Task | Owner | Effort | Done when |
|---|---|---|---|---|
| 1.1 | **Make Rapid Order findable.** In the existing order-history script, insert the Rapid Order section at the top of the account content (`host.insertBefore(wrap, host.firstChild)`) instead of appending it after the order list, and give it the anchor `#rapid-order`. | Dominik | 3 h | Logged-in customer sees Rapid Order first on My Account. |
| 1.2 | Add "Reorder" entry points: (a) mobile mega menu item "Reorder" for logged-in customers (check `customer` via the Storefront API or a body class), (b) a "Reorder from a past order" link in the cart page empty and non-empty states, (c) a "Reorder these items" block on the order confirmation page, (d) a "Reorder" button in the Klaviyo "Loyal Re-Order Accelerator" and post-purchase emails deep-linking to `/account.php#rapid-order`. | Dominik + Michael | 6 h | All four paths land on Rapid Order in one tap. |
| 1.3 | **Fix the silent redirect.** In `addToCart()` replace `window.location.href = url` with an inline error ("Couldn't add — tap to open product") and log the failure to the Vercel app so you can see why adds fail. | Dominik | 2 h | No redirect on failure; failures visible in logs. |
| 1.4 | **Fix the wording, not the option.** Keep the MiniBC option and its values as they are. Show "Delivery" instead of "Recurring Order:", drop "(Required)" on this option, and add one helper line under the dropdown ("One-time is the default. Choose auto-refill to have it shipped on a schedule."). Do it in the theme language file or the existing label-tweak Script Manager script, not by renaming the shared option (MiniBC matches on the value text). Separately, turn on the Cornerstone card-button setting and make card buttons always visible on phones so "Choose Options" is at least tappable from listings. | Michael | 3 h | No "(Required)" next to the delivery choice; card buttons visible on a phone. |
| 1.5 | **Sticky add-to-cart bar on product pages** (price + Recurring Order summary + quantity + Add to Cart) pinned to the bottom of the viewport on ≤800px, appearing once the main button scrolls off-screen. | Dominik | 4 h | Button reachable at any scroll position. |
| 1.6 | **Cart page mobile fixes:** "Continue shopping" button at the top (returns to the last category or Rapid Order), coupon field moved above the totals with helper text ("Clinic members: your 20% is applied automatically at checkout" or whatever is true), and a "Reorder past items" link. | Michael | 3 h | Cart exit rate starts falling in Clarity/analytics. |
| 1.7 | **Guard the hidden Recurring Order select.** In the v4.6 buy-widget script, before the add-to-cart form submits, if `.chp-buy-real` has no value set it to the `data-once` option. | Dominik | 0.3 h | Add works even when a browser restores a blank value. |
| 1.8 | **Watch for a false backorder box.** Seen once in an automated run, not reproduced by hand. If a customer or a Clarity recording ever shows "This item is on backorder" on an in-stock size, check the BeCloudify Stock Notify settings (its check reads `inventory.isInStock`, which can differ from the purchasable flag on untracked variants). | Michael | 0.3 h | Only if it shows up again. |

### Phase 2 — Speed and stability (Weeks 3–5, ~40 hours)
| # | Task | Owner | Effort | Done when |
|---|---|---|---|---|
| 2.1 | **Consolidate Script Manager scripts.** Move the 30+ inline scripts into one versioned file on Vercel (or the BigCommerce CDN via WebDAV) loaded with `defer`, with page-type routing inside (product, category, account, home, services) so each page runs only its own code. Keep the Klaviyo tracking and consent scripts separate. Delete scripts that only existed for the old FusionBot theme. | Dominik | 12 h | Inline JS per page under 20 KB; TBT under 300 ms on product pages. |
| 2.2 | Remove the jQuery CDN include if nothing depends on it; otherwise bundle it into the consolidated file. | Dominik | 2 h | One fewer render-blocking request. |
| 2.3 | **Fix layout shift:** add width/height (or `aspect-ratio`) to product, card, logo and blog images in the theme templates; reserve space for injected blocks (consultation card, free-shipping line, top bar, Searchanise skeleton) with CSS `min-height`. | Dominik | 8 h | CLS under 0.1 on homepage, category, product, search and blog templates in Lighthouse. |
| 2.4 | Lazy-load the mega menu brand images and the hidden Cornerstone brand list (or remove the duplicate theme nav entirely and keep only the custom menu). | Dominik | 3 h | Fewer requests on first paint. |
| 2.5 | Move brand logos off `clear-path-art.lovable.app` onto the BigCommerce CDN (Image Manager). | Michael | 1 h | No third-party preview host in page source. |
| 2.6 | **Third-party app review.** For each of Easy Upsell, SaveYourCart, BeCloudify Stock Notify and Seona: pull 90-day revenue or usage from the app dashboard; keep, defer, or remove. Any kept app loads `async` after `DOMContentLoaded`. | Michael | 4 h | Written keep/remove decision per app. |
| 2.7 | **Blog template:** set fixed dimensions on DropInBlog featured images and the recent-posts widget, defer the DropInBlog embed, and confirm the post template is not loading product-page scripts. | Michael | 4 h | Blog Lighthouse mobile above 70. |
| 2.8 | Fix the Searchanise results skeleton so results render into reserved space; configure Searchanise mobile layout to 2 columns with always-visible Add to Cart (ties to 1.4). | Michael | 4 h | Search page CLS under 0.1, Lighthouse above 60. |
| 2.9 | Clean up Ahrefs technical items that touch mobile UX: 18 4XX pages, 2 5XX, 80 missing alt texts, redirect links in nav/footer. | Michael | 2 h | Ahrefs errors at zero. |

### Phase 3 — Mobile template redesign (Weeks 5–8, ~48 hours)
| # | Task | Owner | Effort | Done when |
|---|---|---|---|---|
| 3.1 | **Header:** sticky on scroll with logo, search icon that expands, account icon (goes to My Account / Rapid Order when logged in, Login when not), cart icon with count that opens a slide-out cart drawer (BigCommerce cart preview) with "Checkout" and "Keep shopping". Replace the duplicate search forms with one. | Dominik | 12 h | One header, one search, one cart path. |
| 3.2 | **Product page order on mobile:** gallery → title/brand/price → Recurring Order (relabeled "Delivery: One time / Every 30 days…") → quantity → Add to Cart → availability/shipping → consultation card → description tabs as accordions → related products. Carry the "Delivery" wording from 1.4 into the redesigned layout. | Dominik | 10 h | Add to Cart visible within the first 1.5 screens on a 390px-wide phone. |
| 3.3 | **Restricted brands:** keep the "Available to clinic members" box but give it a single primary action ("Get free access" → `/access-our-clinic/`) and show the phone number as tappable text. Hide the quantity field and empty button for these products. | Michael | 3 h | No dead add-to-cart states. |
| 3.4 | **Category and search cards:** 2-column grid, always-visible "Add to Cart", price and brand above the fold of the card, "Members save 20%" badge logic if applicable. | Dominik | 8 h | Cards add to cart in one tap. |
| 3.5 | **My Account landing:** Rapid Order → recent orders (collapsible) → subscriptions (MiniBC) → addresses/payment. Add a one-line explanation of where pre-2026 orders are. | Dominik | 6 h | Account page is a reorder page first. |
| 3.6 | **Order confirmation:** "Reorder these items next time" (saves to Rapid Order), "Create a password" for guests, and the consultation CTA referencing Holistic Health Partners and Carolyn Mulally BS, RN, HN-BC. | Michael | 4 h | Guests converted to accounts; CTA present. |
| 3.7 | **Login friction:** enable BigCommerce "Remember me", make the reset-password link prominent, and send a Klaviyo "set your password" email to migrated customers who have never logged in on the new store. | Michael | 5 h | Login views per visitor under 1.5. |

### Phase 4 — Reorder data and lifecycle (Weeks 8–10, ~20 hours)
| # | Task | Owner | Effort | Done when |
|---|---|---|---|---|
| 4.1 | Backfill pre-January 2026 orders into the chp-order-history API from the old store's export so Rapid Order shows a customer's full history. | Dominik | 10 h | Customers no longer write "my old orders are missing". |
| 4.2 | Klaviyo replenishment flow (activate the "CHP Replenishment Field Proof" draft once tested) with a single "Reorder" button to `/account.php#rapid-order`, timed by product day-supply. | Michael | 6 h | Flow live, reorder rate tracked. |
| 4.3 | Coordinate Dominik's "Ship every 30 days" nudge with the relabeled Recurring Order (3.2) so there is one subscription message, and confirm the cancel/discount claims before it goes live. | Michael | 2 h | One consistent subscription message. |
| 4.4 | Update the chp-help-widget answers for "how do I reorder", "where is my order history", "where do I enter my coupon" to link to the new paths. | Dominik | 2 h | Help widget deflects the top three questions. |

### Phase 5 — Measure (ongoing from Week 3)
| KPI | Baseline (30 days to Oct 17) | Target by Week 12 |
|---|---|---|
| Mobile cart → checkout | 44% | 60% |
| Mobile cart viewers → order | 30% | 42% |
| Login views per mobile visitor | 2.5 | under 1.5 |
| Search views per searcher | 4.3 | under 2.5 |
| Lighthouse mobile, top 10 product pages | 31–69 | 75+ |
| CLS on homepage, category, product, search, blog | 0.46–1.47 | under 0.1 |
| Inline JS per page | ~175 KB | under 20 KB |
| New customers reordering within 90 days | ~12% | 18% |
| Chatbot inquiries tagged reorder / add-to-cart / coupon / login | baseline from 0.4 | −50% |

**Total estimate:** roughly 140 developer hours over 10 weeks. Phases 0 and 1 (about 30 hours) address the customer's complaint directly and should ship first.

---

## 5. Device test script (run before Phase 1, repeat after each phase)

Use a real iPhone in Safari and a real Android phone in Chrome, on cellular data, logged out and then logged in as a returning customer.

1. Open the homepage. Does anything shift after load? Is there one search bar or two? Is the cart icon visible after scrolling?
2. Open the menu. Can you reach My Account / Orders / Reorder without logging in first? What do logged-in customers see?
3. Open Clinic Recommended Products → Anti-Inflammation. Tap a product card button. Does "Add to Cart" exist, or does every tap open the product page?
4. Search "garlic". Count how long until results stop moving. Tap "Add to Cart" on a result.
5. Open Garlic Plus 100T. Scroll until you find Add to Cart. How many screens? Tap it once. Did a cart message appear within 1 second? Did any popup (upsell, stock notify) intercept?
6. Open the cart. Find "Continue shopping" and the coupon field without scrolling past the totals. Try to add a second item and return to the cart.
7. Log in. From My Account, find Rapid Order. Count taps and scrolls. Tap "Add to Cart" on a past item. Did it add, or did it redirect to the product page?
8. Place a test order. On the confirmation page, is there a way back to the cart, to reorder, or to add an item?
9. Try to update a saved card's expiration date.
10. Open the blog post "Thorne FiberMend" from search. Time how long until the page stops jumping.

---

## 6. Things I could not verify from here
- An HTTP add-to-cart test I ran on Oct 17 mis-posted the Size option id on three products and failed. Michael confirmed on a phone the same day that Size and Recurring Order both pre-select and Add to Cart works. The product-page issue is wording and layout, not a broken add.
- Whether the card buttons are visible on phones in your current theme build (stock Cornerstone hides them below the medium breakpoint; your theme may have changed this).
- Whether the add-to-cart tap on product pages is being intercepted by Easy Upsell, SaveYourCart or MiniBC on mobile (Clarity recordings in Phase 0 will show this).
- Live cart and checkout page markup (BigCommerce does not let crawlers see them).
- Actual mobile conversion rate from BigCommerce Analytics (the Ahrefs funnel above is a proxy from page views).

---

## 7. Brand and compliance notes
- All CTAs added in Phases 3 and 4 should reference Holistic Health Partners and Carolyn Mulally BS, RN, HN-BC, consistent with the rest of Covenant's content.
- Subscription wording (Phase 4.3) must only promise what MiniBC Recurring Orders actually allow (self-service cancel, fixed price), per Dominik's note on US auto-renewal rules.
- Restricted brands (Ortho Molecular, Xymogen, Quicksilver, etc.) stay gated behind clinic access; the fix is to make the gate clear and one-tap, not to open it.

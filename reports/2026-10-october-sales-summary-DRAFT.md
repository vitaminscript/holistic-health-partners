# October 2026 Sales Summary — DRAFT (handoff)

**Status:** Preliminary. October books are NOT closed in QuickBooks. Calls and appointments are final.
**Pulled:** Nov 1, 2026 (cloud session). Finish on laptop with the BigCommerce MCP.

---

## 1. Revenue & Cost (QuickBooks — Covenant Health Products, LLC)

| | As booked | Notes |
|---|---|---|
| Gross revenue (Product Sales) | $185,984.37 | Missing Sat Oct 31; Fri Oct 30 looks like a lump entry |
| COGS | $110,565.77 | |
| Gross profit | $75,418.60 (40.6%) | |
| Operating expenses (booked) | $32,051.06 | Normal months run $61K–$82K |
| Payroll (ADP, per Michael) | ~$30,000 | **Not posted in QBO yet** |
| Net income (as booked) | $43,369.87 | |
| **Net income incl. ADP payroll** | **~$13,370 (~7.2%)** | |
| If depreciation (~$5,317) + normal interest (~$3,959 more) post | ~$4,100 (~2.2%) | |

Biggest booked expense: Postage & Express Charges $13,806.37.

### Unreconciled items to resolve before final
- [ ] ADP payroll (~$30K) — not in QBO (Salaries / Payroll Taxes = $0 for Oct). Only Payroll Processing Fees $423.55 posted.
- [ ] Depreciation — $0 in Oct (Jan–Sep avg ~$5,317/mo)
- [ ] Interest expense — $448.77 in Oct (Jan–Sep avg ~$4,408/mo)
- [ ] Sat Oct 31 — $0 revenue/COGS posted (weekend sales not entered)
- [ ] Fri Oct 30 — $12,697.00 revenue, round number, 75.6% GM vs ~40% normal → likely batch deposit / missing COGS
- [ ] Bank feed last-sync date — not visible via connector; check QBO → Transactions → Bank transactions ("Updated X ago" + For Review tab)
- [ ] Confirm whether VitaminScript revenue is inside this QBO file or booked elsewhere
- [ ] $30K payroll is above Jan–Sep avg (~$23.5K incl. officers + taxes) — 3-paycheck month / bonus / new hire?

QBO Payroll API returned Forbidden (payroll is ADP). No ADP connector available via Zapier.

---

## 2. Appointments by Type (Calendly, Oct 1–31, ET)

| Type | Count |
|---|---|
| Product Consultation (incl. 1 in-person, 1 FB) | 18 |
| Clinic Access ("Access Our Clinic" 8 + "Gain Access to Our Clinic" 7) | 15 |
| Wellness Consultation (paid) | 3 |
| Diagnostic (Test Kit Selection 1 + Testing Review 1) | 2 |
| Complimentary Mini Wellness | 1 |
| **Total held/booked (not canceled)** | **39** |
| Canceled | 9 |

No-shows not checked.

---

## 3. Phone Calls (RingCentral, Oct 1–31, ET)

**Company:** 915 calls · 630 inbound · 243 outbound · 65.6 hrs talk time · 136 after-hours

Inbound outcomes: 316 answered · 143 missed · 120 voicemail · 93 abandoned → **~56% of inbound not answered live**

| Queue | Calls | Answered | Voicemail | Missed | Abandoned | In SLA |
|---|---|---|---|---|---|---|
| Covenant Health (ext 1) | 459 | 208 | 86 | 94 | 70 | 132 |
| VitaminScript CS (ext 2) | 166 | 84 | 31 | 40 | 9 | 42 |

| Team member | Answered | Outbound | Talk time |
|---|---|---|---|
| Ryanne (x106) | 198 | 68 | 23.5 hrs |
| Chelsea (x105) | 88 | 71 | 18.7 hrs |
| Carolyn (x104) | 8 | 45 | 17.1 hrs |
| Mark (x103) | 20 | 81 | 12.7 hrs |
| Michael (x101) | 2 | 4 | 3.2 hrs |
| Shelby (x7) | 0 | 16 | 0.5 hrs |

Per-person call counts overlap (ring groups) — don't sum them.

---

## 4. STILL TO DO (on laptop, BigCommerce MCP)

- [ ] **Top 100 products** by revenue, Oct 1–31 — Covenant + VitaminScript (split by store?)
- [ ] **Brand revenue** table — **core brands at top**, then all others by revenue
- [ ] Michael to supply the **core brands list**
- [ ] Cross-check BigCommerce October revenue vs QBO $185,984 (incl. Oct 30–31)
- [ ] Re-pull QBO P&L once October is closed
- [ ] Choose final format: Excel workbook / shareable report / markdown

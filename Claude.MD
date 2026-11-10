# CLAUDE.md: Dimiter's personal finances report

This file describes how the personal finances report (originally the "spending report") is built: which files and transactions go into it, the classification decisions Dimiter has made, the output constraints, and the sections. It reflects report version v133, built on 5 November 2026.

## 1. Standing rules

- Australian English. No em dashes. Write plainly and briefly; push back when something looks wrong.
- Never overwrite a file: keep the previous version (`build2_vNNN.py`, `classify2_vNNN.py`, `receipts_vNNN.json`, `acct2/all_YYYY-MM-DD.csv` and so on). Ask before deleting anything.
- Output names carry the build date (Sydney time, `TZ=Australia/Sydney date`) and a version number: `YYYY-MM-DD_spending-report_vNNN.html` and `YYYY-MM-DD_transactions_categorised_vNNN.csv`.
- **Matching principle:** the report is built from transactions. Receipts, invoices, statements and emails are only used to explain or label a transaction that is on Dimiter's accounts. Anything paid by someone else (for example Ayn) or not on his accounts does not appear. The only exceptions are listed in section 5.7 (claimed purchases, a cash receipt, payslips, super).
- **Scope:** only financial years with complete bank and loan records: FY2025-26 and FY2026-27 to date (`CUT = 2025-07-01`). Nothing earlier is used, except where section 5 says so (the Zeiss lens, the loan history from drawdown, payslips and super).
- Financial year of a bill or dividend is decided by the **date it was paid**, not the period it covers.
- Do not state tax figures in chat when asked to add them; put them in the report.
- Future (not started): a git repository holding a central, growing log of all integrated transactions that the report reads from.

## 2. Input files

All working files live in the session scratchpad. Raw uploads are copied into dated folders (`raw_YYYY-MM-DD/`) before being merged.

### 2.1 Bank and loan transactions

| Source | File(s) used | Format | Coverage now |
|---|---|---|---|
| ANZ Access Advantage (transaction account 013-634 172727847, card ending 1900) | `FY 2024 2025 Access Advantage.csv`, `FY 2025 2026 Access Advantage.csv`, `FY 2026 2027 Access Advantage.csv` concatenated newest first into `acct2/all.csv` | ANZ 8-column export: `DD/MM/YYYY,"amount",description,payer,payee,,reference,note`. Notes in columns 4 to 8 are kept | 2025-01-13 to 2026-11-02 |
| ANZ Residential Investment Property Loan (Dudley St, 7356-85696, redraw) | `FY 2024 2025 / 2025 2026 / 2026 2027 Residential Investment Property Loan.csv` concatenated into `loan/loan.csv`, parsed by `loan.py` into `loan.json` | ANZ 3-column export | 2025-01-13 (drawdown) to 2026-11-02 |
| Wise (card and AUD balance) | `FY 2025 2026 transaction-history.csv` + `FY 2026 2027 transaction-history.csv` (one header) into `wise/wise.csv`, converted by `wise_csv.py` into `wise/wise_conv.csv` | Wise transaction history. Only `COMPLETED` rows; card orders and `invoice-` top-ups dropped; UTC converted to Sydney date; note carries the Wise fee and "Wise category" | 2025-10-20 to 2026-11-03 |
| PayPal | `PayPal 2024 07 01 to 2025 06 30.csv`, `PayPal 2025 07 01 to 2026 06 30.csv`, `PayPal 2026 07 01 to 2026 10 20.csv` saved as `paypal/pp_fy25.csv`, `pp_fy26.csv`, `pp_fy27.csv` | PayPal activity export | Used to explain `PAYPAL *` card payments on ANZ (merchant, order) |
| Independent Reserve | `FY 2025 2026 Independent Reserve.csv` (`ir/`) | IR transaction history | The 6 Feb 2026 BTC sale |
| CBA and ME Bank (old accounts) | `cba/`, `me/me.csv` | Parsed statements | Pre-scope. Only used to match pre-FY2025-26 salary deposits to payslips |

The current end of the data is set in `build2.py` (`END`); periods, month lists and labels follow it automatically.

### 2.2 Receipts, invoices and statements

- `receipts.json` (112 entries; groups Personal, Travel, Overwatch, Dudley St, Birrell St). Each entry: `file, supplier, item, inv (date), total, gst, billed, group`, optional `claim`, `paid`, `bank`, `paid_note`. A receipt is matched to a bank transaction of the same amount within -3..+3 days of `inv` when `paid` is set, otherwise -70..+10 days.
- Property bills (strata, council, water, repairs) for both properties, FY2024-25 to 2026-11-02, including the 11 bills sent on 2026-11-03.
- Claimed work purchases: Ted's Cameras (Zeiss lens), Claude Pro (Anthropic), Sena headset (Oceantide), Quad Lock case (Annex), Digidirect Sony A1, Digidirect microphone.
- Google Workspace invoices (billed to Overwatch Pty Ltd).
- Travel bookings (Rail Ninja, Lufthansa, Hotel Ars Vivendi, Bulgaria Air, Cathay screenshot), car-trip bookings for December 2026 (Tuross Motel, Snowy River Homestead / Phalin Pty Ltd, Mountain View Motel Corryong).
- `gym/invoices.json`: 10 One Playground / The Bunker invoices (trainer, gym, admin, extra coach sessions, card fees).
- `etoll/etoll.json`: E-Toll statements parsed into `trips` (date, road, segment, tag/plate, cost) and `tops` (top-ups).
- `leases/`: tenancy agreements (Dudley: Hanlon & McCaw 2024-25, Nay & Rudland 2025-26, the 18-month renewal and release clause; Birrell: Freeman & Petts).
- `trips.txt`: 78 trips with start and end timestamps and a name.
- `receipts_img/2025 11 07 Birrell keys 23.95.jpg` (cash receipt).
- Telstra dividend advices (2025-03-28, 2025-09-25, 2026-03-27, 2026-09-24).
- Last year's tax workbook (`taxin/t.xlsx`, sheet "2025"): first-rented dates, the borrowing-cost schedule, Birrell wardrobes and blinds, and the old work-asset depreciation schedule.
- Tax documents (`tax_docs/`): lodgment summary (receipt 24112 8590 1598), notice of assessment (30 Oct 2026), ANZ BPAY receipt (2 Nov 2026), ATO payment page (income tax PRN 551001993922040821), PAYG instalment letter (4 Nov 2026), Division 293 due-and-payable election (10 Nov 2026, ATO receipt 2411294574816), and the Income tax 551 transaction list Dimiter pasted on 10 Nov.
- Employment (`payslips/`): 44 payslips May 2023 to October 2026, the 2023 job offer, the 28 Apr 2026 Touchstone Point contract, the 4 Jun 2026 side letter. Parsed by `payslip_parse.py` into `payslips.json`.
- Super (`super/`): six AustralianSuper member-transaction CSVs, 2021-07-01 to now.
- RBA cash-rate decisions (pasted by Dimiter) for the loan rate table.

## 3. Pipeline

1. `classify2.py` reads the bank CSVs (`cba/cba.csv:CBA, acct2/all.csv:ANZ, me/me.csv:ME Bank, wise/wise_conv.csv:Wise`) and writes `tx_all.json`. Rules are `(pattern, group, category)`, first match wins; `+`/`-` prefixes restrict to money in/out; `~` matches the note instead of the description.
2. `build2.py <tx_all.json> <out.html> <out.csv|-> "<date>" [SCOPE]` builds one view. `SCOPE` is `all`, `FY2025-26` or `FY2026-27`.
3. `combine.py` merges the three views into one page with the year switcher, prefixing every `id`/`name` per view (`A_`, `V2025_`, `V2026_`).
4. `buildall.sh vNNN "<Date>" YYYY-MM-DD` runs all of the above and writes the outputs to `/mnt/user-data/outputs/`.
5. Every build prints an `ok` reconciliation line per view; the All view must equal the sum of the two years.
6. Check in Chromium (Playwright) for script errors and screenshot changed sections before sending.

## 4. Groups and categories

Groups: Income, Living (living costs), Property (investment property costs), Internal (own transfers, loan redraws and repayments, Wise top-ups), Shared (Ayn), Company (Overwatch Pty Ltd), Unclear.

Unmatched card spending defaults to Living / Eating out and coffee and is reviewed. Unmatched money in defaults to Income / Refunds and cashback. Anything spent during a trip (see 6.4) is moved to Travel, keeping its original category for the trip breakdown.

## 5. Special classifications Dimiter has given

### 5.1 Categories by merchant

- **Industrie** is always Clothes.
- **Helly Hansen** is Sailing, except the $815.00 on 2026-05-14, which is ski clothes (category Skiing).
- **Royal Sydney Yacht Squadron (EZI*RSYS)** is eating out, except the $390.00 on 2026-05-20: sailing clothes (Sailing).
- **David Jones** $319.99 on 2026-06-17: blazer for sailing (Sailing). Other David Jones is Shopping and personal.
- **Java and Coco** is a coffee stand at the Orange Grove markets on Saturday mornings. Several charges on one Saturday are normal. Display name "Java and Coco (Orange Grove markets, Saturdays)".
- **Reuben Hills** is weekday breakfast. Several mornings in a row is normal; weekends would be wrong. Wise card is now used for day-to-day spending to get exact timestamps (ANZ shows posting dates).
- **Digidirect** is a camera shop: camera gear and photo prints. 2026-03-18 $6,257.89 is the Sony A1 body; $153.44 and $99.94 the same day are a camera bag and a strap; 2026-04-27 $97.00 (PayPal, Digital Imaging Express) is photo prints; 2026-08-31 $201.90 is probably a Lexar CFexpress Type A 80GB card (unconfirmed).
- **NX Biz** (PayPal): both Tour de Brisbane entry fees are Bicycles, including the one not attended.
- **Omafiets, Pushys, Trek, Woolys Wheels, Blue Lug**: Bicycles. **Bikebiz, Balmain Motorcycles, Motologic, Adventure Moto, Valosi, the Motorcycle Wrecker**: Motorcycle.
- **SOS Suspension** is Ayn's Honda CB125F (Motorcycle): $580.00 on 2025-09-11 forks; $180.00 on 2025-09-19 kickstand modification.
- **NRMA claims** is the collision claim on Ayn's car (Car).
- **E-Toll** top-ups are Tolls under Vehicles (car and motorcycle), shown by road with proper names (see 7.9).
- **Rego and CTP**: Service NSW / insurer payments in October and November are the motorcycle, May and June the car.
- **Fuel**: own category, not split by vehicle in totals; on a car trip it is the car, otherwise $35 or less is the motorcycle.
- **Advance Vision** payments are rent at Nancarrow ($4,345 a month). The small "1305B water" payments to the same account are lumped in with rent.
- **Bissera Dikova (Mum)**: $390 a month, Loan repayments to Mum (living). Payments she sends back are netted.
- **Phalin Pty Ltd** (Snowy River Homestead) $200.00 on 2026-10-20: Travel, car trip south, 27-28 Dec 2026.
- **One Digi Corp** $149.72 on 2025-07-28: Vietnam e-visa through an agent (Vietnam motorcycle trip).
- **MyCall** (Wise): Norwegian SIM, Travel. **Lime**: Rideshare, taxis and scooters.
- **Wise**: top-ups from "D Nedialkov" are Transfer to Wise (internal) and are matched to the ANZ transfer. Wise card rows go through the normal rules on the merchant name, then fall back to Wise's own category.
- **ATO**: `TAX OFFICE PAYMENT` is Income / Tax paid to the ATO (2026-11-02 $13,512.65 is the FY2025-26 bill). Money from the ATO is Tax refund.
- **Kevin Joyce Pest Services** is Dudley repairs (pest control).
- **Independent Reserve** is Crypto purchase.
- **GoGet in April 2025** is Moving house (van hire for the move to Nancarrow).

### 5.2 Trips and travel

- 2025-10-15 and 2025-12-23 Booking.com: car trip south ("car south Ayn"); the Tuross Heads motel must be booked ahead.
- Natalie's Travel 2026-06-22: $285.60 is Dimiter's travel insurance; $237.15 is Ayn's, and Ayn paid back $238 the same day (netted in the trip).
- Europe Yngling Worlds trip:
  - Bulgaria Air: internal European flights, paid by Dimiter. Ayn's $1,001 Wise top-ups (2026-07-14/15) offset them.
  - Ibis Styles is the London hotel.
  - UK Visas is the UK entry visa.
  - Spond payments are accommodation at Askøy Seilforening.
  - Gary Pearce paid back $1,130.19 for the accommodation Dimiter paid for both of them.
  - The $3,200 ferry and boat rental paid to Gary David Pearce: Ayn's $1,600 on 2026-05-18 is her half.
  - The Cathay flight and the Amsterdam hotel were paid by Ayn and do not appear.
- Trip overrides by date and payee are listed in `build2.py`; every override must name an existing trip (asserted).
- Trip header rows have no details; expanding them shows every transaction, largest first, with details. Trip notes go below the table.

### 5.3 Dates moved

- Zeiss Loxia 35mm lens ($1,999.95, Ted's Cameras via PayPal) was prepaid 2025-05-15 but invoiced and received 2025-07-11. It is dated 2025-07-11 (FY2025-26) and kept out of any trip.

### 5.4 Rent and tenancies

- Dudley (7/62-64 Dudley St, Coogee): first rented 2021-09-13. Hanlon & McCaw to about 2025-07-17 (estimated from the $2,579.76 refunded to Woodwind Carpentry on 2025-07-09), vacant to 2025-08-06, Nay & Rudland from 2025-08-07 at $4,997.02 a month, new 18-month lease from 2026-10-07 at $2,350 a fortnight. Belle Property's $2,042.03 on 2025-08-06 is August rent less fees (statement not seen).
- Birrell (10/304 Birrell St, Bondi): first rented 2025-05-10, Freeman & Petts at $2,600 a fortnight, month to month from 2026-05-10. They have given notice: they move out 2026-11-20 and hand back the keys on 2026-11-21. The 2026-11-06 payment covers 7 to 20 Nov (periods run Saturday to Friday from 2025-05-10), so one day ($185.71) is owed, not the $371.43 they offered.
- Dimiter plans to move back into Birrell. The loan projection still assumes 2027-01-01 (not yet updated to the earlier date).

### 5.5 Tax decisions (FY2025-26 return, lodged 2026-10-25)

- Bills count in the year paid; dividends by the payment date on the statement.
- Work purchases over $300 are depreciated over 3 years, straight line, pro rata by month in the first year (Dimiter's instruction "the depreciation schedule for items is 3 years"). Purchases of $300 or less and subscriptions are claimed in full.
- Microsoft 365 is a yearly subscription ($109 renewal 2026-03-31, claimed in full). Last year's workbook wrongly depreciated it.
- Google Workspace is a monthly subscription billed to Overwatch Pty Ltd: not on Dimiter's return (moved to the Overwatch receipts).
- Sony A1 claim is net of the $500 Sony cashback (2026-04-10).
- Old work assets keep last year's schedules: Socket 370 PC parts (4-year diminishing value), Microsoft Project (5-year straight line), mobile phone and test computer (3 years).
- Property management: $100 per property per month ($1,200 each) under agent fees, on Dimiter's word (no payment on the accounts).
- Interest on the separate loan from Mum, also for Dudley: $390 x 12 = $4,680, added to Dudley's interest on loans.
- Birrell blinds: $580 (EKKO invoice, customer Bissera), 5% a year as in the workbook. The $265 locksmith (2025-04-14) is an initial repair before first letting: cost base, not deductible.
- Birrell keys cut 2025-11-07, $23.95 cash: other expenses.
- Crypto: $100 into Independent Reserve 2025-11-25, sold 2026-02-06 for $67.32 after fees: capital loss carried forward $32.
- Dudley borrowing expenses: $668.80 (FY2025-26 share of the 5-year schedule in last year's workbook).
- Lodged figures: taxable income $225,161; Birrell net rent $58,844.03; Dudley -$30,580.53; deductions $3,733.66; amount payable $13,512.65, paid 2026-11-02.
- Division 293 tax FY2025-26: $398.35, raised 2026-11-09 on Income tax 551, effective 2026-12-07. Taxable income $225,161 + concessional contributions received in FY2025-26 $27,495.00 = $252,656, which is $2,656 over $250,000; 15% = $398.40 (ATO is 5c lower). Dimiter elected on 2026-11-10 to have AustralianSuper (account 29008201) pay it from his super, so it will not appear in the bank data. Expect a withdrawal of $398.35 on the next super export and a matching credit on Income tax 551. Do not pay it by BPAY. Not deductible. Expect Div 293 again for FY2026-27.

### 5.6 Employment and super

- Employer was PassMark Software Pty Ltd (ACN 099 321 392, listed by the ATO as Coogee Code Pty Ltd) to 2026-05-13, then PassMark Software Pty Ltd (ACN 696 978 220) after the sale. The contract names Touchstone Point Pty Ltd; the side letter moved employment to the new subsidiary.
- Salary history: $160,000 (May 2023), $176,000 (Jan 2024), $190,080 (Jan 2025), $199,584 (Jan 2026). $6,000 bonus each December 2023-2025.
- The annual-leave drop of 10.46 hours on the 2026-06-12 payslip is the May leave taken (16 h) less May accrual (5.54 h) at the old company. It is correct.
- Super: the old company's 2026-05-01 payslip super of $1,995.84 (April work) was never paid to AustralianSuper. Its periods jumped from 2026-04-01..04-30 to 2026-05-02..06-14. It was due 2026-07-28. Proof it is the April payslip: the 2026-04-02 payment of $6,612.48 is exactly the Dec ($1,900.80 + $720 on the bonus), Jan and Feb payslips, so the old company pays by pay date and the 2026-05-04 payment is March's. Sally Ram (finance) replied on 2026-11-06 confirming only that 13-31 May was paid; April is still unconfirmed and a follow-up was drafted.
- Super withdrawals: the $398.35 Division 293 release (5.5) will show as a debit on the AustralianSuper export. It is not a contribution and must not affect the owed-vs-received ledger.

### 5.7 Exceptions to the matching principle

- Purchases flagged `claim` in `receipts.json` are listed even without a matching payment (the Sony microphone was paid from another card).
- The Birrell keys cash receipt.
- Payslips and super contributions are shown in full from May 2023.
- The loan history is rebuilt from the 2025-01-13 drawdown.

## 6. Output constraints

### 6.1 Target environment

- One self-contained HTML file. Viewed in **Internet Explorer 6 on Windows XP**, 1600 x 1200, on a 1.4 GHz Pentium III-S.
- Measured by the built-in diagnostics: window 1579 x 1031 px, no SVG, no canvas, no `JSON`, no `querySelector`, no `getElementsByClassName`, no `addEventListener` (only `attachEvent`), `rgba()` fails, `opacity` and `border-radius` work, CSS1Compat mode.
- Installed fonts include Tahoma, Verdana, Arial, Trebuchet MS, Georgia, Courier New, Lucida Console. No Segoe UI, Calibri or Consolas.

### 6.2 Page rules

- Body Tahoma 13px. Do not rely on ClearType.
- Content fits 1524 px wide with no horizontal scrolling.
- JavaScript is ES3 only: `var`, `function`, `getElementById`, `innerHTML`, `style.display`, `window.clipboardData`. Inline `onclick` handlers.
- Charts are built from tables and absolutely positioned `div`s with explicit pixel sizes and `font-size:1px;line-height:1px` on empty blocks.
- Keep it simple, with as few hacks as possible:
  - No lazy rendering.
  - No CSS expressions.
  - No frameworks or external files.
- Load speed does not matter. Smooth scrolling does.
- All three views (All, FY2025-26, FY2026-27) are rendered into the page at build time. Switching only shows or hides a view.
- The year switcher sits in a panel fixed at the top right. The default view is the most recent complete financial year (worked out from the build date in Sydney time), currently FY2025-26.
- Every `id` and `name` is prefixed per view. The global `V` holds the current prefix, and the default view's prefix is set at load.
- Output is ASCII with HTML entities. The file starts with the `saved from url=(0014)about:internet` comment and `X-UA-Compatible: IE=edge`.
- A Diagnostics button shows a panel with browser, screen, fonts, features, timings and script errors, plus a 100 px square and font samples, for copying back.

### 6.3 Interaction and formatting

- **Expanding rows:** collapsible rows use `xtr(id, parents, cells, haschild, cls)` with `[+]`/`[-]` markers. Use "expand all / collapse all" links (`ea`) where a table has many groups. Sections can be collapsed with `xsec`.
- **Copy buttons:** a small "copy" button next to each figure to be typed into myTax copies the bare number (no `$`, no commas; dates as dd/mm/yyyy). The button must not toggle the row it sits in (`event.cancelBubble=true`).
- **Dates:** use ISO 8601 (YYYY-MM-DD) wherever dates must line up, for example in the super ledger.
- **Money:** `$12,345.67` in tables, whole dollars in summaries.
- **Colours:** use the validated palette: #2a78d6, #eb6834, #1baf7a, #eda100, #e87ba4, #008300, #6250d6, with grey #8a99a8 for Other. Use red #c0392b / #f6c9c3 for problems and the yellow `note` box for checks and caveats.
- **Partial month:** the last month in the data is labelled "to N" in the month chart.

## 7. Sections

Sections appear in this order. "Views" says which year views show the section.

1. **Header and year switcher** (all views)
   - Title, the data coverage line (sources and their end dates) and the prepared date.
   - Fixed panel at the top right with All, FY26, FY27 and Diagnostics.

2. **The short version** (all views)
   - Bullet summary: living costs per month by year, housing per month, the biggest costs, health trend, travel, and Dudley against Birrell.

3. **Living costs by category** (all views)
   - One row per category with an amount per month for each period, the total, the share and a bar.

4. **Month by month** (all views)
   - A stacked bar per month: rent, eating out and coffee, trips and travel, everything else.
   - In year views, the six months either side are shown faded.
   - Partial month labelled. A table per month underneath with financial-year subtotals; hovering over a trips figure lists its trips.

5. **Trips** (all views, collapsible)
   - One expandable row per trip: dates, days, total.
   - Expanding shows every transaction, largest first, with details. Header rows have no details.
   - Includes prebooked costs (flights and visas paid months ahead) and Ayn's paybacks netted.
   - A sub-table, "Travel spending outside the trip dates", follows the trip list. Trip notes go below the table.

6. **Eating out and coffee** (all views, collapsible)
   - Places grouped and sorted by total.
   - Expanding a place shows each visit **oldest first**, to show price trends.
   - Count and total of visits of $10 or less.

7. **Biggest one-off purchases** (all views)
   - The 15 largest one-off purchases: not part of a trip, not a regular bill or activity, and not a place paid three or more times **unless the purchase is $500 or more**.
   - Uses transaction labels as names.

8. **Regular commitments now** (all views)
   - Rent, Mum, gym, AHM and subscriptions: average per month over the latest full quarter.

9. **Vehicles** (all views)
   - Rego, CTP and comprehensive cover by year for the VW Polo (CYX80Z) and Yamaha XT250 (KKJ54).
   - "Everything spent through the bank" by vehicle category, with sub-lines. The car's sub-line is "Parts, servicing and parking"; Ayn's car and Honda are separated.
   - Tolls by vehicle (plate or tag from the E-Toll statements).
   - Tolls by road with proper names: Sydney Harbour Bridge and Tunnel, Cross City Tunnel, WestConnex (M4 and M8), M2 Hills Motorway, Westlink M7, M5 South-West Motorway, Eastern Distributor, Lane Cove Tunnel, NorthConnex. Each road expands to a chronological list of date, segment, vehicle and cost.

10. **Fuel** (all views)
    - By financial year and by station.

11. **Investment properties** (all views)
    - Birrell first, then Dudley, then "Both properties". Everything about a property is kept together.
    - Each property has these parts, in order:
      - **Tax return FY2025-26: myTax Rent, in form order** (FY2025-26 view only). Every myTax Rent field in the page's order:
        - Property name and address.
        - Date property genuinely available for rent.
        - Weeks rented.
        - Ownership.
        - Rental income and other rental-related income, then the gross rent total.
        - Each expense field, from advertising to other expenses.
        - Total expenses and net rent.
        - Each figure has a copy button. Each number expands to its evidence: payments, bills, interest charges, schedules or lease periods. Empty fields say "leave empty" with the reason.
      - **Tax return FY2025-26: the bills behind the expenses** (FY2025-26 view only). Bills grouped under the myTax labels, by the date paid. Bills paid outside the year are shown in brackets but not counted.
      - **Cash flow by year:** rent, strata, council, water and repairs, plus loan interest for Dudley, and net.
      - **Rent received:** a bar chart on the shared timeline, $0 to $5,000 scale for both properties, coloured by tenancy type, with the tenancy bar underneath.
      - **Tenancy periods** table.
      - **All rent payments** table.
      - **Receipts** for the property.
      - Dudley only: a pointer to the loan section.
    - **Both properties:** the net of each property, the costs not split by property, and the combined net.

12. **Dudley St loan: history and payoff** (all views)
    - **Tiles:** amount borrowed, amount owing now, rate from the next change, payoff date, interest still to pay.
    - **Assumptions box.** The current assumptions:
      - Payment pattern: average paid in less redrawn over the last six full months, currently $8,935.38 a month.
      - Birrell move date: 2027-01-01. Birrell's rent stops and the Nancarrow rent stops, so the payment drops to $7,647.05.
      - Rate held after the last change.
      - Tax is left out.
    - **Chart:** balance at 30 June each year, actual against projected.
    - **Rate table:** ANZ rate from each change date, the change, the RBA decision 9 days earlier, the cash rate and the margin (2.04% throughout). The next expected change is shown in italics.
    - **Year-by-year table:** expands to each interest period (13th to 13th) with opening balance, borrowed and fees, paid in, redrawn, interest charged, recalculated interest (daily balance x rate / 365, with differences over $1 in red) and closing balance. Projected periods are greyed.

13. **Salary and employment** (all views, full history from May 2023)
    - **Tiles:** salary now, take-home, tax per month, bonus, leave balance.
    - **Pay rises** table.
    - **Every pay:** a stacked bar per payslip (take-home and tax), with bonus months in green.
    - **By financial year:** the year is decided by pay date. Each year expands to its payslips with gross, tax, take-home, super, leave taken, leave balance and the matching bank deposit. Gaps in the bank records are explained.
    - **Super ledger:** owed on the payslips against what AustralianSuper received.
      - Covers six months before the missing payment (from 2025-11-01) to now.
      - **One row per payslip and one per contribution, never grouped.** ISO dates in a monospace font.
      - A running Outstanding column.
      - Rows that belong together share a group letter and alternate white and light blue backgrounds.
      - The missing 2026-05-01 payslip and the 2026-07-28 deadline row are shown in red ("!"). Payslips not yet due are grey.
    - **Checks box:** reconciliation to the tax return, the leave carry-over at the sale, the August 2026 unpaid leave, and the super cap and Division 293.

14. **Subscriptions** (all views)
    - Gym, phone and internet, and apps. One Playground and The Bunker are merged.
    - A timeline of bars coloured by vendor, and a vendor table where each vendor expands to its payments in date order.
    - Gym and coach chart from the One Playground invoices: a stacked bar per invoice of trainer, gym, one-offs (admin fee and extra coach sessions) and card fees, with an invoice table.

15. **Savings and runway** (all views)
    - Balances rebased to 2025-07-01, redraw available, and "If the salary stopped now".

16. **Overwatch Pty Ltd** (all views)
    - Company costs paid personally (ASIC, hosting, Google Workspace).

17. **Telstra dividends** (all views)
    - 1,476 TLS shares. Payment date (from the statement), received date, unfranked, franked, cash and franking credit, with notes (withheld, reissued, conduit foreign income).

18. **Receipts** (all views)
    - Purchases to claim, travel bookings, Overwatch. Each with invoice date, supplier, item, billed to, total, GST, paid (bank) date, and financial year by invoice against paid.
    - A "Worth checking before you claim" box. Property receipts are in the property section.

19. **Tax return FY2025-26** (FY2025-26 view only)
    - **Lodged, assessed and paid** box (figures from the notice of assessment).
    - **myTax Deductions, in form order:** every field. "Other work-related expenses" expands into each item with its own copy button.
    - **myTax Capital gains or losses, in form order.**
    - **Work-related depreciation schedule:** a timeline bar per asset from purchase to fully claimed, split by financial year (grey for claimed, blue for this return, pale blue for still to come), a purchase tick, and the amount left after this year. A table follows with cost, method, claimed before, this year (with copy button) and left after.
    - **Claimed in full:** subscriptions and items of $300 or less, plus greyed past-year items.
    - **Dividends (item 11).**
    - A pointer to the rental sections.

20. **Category details** (all views)
    - Top merchants in each living-cost category.

21. **Footer**
    - Account totals in and out, and a pointer to the CSV.

### 7.1 The CSV

One row per transaction in scope: date, account, amount, group, category, merchant, original description, matched receipt file, trip, note (label).

## 8. Open items

- PAYG instalments from FY2026-27: $3,507 a quarter or 10.94% of gross rent and dividends. The rate option looks better after Birrell's rent stops.
- Missing $1,995.84 of super from the old company (see 5.6). Waiting on Sally's answer about the April payslip.
- Division 293 release of $398.35: check that AustralianSuper has paid it (around 2026-11-24) and that Income tax 551 is back to $0.00 before 2026-12-07. If it hasn't, call the ATO.
- The share of Dudley interest that relates to personal redraws: the 2026-11-02 redraws paid the tax bill and the Nancarrow rent.
- The Belle Property August 2025 statement (agent fees).
- The capital works and depreciation report for both properties.
- The loan projection's move-in date should follow the Birrell handover (2026-11-21).
- Ask ANZ for a rate review (margin 2.04% over cash).

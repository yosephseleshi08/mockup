# Grandma's Recipes — Precision Fixes for Customization, Export & Kitchen Operations

A comprehensive, non-breaking repair plan to diagnose and resolve all six user-reported issues across customization baking, clean exports, ticket generation, and owner dashboard workflows.

## User Review & Critical Decisions

> [!IMPORTANT]
> All 6 items from your feedback are diagnosed to their exact root causes in the codebase and will be implemented cleanly without touching any unrelated features or breaking checkout, modals, or visual styles.

- **Confirmed Decision 1: Text & Image Customization Baking on Clean Export**: Rather than relying solely on fragile `:nth-child` CSS selectors that can miss live edits, `exportCleanStandaloneHtml()` will directly synchronize the live DOM content, sync active `<img>` source data, and update `window.__CONTENT__` so that exported standalone files retain 100% of your edits in any browser or environment.
- **Confirmed Decision 2: Authentic Reservation Ticket PDF**: Replace the raw `.txt` file download with a high-fidelity thermal card PDF formatted via jsPDF (matching customer receipt and kitchen order styling) with party size, date, time, guest details, and brand header.
- **Confirmed Decision 3: Remove Export Toast Message**: Completely eliminate the `"clean ..browser saved..."` / `"Clean site exported..."` toast on export for a silent, distraction-free download.
- **Confirmed Decision 4: Bottom "Open Owner Dashboard" Button**: Bind a universal delegated click listener to `#stripe-owner-cta-btn` that reliably transitions from the customer receipt to the live Kitchen Order Ticket / Owner view every time.
- **Confirmed Decision 5: Kitchen Ticket Download Button**: Re-route `#kt-download-btn` from `buildReceiptHtml()` to `downloadKitchenTicketPdf()` so it strictly downloads the BOH Kitchen Ticket PDF rather than the customer receipt.

---

## 1. Overview & Core Concept

- **What It Does**: Refines the Grandma's Recipes direct ordering web application to ensure that all store owner customizations (brand names, menu descriptions, uploaded photos, color palettes) permanently persist when exporting a clean standalone site, while polishing operational ticketing (Kitchen Order PDFs and Table Reservation PDFs).
- **Target Audience / Persona**: Small bakery and restaurant owners running direct online sales with 0% third-party commission, requiring zero technical hurdles to edit their menu and manage orders.
- **Key Value**: Guarantees WYSIWYG fidelity on export, eliminates workflow friction, and provides print-ready, professional tickets for both front-of-house table reservations and back-of-house kitchen operations.

---

## 2. User Experience & Visual Design

### Key User Flows

1. **Customization & Clean Export Flow**:
   - Owner edits item descriptions, prices, headers, or replaces photos via the toolbar or element inspector.
   - Owner clicks **Clean Export**.
   - The file `grandmas-recipes-direct.html` downloads immediately and silently (zero pop-up toasts).
   - Opening the exported file in any browser or device displays the exact edited text, modified colors, and updated images without any editing toolbar chrome.

2. **Table Reservation Ticket Flow**:
   - Customer or host reserves a table / pop-up pickup slot via `#reservation-overlay`.
   - On submission, the confirmation panel displays **Download Reservation Ticket**.
   - Clicking download produces `Grandmas-Reservation-[ID].pdf`: a clean, thermal ticket card featuring date, formatted 12-hour time, guest count, customer name, contact phone, and store guidelines.

3. **Receipt → Kitchen Order Flow**:
   - When an order completes on the sample checkout, the customer receipt preview appears.
   - Clicking either the top green banner or the bottom **👩‍🍳 Open Owner Dashboard** button instantly brings up the live incoming Kitchen Ticket.
   - Clicking **Download** on the Kitchen Ticket modal generates `Kitchen-Ticket-[ID].pdf` with preparation checklist checkboxes, pickup window, and item modifications (no customer receipt mix-ups).

### Visual Layout & PDF Specifications

| Document | Format | Palette | Key Elements |
| :--- | :--- | :--- | :--- |
| **Customer Receipt PDF** | 340pt × Auto Card | Slate (#0F172A) & Emerald (#059669) | Store header, item breakdown, tax, savings banner, 0% commission note |
| **BOH Kitchen Ticket PDF** | 320pt × 560pt Slip | Warm Stone (#FBF9F5) & Amber (#B45309) | `[PICKUP]` badge, checklist boxes, customer phone, modifier callouts |
| **Reservation Ticket PDF** | 340pt × 520pt Card | Ivory & Emerald Accent | Guest name, party size, reservation date/time, arrival checklist |

---

## 3. Key Product Decisions & Trade-Offs

- **Decision 1: Direct DOM & Content State Synchronization for Export**
  - *Chosen Approach*: During clean export, traverse all live DOM elements and copy `innerHTML` for text and `src` for all `<img>` tags directly to the clone, while baking `window.__CONTENT__` and `window.IS_CLEAN_EXPORT_MODE = true`.
  - *Why*: In previous revisions, `renderAll()` would run on page load and reset elements to the defaults in `window.__CONTENT__`. Synchronizing both DOM and data structures guarantees permanent edits across reloads and exports.
  - *Alternatives Considered*: Storing edits in localStorage only. *Rejected* because standalone exported HTML opened on another machine or file protocol does not share origin localStorage.

- **Decision 2: Dedicated jsPDF Reservation Slip vs. Formatted Text**
  - *Chosen Approach*: Build a dedicated jsPDF layout for reservations with 100% clean ASCII characters, consistent with the existing receipt and kitchen ticket styling.
  - *Why*: Raw `.txt` files looked unpolished and unprofessional compared to the rest of the application.

- **Decision 3: Delegated Document-Level Event Handling for Modal Triggers**
  - *Chosen Approach*: Use delegated event listeners on `document` targeting `#stripe-owner-cta-btn` and `#kt-download-btn`.
  - *Why*: Modal elements are re-rendered and updated dynamically upon checkout. Delegated listeners ensure clicks are never lost due to DOM re-cloning.

---

## 4. Technical Architecture & Data Strategy

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                             DIRECT ORDERING SYSTEM                          │
├───────────────────────────┬─────────────────────────────────────────────────┤
│    Storefront & Editing   │                Operations & Modals              │
│                           │                                                 │
│  ┌─────────────────────┐  │  ┌────────────────────┐   ┌──────────────────┐  │
│  │ Draggable Toolbar   │  │  │ Customer Receipt   │   │ Kitchen Ticket   │  │
│  │  • Colors / Brand   │  │  │  • Top Banner CTA  │──>│  • Live Checklist │  │
│  │  • Element Inspect  │  │  │  • Bottom CTA Btn  │   │  • Download PDF  │  │
│  └──────────┬──────────┘  │  └─────────┬──────────┘   └─────────┬────────┘  │
│             │             │            │                        │           │
│             ▼             │            ▼                        ▼           │
│  ┌─────────────────────┐  │  ┌────────────────────┐   ┌──────────────────┐  │
│  │ exportCleanHtml()   │  │  │ Receipt PDF        │   │ Kitchen Ticket   │  │
│  │  • DOM live sync    │  │  │ (Customer Copy)    │   │ PDF (BOH Copy)   │  │
│  │  • Img src bake     │  │  └────────────────────┘   └──────────────────┘  │
│  │  • Zero toolbars    │  │                                                 │
│  │  • No export toast  │  │  ┌────────────────────┐   ┌──────────────────┐  │
│  └─────────────────────┘  │  │ Reservation Modal  │──>│ Reservation PDF  │  │
│                           │  │  • Date/Time/Party │   │ (Thermal Slip)   │  │
│                           │  └────────────────────┘   └──────────────────┘  │
└───────────────────────────┴─────────────────────────────────────────────────┘
```

### Planned Code Adjustments

1. **Clean Export Baking (`exportCleanStandaloneHtml`)**:
   - Before cloning, sync any focused contenteditable element (`document.activeElement.blur()`).
   - Copy live `img.src` from `document.querySelectorAll('img')` directly into corresponding clone images.
   - Synchronize `window.__CONTENT__` values in clone script tags to match edited text and brand attributes.
   - Remove `showToast(...)` completely from the export function.
2. **Reservation Ticket Generation (`downloadReservationTicketPdf`)**:
   - Replace lines 4867–4905 in `/index.html` with an authentic jsPDF thermal ticket function.
   - Safely encode reservation date, 12-hour formatted time, guest count, and contact information.
3. **Owner Dashboard / Kitchen Ticket Navigation**:
   - Ensure `#stripe-owner-cta-btn` has reliable delegated click handling that triggers `closeStripe()` and `openKitchenTicket()` or `populateOwnerDashboard()`.
4. **Kitchen Ticket Download Binding**:
   - Update line 4511 in `/index.html` to point `#kt-download-btn` directly to `downloadKitchenTicketPdf()`, eliminating the inadvertent receipt download.

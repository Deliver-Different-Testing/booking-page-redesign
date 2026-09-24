# Booking page — full inventory (source audit, 24 Sep 2026)

Source: `booking` repo (git.customd.com, read-only), local mirror dated 7 Aug 2026.
Front end: AngularJS 1.x + ui-router, Bootstrap 3 **and** 4 both loaded, jQuery loaded twice, Font Awesome 5, HERE Maps (Google Places for autocomplete), ui-select, uib-timepicker.
Files that make the booking page: `desktopHomeView.html` (1,935 lines) + `desktopHomeControl.js` (5,460 lines) for desktop; `homeView.html` (1,499) + `homeControl.js` (3,658) for phones. Desktop vs phone is chosen once at boot by a user-agent regex or `innerWidth < 768`.

Redesign canvas: https://claude.ai/artifact/K4xSXPjNaCidQMzS8ivJh5

---

## 1. Screens (desktop)

| Step | What it is |
|---|---|
| Loading | Spinner until clients load and the client address geocodes |
| `client` — "Who are you booking as?" | Client select (or client search for staff), User select, Saved Booking search, buttons Edit Client Settings / Add/Edit Users, "Confirm Client & Contact". Credit-card clients (ids 9196, 35957, 11) get Name / Mobile / Email fields instead of a user, plus a "book on CRECI" warning for staff. Clients without `bookJobPermission` see a "Save Payment Info" Stripe button. Skipped for single-account customers. |
| `address` — the booking form | One fixed white panel (90% wide, min 818px tall, 3px black border) with a top bar and **four 25% columns**: (1) PICK UP FROM + DELIVER TO, (2) PACKAGE, (3) DATE + TIME, TRACKING, REFERENCES, SERVICE, (4) HERE map. Footer: error alert, pink "Accessorial Charges (n)" + teal "Book Job". |
| `jobBooked` — confirmation | Two columns: "Confirmation Details" (from/to, notes, ready date/time, deliver-by, qty/weight, DG, service, price ex/incl GST, estimate warning for speeds 6/7) and "Job Data" (print reminder, job number, tracking link, print label link, update/create saved booking, Stripe charge note, Create Return Booking). "Book Another?" button. |
| Stripe `confirmation` route | Separate page after Stripe checkout; desktop layout only; contains staff-only copy ("Please make sure the client has paid! If not remember to void the job!"). |

### Top bar of the form
Back arrow (change client) · Saved booking search + Update/Add link · "Booking as CODE / Name" · Reset. Optional amber banner "Return booking for JOB · Clear".

## 2. Every field on the form

**Pick up from**: From (company name label with "Add Company Name" toggle + trash icon), address search (Google or Address Book radio, NZ only, shown under DELIVER TO), Extra From Info, Pickup Contact Name *, Pickup Contact Phone, Pickup Notes.
**Deliver to**: To (company name toggle + swap icon), address search, Extra To Info, Delivery Contact Name, Delivery Contact Phone, Delivery Notes (* when Leave my Parcel = Other).
**Package**: Vehicle select (Bike, Car, Cargo Bike, Chilled Truck, Transit, Truck, Van — per tenant) with a chevron to reopen the truck modal; Standard / Custom toggle. Standard: Package Size select (name + L×W×H + weight), Quantity, Total Weight. Custom: table L / W / H / cubic / weight / Qty / item-types tag / remove, max 10 rows, "Package Size (add to custom)" select + plus, "Dimensions are: Per Item / Per Job" switch. "Any Dangerous Goods?" checkbox with chevron to reopen the DG modal.
**Date + time**: one button showing "Ready Now / Deliver ASAP" or the saved times or "On Hold"; opens the time modal.
**Tracking**: Tracking Method (Web Site, Email, Text, Email & Text), Tracking Email * / Tracking Mobile * as needed, Leave my Parcel (Signature Required, Letter Box, Front Door, Back Door, Safe Place, Other).
**References**: Client Ref A / B (label + per-client message + * if mandatory; select if the client has a defined list, else text with suggestions; label click opens the list editor), Client Notes ("for your reference only").
**Service**: empty-state box listing what is still missing (From Address, To Address, Package Size, Vehicle Size); spinner; "No services available … $0.00"; or a list of rate cards (icon, name + Van/Bike suffix, price, description, green/yellow/orange availability pill, and for the selected rate a price breakdown table with Excl GST total).
**Footer**: Book Job (shows total when extras are selected).

## 3. Every modal and popup

| Modal | Contents | Buttons |
|---|---|---|
| Confirm DG | DG checkbox, DG Docs checkbox, DG Class select, Dry Ice Weight + LBS/KG (class 9) | CONFIRM / CLEAR / CLOSE (static backdrop) |
| Confirm Truck Details | Truck checkbox, TailLift (Pickup), TailLift (Dropoff), Deliver To: Business / Residential | CONFIRM / CLEAR / CLOSE |
| Select Prebook Time | Normal / Recurring toggle. Normal: Ready Date, Ready Time (+ time zone outside NZ, economy-run select for NZ economy speeds), Set Deliver By → date/time, Your Current Time Zone, On Hold. Recurring: name, Recurrance Days multi-select, Frequency (Weekly, Fortnightly, First/Second/Third of month, First/Last workday), Initial Days Window, Holiday Behaviour (Don't book / Deliver Next Day / Book Anyway), NZ warning that today is not created | CONFIRM / RESET / CLOSE |
| Add/Edit Reference A (and B) List | paginated editable list with plus/minus | SAVE / CLOSE |
| Accessorial Charges | optional per-leg tabs (Pickup Agent / Flight / Delivery Agent), table: checkbox, Service (+ AUTO / ALWAYS INCLUDED badges), Description, Calculation (flat / per-unit with locked auto values / percentage / quote required), Amount; price breakdown with Excl and Incl GST | SAVE & CLOSE |
| Update/Add Saved Booking | name input, "Save booking time?" checkbox | UPDATE / CREATE / CLOSE |
| Create New Saved Booking | name input, save-time checkbox | CREATE / CLOSE |
| Print your Label | Label Size (PDF A4, 100×174, 100×150) | PRINT / CLOSE |
| Global prompt | "Confirm saved booking update" / "Confirm saved booking creation" | CONFIRM / CANCEL |
| Toast notification | yellow box top-right, 5 s, ~40 different "Error getting …" messages plus success messages |

## 4. Validation on Book Job (in order, first failure stops)
A valid From Address · Pickup Contact Name · valid To Address · delivery notes when Leave = Other · Package Size (standard) · at least one valid custom row (cubic or L/W/H, weight > 0) · a selected service · Ready Date / Time present, in range, not in past · On Hold not allowed on scheduled rates · DG class and DG docs when DG · Deliver-to when truck · Tracking Email / Mobile when required · Client Ref A / B when mandatory. Errors show as one red alert at the panel bottom plus a red border on the field. Delivery contact, phones, weight caps and lead time are only checked by the API.

## 5. Behaviours worth keeping (and where they went in the redesign)
- Rates refresh automatically once from, to, package and vehicle are known → services rail on the right, always visible.
- Auto-select fastest available rate; contact defaults for speed, package size, vehicle, tracking method, hold → unchanged, just visible earlier.
- Economy speeds snap the ready time to the next run → shown as the service description ("Next economy run leaves 2:30 pm").
- Van/bike auto-detection from cubic and weight (NZ) → shown in the "Priced for Van" caption.
- Address book (NZ, `addressBookOnly` tenants) → merged into one address search; results carry an ADDRESS BOOK badge.
- Saved bookings load/update/create → context bar + "Save as template".
- Return booking (inline and `?returnFor=` URL) → "Book the return trip" on the booked screen.
- Recurring, deliver-by, on hold, time zones (US) → inline "When" section.
- Accessorial and per-leg portion charges → "Extras" side sheet.
- Credit-card / Stripe clients, CRECI → gate screen variant (noted, not drawn).

## 6. Things the current build gets wrong that the redesign removes
- 4-column fixed panel that never scrolls the page; 45+ empty spacer divs; 12+ button styles; no focus rings; 2px black square inputs in a rounded card; New York map PNG as the page background on an Auckland app.
- Availability shown by colour only (#00ff00 / yellow / orange).
- Errors only at the bottom of the panel.
- Phone version is a 12-step wizard missing recurring, deliver-by, dry ice, item types, extras, address book, reference lists, weight.
- Staff-only text on customer screens; hard-coded client ids (9196, 35957, 11, 9673) and GST × 1.15 in the template.
- `isSignatureRequired` is always sent as true (reads the wrong scope variable); a precedence bug auto-selects any "delivery" lift-gate charge; return-job prefill silently drops DG and weight.

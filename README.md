# Invoice-Generator
I've struggled with creating Invoiced for my clients, so I've gone ahead and developed and built a system that allows for seamless building and expense calculation for my events.

## Architecture

Everything lives in a single file: `index.html`. CSS, HTML, and JavaScript are all inline — no modules, no bundler, no external dependencies.

### Data flow

1. The sidebar contains static HTML inputs (invoice details, business info, hotel clause, notes) and dynamically rendered event/discount cards.
2. Any input change fires `onChange()` → debounced 80 ms → `readState()` → `render()`.
3. `readState()` walks every DOM input and writes values into the global `state` object.
4. `render()` reads from `state` and overwrites `document.getElementById('invoice').innerHTML` with the full invoice HTML.
5. `renderSidebar()` rebuilds only the events and discounts containers (the parts that are dynamic lists).

### State object shape

```js
state = {
  // invoice fields
  num, status, issued, due, billTo, notes,
  events: [{ id, name, items: [{ name, qty, price, comp, details: [{ name, qty }] }] }],
  discounts: [{ id, name, amount }],
  hotelEnabled, hotelText,
  // business profile (persisted, survives New Invoice / Clear All)
  bizName, bizLogoText, bizLogoImg,  // bizLogoImg is a base64 data URL
  bizAddress, bizPhone, bizEmail,
}
```

Events are header rows (no price). Items inside events are priced line items (`comp: true` renders "COMP" instead of a price). Details inside items are disclosure-only rows (no price, indent level 2).

### Persistence

`saveData()` serialises the full `state` to `localStorage` under the key `'rnd-invoice'`. `loadData()` restores it on page load. `newInvoice()` and `clearAll()` both preserve the `biz*` fields so business identity survives invoice resets.

### Print / CSS

`@media print` CSS hides `#sidebar` and `#preview-pane`'s background, printing only `#invoice`. The invoice is sized at 816 px wide (US Letter at 96 dpi).

### ID conventions for dynamic elements

Dynamic sidebar inputs use predictable IDs so `readState()` can find them:
- `ev-name-{id}` — event name
- `item-name-{eventId}-{i}`, `item-qty-…`, `item-price-…`, `item-comp-…` — item fields
- `detail-name-{eventId}-{itemIdx}-{detailIdx}` — detail fields
- `disc-name-{id}`, `disc-amount-{id}` — discount fields

# Q075 — Your answer

## (a) The trace

Initial book contents:
- `A1`: Sari, WF, WASHING, 14:00
- `A2`: Budi, WDI, READY, 11:00
- `A3`: Ayu, IR, RECEIVED, 16:00
- `A4`: Dewi, WF, READY, 12:00
- `A5`: Rahmat, WDI, WASHING, 13:00
- `A6`: Citra, WF, DRYING, 15:00

Trace of the 10 events:

| Event | Action / Description | Rows Shown / View Update | Explanation |
|:---|:---|:---|:---|
| 0 | attach | `['A2 Budi WDI READY 11:00', 'A4 Dewi WF READY 12:00', 'A5 Rahmat WDI WASHING 13:00', 'A1 Sari WF WASHING 14:00', 'A6 Citra WF DRYING 15:00', 'A3 Ayu IR RECEIVED 16:00']` | Default sort `BY_READY` (11, 12, 13, 14, 15, 16). Initial view attachment. |
| 1 | filter 'ra' | `['A5 Rahmat WDI WASHING 13:00', 'A6 Citra WF DRYING 15:00']` | Customers containing "ra" (case-insensitive): "Rahmat", "Citra". Ordered by ready hour (13:00 < 15:00). |
| 2 | sort by customer | `['A6 Citra WF DRYING 15:00', 'A5 Rahmat WDI WASHING 13:00']` | Sort changed to alphabetical by customer: "Citra" precedes "Rahmat". |
| 3 | add A7 Budi | `(no update)` | A7 has customer name "Budi", which does not contain "ra". Rows remain identical, so view is not notified. |
| 4 | advance A5 | `['A6 Citra WF DRYING 15:00', 'A5 Rahmat WDI DRYING 13:00']` | A5 status advances from `WASHING` to `DRYING`. A5 is in the displayed list, so the rendered string changed; view is updated. |
| 5 | filter 'RA' | `(no update)` | Filter is case-insensitive (`"RA"` normalizes to `"ra"`). Matching rows and sort order do not change; view is not updated. |
| 6 | only DRYING | `(no update)` | Both currently shown rows (A6 Citra and A5 Rahmat) are already in status `DRYING`. Filter result is identical; view is not updated. |
| 7 | advance A6 | `['A5 Rahmat WDI DRYING 13:00']` | A6 status advances from `DRYING` to `READY`. Since `onlyStatus` is `DRYING`, A6 is now excluded. List changes to only A5; view is updated. |
| 8 | filter '' | `(no update)` | Clearing the text filter keeps `onlyStatus == DRYING`. Among all orders, only A5 is in `DRYING`. The displayed row list is still just A5; view is not updated. |
| 9 | only READY | `['A2 Budi WDI READY 11:00', 'A6 Citra WF READY 15:00', 'A4 Dewi WF READY 12:00']` | Orders with status `READY`: A2 (Budi), A4 (Dewi), A6 (Citra). Sorted by customer name alphabetically: Budi (A2) -> Citra (A6) -> Dewi (A4). View is updated. |

**Total:** 6 view updates for 10 events.

---

## (c) Why compare

### Why compare the new rows with the old ones instead of notifying on every event?
1. **Prevents Unnecessary View Repaints & Layout Thrashing:** In GUI programming, repainting widgets or recalculating layout trees is expensive. If a model event does not affect the currently filtered/sorted slice (e.g. an order outside the filter was added or changed, or a filter keystroke produced the same casefold text), skipping the notification avoids unnecessary UI redraws and screen flickering.
2. **Decouples High-Frequency Model Events from the View:** The underlying data store may emit high-frequency change events. Checking `rows.equals(shown)` acts as a deduplication boundary, ensuring notifications are fired strictly when there is a true presentation-state change.

### Why is the view-model easy to test?
1. **Decoupled from GUI Toolkits (Headless Testing):** The view-model has zero dependencies on Swing (`JFrame`, `JTable`, etc.) or Tkinter. It accepts plain callback consumers (`Consumer<List<String>>` or simple functions) and operates purely on POJOs/dataclasses and standard collections.
2. **Determinism and State Inspection:** It can be instantiated, manipulated, and asserted synchronously in simple unit tests (e.g. JUnit, pytest) without launching an Event Dispatch Thread (EDT), creating display windows, or needing UI mock frameworks.

---

## (d) Chapter 9 ideas

The design leverages several key collection and generics ideas from Chapter 9:
1. **Comparator Composition and Chaining:** Using `Comparator.comparing(...).thenComparing(...)` (and `String.CASE_INSENSITIVE_ORDER`) to compose multi-key sorting rules cleanly with tie-breaking without writing nested `if-else` comparator logic.
2. **Functional Pipelines over Collections (Streams & Comprehensions):** Transforming, filtering, and projecting collection data using declarative pipelines (`filter`, `sorted`, `map`, `toList()` in Java Stream API, and list comprehensions in Python) rather than error-prone manual loops.
3. **Defensive Copies & Immutability:** Protecting encapsulation with immutable snapshots such as `List.copyOf(orders)` in Java or `tuple(self._orders)` in Python, preventing external mutation of the model's internal collection.
4. **Generic Functional Interfaces:** Utilizing Java standard generic functional interfaces like `Consumer<List<String>>`, `Comparator<Order>`, and `Runnable` for decoupling callbacks and event listeners.

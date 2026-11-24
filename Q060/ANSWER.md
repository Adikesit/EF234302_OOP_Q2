# Q060 — Your answer

## (a) The trace

Starting state (Step 0): WF selected, Express unticked, field empty, label is `(no quote yet)`, handled = 0.

| Step | Action | `handled` | Result Label |
|:---|:---|:---:|:---|
| 0 | start | 0 | `(no quote yet)` |
| 1 | types 4.2 | 0 | `(no quote yet)` |
| 2 | clicks Quote | 1 | `WF 4.5 kg: Rp31,500, ready in 48 h` |
| 3 | ticks Express | 2 | `WF 4.5 kg express: Rp47,250, ready in 12 h` |
| 4 | chooses WDI | 3 | `WDI 4.5 kg express: Rp67,500, ready in 18 h` |
| 5 | types abc | 3 | `WDI 4.5 kg express: Rp67,500, ready in 18 h` |
| 6 | presses Enter | 4 | `invalid weight: abc` |
| 7 | unticks Express | 5 | `invalid weight: abc` |
| 8 | chooses IR | 6 | `invalid weight: abc` |
| 9 | types 0.4 and clicks Quote | 7 | `IR 2 kg: Rp10,000, ready in 24 h` |

- **Step 1:** Typing into the text field does not fire an `ActionEvent`. No listener is triggered, so `handled` remains 0 and label stays unchanged.
- **Step 2:** Clicking the Quote button fires an `ActionEvent`. `handled` becomes 1, `quoted` is set to `true`. Weight 4.2 kg is rounded up to 4.5 kg. WF rate Rp7,000 * 4.5 = Rp31,500, 48 h.
- **Step 3:** Ticking Express fires an `ItemEvent`. `quoted` is true, so re-quote occurs. `handled` becomes 2. Express price = ⌊3 * 31,500 / 2⌋ = Rp47,250, hours = max(6, ⌊48/4⌋) = 12 h.
- **Step 4:** Choosing WDI fires an `ActionEvent` from the combo box. `handled` becomes 3. WDI min is 3.0 kg, rate Rp10,000, base hours 72 h. Price = 10,000 * 4.5 = 45,000; express price = ⌊3 * 45,000 / 2⌋ = Rp67,500; hours = max(6, ⌊72/4⌋) = 18 h.
- **Step 5:** Typing "abc" into the text field does not fire an `ActionEvent`. No event is delivered, so `handled` remains 3 and label remains showing the quote from step 4.
- **Step 6:** Pressing Enter in the text field fires an `ActionEvent`. `handled` becomes 4. Parsing "abc" fails, producing `invalid weight: abc`.
- **Step 7:** Unticking Express fires an `ItemEvent`. `handled` becomes 5. Parsing "abc" still fails.
- **Step 8:** Choosing IR fires an `ActionEvent` from the combo box. `handled` becomes 6. Parsing "abc" still fails.
- **Step 9:** Typing 0.4 does not fire an event, but clicking Quote fires `ActionEvent`. `handled` becomes 7. For IR, minimum is 2.0 kg. 0.4 kg is raised to 2.0 kg. Rate Rp5,000 * 2.0 = Rp10,000, hours = 24 h.

---

## (b) The stale label

### Why Step 5 leaves a label that contradicts the field:
In Swing (and GUI frameworks in general), a `JTextField` only fires an `ActionEvent` when the user presses **Enter** (or when a button like "Quote" is clicked). Simply typing or modifying characters inside the text field changes the underlying document model, but does **not** deliver an `ActionEvent` to registered `ActionListener`s. Therefore, after the user types `"abc"`, no listener is notified, `refresh` is not called, and the label remains displaying the previous valid quote (`WDI 4.5 kg express...`) even though the text field now displays `"abc"`.

### What event would you listen to in order to fix it:
To detect every keystroke, paste, or text modification immediately:
- In **Java Swing**, you would listen to changes on the field's document model by registering a `DocumentListener` (handling `insertUpdate`, `removeUpdate`, and `changedUpdate`) on `kg.getDocument()`. Alternatively, you could use a `PropertyChangeListener` or a `KeyListener` (specifically `keyReleased`), though `DocumentListener` is the standard and robust Swing mechanism because it catches non-keyboard edits (such as copy-paste) as well.
- In **Python Tkinter**, you would attach a trace on the `StringVar` associated with the entry using `kg.trace_add("write", callback)` (or bind `<KeyRelease>`).

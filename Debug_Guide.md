# Development Guide: quirk_util.html

This document is a comprehensive reference for any developer who wishes to maintain or extend the custom Quirk visualizer for QCraft. It explains each logical component, data flow, and technical integration of the file.

---

## 1. Global Variables and System State

These variables maintain the application state in memory during the user session.

- `window.urlAntesDeCadena`: Stores the Quirk URL without the QCraft parameters. It is vital for reconstructing the pure circuit.
- `window.cadenaCircuito`: The unique ID (string) of the circuit in the QCraft database.
- `window.debugBreakpoint`: Integer indicating the index of the column where a breakpoint has been set. If `null`, there is no breakpoint.
- `ConjuntoResultados`: An array where each index `i` contains a JSON string with the probabilities for column `i`.
- `dataGlobal`: Structured object containing all the circuit "slices" (state snapshots) processed from `ConjuntoResultados`.
- `debugAlreadySent`: Boolean flag to prevent duplicate POST requests to the backend.

---

## 2. URL Handling Utilities (Lines 745-793)

Quirk encodes the circuit in the `#` fragment of the URL. QCraft adds extra parameters like `&cadena=`.

- `urlContieneCadena()`: Returns a boolean. Checks if the circuit is already persisted in the DB by looking for `&cadena=` in the URL.
- `obtenerValorParametroCadena()`: Uses `URLSearchParams` on the fragment to extract the circuit ID string.
- `obtenerURLAntesDeCadena()`: Cleans the URL by deleting everything after the QCraft ID, allowing Quirk to process the base JSON.
- `obtenerURLYCadena()`: Main validator. If the circuit is empty (`:[]`), it triggers an `alert` preventing translation.

---

## 3. Integration with React and Persistence (Lines 795-822)

Communication with the parent React component (`MisCircuitos.js`) is done via `postMessage`.

- **Dynamic Button Text**: On initialization, if `urlContieneCadena()` is true, the button text is set to "Guardar cambios" (Save changes). Otherwise, it defaults to "Guardar como nuevo" (Save as new).
- **'click' event on `#enviar-url`**:
  - If the button says "Guardar como nuevo", it sends `{ type: 'get url' }` to trigger the creation of a new entry in the DB.
  - If it says "Guardar cambios", it sends `{ type: 'edit circuit' }` to update the existing entry.
- **Visual Feedback**: The button reflects the persistence state immediately upon loading or URL change.

---

## 4. Debug Architecture (Breakpoints)

This section uses **Monkey Patching** to modify the original behavior of the Quirk engine (Traceur Runtime) without touching the compiled source code.

### 4.1 Breakpoint Rendering (Lines 1885-1922)

`DisplayedCircuit.prototype.paint` is overwritten:

1.  Calls the original function `originalPaint.call(...)` to draw the base circuit.
2.  Calculates the `y` position below the circuit (`circuitBottom + yOffset`).
3.  Draws a circle for each column using `painter.ctx`.
4.  If the column matches `window.debugBreakpoint`, it is filled with red.

### 4.2 Click Interception and UI Blocking (Lines 1924-1984)

`DisplayedCircuit.prototype.tryClick` is overwritten:

1.  Creates an invisible rectangle (`hitRect`) over each breakpoint circle.
2.  If the user's click (`hand.pos`) is within that rectangle:
    - Toggles the value of `window.debugBreakpoint`.
    - Synchronizes `breakpointColumn`.
    - Dispatches an artificial `mousemove` event to force a redraw.
3.  **UI Blocking**: In debug mode, standard Quirk buttons (Menu, Export, etc.) have their `click` and `contextmenu` (right click) events blocked via `stopImmediatePropagation` to prevent state corruption.

---

## 5. Result Visualization and Graphs (Lines 1568-1706)

Results are rendered dynamically every time the circuit changes or during the debug step-through.

- `pintarGrafica(sliceName, data)`:
  - Calculates the maximum value (`maxValue`) to scale the Y-axis.
  - Determines the Y-axis step (`yStep`) automatically (1, 5, or 10).
  - Creates dynamic `div` and `span` elements for the bars.
  - Uses Flexbox for alignment in the `#chart` container.
- `mostrarEtiquetasSlice()`: Prints the raw JSON in the text results container (`#contenedor`).
- `pintarRectanguloConDatosAPI()`: Orchestrates the mapping from `ConjuntoResultados` to the visual components.

---

## 6. Assertion Engine (Lines 2008-2182)

Allows the user to validate if the circuit behaves as expected in real-time.

- `compareResults()`:
  1.  Takes the user input from `#inputResultadoEsperado`.
  2.  Supports both comma-separated bit strings (e.g., `00, 11`) and raw JSON.
  3.  Applies `padStart` logic to bit strings to ensure alignment with the register width.
  4.  **Style Logic**: Sets a green border (`#28a745`) and background for success, or red (`#dc3545`) for failure.
- **Auto-Sync**: The engine hooks into `window.pintarGrafica` to re-run comparisons automatically whenever the data changes.

---

## 7. Backend Synchronization and Persistence (Lines 1758-1852)

Communicates with the Django server for simulation and storage.

- `comprobarCircuitoBD(id)`: Checks if cached results exist for the circuit ID.
  - **Crucial**: If found, it calls `leerURL()` to synchronize the `columnasURL` length, ensuring navigation and overlay limits match the loaded circuit.
- `sendDebugData()`: Persists results to `/resultsDebug/`.
  - **Full Dataset**: It sends the _entire_ `ConjuntoResultados` array even if a breakpoint is active, allowing full navigation (back/forward) once the record is re-loaded in the future.
- `sacarID` / `sacarResultado`: Asynchronous polling logic to wait for the quantum simulator worker to finish.

---

## Quick Extension Guide

| If you want to...               | Touch the function / area                                             |
| :------------------------------ | :-------------------------------------------------------------------- |
| **Change the breakpoint color** | `DisplayedCircuit.prototype.paint` (Line 1915)                        |
| **Modify API endpoints**        | `sacarID` or `sendDebugData` (Lines 1259 / 1865)                      |
| **Change the assertion format** | `compareResults` (Line 2089)                                          |
| **Add buttons to the top bar**  | `insertarBotonesDebug` listener section (Line 964)                    |
| **Modify gate logic**           | Modify `UpdateQuirk/Quirk/src/gates/` and **not** this file directly. |

---

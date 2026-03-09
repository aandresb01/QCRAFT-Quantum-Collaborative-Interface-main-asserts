# Guía de Desarrollo: quirk_util.html

Este documento es una referencia exhaustiva para cualquier desarrollador que desee mantener o ampliar el visualizador personalizado de Quirk para QCraft. Explica cada componente lógico, el flujo de datos y la integración técnica del archivo.

---

## 1. Variables Globales y Estado del Sistema

Estas variables mantienen el estado de la aplicación en memoria durante la sesión del usuario.

- `window.urlAntesDeCadena`: Almacena la URL de Quirk sin los parámetros de QCraft. Es vital para reconstruir el circuito puro.
- `window.cadenaCircuito`: El ID único (string) del circuito en la base de datos de QCraft.
- `window.debugBreakpoint`: Entero que indica el índice de la columna donde se ha puesto un breakpoint. Si es `null`, no hay breakpoint.
- `ConjuntoResultados`: Un array donde cada índice `i` contiene un string JSON con las probabilidades de la columna `i`.
- `dataGlobal`: Objeto estructurado que contiene todos los "slices" (instantáneas de estado) del circuito procesados a partir de `ConjuntoResultados`.
- `debugAlreadySent`: Bandera booleana para evitar peticiones POST duplicadas al backend.

---

## 2. Utilidades de Manejo de URL (Líneas 745-793)

Quirk codifica el circuito en el fragmento `#` de la URL. QCraft añade parámetros extra como `&cadena=`.

- `urlContieneCadena()`: Devuelve un booleano. Verifica si el circuito ya está persistido en la BD buscando `&cadena=` en la URL.
- `obtenerValorParametroCadena()`: Usa `URLSearchParams` sobre el fragmento para extraer el string del ID del circuito.
- `obtenerURLAntesDeCadena()`: Limpia la URL borrando todo lo que haya después del ID de QCraft, permitiendo que Quirk procese el JSON base.
- `obtenerURLYCadena()`: Validador principal. Si el circuito está vacío (`:[]`), lanza un `alert` impidiendo la traducción.

---

## 3. Integración con React y Persistencia (Líneas 795-822)

La comunicación con el componente React padre (`MisCircuitos.js`) se realiza mediante `postMessage`.

- **Texto Dinámico del Botón**: En la inicialización, si `urlContieneCadena()` es verdadero, el texto del botón se establece en "Guardar cambios". De lo lo contrario, por defecto es "Guardar como nuevo".
- **Evento 'click' en `#enviar-url`**:
  - Si el botón dice "Guardar como nuevo", envía `{ type: 'get url' }` para disparar la creación de una nueva entrada en la BD.
  - Si dice "Guardar cambios", envía `{ type: 'edit circuit' }` para actualizar la entrada existente.
- **Feedback Visual**: El botón refleja el estado de persistencia inmediatamente al cargar o al cambiar la URL.

---

## 4. Arquitectura de Debug (Breakpoints)

Esta sección utiliza **Monkey Patching** para modificar el comportamiento original del motor Quirk (Traceur Runtime) sin tocar el código fuente compilado.

### 4.1 Renderizado de Breakpoints (Líneas 1885-1922)

Se sobrescribe `DisplayedCircuit.prototype.paint`:

1.  Llama a la función original `originalPaint.call(...)` para dibujar el circuito base.
2.  Calcula la posición `y` debajo del circuito (`circuitBottom + yOffset`).
3.  Dibuja un círculo por cada columna usando `painter.ctx`.
4.  Si la columna coincide con `window.debugBreakpoint`, se rellena de rojo.

### 4.2 Intercepción de Clics y Bloqueo de UI (Líneas 1924-1984)

Se sobrescribe `DisplayedCircuit.prototype.tryClick`:

1.  Crea un rectángulo invisible (`hitRect`) sobre cada círculo de breakpoint.
2.  Si el clic del usuario (`hand.pos`) está dentro de ese rectángulo:
    - Alterna el valor de `window.debugBreakpoint`.
    - Sincroniza `breakpointColumn`.
    - Dispara un evento `mousemove` artificial para forzar el redibujado.
3.  **Bloqueo de UI**: En modo debug, los botones estándar de Quirk (Menu, Export, etc.) tienen sus eventos `click` y `contextmenu` (clic derecho) bloqueados mediante `stopImmediatePropagation` para evitar la corrupción del estado.

---

## 5. Visualización de Resultados y Gráficas (Líneas 1568-1706)

Los resultados se renderizan dinámicamente cada vez que el circuito cambia o durante el paso a paso del debug.

- `pintarGrafica(sliceName, data)`:
  - Calcula el valor máximo (`maxValue`) para escalar el eje Y.
  - Determina el paso del eje Y (`yStep`) automáticamente (1, 5 o 10).
  - Crea elementos `div` y `span` dinámicos para las barras.
  - Usa Flexbox para la alineación en el contenedor `#chart`.
- `mostrarEtiquetasSlice()`: Imprime el JSON crudo en el contenedor de resultados de texto (`#contenedor`).
- `pintarRectanguloConDatosAPI()`: Orquesta el mapeo de `ConjuntoResultados` a los componentes visuales.

---

## 6. Motor de Aserciones (Líneas 2008-2182)

Permite al usuario validar si el circuito se comporta como se espera en tiempo real.

- `compareResults()`:
  1.  Toma la entrada del usuario de `#inputResultadoEsperado`.
  2.  Soporta tanto cadenas de bits separadas por comas (ej. `00, 11`) como JSON crudo.
  3.  Aplica lógica de `padStart` a las cadenas de bits para asegurar la alineación con el ancho del registro.
  4.  **Lógica de Estilo**: Establece un borde verde (`#28a745`) y fondo para el éxito, o rojo (`#dc3545`) para el fallo.
- **Auto-Sincronización**: El motor se engancha en `window.pintarGrafica` para volver a ejecutar las comparaciones automáticamente cada vez que los datos cambian.

---

## 7. Sincronización con el Backend y Persistencia (Líneas 1758-1852)

Se comunica con el servidor Django para la simulación y el almacenamiento.

- `comprobarCircuitoBD(id)`: Verifica si existen resultados cacheados para el ID del circuito.
  - **Crucial**: Si se encuentran, llama a `leerURL()` para sincronizar la longitud de `columnasURL`, asegurando que los límites de navegación y overlay coincidan con el circuito cargado.
- `sendDebugData()`: Persiste los resultados en `/resultsDebug/`.
  - **DataSet Completo**: Envía el array _completo_ `ConjuntoResultados` incluso si hay un breakpoint activo, permitiendo la navegación completa (atrás/adelante) una vez que el registro se vuelva a cargar en el futuro.
- `sacarID` / `sacarResultado`: Lógica de sondeo asíncrono para esperar a que el worker del simulador cuántico termine.

---

## Guía Rápida de Extensión

| Si quieres...                       | Toca la función / área                                                 |
| :---------------------------------- | :--------------------------------------------------------------------- |
| **Cambiar el color del breakpoint** | `DisplayedCircuit.prototype.paint` (Línea 1915)                        |
| **Modificar endpoints de la API**   | `sacarID` o `sendDebugData` (Líneas 1259 / 1865)                       |
| **Cambiar el formato de aserción**  | `compareResults` (Línea 2089)                                          |
| **Añadir botones a la barra sup.**  | Sección de listeners en `insertarBotonesDebug` (Línea 964)             |
| **Modificar lógica de puertas**     | Modifica `UpdateQuirk/Quirk/src/gates/` y **no** este archivo directo. |

---

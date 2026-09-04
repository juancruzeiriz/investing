---
name: finance-monthly-report
description: Reporte financiero mensual — verifica tasas reales de Binance/Nexo, inflación y precios, y actualiza el tablero.
---

Generá el reporte financiero mensual. Español rioplatense (voseo), tono directo, sin marketing inflado.

## Contexto

Leé primero:

- `D:\workspace\investing\.env` — tenencias, aporte mensual, webhook y `ARTIFACT_URL`. **Nunca
  copies valores de este archivo a nada que se commitee: el repo es público.**
- `D:\workspace\investing\sources.md` — los endpoints exactos y sus trampas.
- `D:\workspace\investing\marcos-de-analisis.md` — lentes de lectura (riesgo, costos, liquidez,
  psicología del inversor, calidad contable, etc.) sacados de libros de finanzas/economía y
  parafraseados por Juan. **Son lentes para leer los datos, no una fuente de recomendaciones.**
  Usalos para enriquecer la sección de análisis (por ejemplo: si una tasa parece generosa,
  aplicar el eje 7 y preguntar qué parte compensa riesgo de default; si un producto promete
  rendimiento sin volatilidad visible, aplicar el eje 1). Nunca cites una oración textual de ese
  archivo de más de 15 palabras, y nunca lo uses para decir qué comprar — eso sigue prohibido
  sin excepción (ver Alcance).
- El reporte más reciente en `D:\workspace\investing\reports\` para poder hacer el diff.

## Alcance — la línea, no la cruces

**SÍ:** datos verificados de APIs y del código de las páginas (nunca del cartel), relevamiento de
instrumentos y de sus costos, alertas de cambio contra la corrida anterior, análisis e inferencias
sobre lo que significan los datos observados, comparaciones aritméticas entre productos, puntos de
equilibrio, y la tabla de escenarios completa.

**NO:** emitir órdenes de compra o de venta —ni "comprá X ahora", ni "salí de Y"—. El reporte
entrega el dato y el análisis; la decisión es del lector. La tabla de escenarios muestra **todas**
las filas.

Los retornos históricos de renta variable se citan **siempre etiquetados como históricos**, nunca
como pronóstico.

Si un dato no se pudo verificar, marcalo como hueco abierto. **Nunca lo estimes ni lo inventes.**

## Pasos

1. **Traé los datos** con el navegador. Para las llamadas `bapi/*` de Binance navegá primero a
   `https://www.binance.com/en/earn/simple-earn` o fallan por CORS. Para los máximos públicos de
   Nexo (sin login) navegá a `https://nexo.com/es-ar/earn-crypto/usdt` y extraé el payload RSC con
   el regex de `sources.md` (`fixedRate` es el **incremento**, no la tasa total). **Para la tasa
   real de Nexo por nivel** —esto ya no es un hueco— andá directo a
   `https://platform.nexo.com/savings-breakdown` con sesión iniciada, elegí el nivel de `.env`
   (`NEXO_NIVEL`) y leé las columnas Flexible/Plazo Fijo/Bonificación por activo (ver `sources.md`
   para la tabla completa por nivel y la trampa del "Rendimiento a Plazo" que rinde menos que el
   techo de Flexible). Sumá dólar (dolarapi.com), IPC del INDEC, CPI de EE.UU., precios BTC/ETH,
   **tasas en pesos del BCRA** (`v4.0`, ojo con el anidado `results[0].detalle[]`), **FCI money
   market diario** (`curl` directo a `api.pub.cafci.org.ar/pb_get`, CORS no aplica fuera del
   navegador — parsear con `openpyxl`, ver `sources.md` para el mapeo de columnas) y **comisiones
   de retiro** (`getNetworkCoinAll`).

1c. **Traé los datos de IOL/CEDEARs** — ya no es referencia estática, se re-verifica cada mes
   igual que Binance y Nexo. Dos páginas públicas, sin login (ver `sources.md` para el detalle):
   `iol.invertironline.com/mercado/cotizaciones/argentina/cedears/todos` para spreads reales
   (compra/venta) de CEDEARs puntuales, y `invertironline.com/tarifas` para la comisión vigente.
   **La comisión depende del volumen operado en el mes anterior** (Gold/Platinum/Black) — no es
   un número fijo. Recalculá el costo ida y vuelta del perfil Gold (el default,
   `(comisión + 0,05%×1,21) × 2`) en vez de reusar el de la edición anterior sin chequear.

2. **Calculá el rendimiento efectivo** sobre el monto de `.env`, aplicando el escalón:
   `efectivo = (200 × APR_cartel + (monto − 200) × marketApr) / monto`.
   Calculá los puntos de cruce contra BFUSD y RWUSD.

2b. **Compará también entre plataformas, no solo dentro de Binance.** Nexo nivel Base y Binance
   Simple Earn son ambos stablecoin flexible, pero **no son el mismo perfil de riesgo** — en Nexo
   la titularidad del activo se transfiere a Keystone Capital Fund Inc. (Panamá). Publicá la
   diferencia aritmética **y** la diferencia de contraparte en la misma fila. Nunca presentes la
   diferencia de tasa sola, porque leída sin el riesgo parece plata gratis.

3. **Armá la tabla de escenarios** con las mezclas 100/0, 75/25, 50/50, 25/75, 0/100 sobre el
   aporte de largo plazo (el % de `HORIZONTE_LARGO_PCT`, convertido a USD con el dólar cripto del
   día). Columnas: nominal, real vs inflación EE.UU., valor a 10 años, peor caída histórica
   ponderada, y valor si esa caída pega al final. Usá 10 % nominal / 6,6 % real como promedio
   histórico del S&P 500 —**siempre etiquetado como histórico, no pronóstico**— y −57 % como peor
   caída (2007-09).

3b. **Calculá el punto de equilibrio de los pesos.** Con la tasa de plazo fijo minorista (BCRA
   `idVariable` 12) pasada a TEA —`TEA = (1 + TNA/12)^12 − 1`, porque se renueva todos los meses—
   calculá cuánto puede subir el dólar en el año antes de que dolarizar convenga más:
   `equilibrio = TEA_pesos / (1 + rendimiento_USD) − 1`.

   **Publicá el punto de equilibrio, nunca un pronóstico de devaluación.** La fila dice "el plazo
   fijo en pesos gana solo si el dólar sube menos que X %". Cuán probable es ese X lo decide quien
   lee. Sumá también la TEA contra el IPC argentino, que es la otra mitad del cuadro.

4. **Costeá cada movimiento que propongas.** Ninguna optimización se publica sin su costo al lado
   y su período de repago. Distinguí siempre: mover **entre productos de Binance** es interno y no
   paga red; **salir de la plataforma** paga comisión de retiro, y esa comisión varía 150× según la
   red (0,01 por BSC contra 1,50 por TRC20 en USDT). Decí siempre por qué red está calculado.

5. **Cuantificá los impuestos que efectivamente aplican.** Compará la tenencia total contra el
   mínimo no imponible de Bienes Personales del período vigente, convertido a USD al CCL del día,
   y decí a qué porcentaje del umbral está. Separá siempre los dos hechos imponibles de Ganancias:
   la **venta** de criptoactivos (5 % fuente argentina / 15 % extranjera) y el **rendimiento** que
   paga la plataforma (segunda categoría). Re-verificá el mínimo no imponible cada año: se
   actualiza solo por IPC. Mantené el descargo de que la estructura fiscal amerita contador.

6. **Vigilá cambios de reglas, no de precios.** Guardá en `history.json` una huella de las páginas
   de reglas que ya visitás —Anexo III de los términos de Nexo, página de campañas, escalones de
   Binance— y compará contra la corrida anterior. Si la huella cambió, leé qué cambió y escribilo
   en Alertas. **Esto no predice nada: detecta que cambió la letra chica.** Es lo que hoy las
   alertas no ven, porque solo comparan números que ya se scrapean.

7. **Escribí el markdown** en `D:\workspace\investing\reports\YYYY-MM.md` con las secciones:
   Qué cambió · Tu rendimiento real · Tabla de escenarios · Pesos vs. dólares · Binance · Nexo ·
   Inflación y tipo de cambio · BTC/ETH · IOL/CEDEARs · Impuestos · Alertas. Cerrá con huecos
   abiertos y descargo.

8. **Actualizá `reports\history.json`.** Es un array, **un objeto por corrida** — agregá una
   entrada nueva al final, nunca reescribas ni borres las anteriores. La clave es `corrida`
   (fecha ISO), no el mes: si hay dos corridas en el mismo mes, son **dos entradas**, no una que
   pisa a la otra. Campos mínimos que alimentan los gráficos del tablero: `corrida`,
   `binance.rendimiento_real`, `nexo.flexible_usdt_pct`, `dolar.ccl_venta`, `pesos.plazo_fijo_tea`.
   Este archivo tampoco se commitea — vive en `reports/`, que está gitignoreado entero.

9. **Actualizá el tablero.** Editá el HTML en `reports\index.html`:
   - Los números del mes (stats, tablas, callouts) van con los datos frescos.
   - La sección **"Serie histórica"** tiene que graficar **todos** los puntos de
     `reports\history.json` actualizado en el paso anterior, no solo la última corrida — copiá el
     array completo (o su versión resumida a los campos de arriba) dentro del `const HISTORY`
     del `<script>` al final del archivo. Con una sola corrida el gráfico muestra un punto; a
     partir de la segunda ya traza la línea.
   - Republicá con la herramienta Artifact pasando `url` = el valor de `ARTIFACT_URL` del `.env`,
     para que mantenga la **misma** URL. Sin ese `url` se crea un artefacto nuevo. Mantené el
     favicon 🔍 y el título "Tasa Real".

10. **Mandá el resumen a Discord** usando `DISCORD_WEBHOOK_URL` del `.env`. Si no está, **no
   falles**: escribí el reporte igual y avisá que falta. Si está, mandá un embed **fácil de leer
   sin abrir el tablero**, en español rioplatense llano (nada de jerga sin explicar):
   - Título: `Rendimientos — {Mes} {Año}`.
   - Primera línea: una frase de una oración diciendo qué es esto (p. ej. "Verificación mensual
     de tasas reales vs. lo que muestran las apps — no es una recomendación.").
   - 3 a 5 campos cortos con el número y qué significa en criollo, no solo el porcentaje pelado
     — ejemplo: `Rendimiento real: −0,05% (empatando con la inflación de EE.UU.)`, no
     `Rendimiento real: -0.05%`.
   - Si hubo alertas ese mes, un campo aparte listándolas; si no hubo, no incluyas el campo.
   - Último campo, siempre: `🔍 Ver el tablero completo → {ARTIFACT_URL}` — con esas palabras
     exactas, para que quede claro que ahí está el detalle, los gráficos y la serie histórica.

11. **Commiteá** en `D:\workspace\investing` solo lo que corresponde. `reports/` y `.env` están
   gitignoreados y **tienen que seguir estándolo** — verificá con `git status` antes de commitear
   que no aparezca ningún monto.

## Reglas de calidad

- **"Qué cambió" queda vacío si nada se movió.** No inventes novedad para llenar espacio.
- Alerta solo si el cambio es material: tasa que cae más de 0,5 puntos, cambio de términos o de
  contraparte, campaña que aparece o vence, comisión que se mueve, o punto de cruce que se corre
  por encima/debajo del monto en cartera.
- Dejá escrito el control aritmético: recompensa diaria × 365 ÷ monto debe reproducir el
  rendimiento efectivo observado.
- Re-verificá que la inflación relevante es la de EE.UU. y no la argentina, porque la plata está
  dolarizada. Es el dato que más distorsiona la percepción de urgencia.
- Si Nexo cambió el Anexo III o el proveedor Earn dejó de ser Keystone Capital Fund Inc. (Panamá),
  eso es alerta de máxima prioridad.
- Si la tarifa de IOL para el perfil Gold cambió, o el umbral de volumen para pasar a Platinum se
  movió, es alerta — afecta el costo real de cualquier operación en CEDEARs.
- **Todo rendimiento se acompaña de su riesgo de contraparte en la misma línea**, no en un párrafo
  aparte (`marcos-de-analisis.md`, eje 7).
- **Toda tasa lleva fecha y hora de verificación, y la fuente concreta** — nunca un número sin
  saber de cuándo es (`marcos-de-analisis.md`, reglas de redacción).
- **Todo producto lleva declarado su plazo de rescate y su penalidad de salida junto a la tasa**
  — la iliquidez es un riesgo aparte del riesgo de precio, y no se ve en el número
  (`marcos-de-analisis.md`, eje 11).
- Cerrá recordando que asignación y estructura fiscal ameritan asesor matriculado (AAGI de CNV) y contador.

Al terminar, resumen de cinco líneas: rendimiento real, qué cambió, alertas, huecos abiertos y el link al tablero.

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
- El reporte más reciente en `D:\workspace\investing\reports\` para poder hacer el diff.

## Alcance — la línea, no la cruces

**SÍ:** datos verificados de APIs y del código de las páginas (nunca del cartel), alertas de
cambio contra el mes anterior, inferencias sobre lo que significan los datos observados,
comparaciones aritméticas entre productos del mismo perfil de riesgo, tabla de escenarios completa.

**NO, nunca, aunque parezca que el reporte lo pide:**

- Recomendar un porcentaje de asignación. La tabla muestra **todas** las filas; la elección es del lector.
- Elegir acciones o CEDEARs concretos. SPIVA: 84,3 % de gestores profesionales pierden contra el índice a 10 años.
- Predecir precios a partir de noticias, política o eventos.
- Recomendar momentos de compra o venta.

Si un dato no se pudo verificar, marcalo como hueco abierto. **Nunca lo estimes ni lo inventes.**

## Pasos

1. **Traé los datos** con el navegador. Para las llamadas `bapi/*` de Binance navegá primero a
   `https://www.binance.com/en/earn/simple-earn` o fallan por CORS. Para Nexo navegá a
   `https://nexo.com/es-ar/earn-crypto/usdt` y extraé el payload RSC con el regex de `sources.md`
   (`fixedRate` es el **incremento**, no la tasa total). Sumá dólar (dolarapi.com), IPC del INDEC,
   CPI de EE.UU. y precios BTC/ETH.

2. **Calculá el rendimiento efectivo** sobre el monto de `.env`, aplicando el escalón:
   `efectivo = (200 × APR_cartel + (monto − 200) × marketApr) / monto`.
   Calculá los puntos de cruce contra BFUSD y RWUSD.

3. **Armá la tabla de escenarios** con las mezclas 100/0, 75/25, 50/50, 25/75, 0/100 sobre el
   aporte de largo plazo (el % de `HORIZONTE_LARGO_PCT`, convertido a USD con el dólar cripto del
   día). Columnas: nominal, real vs inflación EE.UU., valor a 10 años, peor caída histórica
   ponderada, y valor si esa caída pega al final. Usá 10 % nominal / 6,6 % real como promedio
   histórico del S&P 500 —**siempre etiquetado como histórico, no pronóstico**— y −57 % como peor
   caída (2007-09).

4. **Escribí el markdown** en `D:\workspace\investing\reports\YYYY-MM.md` con nueve secciones:
   Qué cambió · Tu rendimiento real · Tabla de escenarios · Binance · Nexo · Inflación y tipo de
   cambio · BTC/ETH · IOL/CEDEARs · Alertas. Cerrá con huecos abiertos y descargo.

5. **Actualizá el tablero.** Editá el HTML en `reports\index.html` y republicá con la herramienta
   Artifact pasando `url` = el valor de `ARTIFACT_URL` del `.env`, para que mantenga la **misma**
   URL. Sin ese `url` se crea un artefacto nuevo. Mantené el favicon 🔍 y el título "Tasa Real".

6. **Mandá el resumen a Discord** usando `DISCORD_WEBHOOK_URL` del `.env`. Si no está, **no
   falles**: escribí el reporte igual y avisá que falta. Si está, mandá un embed con rendimiento
   efectivo, rendimiento real, alertas del mes y el link al tablero.

7. **Commiteá** en `D:\workspace\investing` solo lo que corresponde. `reports/` y `.env` están
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
- Cerrá recordando que asignación y estructura fiscal ameritan asesor matriculado (AAGI de CNV) y contador.

Al terminar, resumen de cinco líneas: rendimiento real, qué cambió, alertas, huecos abiertos y el link al tablero.

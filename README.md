# investing

Verificador mensual de rendimientos. Lee las tasas reales desde las APIs y desde el código de
cada página —**nunca del cartel**— y las recalcula sobre el monto que uno realmente tiene.

## El problema

Las tasas que muestran las plataformas casi nunca son las que se cobran. Tres casos verificados
en agosto de 2026:

| Lo que muestra la pantalla | Lo que se cobra |
|---|---|
| Binance, USDC flexible: **6,91 %** | **1,91 %** — el bono corre solo sobre los primeros 200 USDC |
| Nexo, USDT: **"hasta 11,5 %"** | **5,50 %** en nivel Base, con la contraparte en Panamá |
| "La inflación se come todo" | Si la plata está dolarizada, la que aplica es la de EE.UU.: **3,4 %** |

Ninguno de los tres es visible en la interfaz del producto. Verificarlos a mano lleva horas;
automatizarlo una vez por mes es barato.

## Qué hace

- Trae tasas de las APIs públicas de Binance y del payload RSC de Nexo
- Recalcula el **rendimiento efectivo real** aplicando el escalón de 200 unidades
- Calcula los **puntos de cruce** entre productos según el monto en cartera
- Compara contra la inflación que efectivamente aplica
- Modela una **tabla de escenarios** con todo el abanico de mezclas: rendimiento, rendimiento
  real, peor caída histórica, y valor si esa caída pega al final del horizonte
- Detecta cambios materiales mes contra mes y avisa por Discord

## Qué no hace

- **No recomienda un porcentaje de asignación.** La tabla muestra todas las filas; la elección
  es de quien lee. La columna "peor caída" descarta filas sola.
- **No elige acciones ni CEDEARs.** SPIVA: 84,3 % de los gestores profesionales pierden contra
  el índice a 10 años, 89,5 % a 15. La selección individual es lo que la evidencia desaconseja.
- **No predice precios a partir de noticias.** No es una limitación técnica: nadie lo hace bien
  de forma consistente, y fingirlo genera confianza falsa.
- Los retornos históricos de renta variable que usa **no son pronósticos**, y el reporte lo dice
  cada vez.

## Lo más útil de este repo

[`sources.md`](sources.md) — los endpoints, con sus trampas documentadas. Encontrarlos la primera
vez llevó una sesión entera:

- Las llamadas `bapi/*` de Binance tienen que salir desde el origen `binance.com` o fallan por CORS
- `latestAnnualInterestRate = marketApr + tier[0]`, y el bono muere exactamente en `endAmount`
- Las tasas de Nexo **no están en el DOM**: los contadores `<number-flow-react>` quedan vacíos si
  la pestaña no compone. Los valores viven en el payload RSC dentro de los `<script>`
- En Nexo, `fixedRate` es el **incremento** del plazo fijo, no la tasa total. Confundirlos hace
  ver un plazo fijo que paga menos que la cuenta flexible
- El Help Center de Nexo está geobloqueado desde IP argentina

## Setup

```bash
cp .env.example .env
```

Completar `.env` con las tenencias y el webhook. **`.env` está en `.gitignore` y no se commitea
nunca** — este repo es público.

Los reportes se generan en `reports/`, que también está ignorado: contienen la posición real.

## Ejecución

El análisis corre como tarea programada de Claude Code, el día 16 de cada mes (el INDEC publica
el IPC entre el 12 y el 15, así el dato del mes anterior ya está disponible).

El prompt de la tarea está en [`task/SKILL.md`](task/SKILL.md) y es autocontenido: cada corrida
arranca sin memoria de la conversación que la creó.

## Estructura

| Ruta | Qué es | Commiteado |
|---|---|---|
| `sources.md` | Endpoints verificados y sus trampas | Sí |
| `task/SKILL.md` | Prompt de la tarea mensual | Sí |
| `.env.example` | Plantilla de configuración | Sí |
| `.env` | Tenencias, montos y webhook | **No** |
| `reports/` | Reportes generados con la posición real | **No** |

## Descargo

Esta herramienta verifica y compara datos publicados. **No constituye asesoramiento de
inversión.** Decisiones de asignación y estructura fiscal ameritan un asesor matriculado y un
contador.

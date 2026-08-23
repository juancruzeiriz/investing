# Fuentes de datos — verificadas 2026-08-22

Todas públicas, sin login. Documentadas acá para que ninguna corrida futura tenga que
redescubrirlas — encontrarlas la primera vez llevó una sesión entera.

---

## Binance

Las llamadas `bapi/*` **tienen que salir desde el origen `binance.com`** (navegar ahí primero;
desde otro origen fallan por CORS). Header requerido: `clienttype: web`.

### Tasas por activo

```
GET https://www.binance.com/bapi/earn/v1/friendly/lending/daily/product/list
    ?pageSize=20&pageIndex=1&asset={USDT|USDC|FDUSD}
```

Campos que importan en `data[0]`:

| Campo | Qué es |
|---|---|
| `latestAnnualInterestRate` | El APR de cartel. **Solo aplica al primer tramo.** |
| `marketApr` | La tasa real sobre el excedente. Es la que se cobra de verdad. |
| `tierAnnualInterestRateList` | El bono y hasta qué monto llega (`endAmount`). |

**La relación clave:** `latestAnnualInterestRate = marketApr + tier[0].annualInterestRate`.
El bono es aditivo y muere en `endAmount`. Comprobado en USDT, USDC, BTC, ETH, BNB.

### Tabla de escalones (todos los activos de una)

```
GET https://www.binance.com/bapi/earn/v1/friendly/lending/daily/product/tierApy
```

### Todos los productos de un activo (BFUSD, RWUSD, locked, on-chain)

```
GET https://www.binance.com/bapi/earn/v1/friendly/finance-earn/simple-earn/homepage/details
    ?includeBFUSD=true&includeRWUSD=true&includeEthStaking=true&includeSolStaking=true
    &includeP2pLoan=true&includeP2pLoanSupply=true&requestSource=WEB&pageIndex=1&pageSize=50
```

`data.list[].productDetailList[]` → `productType` (`LENDING_FLEXIBLE`, `BFUSD`, `RWUSD`,
`POS_FIXED`, `FIXED_LOAN_SUPPLY`), `apy`, `marketApr`, `apyTierOption`.

### Precios y comisiones

```
GET https://api.binance.com/api/v3/ticker/24hr?symbols=["BTCUSDT","ETHUSDT"]   # sin header
GET https://www.binance.com/bapi/capital/v1/public/capital/getNetworkCoinAll   # comisiones de retiro
```

`getNetworkCoinAll` devuelve **947 monedas**; buscar `data[].coin === 'USDT'` y recorrer
`networkList[]` filtrando por `withdrawEnable`. Campos: `network`, `name`, `withdrawFee`,
`withdrawMin`.

> **La comisión de retiro varía 150× según la red.** Verificado 2026-08-23 en USDT: BSC (BEP20)
> cobra **0,01**, Polygon 0,07, Arbitrum 0,10, Ethereum (ERC20) 0,30, y **Tron (TRC20) 1,50** —
> que es justo la red que la mayoría elige por costumbre. Cualquier cálculo de "conviene mover
> los fondos" tiene que decir **por qué red**, o el número no significa nada.

**Mover entre productos dentro de Binance no paga red.** Simple Earn ↔ BFUSD ↔ RWUSD son
transferencias internas: comisión cero. La comisión de retiro solo aplica al salir de la
plataforma (p. ej. hacia Nexo).

---

## Nexo

**Las tasas no están en el DOM renderizado.** Los contadores usan `<number-flow-react>` y quedan
vacíos si la pestaña no está compositando. Los valores reales viven en el payload RSC dentro de
los `<script>`.

```
GET https://nexo.com/es-ar/earn-crypto/usdt
```

```js
const s = [...document.querySelectorAll('script')].map(x => x.textContent).join('\n');
const re = /flexRate\\":([0-9.]+),\\"fixedRate\\":([0-9.]+)[^]{0,900}?slugUpper\\":\\"([A-Z0-9]+)\\"/g;
// m[1] = tasa flexible máxima, m[2] = INCREMENTO por plazo fijo, m[3] = ticker
```

> **`fixedRate` es el incremento, no la tasa total.** USDT: `0.095 + 0.02 = 0.115`, que es
> exactamente el "hasta 11,5%" que publica la página. Confundirlos hace ver un plazo fijo que
> paga menos que la cuenta flexible, lo cual no tiene sentido económico.

Todo lo que publica Nexo son **máximos** (nivel Platinum). La tasa del nivel Base solo se ve
dentro de la app logueado — el reporte marca ese hueco, no lo estima.

### Términos y campañas

- `https://nexo.com/es-ar/terms` — Anexo III art. I.3 nombra los proveedores Earn reales.
  Buscar `Proveedores Earn serán los siguientes` y `Riesgos del Producto Earn Interest`.
- `https://nexo.com/es-ar/usdt-yield-boost-campaign` — campañas activas. El art. 4.4 de la
  campaña de agosto 2026 fue la única fuente pública de una tasa que no era "hasta".

**El Help Center está geobloqueado desde IP argentina.** `support.nexo.com` devuelve el shadow
root `c-sc-article-content` vacío y un `c-sc-geofence`. No perder tiempo ahí.

---

## Macro

| Dato | Fuente |
|---|---|
| Dólar (todas las cotizaciones) | `GET https://dolarapi.com/v1/dolares` — sin auth, devuelve oficial, blue, MEP, CCL, cripto, tarjeta |
| IPC Argentina | `indec.gob.ar` — publica entre el 12 y el 15 del mes siguiente |
| CPI Estados Unidos | `bls.gov/news.release/PDF/cpi.PDF` o TradingEconomics |
| Verificar entidades reguladas | `cnv.gov.ar/SitioWeb/RegistrosPublicos` |

### Tasas en pesos — API del BCRA

```
GET https://api.bcra.gob.ar/estadisticas/v4.0/monetarias?limit=1000   # catálogo, 1.610 variables
GET https://api.bcra.gob.ar/estadisticas/v4.0/monetarias/{id}?limit=1 # último valor
```

> **`v3.0` devuelve `410 Gone`.** Hay que usar **`v4.0`**. Sin auth, sin headers especiales.

> **La respuesta anida distinto de lo que parece.** No es `results[]` con los valores: es
> `results[0].detalle[]` → `{fecha, valor}`. Leer `results[]` directo devuelve objetos vacíos sin
> tirar error, que es la peor forma de fallar.

| `idVariable` | Qué es | Ojo con |
|---|---|---|
| **12** | Tasa de depósitos a 30 días — **el plazo fijo minorista**, TNA | Es la que aplica a un ahorrista común |
| 7 | BADLAR bancos privados, TNA | **Mayorista: depósitos de más de 1 millón de ARS.** No es la tasa que le pagan a una persona con 300 mil |
| 160 / 161 | Tasa de política monetaria, TNA / TEA | **Congelada desde 2025-07-10** — el BCRA dejó de publicarla. Devuelve un valor viejo sin avisar. No usarla como dato corriente |

**TNA no es lo que se cobra.** El plazo fijo se renueva mensualmente, así que la comparación
honesta es la TEA: `TEA = (1 + TNA/12)^12 − 1`. Con TNA 21,08% la TEA es 23,24% — más de dos
puntos de diferencia que se pierden si se compara la TNA contra un rendimiento anual.

**FCI money market y LECAPs: hueco abierto.** No se encontró fuente pública sin login que
publique el rendimiento de FCI de liquidez inmediata ni el de las letras. Se marca como hueco,
no se estima.

---

## Impuestos (ARCA, ex AFIP)

| Dato | Valor verificado | Fecha |
|---|---|---|
| Bienes Personales — mínimo no imponible, período fiscal 2025 | **ARS 384.728.044,57** | 2026-08-23 |
| Bienes Personales — alícuotas | 0,50 % a 1,00 %, progresivas | 2026-08-23 |
| Ganancias — venta de criptoactivos | **5 %** fuente argentina · **15 %** fuente extranjera (art. 98 LIG) | 2026-08-23 |
| Ganancias — rendimientos de depósitos cripto | Renta de **segunda categoría** | 2026-08-23 |
| CEDEARs y ADRs en Bienes Personales | **No computables** (dictamen ARCA 03/2024) | 2026-08-23 |
| Umbral de reporte de PSAVs a ARCA | ARS 50 millones mensuales, personas humanas (RG 4614/2019, act. RG 5804/2025) | 2026-08-23 |

> **El mínimo no imponible se actualiza solo por IPC** desde la reforma de la Ley 27.743 — ya no
> hace falta una ley cada año. Hay que re-verificarlo igual: cambia todos los años.

> **La tenencia sola de cripto no genera hecho imponible en Ganancias.** Lo que tributa es la
> venta, y por separado el rendimiento que paga la plataforma. Son dos cosas distintas y se
> liquidan distinto.

---

## Referencias fijas (no cambian mes a mes)

| Dato | Valor |
|---|---|
| Máximo histórico BTC | USD 126.198,07 — 6 de octubre de 2025 |
| Máximo histórico ETH | USD 4.946 — 25 de agosto de 2025 (superó el de nov-2021 tras 3 años y 9 meses) |
| Caídas históricas BTC | −93% (2011), −86% (2015), −84% (2018), −77% (2022). Recuperación ≈3 años en 3 de 4 |
| Caídas históricas S&P 500 | −49% (2000-02), −57% (2007-09), −34% (2020), −25% (2022) |
| S&P 500 plano | Del pico de 2000 hasta ≈2013 en términos reales |
| SPIVA | 84,3% de gestores activos pierden contra el índice a 10 años; 89,5% a 15 |
| IOL | INVERTIRONLINE S.A.U., ALyC Integral **matrícula 273** (no 203, que circula mal en la web) |
| Retorno histórico S&P 500 | ≈10% nominal, ≈6,6% real anual en el largo plazo. **Histórico, no pronóstico.** |

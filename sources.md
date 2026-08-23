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

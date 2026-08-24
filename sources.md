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

### App logueada: lo que sí y no se pudo sacar (verificado 2026-08-24)

Con sesión iniciada en `platform.nexo.com` (vía extensión de Chrome, sin credenciales manuales):

- **Los badges "Ganá hasta X%" que muestra la app logueada son el mismo máximo de Platinum**
  que ve un visitante sin cuenta — no cambian por tener sesión iniciada ni por el nivel real.
- **Nivel confirmado por API**, no por UI: `POST /api/platform/loyalty/v1/tiers` (sin body)
  devuelve `tiers[].name` con los 4 niveles (`base`, `silver`, `gold`, `platinum`); no trae
  `min_usd_balance` — el umbral real no vive ahí, se calcula en otro lado.
- **La confirmación de tasa por nivel no está en ningún endpoint público del propio front.**
  Se probaron `loyalty/v1/rates`, `loyalty/v1/config`, `earn/v1/rates`, `savings/v1/rates`,
  `fincore/v1/interest-rates` — todos devuelven la SPA (HTML) en vez de datos, es decir, no
  son rutas reales de API.
- **Hallazgo real: el "hasta 13% en USDC / 7,5% en ETH"** que aparece en el modal de fidelización
  (`Mejora tu experiencia en Nexo`) **no es la tasa estándar de Platinum** — el propio modal
  aclara "Habilitá 'Ganar en NEXO' para potenciar aún más la generación de intereses": ese % más
  alto exige aceptar el interés pagado **en token NEXO**, no en la moneda depositada. Es un
  mecanismo distinto (exposición al token de la plataforma), no comparable línea a línea con el
  11,50%/10,50% ya verificado en la página pública.
- **Trampa de automatización:** los `<span>` de texto en React a veces no responden a un click
  por coordenadas de pantalla ni por `ref.click()` de accesibilidad — hace falta
  `document.querySelectorAll` + `.click()` sobre el nodo de texto exacto para disparar el
  handler.
- **Trampa de API:** `fetch()` a rutas `/api/1/*` con **GET** devuelve `access denied` (HTML);
  las mismas rutas con **POST** y body `{}` sí funcionan (`get_balances` confirmado así).

**No se pudo cerrar:** la tasa real de Base para USDT/USDC, ni la de Platinum sin el boost de
NEXO. La cuenta usada para probar tiene saldo 0, lo que bloquea llegar a la pantalla real de
creación de plazo fijo (cualquier click en la fila del activo abre "Recibir", no "Ganar").

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

### FCI money market — CAFCI

```
GET https://estadisticas.cafci.org.ar/v2/fondos-mercado-de-dinero.json?fecha=YYYY-MM-DD
```

Sin auth. Devuelve `clases[]` con `rendimientos.{dia,dias_7,dias_30,dias_90,dias_180,meses_12,ytd}`,
cada uno con `directo` y `tna`.

> **El parámetro `fecha` es cosmético.** Probado con fechas del 20 al 23 de agosto de 2026: todas
> devolvieron `fecha_base: "2026-07-31"`. Es el ranking RG1121, que se actualiza **una vez por
> mes**, no a diario. `fechas_disponibles[]` lo confirma — solo hay un valor por mes. **Nunca
> presentar esto como "tasa de hoy".** Usar `fecha_base` del propio JSON para fechar el dato, no
> la fecha que se pidió.

**Hay una versión diaria y no se pudo usar.** `https://api.pub.cafci.org.ar/pb_get` devuelve una
planilla XLSX con variación de cuotaparte día a día, genuinamente fresca — pero está en un
subdominio distinto (`api.pub` vs `estadisticas`) sin CORS habilitado entre ellos, y la navegación
directa dispara una descarga en vez de una respuesta legible. Requeriría parsear XLSX fuera del
navegador. **Queda como mejora futura, no como hueco cerrado ni estimado.**

### LECAPs — precio en vivo + TEM publicada

```
GET https://data912.com/live/arg_notes
```

Sin auth, CORS abierto, límite 120 req/min. Devuelve precio bid/ask en vivo por ticker
(`symbol`, `px_bid`, `px_ask`, `c`) para más de 20 letras, LECAPs incluidas — filtrar por el
patrón de ticker `S{día}{mes en letra}{año}` (ej. `S15S6` = vence 15-sep-2026). **Da precio, no
tasa** — la TIR hay que calcularla contra el valor de rescate, que no viene en la respuesta.

```
GET https://www.rava.com/perfil/{TICKER}
```

Página pública, sin login. **La TEM (Tasa Efectiva Mensual) viene en el `<title>`**, no en el
cuerpo — ej. `S15S6 LECAP $ TEM 1,99% Vto. 15.09.2026 $105,63`. Suficiente para leer el dato sin
parsear la tabla completa, que carga por JS. No hay endpoint que liste todos los tickers de una:
hay que conocer el ticker de antemano (sacarlo de `data912.com`) y consultar rava uno por uno.

> **La curva de TEM no sale monótona ni prolija.** Verificado 2026-08-23: S15S6 (23 días) 1,99%,
> S30S6 (38 días) 2,53%, S13N6 (82 días) 2,10%. No es un error de lectura — es así como cotiza
> cada letra según su propia demanda. No "corregir" el dato para que parezca una curva suave.

**Riesgo que no tiene el plazo fijo bancario:** la TEM solo se realiza si se mantiene hasta el
vencimiento. Vender antes expone a riesgo de precio de mercado, además del riesgo soberano
(es deuda del Tesoro). El plazo fijo bancario no tiene riesgo de precio si se mantiene el plazo.

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

# Marcos de análisis para leer un reporte de tasas y escenarios

> Lentes de lectura, no recomendaciones. Este documento no dice qué comprar: dice qué preguntarle
> a un número antes de creerle. Cada punto está trazado al autor de donde sale la idea, redactado
> como síntesis propia.
>
> Uso previsto: alimentar el instructivo de un reporte automatizado que verifica tasas reales
> (cripto, plazo fijo/money market, renta fija local, acciones y CEDEARs) y arma tablas de
> escenarios para que el lector decida.
>
> **Versión 3.** Cambios respecto de la v1: se agregó el eje 3 bis (concentración del retorno,
> Bessembinder) y los ejes 11 (liquidez) y 12 (apalancamiento); se corrigieron y ampliaron los
> números de McLean-Pontiff. Respecto de la v2: el eje 9 se amplía con Schilit, ahora sí verificado
> contra el texto. Ver nota de trazabilidad al final.

---

## 1. Riesgo y volatilidad — más allá de "sube o baja"

**El riesgo se declara antes de mirar el retorno, y se declara en plata, no en porcentaje.**
Carver plantea que la pregunta "¿cuánto riesgo querés tomar?" no tiene respuesta hasta definir
horizonte y unidad: no es lo mismo lo máximo que podés perder mañana, en un año, o en el peor de
cada veinte días. La forma operativa es fijar un objetivo de volatilidad expresado en dinero.
*Lente: ante un rendimiento, preguntarse cuánto es la pérdida típica de un año malo en la misma
unidad — y si esa cifra es tolerable.*
(Carver, *Systematic Trading*)

**La volatilidad anualizada no es el peor caso, es la dispersión promedio.** Carver es explícito
en que es bastante probable perder más que eso en algún año. *Lente: un desvío estándar no es un
techo de pérdida.*
(Carver)

**El drawdown máximo no escala linealmente con la exposición.** Chan muestra un caso donde
reducir el apalancamiento a la mitad casi no mueve el drawdown; hizo falta dividirlo por siete para
partirlo al medio. *Lente: "invierto la mitad y arriesgo la mitad" no se cumple.*
(Chan, *Algorithmic Trading*)

**La asimetría importa tanto como la magnitud.** Carver la señala como la característica más
ignorada de una estrategia: dos inversiones con la misma volatilidad se comportan de forma opuesta
si una acumula ganancias chicas con pérdidas raras y enormes, y la otra al revés. Los productos que
pagan una tasa fija y constante suelen tener el primer perfil. *Lente: si un rendimiento se ve suave
y sin sobresaltos, preguntarse dónde está guardada la pérdida.*
(Carver)

**La volatilidad es un mal punto de partida, pero no tan malo.** Ilmanen argumenta que el marco de
distribución normal, con sus defectos conocidos, sirve sorprendentemente bien como base, siempre
que se lo complemente con drawdown máximo, comportamiento en eventos de cola y estimaciones de
liquidez. *Lente: no descartar un número de volatilidad por imperfecto; sí exigir que venga
acompañado.*
(Ilmanen, *Investing Amid Low Expected Returns*)

**Los retornos artificialmente suaves esconden riesgo real.** Ilmanen: en activos sin marcación a
mercado los retornos alisados subestiman el riesgo y "halagan" su aparente capacidad de
diversificar. *Lente: aplicable a cualquier producto cuyo precio lo publica el propio emisor — un
rendimiento sin volatilidad observable no es un rendimiento sin riesgo.*
(Ilmanen)

**El riesgo reduce el crecimiento compuesto, siempre.** Chan lo demuestra con un caso límite: un
activo con 50% de probabilidad de subir 1% y 50% de bajar 1% cada minuto no queda plano en el largo
plazo — pierde, porque la media geométrica es la aritmética menos la mitad de la varianza.
*Lente: comparar dos productos por su rendimiento promedio sin mirar su volatilidad sobreestima
sistemáticamente al más volátil.*
(Chan, *Quantitative Trading*)

---

## 2. Inflación real vs. nominal — errores al comparar rendimientos

**Un rendimiento histórico no es un rendimiento esperado.** Es la idea central de Ilmanen: los
retornos de las últimas décadas incluyen ganancias extraordinarias por revalorización (los precios
subieron porque las tasas de descuento bajaron), y esa fuente ya se consumió. El rendimiento
esperado se estima desde los *rendimientos iniciales de hoy*, no desde el promedio del pasado.
*Lente: "esto rindió X% histórico" es dato del pasado; el escenario futuro se construye desde la
tasa vigente.*
(Ilmanen)

**El promedio histórico está sesgado si la muestra empieza o termina en valuaciones extremas.**
Ilmanen propone corregirlo regresando retornos pasados contra el cambio de valuación
contemporáneo. *Lente: preguntar siempre desde qué fecha arranca la serie y qué pasaba con los
precios en ese punto.*
(Ilmanen)

**Descomponer el retorno revela de dónde saldría.** Grinold y Kahn lo tratan como identidad
contable, no como supuesto: retorno total = ingreso corriente + crecimiento nominal +
revalorización. Quien afirma un retorno esperado está afirmando implícitamente valores para las
tres partes. *Lente: desarmar toda proyección — si los tres componentes no cierran, es un deseo.*
(Grinold & Kahn, *Advances in Active Portfolio Management*)

**El rendimiento real es la única comparación válida entre monedas.** Ilmanen documenta que la
tasa real de corto plazo puede ser negativa por períodos largos y que eso es una realidad
persistente, no una anomalía. *Lente: ningún par de tasas en monedas distintas se compara sin
traerlas a real; y el índice de inflación usado debe ser el del lugar donde se va a gastar.*
(Ilmanen)

**La inflación no se traslada completa a los ingresos indexados.** En su análisis de real estate
Ilmanen encuentra lo que llama "traslado insuficiente de inflación": el crecimiento nominal de los
alquileres quedó por debajo de la inflación general, dando crecimiento real negativo sostenido
durante décadas. *Lente: cuando un instrumento promete ajuste por inflación, preguntar por qué
índice, con qué rezago, y si históricamente ese ajuste igualó a la inflación o quedó atrás.*
(Ilmanen)

**El proceso inflacionario cambia de régimen; el promedio de una era no describe la siguiente.**
Ilmanen recorre cómo la inflación pasó de paseo aleatorio a persistente a anclada cerca del 2%, y
menciona a la Argentina entre los casos de hiperinflación de posguerra. *Lente: una serie larga
promedia regímenes incompatibles; para proyectar, usar el régimen vigente y declararlo.*
(Ilmanen)

---

## 3. Diversificación — qué la hace efectiva o inefectiva

**Lo que importa no es cuántas posiciones hay, sino cuántas apuestas independientes.** Carver lo
formaliza con el multiplicador de diversificación: con correlación promedio 0, veinte activos
equivalen a 4,5 apuestas independientes; con correlación 0,5, apenas 1,38 — casi lo mismo que una
sola. Cincuenta activos correlacionados al 0,75 dan 1,19. *Lente: ante una lista larga de
posiciones, calcular la correlación promedio antes de llamarla diversificada.*
(Carver)

**Dentro de una misma clase de activo la diversificación tiene techo.** Ilmanen: entre acciones
las correlaciones de a pares suelen superar 0,5 por el riesgo sistemático común; entre bonos son aún
mayores. Las oportunidades buenas aparecen *entre* clases de activo. *Lente: una cartera de muchas
acciones de distintos sectores sigue siendo una sola apuesta grande al mercado accionario.*
(Ilmanen)

**Las correlaciones suben justo cuando se las necesita bajas.** Carver advierte que en una crisis
como 2008 saltan hacia arriba, y por eso recomienda topear el multiplicador. *Lente: todo escenario
de estrés debe calcularse asumiendo correlaciones más altas que las del período tranquilo.*
(Carver)

**La diversificación reduce la varianza del resultado, no eleva el retorno esperado.** Grinold y
Kahn lo ilustran con el casino: la misma ventaja porcentual repartida en 100.000 apuestas chicas en
vez de una grande no cambia la ganancia esperada, pero convierte una probabilidad de pérdida del
47% en una virtual certeza de ganancia. *Lente: si un producto promete "más retorno por
diversificar", desconfiar; lo que da es más previsibilidad.*
(Grinold & Kahn)

**El marco angosto destruye la lectura de cartera.** Ilmanen lo lista entre los sesgos centrales:
mirar la ganancia de cada posición por separado en vez de su efecto sobre el patrimonio total. Lo
vincula directamente con la subdiversificación. *Lente: un reporte que lista rendimientos posición
por posición fomenta este sesgo; la pregunta correcta es qué le hace cada posición al total.*
(Ilmanen)

**Diversificar también los criterios, no solo los activos.** Ilmanen enumera prácticas concretas:
usar varias señales por factor en vez de una, escalonar los rebalanceos para evitar la "suerte de
timing del rebalanceo", igualar volatilidades en vez de montos nominales. *Lente: dos productos con
el mismo monto invertido no aportan el mismo riesgo; la comparación correcta es a volatilidad
equiparada.*
(Ilmanen)

---

## 3 bis. Concentración del retorno — por qué la mayoría de los activos individuales pierde

Este eje merece lugar propio: es el hallazgo empírico que más justifica indexar en vez de elegir.

**La mayoría de las acciones individuales rinde menos que una letra del Tesoro.** Bessembinder
analiza toda la base CRSP entre 1926 y 2016 y encuentra que solo el 42,6% de las acciones tuvo un
retorno de vida completa (buy-and-hold, con dividendos reinvertidos) superior al de la letra del
Tesoro a un mes en el mismo período. Menos de la mitad tuvo siquiera retorno positivo. La mediana
del retorno de vida completa fue **–2,29%**. *Lente: el retorno promedio de un mercado no describe
al activo típico de ese mercado; describe a un puñado de excepciones.*
(Bessembinder, *Do Stocks Outperform Treasury Bills?*, Journal of Financial Economics, 2018)

**Toda la creación neta de riqueza vino del 4% de las empresas.** Bessembinder: las 1.092 empresas
de mejor desempeño —poco más del 4% del total— explican la totalidad de la ganancia neta del
mercado accionario estadounidense desde 1926. El resto, en conjunto, empató con las letras del
Tesoro. *Lente: si el retorno del mercado se concentra en tan pocos nombres, cualquier cartera que
no los contenga se queda afuera del retorno del mercado, aunque tenga decenas de posiciones.*
(Bessembinder)

**Elegir una acción al azar perdió contra la letra del Tesoro en el 73% de los casos.**
Bessembinder repite el ejercicio de selección aleatoria de una sola acción muchas veces sobre el
período 1926–2016. La causa es la asimetría positiva extrema de los retornos de largo plazo, que
proviene tanto de la asimetría mensual como del efecto de la capitalización compuesta. *Lente: la
media aritmética exagera el desempeño real de quien compra y mantiene; ante una cartera de
posiciones individuales, la pregunta es si es una apuesta diversificada al mercado o una colección
de tickets con mediana negativa.*
(Bessembinder)

**Esto explica por qué las estrategias activas poco diversificadas suelen perder contra el
índice.** Es la conclusión que el propio autor extrae de sus resultados. *Lente: la comparación de
una cartera individual contra su índice no es una cuestión de habilidad; es primero una cuestión
aritmética de cuántas de las pocas ganadoras contiene.*
(Bessembinder)

---

## 4. Horizonte temporal — cómo cambia el análisis según plazo

**Cada horizonte tiene sus propios predictores.** Ilmanen es específico: a 5–10 años mandan los
rendimientos iniciales y las valuaciones; entre un mes y un año pesan más el momentum, el ciclo
macro y las noticias, cuyo efecto se diluye o revierte en una década; a horizontes multi-década
hasta el efecto valor se diluye. *Lente: antes de aceptar una proyección, preguntar a qué plazo
aplica el predictor que la genera.*
(Ilmanen)

**La reversión a la media plurianual es lo contrario del momentum de meses.** Ilmanen muestra que
los mercados tienden a continuar hasta un año pero a revertir en horizontes plurianuales — y que es
justo en esos horizontes donde se toman las decisiones de reasignación. *Lente: "viene subiendo hace
tres años" es un argumento de signo opuesto a "viene subiendo hace seis meses".*
(Ilmanen)

**Los drawdowns largos son mucho más largos de lo que sugiere el caso estadounidense.** Ilmanen
señala que en el siglo XX los drawdowns reales se extendieron 55 años en Alemania, 51 en Japón, 22
en el Reino Unido y en el índice mundial. *Lente: cuando un escenario asuma "en el largo plazo se
recupera", preguntar de qué país y qué siglo sale ese largo plazo.*
(Ilmanen)

**El horizonte declarado y el real no coinciden bajo estrés.** Ilmanen desarma el argumento de que
al inversor paciente solo debería importarle la pérdida permanente: todos tienen un punto de
quiebre, y los horizontes se acortan cuando llegan los momentos difíciles. *Lente: un escenario a
diez años solo es válido si el flujo de caja aguanta diez años sin tocar el capital — eso se
verifica, no se supone.*
(Ilmanen)

**La volatilidad escala con la raíz del tiempo, salvo autocorrelación.** Ilmanen da la regla: 1%
mensual equivale a ~3,46% anual (√12). *Lente: llevar todo a la misma unidad temporal antes de
comparar.*
(Ilmanen)

---

## 5. Costos y fricciones — por qué importan más de lo que parece

**El costo anual no es la comisión: es la comisión por la rotación.** Ilmanen da la fórmula: una
estrategia con 200% de rotación anual y 20 puntos básicos por operación paga 40 pb al año. La
consecuencia es contraintuitiva — una inversión que rota dos veces por década puede tener costos
anuales más altos que una estrategia de momentum líquida. *Lente: calcular el costo anualizado, no
el de la operación aislada.*
(Ilmanen)

**El costo se come primero lo que más promete.** Jegadeesh y Titman, en el paper fundacional de
momentum, reportan rotación semestral del 84,8% y calculan explícitamente el retorno después de un
costo de 0,5% por lado. *Lente: toda diferencia de rendimiento entre productos se compara después
de costos de rotación, no antes.*
(Jegadeesh & Titman, *Journal of Finance*, 1993)

**Minimizar costos no es el objetivo; maximizar el retorno neto sí.** Ilmanen insiste en la
distinción y muestra que operar demasiado y operar demasiado poco son ambos subóptimos. Advierte
además que restar ingenuamente el costo estimado del retorno bruto subestima la utilidad de una
estrategia. *Lente: la pregunta no es "cuál cobra menos" sino "cuál queda mejor después de todo".*
(Ilmanen)

**Los costos incluyen componentes invisibles en el resumen de cuenta.** Chan desglosa: comisión,
spread bid-ask, impacto de mercado, costo de oportunidad de las órdenes límite no ejecutadas, y
slippage por demora, que en promedio juega en contra. *Lente: el costo declarado por el broker es un
piso, no el total.*
(Chan, *Quantitative Trading*)

**Existe un límite de velocidad por instrumento.** Carver propone estandarizar el costo por
volatilidad para hacerlo comparable entre instrumentos, y fijar un tope de cuánto Sharpe se está
dispuesto a pagar en costos por año. *Lente: un instrumento caro de operar impone un horizonte
mínimo de tenencia; si la idea requiere rotar más rápido, el instrumento no sirve para la idea.*
(Carver)

**El costo escala peor de lo esperado cuando crece el monto.** Grinold y Kahn modelan cómo el
proceso de gestión responde reduciendo riesgo activo, rotación y velocidad de ejecución. *Lente: los
rendimientos publicados corresponden a un tamaño que puede no ser el propio.*
(Grinold & Kahn)

---

## 6. Psicología del inversor — sesgos que distorsionan la lectura de un reporte

**Perseguir rendimientos plurianuales es, según Ilmanen, el sesgo principal.** Comprar lo que ganó
los últimos años y vender lo que perdió, justo en el horizonte donde los mercados tienden a
revertir. Cita evidencia de que los flujos están sistemáticamente mal calibrados y que el retorno
ponderado por dinero queda por debajo del ponderado por tiempo. *Lente: el producto que más rindió
en los últimos tres años es el dato más peligroso de la página, no el más útil.*
(Ilmanen)

**El exceso de confianza es el sesgo que alimenta a los demás.** Ilmanen lo señala como la
explicación principal del sobre-trading, sostenido por autoatribución y confirmación. Carver agrega
la ilusión de superioridad y observa que operar seguido puede volverse conductualmente
indistinguible del juego problemático. *Lente: si después de leer el reporte dan ganas de operar,
esa gana es el sesgo, no la conclusión.*
(Ilmanen; Carver)

**La falacia narrativa hace ver patrones donde no los hay.** Carver: nuestro cerebro ve más
predictibilidad en los precios pasados de la que realmente existía. *Lente: una explicación
convincente de por qué algo rindió bien no es evidencia de que vaya a seguir rindiendo bien.*
(Carver)

**La preferencia por la lotería explica por qué lo más llamativo rinde peor.** Ilmanen conecta la
sobreponderación de eventos de baja probabilidad con la preferencia por inversiones de sesgo
positivo y con su mal desempeño de largo plazo. *Lente: si un producto es atractivo porque "podría"
multiplicarse, ese atractivo es el sesgo operando.*
(Ilmanen)

**La impaciencia rompe el plan antes que el mercado.** Ilmanen la vincula al ahorro insuficiente, y
observa que la gente ajusta gastos y aspiraciones al alza apenas sube el ingreso. *Lente: la mejor
estrategia es la que se puede sostener; una que exige aguantar más de lo que el lector aguanta no es
mejor, es peor.*
(Ilmanen)

**Los expertos con herramientas caen en el mismo pozo.** Carver observa que los optimizadores
asumen conocer los retornos con precisión, que pequeños cambios en los supuestos producen carteras
extremas, y que muchos profesionales terminan retocando el resultado hasta que coincida con lo que
esperaban ver. *Lente: un output cuantitativo no es más confiable que los supuestos que se le
metieron.*
(Carver)

---

## 7. Riesgo de contraparte y estructura — qué preguntar antes de confiar en una tasa publicada

**Un spread de tasa es, en parte, compensación por pérdidas esperadas.** Ilmanen desarma el spread
de crédito: parte cubre pérdidas por default esperadas y parte es prima genuina. La "captura de
spread" histórica rondó el 56% en grado de inversión y el 54% en alto rendimiento. *Lente: ante una
tasa por encima de la libre de riesgo, preguntar qué parte de ese exceso paga por la probabilidad de
no cobrar.*
(Ilmanen)

**Buscar rendimiento sin mirar la estructura lleva a los peores lugares posibles.** Ilmanen lo
enuncia crudamente: el yield ingenuo empuja hacia un mercado con hiperinflación, hacia la deuda de
una empresa al borde del default, o hacia un bono estructurado con opciones cortas embebidas y
costos tales que el retorno esperado realista es negativo. *Lente: la tasa más alta de la tabla es
la que más explicación requiere, no la que menos.*
(Ilmanen)

**El rendimiento sobrestima el retorno esperado en activos riesgosos y lo subestima en otros.**
Ilmanen: para bonos riesgosos el yield exagera (en high yield, históricamente por cerca de la
mitad); para acciones el dividend yield subestima porque ignora el crecimiento. *Lente: "tasa
publicada" y "retorno esperado" son cantidades distintas, y la brecha cambia de signo según el
instrumento.*
(Ilmanen)

**Los spreads se comprimen en la calma y se abren de golpe en la crisis.** Ilmanen documenta la
ciclicidad: angostos en tiempos tranquilos, ensanchándose bruscamente en recesiones, con frecuencia
frenados por intervención de bancos centrales. *Lente: una tasa que hoy parece generosa relativa a
su riesgo puede estar reflejando complacencia general.*
(Ilmanen)

**El riesgo de crédito de corto plazo se comporta distinto bajo estrés.** Ilmanen advierte que los
créditos de plazo corto son los más vulnerables a un salto en el riesgo de default real, porque pasan
a operar por precio y no por tasa. *Lente: "es a corto plazo, así que es seguro" no se sostiene
cuando el riesgo es de contraparte y no de tasa.*
(Ilmanen)

---

## 8. Cómo leer una evidencia empírica o un backtest

**La mayoría de los hallazgos publicados no se replican.** Hou, Xue y Zhang reproducen 447
anomalías con un procedimiento común y encuentran que entre el 64% y el 85% resultan
estadísticamente insignificantes; identifican como causa principal la sobreponderación de empresas
microcap, que son el 61% de la cantidad de acciones pero apenas el 3,3% de la capitalización.
*Lente: un resultado publicado no es un resultado verificado; preguntar con qué universo y qué
ponderación se calculó.*
(Hou, Xue & Zhang, NBER w23394)

**Cuantas más configuraciones se prueban, más alto tiene que ser el resultado para significar
algo.** López de Prado lo formula como ley: todo backtest debe reportarse junto con la cantidad de
pruebas hechas para producirlo; sin ese dato es imposible evaluar la probabilidad de falso
descubrimiento. Con solo diez configuraciones probadas, el mejor resultado esperado en muestra ronda
un Sharpe de 1,57 aunque el verdadero sea cero. *Lente: ante un rendimiento óptimo, preguntar
cuántas alternativas se descartaron para llegar a él.*
(López de Prado, *Advances in Financial Machine Learning*; Bailey, Borwein, López de Prado & Zhu,
*Pseudo-Mathematics and Financial Charlatanism*)

**Hace falta muchísima historia para distinguir habilidad de suerte.** Carver calcula el tiempo
promedio para pasar un test de significancia: con Sharpe verdadero de 0,5, más de veinte años; con
0,3 —realista para una regla sobre un instrumento— cerca de cuarenta. Chan da la contraparte: para
tener confianza al 95% de que el Sharpe verdadero supera 1, hacen falta unos 2.739 datos diarios,
casi once años. *Lente: un track record de dos o tres años no distingue entre un producto bueno y
uno afortunado.*
(Carver; Chan, *Quantitative Trading*)

**Las anomalías se degradan después de publicarse, y ahora con números.** McLean y Pontiff estudian
97 variables predictoras: el retorno promedio de la estrategia cae **26% fuera de muestra** y **58%
después de la publicación**. La diferencia entre ambas caídas —unos 32 puntos— la atribuyen a que
los inversores operan informados por la publicación; el 26% restante es un límite superior del
efecto de minería de datos. La degradación es mayor en predictores con mejor desempeño en muestra y
en activos de baja liquidez. *Lente: si una oportunidad es conocida y está siendo promocionada, el
rendimiento que se le atribuye es probablemente el de antes de que se conociera.*
(McLean & Pontiff, *Journal of Finance*)

---

## 9. Calidad de lo que genera el rendimiento

Aplica cuando el reporte compare instrumentos cuyo rendimiento depende de una empresa o un emisor,
no de una tasa de mercado.

**Un múltiplo barato sobre ganancias infladas no es barato.** Penman ordena el análisis: primero la
calidad contable, después la valuación. Su señal central es la divergencia entre resultado devengado
y flujo de caja operativo — cuando la ganancia crece mientras el flujo de caja cae, el componente de
accruals está creciendo, y ahí es donde vive la manipulación. *Lente: comparar ganancia contra flujo
de caja antes de mirar cualquier ratio.*
(Penman, *Financial Statement Analysis and Security Valuation*)

**Las señales de alerta son observables y enumerables.** Penman lista, entre otras: cuentas por
cobrar que crecen mientras las ventas caen; acumulación de inventario sobre ventas en baja;
depreciación que no acompaña el crecimiento de las inversiones de capital; ingresos diferidos que
bajan (revenue del pasado entrando al resultado presente); ganancias por venta de activos
presentadas junto a las operativas. *Lente: cada una es una pregunta concreta que se le puede hacer
a un balance publicado.*
(Penman)

**La rotación de activos que cae indica gastos diferidos.** Penman: si los activos operativos netos
crecen más rápido que las ventas, puede ser porque se están registrando menos gastos de los que
corresponden, dejando costo en el balance en vez de en el resultado. *Lente: una mejora de margen
acompañada de caída en rotación es sospechosa, no una buena noticia.*
(Penman)

**El precio se puede ingeniar al revés para ver qué crecimiento está asumiendo el mercado.**
Penman muestra el procedimiento: partiendo del precio, del valor libro y de la rentabilidad sobre el
patrimonio, se despeja la tasa de crecimiento implícita de las ganancias residuales, y esa tasa se
contrasta contra un referente razonable (por ejemplo, el crecimiento del PBI). *Lente: en vez de
preguntar "¿está caro?", preguntar "¿qué crecimiento hay que creer para que este precio tenga
sentido, y es creíble?".*
(Penman)

**Toda manipulación contable persigue una de dos estrategias, y la segunda es contraintuitiva.**
Schilit las reduce a dos: inflar el resultado del período actual (subiendo ingresos o bajando
gastos), o *deflactarlo* a propósito, bajando ingresos actuales o inflando gastos actuales. La
segunda parece absurda hasta entender el objetivo: correr ganancias hacia un período futuro donde
harán falta. *Lente: un resultado sorpresivamente malo, sobre todo con cargos extraordinarios
grandes, puede ser preparación para resultados futuros artificialmente buenos.*
(Schilit, *Financial Shenanigans*, 2ª ed.)

**Las técnicas son finitas y están catalogadas.** Schilit organiza treinta técnicas en siete
categorías: registrar ingresos demasiado pronto o de calidad dudosa; registrar ingresos ficticios;
inflar el resultado con ganancias por única vez; correr gastos del período actual a otro período;
omitir o reducir indebidamente pasivos; correr ingresos actuales a un período posterior; y adelantar
gastos futuros al período actual como cargo especial. *Lente: ante una anomalía en un balance, la
pregunta operativa es a cuál de estas siete categorías se parece — es una lista cerrada, no un
universo infinito.*
(Schilit)

**Hay tres condiciones que hacen probable la manipulación, y ninguna es contable.** Schilit las
enuncia como: conviene hacerlo, es fácil hacerlo, y es improbable que te descubran. Las traduce en
señales tempranas observables: entorno de control débil (falta de directores independientes o de un
auditor externo competente e independiente), gerencia bajo presión competitiva extrema, y gerencia
de carácter cuestionable. *Lente: el riesgo de calidad contable se evalúa mirando incentivos y
gobernanza, no solo estados financieros.*
(Schilit)

**Cuatro perfiles de empresa concentran el riesgo.** Schilit señala: empresas de crecimiento
acelerado cuyo crecimiento real empieza a desacelerar; empresas en dificultades que luchan por
sobrevivir; empresas recién salidas a bolsa; y empresas privadas, sobre todo las de capital cerrado
sin auditoría. Sobre las primeras es explícito: todo crecimiento rápido termina desacelerando, y es
en ese punto donde aparece la tentación. *Lente: la pregunta no es solo qué dice el balance, sino
en cuál de estas cuatro situaciones está la empresa.*
(Schilit)

**La información importante está en el material que rodea a los estados financieros.** Schilit
recomienda empezar la búsqueda no por el balance ni por el estado de resultados, sino por el informe
del auditor (¿hay opinión con salvedades, en particular sobre empresa en marcha?), el acta de
asamblea (litigios, remuneración de ejecutivos, operaciones con partes relacionadas), las notas
(políticas contables y sus cambios, contingencias, compromisos) y las comunicaciones de hechos
relevantes (desacuerdos sobre políticas contables). *Lente: si un análisis solo mira los números
principales, está mirando donde la información fue puesta para ser vista.*
(Schilit)

**Las políticas contables se pueden clasificar en un eje conservador-agresivo, ítem por ítem.**
Schilit da la tabla: reconocer ingresos después de la venta cuando el riesgo pasó al comprador es
conservador, hacerlo en la venta con riesgo remanente es agresivo; depreciación acelerada en plazo
corto vs. lineal en plazo largo; estimaciones altas de garantías e incobrables vs. bajas; gastar
publicidad vs. capitalizarla; devengar una contingencia de pérdida vs. solo mencionarla en nota.
*Lente: la agresividad contable no es un juicio global; se arma sumando decisiones individuales que
están declaradas en las notas.*
(Schilit)

---

## 10. Moneda y calce

**El activo se elige contra el pasivo, no en el vacío.** Ilmanen señala que para un inversor
cubierto por moneda es la tasa doméstica la que ancla los retornos. La generalización útil: la
comparación relevante no es entre dos rendimientos, sino entre el rendimiento y la moneda en la que
se van a pagar los compromisos futuros. *Lente: antes de comparar un rendimiento en dólares con uno
en pesos, definir en qué moneda están los gastos y las deudas que ese dinero va a cubrir.*
(Ilmanen)

**Un pasivo indexado por inflación no se cubre con un activo dolarizado, salvo que el traslado sea
completo y simultáneo.** Se desprende de combinar dos observaciones de Ilmanen: el traslado
insuficiente de inflación en activos supuestamente indexados, y la variación de régimen
inflacionario. Los períodos de apreciación real —inflación corriendo mientras el tipo de cambio se
queda— son exactamente donde ese calce falla. *Lente: al comparar un rendimiento en moneda dura
contra una obligación indexada por precios locales, incluir un escenario de apreciación real, que es
el caso donde la comparación se da vuelta.*
(Ilmanen, aplicado — ver nota de trazabilidad)

---

## 11. Liquidez — un riesgo distinto, no una variante del riesgo de precio

**La iliquidez tiene tres caras y ninguna se ve en la tasa.** Ilmanen las separa: costo alto de
operar, imposibilidad de operar montos grandes, y períodos de bloqueo durante los cuales solo se
puede salir pagando una penalidad. *Lente: ante un producto con rendimiento atractivo, preguntar
por separado cuánto cuesta salir, cuánto se puede sacar de una vez, y en cuánto tiempo.*
(Ilmanen)

**La prima por iliquidez que se espera cobrar suele ser mayor que la que se cobra.** Ilmanen
sostiene que la evidencia empírica de primas por iliquidez es sorprendentemente escasa, y da su
mejor explicación: la ausencia de marcación a mercado hace que el riesgo medido parezca
artificialmente bajo, y los inversores terminan pagando por esa comodidad aceptando un premio menor.
Menciona incluso la posibilidad de que el signo se invierta y se termine cobrando un *descuento* por
iliquidez. *Lente: "está bloqueado pero paga más" no garantiza que pague lo suficiente por estar
bloqueado.*
(Ilmanen)

**Hay un orden de liquidez razonablemente estable entre activos.** Ilmanen lo enumera: efectivo,
futuros, bonos, acciones, fondos de cobertura, activos privados; y dentro de cada grupo, gobierno
antes que crédito, gran capitalización antes que pequeña, mercados desarrollados antes que
emergentes. *Lente: ubicar cada producto del reporte en ese orden antes de comparar sus tasas.*
(Ilmanen)

---

## 12. Apalancamiento — por qué el óptimo teórico es un techo, no una meta

**La fórmula de Kelly da el apalancamiento que maximiza el crecimiento compuesto, y equivocarse
para arriba es catastrófico.** Chan explica la asimetría: subestimar el apalancamiento óptimo
cuesta apenas un crecimiento menor al máximo; sobrestimarlo —por sobrestimar la media o subestimar
la varianza— lleva eventualmente a la ruina, con el capital en cero. Por eso muchos operadores usan
la mitad de lo que la fórmula recomienda. *Lente: ante cualquier producto que ofrezca rendimiento
amplificado, preguntar qué pasa si el retorno esperado que asume resulta ser más bajo de lo
estimado.*
(Chan, *Quantitative Trading* y *Algorithmic Trading*)

**El apalancamiento óptimo se usa como cota superior, no como recomendación.** Chan cuenta que en
su experiencia el Kelly del backtest muchas veces supera lo que el bróker permite, y que en otros
casos habría fundido la cuenta incluso dentro del propio backtest, por la no normalidad de los
retornos. Su ejemplo concreto: calculó un Kelly cercano a 1,8 para los índices Russell 1000 y 2000,
mientras existen ETFs apalancados 3x sobre esos mismos índices — con riesgo real de que su valor
llegue a cero. *Lente: un producto apalancado por encima del óptimo teórico del activo subyacente
no es una versión más agresiva de la misma apuesta; es una apuesta con destino distinto.*
(Chan, *Algorithmic Trading*)

**La gestión de riesgo obliga a vender en la pérdida.** Chan lo señala como consecuencia
incómoda pero necesaria: mantener el apalancamiento objetivo implica reducir posición cuando el
capital cae, realizando la pérdida. *Lente: cualquier esquema de riesgo que se declare por
adelantado tiene que especificar qué se hace en la caída, no solo en la suba.*
(Chan)

---

## Reglas de redacción para el reporte que use estos marcos

1. Todo rendimiento se acompaña de su riesgo de contraparte en la misma línea, no en un párrafo
   aparte.
2. Todo histórico se etiqueta como histórico; toda proyección declara el supuesto y su origen.
3. Toda comparación entre productos se hace después de costos y de impuestos, y a volatilidad
   equiparada cuando corresponda.
4. Toda tasa lleva fecha y hora de verificación, y la fuente concreta.
5. Todo producto lleva declarado su plazo de rescate y su penalidad de salida junto a la tasa.
6. Ningún dato que no se pudo verificar se estima: se marca como hueco abierto con el nombre de la
   fuente que habría que consultar.
7. El reporte entrega datos, escenarios y las preguntas pertinentes. La decisión es de quien lee.

---

## Nota de trazabilidad

Honestidad sobre de dónde sale cada cosa, porque el documento es público:

- **Todo está parafraseado.** No hay citas textuales de los libros. Los números específicos
  (porcentajes, cantidades de empresas, años) sí son datos reportados por los autores y se pueden
  verificar en las fuentes.
- **Historia de la atribución a Schilit.** La v1 le atribuía un catálogo de señales de alerta sin
  haber verificado el texto; el archivo disponible entonces estaba vacío, así que en la v2 se
  eliminó la mención. Con el libro ya disponible (2ª edición, 2002), el eje 9 se amplió en la v3 con
  material leído directamente: las dos estrategias subyacentes, las siete categorías, las tres
  condiciones que hacen probable la manipulación, los cuatro perfiles de empresa y la tabla de
  políticas conservadoras vs. agresivas.
- **El libro de Schilit es de 2002.** El marco conceptual (incentivos, categorías de manipulación,
  dónde mirar) envejece bien, pero las referencias regulatorias concretas —boletines de la SEC,
  plazos de presentación, tratamiento del goodwill a 40 años, LIFO/FIFO— corresponden a la
  normativa estadounidense de ese momento y no se pueden aplicar sin verificar contra las reglas
  vigentes, menos aún fuera de Estados Unidos.
- **El punto sobre pasivos indexados vs. activos dolarizados (eje 10) es una conclusión propia**
  construida combinando dos observaciones separadas de Ilmanen. Él no formula esa conexión. Está
  marcado como "aplicado" para que no se lea como cita.
- **Las referencias a Penman provienen de las soluciones de los ejercicios del capítulo 17 y de los
  capítulos 5 y 13**, no del desarrollo teórico principal. El contenido es del autor, pero la
  fuente es de menor jerarquía que el cuerpo del libro.
- **Ilmanen está sobrerrepresentado** respecto de los demás autores: es el libro que más
  directamente aborda las preguntas de este documento. No es un juicio sobre la importancia relativa
  de las fuentes.
- **Quedan fuera deliberadamente**: la maquinaria de gestión activa de Grinold & Kahn (transfer
  coefficient, ley fundamental) y casi todo López de Prado (machine learning aplicado a trading),
  por no aplicar a un reporte de comparación de instrumentos. Si el reporte evoluciona hacia
  comparar estrategias en vez de productos, conviene volver a esas fuentes.

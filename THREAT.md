# Lab 17 — Threat model de Pactrail

**Flujo elegido: crear y financiar un contrato de escrow.** En Pactrail, el usuario conecta una billetera EVM, indica beneficiario, verificador, importe, vencimiento y requisitos, y pulsa “Crear en blockchain”. La aplicación solicita a la billetera firmar `createEscrow`; después, el pagador aprueba el importe del token y firma `fund`. El venture lee la configuración del contrato y su estado desde la red pública. No se asumen endpoints ni datos que no estén en el repositorio.

## HTTPS, Tor e I2P

HTTPS cifra el tráfico entre el cliente y el sitio. Tor oculta la IP de origen frente al sitio mediante guard, middle y exit, pero el exit todavía puede observar destino y metadatos de conexión. Si el sitio usa HTTPS, el exit no puede leer el payload protegido por TLS. I2P es una red separada; aquí se describe solo conceptualmente y no es necesario instalarla ni configurarla.

| Elemento | Solo HTTPS | Tor sobre clearnet | I2P dentro de I2P |
|---|---|---|---|
| IP del usuario | El sitio y sus proveedores de red pueden asociarla con la conexión del usuario. | El sitio ve la IP del exit; la red local/guard pueden ver la conexión de origen. | El destino I2P ve un punto de salida de I2P, no la IP directa; no aplica al sitio clearnet observado. |
| Destino | El proveedor y el sitio pueden inferir que se contacta al servicio; el sitio recibe la conexión directa. | El exit conecta con el sitio clearnet; el sitio ve al exit. | El destino debe ser un servicio I2P; no es una ruta equivalente para `github.com` o un servicio clearnet. |
| Payload | HTTPS protege formulario, respuestas y solicitudes frente a intermediarios. | HTTPS sigue protegiendo el payload entre navegador y sitio; Tor encapsula el transporte. | I2P aporta transporte dentro de su red, pero la protección del payload depende también de HTTPS/autenticación del servicio. |
| Exit o relay malicioso | No existe exit de Tor; un intermediario de red puede ver metadatos, no el payload TLS. | Un exit puede ver destino, tiempos y volumen; no puede leer el payload HTTPS protegido. Un relay no debe conocer por sí solo todo el flujo. | Un relay malicioso puede observar metadatos de su tramo; la ruta no elimina correlación ni riesgos del servicio final. |
| Login | La cuenta, firma y datos entregados llegan al venture; las cookies siguen identificando la sesión. | Igual: Tor no elimina la identidad que el usuario entrega ni la cuenta asociada a la billetera. | Igual, si el servicio requiere cuenta; I2P no reemplaza controles de identidad del venture. |
| Costo para el usuario | Coste y latencia habituales de HTTPS y de las transacciones de blockchain. | Mayor latencia y posible fricción de acceso; además siguen existiendo gas y firmas de la billetera. | Mayor complejidad/latencia y necesidad de un servicio I2P; no se requiere para este lab. |
| Qué sigue viendo el venture | Cuenta, dirección de billetera, parámetros del escrow, estado, requisitos y actividad necesaria para operar. | Lo mismo a nivel de aplicación; normalmente no ve la IP real de origen, pero puede ver la IP del exit y metadatos del cliente. | Lo mismo a nivel de aplicación; la red de transporte no cambia la información que el venture necesita recibir. |

## Circuito real observado

La observación se hizo con Tor Browser oficial, configuración predeterminada, visitando `https://github.com`.

| Componente | Observación real |
|---|---|
| Navegador | Tor Browser oficial |
| Sitio | `github.com` |
| Guard inicial | Alemania |
| Middle inicial | Alemania |
| Exit inicial | Alemania |
| Guard posterior | Estados Unidos |
| Middle posterior | Francia |
| Exit posterior | Países Bajos |
| Cambio observado | Cambiaron los tres hops después de “New Tor Circuit for this Site”. La interfaz advertía que el guard podía no cambiar. |
| IPs completas | No documentadas por privacidad |

## Quién ve qué

| Componente | Puede observar | No puede observar directamente |
|---|---|---|
| Red local | Que el usuario se conecta a Tor y metadatos básicos. | El sitio final y el payload HTTPS. |
| Guard | La conexión del usuario y su IP de origen. | El contenido final protegido por HTTPS. |
| Middle | Que participa en el circuito. | La identidad completa del usuario y el payload final. |
| Exit | El destino, tiempos y volumen del tráfico. | El payload protegido por HTTPS. |
| Sitio | Una conexión proveniente del exit. | La IP real del usuario de Tor. |

“Si el exit fuera malicioso, con este sitio HTTPS podría ver el destino, los tiempos y otros metadatos de conexión, pero no podría leer el payload protegido por HTTPS.”

En el navegador normal, `github.com` recibe directamente una conexión desde la red del usuario; la captura mostró HTTPS válido y certificado válido, pero no un circuito de Tor. En Tor Browser, el sitio observa la conexión desde el exit. En ambos casos HTTPS protege el payload frente a intermediarios: Tor agrega protección de ubicación/IP frente al sitio, pero no garantiza anonimato absoluto.

## Adversarios y límites

El ISP y la red local pueden observar la conexión de acceso a Tor y sus metadatos. El sitio de destino puede observar la IP del exit, el comportamiento de la cuenta y la actividad del flujo. Un operador de guard puede observar el origen; un operador de exit puede observar destino y metadatos, aunque no el payload HTTPS. Un gobierno con visibilidad parcial puede observar algunos tramos o extremos. La correlación end-to-end de tiempos y volumen sigue siendo un límite importante de Tor, especialmente frente a un adversario con visibilidad suficiente.

Tor no protege, en este flujo, al menos estas tres cosas: (1) fingerprinting del navegador y señales de sesión; (2) cookies, cuentas y el vínculo entre una billetera y la actividad si el usuario inicia sesión o entrega la dirección; y (3) malware o compromiso del dispositivo, ni la información que el usuario debe entregar para crear y financiar el escrow. Tampoco oculta los datos que el venture necesita para ejecutar sus reglas.

## Liability y Persona Obligada

Aunque el usuario use Tor, el venture debe seguir viendo la identidad de la cuenta o billetera, beneficiario, verificador, importe, vencimiento, requisitos, estado del escrow, firmas y el historial de actividad necesarios para operar y auditar el contrato. Tor puede ocultar la IP de origen, pero no elimina la identidad de la cuenta, el correo si se solicita, los datos de una orden, la información de soporte ni la información necesaria para cumplir obligaciones legales o de Persona Obligada. Esta evaluación se documenta teniendo presente el marco PSAV / 15-2026.

## Evidencia y aprendizaje

- Captura del circuito inicial: [`tor1.png`](./tor1.png), hops Alemania → Alemania → Alemania, con opción “New Tor circuit for this site”.
- Captura del circuito posterior: [`tor2.png`](./tor2.png), Estados Unidos → Francia → Países Bajos.
- Captura de contraste HTTPS: [`normal.png`](./normal.png), navegador normal en `github.com`, con conexión HTTPS y certificado válido.

Las capturas están disponibles dentro de `Lab17/` y ya tienen las IPs cubiertas. Deben conservarse redactadas al subirlas al carnet. El aprendizaje principal es que el payload es el contenido protegido —por ejemplo, solicitudes del flujo y respuestas— mientras que la metadata incluye destino, tiempos, volumen y participantes de red. El venture continúa viendo la información de aplicación que necesita, aunque deje de ver la IP de origen directa.

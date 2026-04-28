# vault-privategit-design

[ 🇬🇧 Read this document in English ](./README.md)

Entrada de catálogo para la forma de despliegue del sustrato del
sistema de diseño conforme a la afirmación doctrinal #38. Cada
instancia atiende a un único inquilino desde un vault gestionado en
Git que contiene tokens de diseño en formato DTCG, recetas de
componentes HTML+CSS+ARIA, temas por marca, investigación de
decisiones de diseño legible por IA, y exportaciones derivadas.

La instancia canónica de PointSav se ejecuta en `design.pointsav.com`.
Cada cliente PYME que bifurca el sustrato ejecuta su propia
instancia en su propio dominio. Un solo código, una sola forma de
despliegue, dos contextos (vitrina del proveedor, instancia del
cliente).

## Contenido del catálogo

| Archivo | Propósito |
|---|---|
| `MANIFEST.md` | Definición a nivel de catálogo (instancias, infraestructura, layout del vault) |
| `GUIDE-deploy-design-substrate.md` | Runbook operativo para el lanzamiento y procedimiento de bifurcación |
| `README.md` / `README.es.md` | Este par de READMEs |

## Documentación operativa completa en `README.md` (inglés)

Per `~/Foundry/DOCTRINE.md` §XII, los archivos en español son
resúmenes estratégicos. La documentación operativa completa está en
`README.md` y `GUIDE-deploy-design-substrate.md` (inglés).

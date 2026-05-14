# PointSav Digital Systems: Despliegue de Marketing y Medios

[ 🇬🇧 Read this document in English ](./README.md)

Este directorio contiene los archivos de despliegue operativo para la interfaz digital principal de PointSav. Sirve como implementación de referencia genérica de la interfaz pública.

## Guías operativas

- [➔ guide-Telemetry-Operations.md](./guide-Telemetry-Operations.md): Operaciones de telemetría soberana y cumplimiento.

## Arquitectura

- `index.html`: La interfaz de aplicación monolítica, sin cookies y habilitada para PWA.
- `app-mediakit-telemetry/`: El dominio de ejecución para los binarios Rust compilados `telemetry-daemon` y `omni-matrix-engine`.

*Nota: Las modificaciones a la interfaz UI/UX deben adherirse a los estándares de Brutalismo Institucional.*

---
*© 2026 PointSav Digital Systems™.*

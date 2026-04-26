---
schema: foundry-doc-v1
document_version: 0.1.0
deployment_name: media-knowledge-documentation
catalog_tier: pointsav-fleet-deployment
canonical_language: en
language: es
last_updated: 2026-04-26
---

# media-knowledge-documentation

Entrada de catálogo para la forma de despliegue del wiki de conocimiento
de PointSav. Cada instancia de este nombre ejecuta un binario
`app-mediakit-knowledge` que sirve un árbol de contenido en Markdown
como una superficie de lectura y edición con la forma de Wikipedia.

El binario se enlaza al interfaz de loopback por defecto
(`127.0.0.1:9090`). La exposición pública requiere la configuración
de un proxy inverso, lo cual es una decisión del operador que está
fuera del alcance de esta entrada de catálogo.

## Estructura de operación

- Una instancia corresponde a un binario, un directorio de contenido
  y un operador
- El contenido es canónico en disco y en Git; los índices y bases de
  datos son estado derivado, reconstruible desde el árbol de archivos
- Las instancias numeradas viven bajo
  `~/Foundry/deployments/media-knowledge-documentation-N/` y no se
  rastrean en Git

## Guías de operación

- `guide-provision-node.md` — aprovisionar usuario del sistema,
  directorios, binario y unidad systemd en el host
- `guide-deployment.md` — aprovisionar un directorio de instancia
  numerada y conectarlo a un árbol de contenido

## Postura de divulgación

Esta entrada de catálogo se mantiene bajo la postura de divulgación
continua descrita en `conventions/bcsc-disclosure-posture.md`. Las
declaraciones prospectivas llevan lenguaje de "planificado" o
"previsto", una base razonable declarada y las suposiciones
materiales correspondientes.

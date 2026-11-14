# Arquitectura UML — Grupo 3

Este directorio contiene la aportación del **diagrama UML de clases** para la Práctica 1 de Ingeniería del Software.

## Contenido

- `diagramas/clases/`: fuentes PlantUML (`.puml`) y exportaciones SVG/PNG de las cuatro vistas.
- `documentacion/documentacion_diagrama_clases.md`: explicación del diseño, capas, clases y trazabilidad.
- `documentacion/revision_diagrama_clases.md`: checklist para revisión manual. Los puntos pendientes no deben marcarse como validados sin comprobarlos.
- `referencias/`: enunciado PDF y Excel de requisitos usados como fuentes.
- `guias/`: instrucciones de integración y traspaso a la segunda arquitecta.

## Vistas de clases

1. `diagrama_clases`: visión global simplificada de las cuatro capas.
2. `detalle_clases_dominio`: entidades y relaciones del dominio ferroviario.
3. `detalle_clases_aplicacion`: servicios, DTOs y puertos.
4. `detalle_clases_externas`: controladores web y adaptadores de infraestructura.

## Arquitectura reflejada

Monolito modular organizado en cuatro capas:

- Presentación
- Aplicación
- Dominio
- Infraestructura

El modelo declara 22 clases, 2 interfaces/puertos y 3 enumeraciones (27 tipos modelados). Este recuento debe volver a comprobarse si se modifica el UML.

## Estado

La aportación de clases está preparada para integrarse con el diagrama de paquetes y el diagrama de secuencia de la otra arquitecta. No equivale a una validación académica final: quedan pendientes los contrastes manuales indicados en la checklist, especialmente RF1.4, el formato CSV y la integración entre diagramas.

## Regla de edición

Los `.puml` son las fuentes editables. Si se cambia alguno, deben regenerarse sus SVG y PNG y actualizarse la documentación y la checklist para que coincidan.

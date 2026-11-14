# Guía y Checklist de Revisión Manual del Diseño UML de Clases
## Sistema Ferroviario de Detección y Predicción de Incidencias
### Ingeniería del Software · Práctica 1 · Grupo 3

> **AVISO METODOLÓGICO**:
> Conforme a la normativa académica de la práctica, el uso de herramientas asistidas exige una **revisión manual exhaustiva y documentada** por parte de los integrantes del grupo antes de la entrega formal.
> Este documento recoge las comprobaciones del **cierre definitivo del diseño de clases**, certificando la coherencia entre el Diagrama Global Simplificado y las Tres Vistas Complementarias de Detalle. **Ningún apartado debe marcarse como validado hasta haber sido contrastado personalmente por los miembros del grupo**.

---

## 1. Identificación del Entregable y Conjunto de Vistas

- **Diagrama Global Simplificado**:
  - Código fuente: `diagrama_clases.puml`
  - Exportaciones gráficas: `diagrama_clases.svg` y `diagrama_clases.png`
- **Vistas Complementarias de Detalle**:
  - *Vista de Dominio*: `detalle_clases_dominio.puml` (`.svg` / `.png`)
  - *Vista de Aplicación (3 Bloques)*: `detalle_clases_aplicacion.puml` (`.svg` / `.png`)
  - *Vista de Presentación e Infraestructura*: `detalle_clases_externas.puml` (`.svg` / `.png`)
- **Documentación de Soporte**: `documentacion_diagrama_clases.md`.
- **Revisores Manuales**: Alumnos con el rol de Arquitectos Software.
- **Fecha de Revisión Manual Presencial**: `[Pendiente de realización por el grupo]`
- **Estado Global**: `PENDIENTE DE VALIDACIÓN MANUAL POR EL ALUMNO`

### Recuento Oficial de Elementos (Coincidente en Todo el Sistema)
- **Capas Arquitectónicas**: **4** (`Presentacion`, `Aplicacion`, `Dominio`, `Infraestructura`).
- **Total de Clases**: **22**
  - Presentación (2): `VisualizacionController`, `IncidenciasController`.
  - Aplicación (10): `PublicarIncidenciaDTO`, `IncidenciaDTO`, `SuscripcionDTO`, `IncidenciaGraficaDTO`, `DatosGraficaMinuto`, `ServicioAdquisicionDatos`, `ServicioIncidencias`, `ServicioPrediccion`, `ServicioSuscripciones`, `ServicioVisualizacion`.
  - Dominio (8): `ViaFerroviaria`, `DispositivoVia`, `MedicionElectrica`, `Incidencia`, `PrediccionIncidencia`, `Usuario`, `Gestor`, `Suscripcion`.
  - Infraestructura (2): `LectorCSV`, `ModuloMachineLearning`.
- **Total de Interfaces / Puertos**: **2** (`IModuloMachineLearning`, `ILectorCSV` en Aplicación).
- **Total de Enumeraciones**: **3** (`EstadoOcupacion`, `TipoIncidencia`, `OrigenIncidencia` en Dominio).
- **Total de Tipos Modelados**: **27 tipos**.

---

## 2. Checklist para la Revisión Manual del Equipo

### A. Comprobaciones Específicas de Cierre

| ID | Criterio de Cierre a Contrastar | Elementos a Inspeccionar | Estado Manual | Notas del Revisor |
|---|---|---|:---:|---|
| **CIE-01** | ¿`ServicioVisualizacion` representa explícitamente sus dependencias hacia `ServicioIncidencias` y `ServicioPrediccion`? | Relaciones de `ServicioVisualizacion` en Global y Aplicación | `[ ] Validado`<br>`[ ] Requiere ajuste` | Verificar que se visualiza de dónde obtiene incidencias y predicciones antes de transformarlas a `IncidenciaGraficaDTO`. |
| **CIE-02** | ¿Se ha eliminado motivadamente `filtrarGestoresPorTipo()` de `ServicioSuscripciones` por no responder a requisitos? | Signaturas de `ServicioSuscripciones` | `[ ] Validado`<br>`[ ] Requiere ajuste` | Constatar que solo existen operaciones de alta, cancelación y consulta de suscripciones. |
| **CIE-03** | ¿`ServicioPrediccion.generarPredicciones()` retorna `List<PrediccionIncidencia>` (entidad de Dominio)? | Signatura de `ServicioPrediccion` | `[ ] Validado`<br>`[ ] Requiere ajuste` | Confirmar que el servicio de predicción no mezcla lógica de vista. |
| **CIE-04** | ¿`ServicioSuscripciones.suscribirGestor()` recibe tipos primitivos (`String`) y valida `TipoIncidencia` internamente? | Signatura de `ServicioSuscripciones` | `[ ] Validado`<br>`[ ] Requiere ajuste` | Verificar que Presentación no importa enums de Dominio. |

---

### B. Aislamiento entre Capas Arquitectónicas

| ID | Regla de Aislamiento | Elementos a Inspeccionar | Estado Manual | Notas del Revisor |
|---|---|---|:---:|---|
| **AIS-01** | ¿Presentación interactúa únicamente con Servicios y DTOs de Aplicación? | `VisualizacionController`, `IncidenciasController` | `[ ] Validado`<br>`[ ] Requiere ajuste` | Ninguna entidad de Dominio en firmas de controladores. |
| **AIS-02** | ¿Aplicación no contiene dependencias hacia la capa de Presentación? | Paquete `Aplicacion` | `[ ] Validado`<br>`[ ] Requiere ajuste` | Los DTOs pertenecen formalmente a Aplicación. |
| **AIS-03** | ¿Dominio está completamente aislado de las tres capas restantes? | Paquete `Dominio` | `[ ] Validado`<br>`[ ] Requiere ajuste` | Entidades y Value Objects puros del ferrocarril. |
| **AIS-04** | ¿Infraestructura realiza los puertos de Aplicación mediante Inversión de Dependencias (DIP)? | `LectorCSV`, `ModuloMachineLearning` | `[ ] Validado`<br>`[ ] Requiere ajuste` | Relaciones `..|>` hacia `ILectorCSV` e `IModuloMachineLearning`. |

---

### C. Coherencia entre el Diagrama Global y las Tres Vistas de Detalle

| ID | Criterio de Coherencia | Vistas Involucradas | Estado Manual | Notas del Revisor |
|---|---|---|:---:|---|
| **COH-01** | ¿Las signaturas de `ServicioPrediccion`, `ServicioVisualizacion` y `ServicioSuscripciones` coinciden en Global y Aplicación? | `diagrama_clases` vs `detalle_clases_aplicacion` | `[ ] Validado`<br>`[ ] Requiere ajuste` | Comprobar paridad de operaciones. |
| **COH-02** | ¿Las operaciones de controladores y adaptadores coinciden en Global y en la Vista Externa? | `diagrama_clases` vs `detalle_clases_externas` | `[ ] Validado`<br>`[ ] Requiere ajuste` | Comprobar paridad de signaturas. |
| **COH-03** | ¿El modelo conceptual coincide exactamente en el Global y en la Vista de Dominio? | `diagrama_clases` vs `detalle_clases_dominio` | `[ ] Validado`<br>`[ ] Requiere ajuste` | Comprobar multiplicidades y herencia. |

---

### D. Trazabilidad con los Requisitos Oficiales del Enunciado

| Requisito | Descripción en el Excel | Cobertura en el Conjunto de Vistas | ¿Validado? | Firma / Comentario |
|---|---|---|:---:|---|
| **RF1 / RF1.1** | Detección y predicción ML | `IModuloMachineLearning`, `ModuloMachineLearning` (métodos explícitos de detección y predicción). | `[ ] Sí  [ ] No` | |
| **RF1.2** | Tipos considerados | `TipoIncidencia` con `BLOQUEO_PROLONGADO` y `SALTO_FRECUENCIA`. | `[ ] Sí  [ ] No` | |
| **RF1.3** | Bloqueo prolongado > 2 min | `ModuloMachineLearning.analizarBloqueoProlongado(...)`. | `[ ] Sí  [ ] No` | |
| **RF1.4** | Salto de al menos 0.5 V | `ModuloMachineLearning.analizarSaltoFrecuencia(...)` (con nota de ambigüedad física). | `[ ] Sí  [ ] No` | |
| **RF1.5 / RD1** | Aplicación web HTTP/REST | Controladores `VisualizacionController` e `IncidenciasController`. | `[ ] Sí  [ ] No` | |
| **RF2 / RF2.1 / RF2.2** | Ingesta CSV, hora HH:MM:SS:MS, estado 0/1 | `ILectorCSV`, `LectorCSV`, `MedicionElectrica`, `EstadoOcupacion`. | `[ ] Sí  [ ] No` | |
| **RF3 / RF3.1 / RF3.2** | Dispositivos cada 200 o 500 m con ID | `ViaFerroviaria`, `DispositivoVia` (`separacionSiguienteMetros`, `idDispositivo`). | `[ ] Sí  [ ] No` | |
| **RF4 / RF4.1 / RF4.2** | Publicación y suscripción a incidencias | `ServicioIncidencias.registrarIncidenciaManual`, `ServicioSuscripciones.suscribirGestor`. | `[ ] Sí  [ ] No` | |
| **RF5 / RF5.1-RF5.3** | Visualización gráfica por minuto | `VisualizacionController`, `ServicioVisualizacion`, `DatosGraficaMinuto`, `IncidenciaGraficaDTO`. | `[ ] Sí  [ ] No` | |

---

## 3. Cuestiones a Contrastar con los Analistas del Grupo

1. **Ratificación terminológica de RF1.4**:
   - Preguntar a los analistas si se mantiene la expresión literal *«salto de frecuencia de al menos 0.5 voltios»* con nota técnica al pie en la memoria, o si prefieren redactar una fe de erratas explícita sobre tensión eléctrica.
2. **Estructura definitiva del fichero CSV**:
   - Confirmar si el orden convenido de columnas en los datos de prueba es `Hora,Voltaje,Estado` o `Hora,Estado,Voltaje`.
3. **Validación de la integración con la compañera**:
   - Asegurar que los nombres de los paquetes de las cuatro capas coinciden exactamente con los de su **Diagrama de Paquetes** y que las signaturas de los servicios se corresponden con las llamadas del **Diagrama de Secuencia**.

---

## 4. Registro de Sesión de Revisión Presencial del Equipo

*(Completar por los alumnos durante la sesión de trabajo)*

| Fecha | Asistentes | Apartado Evaluado | Acuerdo / Decisión Adoptada | Visto Bueno |
|---|---|---|---|:---:|
| `[DD/MM/2026]` | `[Nombres]` | `[Dependencias ServicioVisualizacion]` | `[Aprobado por unanimidad]` | `[Firma]` |
| `[DD/MM/2026]` | `[Nombres]` | `[Eliminación filtrarGestoresPorTipo]` | `[Aprobado por estricta fidelidad a RF4]` | `[Firma]` |

---
*Checklist de control y calidad · Práctica 1 de Ingeniería del Software · Curso 2026-2027.*

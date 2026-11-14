# Documentación del Diseño UML de Clases del Sistema Ferroviario
## Arquitectura en Capas, Vistas UML y Cierre Definitivo de Diseño
### Ingeniería del Software · Práctica 1 · Grupo 3

---

## 1. Estrategia de Visualización: Diagrama Global y Vistas Complementarias

Para lograr el equilibrio idóneo entre legibilidad visual y exhaustividad técnica, el diseño se organiza mediante una **estrategia jerárquica en dos niveles**:

1. **Diagrama Global de Clases Simplificado (`diagrama_clases.puml` / `.svg` / `.png`)**:
   - **Propósito**: Proporcionar una visión panorámica completa del sistema y su organización modular en capas.
   - **Enfoque de simplificación**: Contiene la totalidad de los 27 tipos del modelo (22 clases, 2 interfaces y 3 enumeraciones) distribuidos en sus cuatro capas, con sus estereotipos y relaciones estructurales esenciales (composiciones, agregaciones, herencia, realizaciones de interfaces y dependencias entre capas). Prescinde de getters, setters y atributos o métodos triviales para permitir que el diagrama se lea de un vistazo y pueda insertarse en la memoria académica oficial.
2. **Tres Vistas Complementarias de Detalle**:
   - **Vista de Dominio (`detalle_clases_dominio.puml` / `.svg` / `.png`)**: Detalla el modelo conceptual del ferrocarril, con todas las entidades, objetos de valor, enumeraciones, atributos y operaciones de negocio.
   - **Vista de Aplicación (`detalle_clases_aplicacion.puml` / `.svg` / `.png`)**: Detalla la orquestación de casos de uso (Servicios), los modelos de transferencia (DTOs) y los contratos técnicos (Puertos), estructurada en 3 bloques independientes para facilitar su seguimiento.
   - **Vista de Presentación e Infraestructura (`detalle_clases_externas.puml` / `.svg` / `.png`)**: Detalla los puntos de interacción con el exterior: controladores HTTP/REST web y adaptadores técnicos para ficheros CSV y Machine Learning.

```
+---------------------------------------------------------------------------------------+
|                    DIAGRAMA GLOBAL SIMPLIFICADO (diagrama_clases)                     |
|            Visión integral de las 4 capas, 22 clases, 2 interfaces y 3 enums          |
+---------------------------------------------------------------------------------------+
                |                                  |                               |
                v                                  v                               v
+-------------------------------+  +-------------------------------+  +-------------------------------+
|     VISTA DETALLE DOMINIO     |  |   VISTA DETALLE APLICACIÓN    |  |  VISTA PRESENT. E INFRAESTR.  |
| (detalle_clases_dominio.puml) |  | (detalle_clases_aplicacion)   |  | (detalle_clases_externas.puml)|
| - Entidades ferroviarias      |  | - Bloque A: Servicios (casos) |  | - Controladores REST Web      |
| - Atributos y operaciones     |  | - Bloque B: DTOs              |  | - Adaptadores CSV y ML        |
| - Enumeraciones de estado     |  | - Bloque C: Puertos DIP       |  | - Realización de contratos    |
+-------------------------------+  +-------------------------------+  +-------------------------------+
```

---

## 2. Decisiones y Comprobaciones de Diseño de Cierre

Durante la fase de consolidación definitiva se analizaron y resolvieron con rigor los siguientes aspectos arquitectónicos:

### A. Dependencias y Flujo de `ServicioVisualizacion`
- **Problema evaluado**: `ServicioVisualizacion` es responsable de entregar los datos agregados por minuto a la interfaz web (RF5). Para ello, debe recopilar tanto las incidencias detectadas como las predicciones analíticas sobre un dispositivo de vía y transformarlas en el DTO de vista `IncidenciaGraficaDTO`.
- **Resolución en el diseño**:
  - `ServicioPrediccion.generarPredicciones(idDispositivo)` devuelve formalmente `List<PrediccionIncidencia>`, garantizando que el servicio de predicción se enfoque exclusivamente en la inferencia algorítmica de dominio sin asumir tareas de presentación.
  - `ServicioIncidencias.evaluarIncidenciasDispositivo(idDispositivo)` gestiona las incidencias detectadas.
  - `ServicioVisualizacion` colabora explícitamente con ambos servicios mediante dependencias UML:
    $$\text{ServicioVisualizacion} \dashrightarrow \text{ServicioIncidencias}\quad (\text{consulta incidencias})$$
    $$\text{ServicioVisualizacion} \dashrightarrow \text{ServicioPrediccion}\quad (\text{consulta predicciones})$$
    $$\text{ServicioVisualizacion} \dashrightarrow \text{DispositivoVia}\quad (\text{consulta mediciones/estado})$$
  - Una vez obtenidas las entidades del dominio, `ServicioVisualizacion` las mapea y transforma a `IncidenciaGraficaDTO`, componiéndolas dentro de `DatosGraficaMinuto`.
  - El pipeline queda perfectamente delimitado, transparente y documentado tanto en el diagrama global como en la vista de aplicación.

### B. Análisis y Justificación de la Operación en `ServicioSuscripciones`
- **Problema evaluado**: Se revisó la pertinencia de la operación previa `filtrarGestoresPorTipo()` en `ServicioSuscripciones`.
- **Resolución en el diseño**:
  - **Eliminación motivada**: Dicha operación no respondía a ningún requisito funcional explícito (RF4 y RF4.2 exigen la suscripción y gestión de las mismas, no un filtrado masivo de gestores). Además, retornaba colecciones de la entidad de Dominio `Gestor`, lo que generaba riesgo de fuga del modelo interno.
  - Al no existir en el alcance del sistema ningún canal de notificación push ni envío de correos que requiera localizar destinatarios masivos, la operación carecía de justificación funcional.
  - **Signaturas definitivas de `ServicioSuscripciones`**:
    - `+ suscribirGestor(idGestor: String, tipoIncidencia: String): SuscripcionDTO`
    - `+ cancelarSuscripcion(idSuscripcion: String): void`
    - `+ obtenerSuscripcionesGestor(idGestor: String): List<SuscripcionDTO>`
  - El servicio recibe tipos primitivos (`String`), valida y mapea internamente a la enumeración de dominio `TipoIncidencia`, administra la entidad `Gestor` y retorna `SuscripcionDTO`. Cero clases de Dominio expuestas a Presentación.

### C. Estructuración en Tres Bloques de la Vista de Aplicación
Para garantizar un lienzo limpio y legible en `detalle_clases_aplicacion.puml`, los elementos se estructuran en tres paquetes visuales:
- **Bloque A: Servicios de Aplicación (Casos de Uso)**: Los 5 servicios orquestadores del sistema.
- **Bloque B: Modelos de Transferencia de Datos (DTOs)**: Los 5 DTOs (`PublicarIncidenciaDTO`, `IncidenciaDTO`, `SuscripcionDTO`, `IncidenciaGraficaDTO`, `DatosGraficaMinuto`).
- **Bloque C: Puertos / Interfaces Técnicas**: Los 2 contratos abstractos (`IModuloMachineLearning`, `ILectorCSV`).

---

## 3. Recuento Oficial y Coincidente de Elementos en el Modelo

El recuento real y exacto en los archivos PlantUML definitivos y en esta memoria es:

- **Total de Paquetes / Capas**: **4** (`Presentacion`, `Aplicacion`, `Dominio`, `Infraestructura`).
- **Total de Clases**: **22**
  - *Capa de Presentación (2)*: `VisualizacionController`, `IncidenciasController`.
  - *Capa de Aplicación (10)*: `PublicarIncidenciaDTO`, `IncidenciaDTO`, `SuscripcionDTO`, `IncidenciaGraficaDTO`, `DatosGraficaMinuto`, `ServicioAdquisicionDatos`, `ServicioIncidencias`, `ServicioPrediccion`, `ServicioSuscripciones`, `ServicioVisualizacion`.
  - *Capa de Dominio (8)*: `ViaFerroviaria`, `DispositivoVia`, `MedicionElectrica`, `Incidencia`, `PrediccionIncidencia`, `Usuario`, `Gestor`, `Suscripcion`.
  - *Capa de Infraestructura (2)*: `LectorCSV`, `ModuloMachineLearning`.
- **Total de Interfaces / Puertos**: **2** (`IModuloMachineLearning`, `ILectorCSV` en Aplicación).
- **Total de Enumeraciones**: **3** (`EstadoOcupacion`, `TipoIncidencia`, `OrigenIncidencia` en Dominio).
- **Total de Tipos Modelados**: **27 tipos**.

---

## 4. Atributos y Operaciones Principales

| Clase / Elemento | Atributos Principales | Operaciones Principales |
|---|---|---|
| **`VisualizacionController`** | `- servicioVisualizacion : ServicioVisualizacion` | `+ obtenerDatosGrafica(idDispositivo : String, minuto : String) : DatosGraficaMinuto` |
| **`IncidenciasController`** | `- servicioIncidencias : ServicioIncidencias`<br>`- servicioSuscripciones : ServicioSuscripciones` | `+ publicarIncidencia(dto : PublicarIncidenciaDTO) : IncidenciaDTO`<br>`+ listarIncidencias(idDispositivo : String) : List<IncidenciaDTO>`<br>`+ suscribirGestor(idGestor : String, tipo : String) : SuscripcionDTO`<br>`+ cancelarSuscripcion(idSuscripcion : String) : void`<br>`+ listarSuscripcionesGestor(idGestor : String) : List<SuscripcionDTO>` |
| **`PublicarIncidenciaDTO`** | `- idUsuario : String`<br>`- idDispositivo : String`<br>`- tipoIncidencia : String`<br>`- descripcion : String` | `+ getIdUsuario() : String`<br>`+ getIdDispositivo() : String`<br>`+ getTipoIncidencia() : String`<br>`+ getDescripcion() : String` |
| **`IncidenciaDTO`** | `- idIncidencia : String`<br>`- idDispositivo : String`<br>`- fechaHora : String`<br>`- tipo : String`<br>`- origen : String`<br>`- descripcion : String` | `+ getIdIncidencia() : String`<br>`+ getIdDispositivo() : String`<br>`+ getFechaHora() : String`<br>`+ getTipo() : String`<br>`+ getOrigen() : String`<br>`+ getDescripcion() : String` |
| **`SuscripcionDTO`** | `- idSuscripcion : String`<br>`- idGestor : String`<br>`- tipoIncidencia : String`<br>`- fechaAlta : String` | `+ getIdSuscripcion() : String`<br>`+ getIdGestor() : String`<br>`+ getTipoIncidencia() : String`<br>`+ getFechaAlta() : String` |
| **`IncidenciaGraficaDTO`** | `- hora : String`<br>`- tipo : String`<br>`- esPrediccion : boolean`<br>`- probabilidad : double`<br>`- descripcion : String` | `+ getHora() : String`<br>`+ getTipo() : String`<br>`+ esPrediccion() : boolean`<br>`+ getProbabilidad() : double`<br>`+ getDescripcion() : String` |
| **`DatosGraficaMinuto`** | `- minutoReferencia : String`<br>`- marcasTiempo : List<String>`<br>`- estadosOcupacion : List<int>`<br>`- valoresVoltaje : List<double>`<br>`- incidenciasVisualizacion : List<IncidenciaGraficaDTO>` | `+ getMinutoReferencia() : String`<br>`+ getMarcasTiempo() : List<String>`<br>`+ getEstadosOcupacion() : List<int>`<br>`+ getValoresVoltaje() : List<double>`<br>`+ getIncidenciasVisualizacion() : List<IncidenciaGraficaDTO>` |
| **`IModuloMachineLearning`** | *(Puerto de Aplicación)* | `+ detectarIncidencias(mediciones : List<MedicionElectrica>) : List<Incidencia>`<br>`+ predecirIncidencias(historico : List<MedicionElectrica>) : List<PrediccionIncidencia>` |
| **`ILectorCSV`** | *(Puerto de Aplicación)* | `+ leerMediciones(rutaFichero : String) : List<MedicionElectrica>` |
| **`ServicioAdquisicionDatos`** | `- lectorCSV : ILectorCSV` | `+ procesarFicheroCSV(rutaFichero : String, idDispositivo : String) : int` |
| **`ServicioIncidencias`** | `- moduloML : IModuloMachineLearning` | `+ evaluarIncidenciasDispositivo(idDispositivo : String) : List<IncidenciaDTO>`<br>`+ registrarIncidenciaManual(dto : PublicarIncidenciaDTO) : IncidenciaDTO`<br>`+ obtenerIncidenciasDispositivo(idDispositivo : String) : List<IncidenciaDTO>` |
| **`ServicioPrediccion`** | `- moduloML : IModuloMachineLearning` | `+ generarPredicciones(idDispositivo : String) : List<PrediccionIncidencia>` |
| **`ServicioSuscripciones`** | *(Servicio de aplicación)* | `+ suscribirGestor(idGestor : String, tipoIncidencia : String) : SuscripcionDTO`<br>`+ cancelarSuscripcion(idSuscripcion : String) : void`<br>`+ obtenerSuscripcionesGestor(idGestor : String) : List<SuscripcionDTO>` |
| **`ServicioVisualizacion`** | *(Servicio de aplicación)* | `+ prepararDatosMinuto(idDispositivo : String, minuto : String) : DatosGraficaMinuto` |
| **`ViaFerroviaria`** | `- idVia : String`<br>`- nombre : String` | `+ getIdVia() : String`<br>`+ getNombre() : String`<br>`+ agregarDispositivo(dispositivo : DispositivoVia) : void`<br>`+ obtenerDispositivos() : List<DispositivoVia>` |
| **`DispositivoVia`** | `- idDispositivo : String`<br>`- separacionSiguienteMetros : int`<br>`- puntoKilometrico : double` | `+ getIdDispositivo() : String`<br>`+ getSeparacionSiguienteMetros() : int`<br>`+ getPuntoKilometrico() : double`<br>`+ registrarMedicion(medicion : MedicionElectrica) : void`<br>`+ asociarIncidencia(incidencia : Incidencia) : void`<br>`+ asociarPrediccion(prediccion : PrediccionIncidencia) : void`<br>`+ obtenerMedicionesMinuto(minuto : String) : List<MedicionElectrica>`<br>`+ obtenerTodasMediciones() : List<MedicionElectrica>` |
| **`MedicionElectrica`** | `- hora : String`<br>`- voltaje : double`<br>`- estado : EstadoOcupacion` | `+ getHora() : String`<br>`+ getVoltaje() : double`<br>`+ getEstado() : EstadoOcupacion`<br>`+ estaOcupada() : boolean` |
| **`Incidencia`** | `- idIncidencia : String`<br>`- fechaHora : DateTime`<br>`- tipo : TipoIncidencia`<br>`- origen : OrigenIncidencia`<br>`- descripcion : String` | `+ getIdIncidencia() : String`<br>`+ getFechaHora() : DateTime`<br>`+ getTipo() : TipoIncidencia`<br>`+ getOrigen() : OrigenIncidencia`<br>`+ getDescripcion() : String` |
| **`PrediccionIncidencia`** | `- idPrediccion : String`<br>`- fechaHoraCalculo : DateTime`<br>`- tipoPronosticado : TipoIncidencia`<br>`- probabilidad : double`<br>`- horizonteEstimadoMinutos : int` | `+ getIdPrediccion() : String`<br>`+ getFechaHoraCalculo() : DateTime`<br>`+ getTipoPronosticado() : TipoIncidencia`<br>`+ getProbabilidad() : double`<br>`+ getHorizonteEstimadoMinutos() : int` |
| **`Usuario`** | `- idUsuario : String`<br>`- nombre : String`<br>`- email : String` | `+ getIdUsuario() : String`<br>`+ getNombre() : String`<br>`+ getEmail() : String` |
| **`Gestor`** | *(Hereda atributos de Usuario)* | `+ agregarSuscripcion(suscripcion : Suscripcion) : void`<br>`+ eliminarSuscripcion(idSuscripcion : String) : void`<br>`+ getSuscripciones() : List<Suscripcion>` |
| **`Suscripcion`** | `- idSuscripcion : String`<br>`- fechaAlta : DateTime`<br>`- tipoIncidencia : TipoIncidencia` | `+ getIdSuscripcion() : String`<br>`+ getFechaAlta() : DateTime`<br>`+ getTipoIncidencia() : TipoIncidencia`<br>`+ esAplicableA(tipo : TipoIncidencia) : boolean` |
| **`LectorCSV`** | *(Adaptador de Infraestructura)* | `+ leerMediciones(rutaFichero : String) : List<MedicionElectrica>`<br>`- parsearHora(token : String) : String`<br>`- parsearVoltaje(token : String) : double`<br>`- parsearEstado(token : String) : EstadoOcupacion` |
| **`ModuloMachineLearning`** | *(Adaptador de Infraestructura)* | `+ detectarIncidencias(mediciones : List<MedicionElectrica>) : List<Incidencia>`<br>`+ predecirIncidencias(historico : List<MedicionElectrica>) : List<PrediccionIncidencia>`<br>`- analizarBloqueoProlongado(mediciones : List<MedicionElectrica>) : List<Incidencia>`<br>`- analizarSaltoFrecuencia(mediciones : List<MedicionElectrica>) : List<Incidencia>` |

---

## 5. Matriz de Trazabilidad con los Requisitos Oficiales

| Requisito | Denominación Oficial | Elementos Responsables | Cobertura en el Diseño |
|---|---|---|---|
| **RF1 / RF1.1** | Detección y predicción con Machine Learning | `IModuloMachineLearning`, `ModuloMachineLearning`, `ServicioIncidencias`, `ServicioPrediccion` | Coordinación de detección reactiva y predicción futura. |
| **RF1.2** | Tipos de incidencias considerados | `TipoIncidencia` (`BLOQUEO_PROLONGADO`, `SALTO_FRECUENCIA`) | Catálogo formalizado como enum de dominio. |
| **RF1.3** | Detección de bloqueo prolongado > 2 min | `ModuloMachineLearning.analizarBloqueoProlongado(...)`, `Incidencia` | Detección automática por ausencia de emisión $> 120$ s. |
| **RF1.4** | Detección de saltos de al menos 0.5 V | `ModuloMachineLearning.analizarSaltoFrecuencia(...)`, `Incidencia` | Detección de variaciones de potencial $\ge 0.5$ V (con nota de ambigüedad física). |
| **RF1.5 / RD1** | Aplicación web HTTP/REST | `VisualizacionController`, `IncidenciasController` | Endpoints REST web desacoplados de dominio mediante DTOs. |
| **RF2** | Lectura de datos desde CSV | `ILectorCSV`, `LectorCSV`, `ServicioAdquisicionDatos` | Ingesta desacoplada mediante contrato de lectura y adaptador. |
| **RF2.1** | Formato de la hora | `MedicionElectrica.hora: String`, `LectorCSV.parsearHora(...)` | Soporte expreso del formato horario `HH:MM:SS:MS`. |
| **RF2.2** | Estado binario | `EstadoOcupacion` (0: Ocupado, 1: Libre), `MedicionElectrica.estado` | Tipado enumerado que preserva la semántica binaria ferroviaria (0=tren en vía, 1=vía libre). |
| **RF3** | Dispositivos de vía | `ViaFerroviaria`, `DispositivoVia`, `MedicionElectrica` | Estructuración física y composición de vía y sensores. |
| **RF3.1** | Separación entre dispositivos | `DispositivoVia.separacionSiguienteMetros: int` | Parametrización admisible de 200 o 500 metros entre dispositivos. |
| **RF3.2** | Identificador de dispositivo | `DispositivoVia.idDispositivo: String` | Clave identificativa unívoca por dispositivo. |
| **RF4** | Suscripción a incidencias | `ServicioSuscripciones`, `Suscripcion`, `Gestor` | Caso de uso dedicado para gestionar suscripciones de gestores. |
| **RF4.1** | Publicación de incidencias | `ServicioIncidencias.registrarIncidenciaManual(...)`, `PublicarIncidenciaDTO` | Caso de uso con DTO web para reporte manual de usuarios. |
| **RF4.2** | Suscripción a tipos de incidencias | `ServicioSuscripciones.suscribirGestor(...)`, `Gestor`, `SuscripcionDTO` | Habilita a los gestores a filtrar y suscribirse a categorías de incidencias. |
| **RF5** | Visualización gráfica | `VisualizacionController`, `ServicioVisualizacion`, `DatosGraficaMinuto` | Pipeline de preparación y entrega de métricas visuales. |
| **RF5.1** | Contenido de la gráfica | `DatosGraficaMinuto` (horas, estados 0/1, voltajes) | Empaquetamiento simultáneo de variables temporales. |
| **RF5.2** | Indicación de incidencias | `DatosGraficaMinuto.incidenciasVisualizacion`, `IncidenciaGraficaDTO` | Señalización gráfica del instante del fallo y su categoría sin acoplamiento a dominio. |
| **RF5.3** | Frecuencia de visualización cada minuto | `DatosGraficaMinuto.minutoReferencia`, `ServicioVisualizacion.prepararDatosMinuto(...)` | Intervalos discretos de refresco de 60 segundos. |

---

## 6. Ambigüedades y Decisiones Pendientes de Validación con Analistas

Se mantienen registradas formalmente las cuestiones del enunciado pendientes de ratificación por el equipo de analistas:

1. **Ambigüedad física en RF1.4: «Saltos de frecuencia de al menos 0.5 voltios»**:
   - La frecuencia se mide en Hertzios (Hz) y la tensión en voltios (V). El sistema evalúa variaciones de potencial eléctrico ($\Delta V \ge 0.5\text{ V}$) manteniendo la etiqueta literal `SALTO_FRECUENCIA`. Se debe ratificar si se incluirá una fe de erratas formal en la memoria.
2. **Discrepancia en columnas del CSV**:
   - El anexo del PDF cita únicamente *hora y estado*, mientras que el Excel (RF2, RF5.1) exige lecturas continuas de *voltaje*. El modelo procesa las tres variables. Queda pendiente acordar con analistas el orden físico de columnas (`Hora,Voltaje,Estado` vs `Hora,Estado,Voltaje`) y el carácter delimitador.
3. **Ausencia de identificador de dispositivo en el fichero CSV**:
   - El CSV no posee una columna de identificación de sensor. El método `procesarFicheroCSV(ruta, idDispositivo)` realiza la vinculación contextual durante la carga.

---

## 7. Integración con los Diagramas de Paquetes y Secuencia

- **Diagrama de Paquetes**: Las 4 capas (`Presentacion`, `Aplicacion`, `Dominio`, `Infraestructura`) se mapean de forma unívoca a los paquetes asignados a la compañera de equipo, sin dependencias circulares.
- **Diagrama de Secuencia**: Las operaciones expuestas en los servicios y controladores proporcionan signaturas bien tipadas con DTOs de transporte para documentar los flujos dinámicos de *Ingesta y Detección*, *Reporte Manual*, *Gestión de Suscripciones* y *Consulta de Gráficas*.

---
*Memoria técnica oficial y definitiva de Diseño UML · Curso 2026-2027.*

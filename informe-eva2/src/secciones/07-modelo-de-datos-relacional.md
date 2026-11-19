# Modelo de Datos Relacional

<!-- GUÍA · Sección 8 de la pauta · Criterio 2.1.4 (10 pts, junto con NoSQL).
Cubre: DER, diccionario, claves, cardinalidades, reglas de integridad; C-02/CAL-02 (propagación a todos o a algunos),
C-06/CAL-03 (clonación), CAL-08 (ciclo de vida y sincronización cada 10 min); cuentas y enlaces; coexistencia (RF-08).
El esquema real lo enviará el docente: este modelo se ajustará cuando llegue. -->

El modelo relacional parte del esquema que ya existe. En el legado, cada **sistema** (una de las 37 aplicaciones) tiene opciones de acceso que agrupan **transacciones** (opciones de menú), y a cada usuario se le asignan una por una, sistema por sistema (`tb_usuario_sistema_perfil`). En este informe, cada una de esas opciones de acceso se llama **permiso**. Lo que falta es el **perfil**: un paquete de permisos de varias aplicaciones que se asigna de una vez. El diseño conserva el catálogo legado y agrega encima, en un esquema `rbac`, las tablas de perfiles, colaboradores, cuentas, excepciones, proyección y auditoría.

En este modelo, **un permiso es el acceso a una aplicación con una de sus opciones** y corresponde a un registro del catálogo legado `tb_perfil` (columna `dc_perfil`). Los nombres de las tablas legadas se mantienen porque los satélites dependen de ellos. Así, los permisos efectivos se entregan a los satélites con las mismas claves que ya conocen. El docente se comprometió a enviar el esquema real; cuando llegue, se verificará que las tablas legadas referenciadas coincidan con este modelo.

## Diagrama Entidad-Relación

El modelo se presenta en tres diagramas. La Figura 16 muestra el núcleo RBAC; las tablas con prefijo `tb_` pertenecen al esquema legado `dbo` y se mantienen sin cambios.

![Diagrama entidad-relación del núcleo RBAC](assets/diagramas/06-der.png "Figura 16. DER del núcleo RBAC: catálogo, versiones de perfil, asignaciones, excepciones y proyección. Fuente: Elaboración propia (2026).")

La Figura 17 muestra las identidades, las cuentas del portal y la sincronización con el directorio:

![Diagrama entidad-relación de cuentas, acceso y sincronización](assets/diagramas/10-der-cuentas.png "Figura 17. DER de colaboradores, cuentas, solicitudes de acceso, enlaces y sincronización. Fuente: Elaboración propia (2026)."){.img-h12}

La Figura 18 muestra la auditoría y la recertificación:

![Diagrama entidad-relación de auditoría y recertificación](assets/diagramas/07-der-auditoria.png "Figura 18. DER de auditoría y recertificación. Fuente: Elaboración propia (2026)."){.img-h12}

## Diccionario de Datos

Las tablas nuevas usan nombres en español sin prefijos; los prefijos del legado (`dc_` código, `dg_` glosa) se conservan solo en las columnas que referencian tablas legadas. Las tablas que se modifican desde el portal incluyen además una columna `fila_version` (`rowversion`) para el control de concurrencia de la sección 4.

**`rbac.colaborador`**: identidad de una persona, sincronizada desde el directorio.

| Columna | Tipo | Nulo | Descripción y regla |
| --- | --- | --- | --- |
| `id_colaborador` | `int` identidad | No | Clave primaria. |
| `id_directorio` | `varchar(100)` | No | Identificador de la persona en AD. Único: no hay colaborador sin identidad corporativa (CAL-08). |
| `dc_usuario` | `varchar(50)` | No | Usuario legado (`dbo.tb_usuario`), la clave que usan los satélites. Único. |
| `nombre`, `correo`, `departamento_ad` | `nvarchar(150)` | No | Copia desde el directorio en cada sincronización. `departamento_ad` es informativo: el departamento del perfil es el del catálogo propio. |
| `estado` | `varchar(12)` | No | `ACTIVO` o `DESACTIVADO`. |
| `desactivado_en`, `desactivado_por` | `datetime2`, `varchar(50)` | Sí | Quién lo desactivó: un administrador o `PROCESO_SINCRONIZACION`. |

**`rbac.cuenta`**: credencial para ingresar al portal. Se implementa con las tablas de usuarios de ASP.NET Core Identity, extendidas con estas columnas.

| Columna | Tipo | Nulo | Descripción y regla |
| --- | --- | --- | --- |
| `id_cuenta` | `int` identidad | No | Clave primaria. |
| `id_colaborador` | `int` | No | Único: una cuenta por colaborador. |
| `nombre_usuario` | `varchar(50)` | No | Único. Coincide con el usuario corporativo. |
| `hash_contrasena` | `varchar(400)` | Sí | *Hash* con sal generado por Identity (PBKDF2 con miles de iteraciones). Es nulo hasta que la persona define su contraseña. Nunca existe una columna con la contraseña. |
| `rol` | `varchar(20)` | No | `ADMINISTRADOR_TI` o `COLABORADOR`. Un solo valor por cuenta. |
| `estado` | `varchar(12)` | No | `PENDIENTE_ACTIVACION`, `ACTIVA`, `BLOQUEADA` o `DESACTIVADA`. |
| `intentos_fallidos`, `bloqueada_hasta` | `int`, `datetime2` | Sí en `bloqueada_hasta` | Control de fuerza bruta: 5 intentos y 15 minutos. |
| `sello_seguridad` | `varchar(100)` | No | Cambia al definir una contraseña nueva, lo que invalida todas las sesiones abiertas. |

**`rbac.solicitud_acceso`** y **`rbac.enlace_un_uso`**: flujo de activación y recuperación.

| Columna | Tipo | Nulo | Descripción y regla |
| --- | --- | --- | --- |
| `solicitud_acceso.estado` | `varchar(10)` | No | `PENDIENTE`, `APROBADA` o `RECHAZADA`. Índice único filtrado: una sola solicitud pendiente por colaborador. |
| `solicitud_acceso.creada_por` | `int` | Sí | Nulo si la creó el propio colaborador; si no, la cuenta del administrador que la creó. |
| `solicitud_acceso.resuelta_por` | `int` | Sí | Restricción `CHECK`: distinta de `creada_por` (separación de funciones). |
| `solicitud_acceso.id_perfil_inicial` | `int` | Sí | Perfil asignado al aprobar; obligatorio si el estado es `APROBADA`. |
| `enlace_un_uso.tipo` | `char(1)` | No | `A` activación o `R` restablecimiento. |
| `enlace_un_uso.hash_token` | `binary(32)` | No | Solo se guarda el *hash* SHA-256 del enlace; el enlace completo existe únicamente en el correo. |
| `enlace_un_uso.expira_en`, `usado_en` | `datetime2` | Sí en `usado_en` | Vence a los 15 minutos. Un enlace usado o vencido no se acepta. Al crear uno nuevo, los anteriores de la misma cuenta quedan invalidados. |

**`rbac.perfil`**, **`rbac.perfil_version`** y **`rbac.perfil_version_permiso`**: perfil corporativo y sus plantillas versionadas.

| Columna | Tipo | Nulo | Descripción y regla |
| --- | --- | --- | --- |
| `perfil.id_perfil`, `perfil.id_departamento`, `perfil.nombre` | `int`, `int`, `nvarchar(100)` | No | Nombre único dentro del departamento. Un perfil con asignaciones vigentes solo se desactiva. |
| `perfil_version.version` | `int` | No | Clave primaria compuesta con `id_perfil`. Cada cambio de plantilla crea una versión nueva; las anteriores no se modifican. |
| `perfil_version_permiso.dc_perfil` | `int` | No | Permisos de esa versión de la plantilla. |

**`rbac.asignacion_perfil`**: perfil y versión que tiene un colaborador.

| Columna | Tipo | Nulo | Descripción y regla |
| --- | --- | --- | --- |
| `id_asignacion` | `bigint` identidad | No | Clave primaria. |
| `id_colaborador`, `id_perfil`, `version` | `int` | No | La versión indica qué plantilla aplica a este colaborador. Índice único filtrado: un perfil no puede estar vigente dos veces para la misma persona. |
| `fecha_inicio`, `fecha_fin` | `datetime2` | Sí en `fecha_fin` | Vigente mientras `fecha_fin` sea nula. Nunca se borra: se cierra. |
| `id_evento_origen` | `bigint` | No | Evento de auditoría que la creó (asignación, clonación, aprobación de acceso o carga inicial). |

**`rbac.excepcion`**: permiso individual fuera del perfil.

| Columna | Tipo | Nulo | Descripción y regla |
| --- | --- | --- | --- |
| `id_colaborador`, `dc_perfil` | `int` | No | A quién y sobre qué permiso. |
| `tipo` | `char(1)` | No | `C` concede un permiso que el perfil no da; `R` retira uno que el perfil sí da. |
| `justificacion`, `ticket` | `nvarchar(500)`, `varchar(30)` | No | Obligatorios. |
| `fecha_inicio`, `fecha_termino` | `date` | No | Término posterior al inicio y a no más de 180 días (propuesta por validar). |
| `estado` | `varchar(12)` | No | `PENDIENTE`, `APROBADA`, `RECHAZADA`, `VENCIDA` o `REVOCADA`. |
| `solicitada_por`, `resuelta_por` | `int` | Sí en `resuelta_por` | Restricción `CHECK`: distintos. |

**`rbac.permiso_efectivo`**: proyección de lo que cada usuario puede hacer. Solo la escribe el motor de permisos.

| Columna | Tipo | Nulo | Descripción y regla |
| --- | --- | --- | --- |
| `dc_usuario`, `dc_sistema`, `dc_perfil` | `varchar(50)`, `int`, `int` | No | Clave primaria compuesta en el mismo orden que `tb_usuario_sistema_perfil`, para que las consultas de los satélites usen el índice. |
| `origen` | `varchar(10)` | No | `PERFIL`, `EXCEPCION` o `MIGRADO`. |
| `calculado_en` | `datetime2` | No | Momento del último recálculo. |

**`rbac.ejecucion_sincronizacion`**: registro de cada sincronización con el directorio (RNF-13). Guarda inicio, fin, cantidad de activos recibidos, altas, bajas, si el freno se activó y el motivo.

**`rbac.evento_auditoria`** y **`rbac.adjunto_auditoria`**: bitácora inmutable.

| Columna | Tipo | Nulo | Descripción y regla |
| --- | --- | --- | --- |
| `id_evento` | `bigint` identidad | No | Clave primaria. |
| `id_operacion` | `uniqueidentifier` | No | Agrupa los eventos de una misma operación, por ejemplo un cambio de plantilla aplicado a 37 colaboradores. |
| `fecha_utc` | `datetime2` | No | Asignada por el servidor de base de datos. |
| `tipo_evento` | `varchar(30)` | No | Por ejemplo, `LOGIN_FALLIDO`, `ACCESO_APROBADO`, `ASIGNAR_PERFIL`, `CLONACION`, `BAJA_AUTOMATICA` o `ANULACION`. |
| `id_operador`, `id_colaborador` | `varchar(50)`, `int` | Sí en `id_colaborador` | Quién (una cuenta o el proceso de sincronización) y a quién. |
| `ticket`, `justificacion` | `varchar(30)`, `nvarchar(500)` | Sí | Obligatorios en los cambios hechos por personas. |
| `detalle` | `nvarchar(max)` | No | JSON con el estado anterior y posterior; `CHECK (ISJSON(detalle) = 1)`. Nunca contiene contraseñas ni enlaces. |
| `id_evento_anulado` | `bigint` | Sí | Una corrección crea un evento `ANULACION` que referencia al original (RNF-04). |
| `adjunto_auditoria.sha256` | `binary(32)` | No | Huella del archivo de respaldo, de máximo 5 MB. |

**`rbac.departamento`**, **`rbac.campana_recertificacion`** y **`rbac.decision_recertificacion`** completan el modelo para HU06 y HU11. Una campaña pertenece a un departamento y está `ABIERTA` o `CERRADA`; cada decisión registra, para un colaborador y un permiso, si se confirma o se revoca, quién decidió y cuándo.

## Claves, Cardinalidades y Reglas de Integridad

Las cardinalidades principales del modelo son:

- Un **departamento** agrupa cero o más perfiles, y cada perfil pertenece a uno.
- Un **perfil** tiene una o más versiones, y cada versión tiene uno o más permisos.
- Un **colaborador** corresponde a un usuario legado (uno a uno), tiene como máximo una cuenta, y puede tener cero o más solicitudes, asignaciones, excepciones y permisos efectivos.
- Una **cuenta** puede tener cero o más enlaces, pero solo uno vigente.
- Un **evento de auditoría** puede tener cero o más adjuntos, y puede anular a un evento anterior.

Las reglas que la base de datos puede asegurar se implementan como restricciones, para que se cumplan aunque exista un error en la aplicación:

| ID | Regla | Dónde se asegura |
| --- | --- | --- |
| RI-01 | No existe un colaborador sin identidad en el directorio, ni dos con la misma. | `NOT NULL` y `UNIQUE` sobre `id_directorio`. |
| RI-02 | Un colaborador no tiene dos asignaciones vigentes del mismo perfil. | Índice único filtrado `WHERE fecha_fin IS NULL`. |
| RI-03 | Quien resuelve una solicitud de acceso o una excepción no es quien la creó. | Restricciones `CHECK` y validación en la API. |
| RI-04 | Una excepción siempre tiene fecha de término, posterior a su inicio. | `CHECK` y `NOT NULL`. |
| RI-05 | Un colaborador no tiene a la vez una concesión y un retiro vigentes del mismo permiso. | Validación en el dominio antes de aprobar. |
| RI-06 | La bitácora no se modifica ni se borra. | Tabla *ledger append-only* de SQL Server 2022; si la versión corporativa no la soporta, `DENY UPDATE, DELETE` y un *trigger* que rechaza ambas operaciones. |
| RI-07 | Ningún cambio de permisos existe sin su evento de auditoría. | El cambio y el evento se escriben en la misma transacción. |
| RI-08 | Un perfil, permiso o aplicación en uso no se borra. | Claves foráneas sin borrado en cascada; los registros se desactivan. |
| RI-09 | Ninguna contraseña ni enlace se guarda en forma legible. | Solo existen columnas de *hash*; revisión del modelo en cada *pull request*. |
| RI-10 | Dos operaciones no modifican al mismo colaborador o perfil al mismo tiempo. | Bloqueo de fila al inicio de la transacción y verificación de `fila_version` (sección 4). |

Fuente: Elaboración propia (2026).

## Reglas de Negocio: Propagación y Clonación

Esta subsección responde a los compromisos C-02 y C-06. Ambas reglas se apoyan en la misma definición:

> Permisos efectivos de un colaborador = (permisos de la versión asignada de cada uno de sus perfiles vigentes) + (concesiones aprobadas y vigentes) − (retiros aprobados y vigentes). Si el colaborador está `DESACTIVADO`, sus permisos efectivos son vacíos.

**Regla de propagación al modificar la plantilla de un perfil (C-02, CAL-02).** Fue acordada con el cliente:

1. **Cada cambio crea una versión nueva del perfil.** Las versiones anteriores no se modifican.
2. **El administrador elige a quién aplicarla:** a todos los colaboradores que tienen el perfil, o solo a los que seleccione. Los no seleccionados siguen en la versión anterior y la pantalla los muestra como desactualizados, para actualizarlos después si corresponde. Las asignaciones nuevas siempre reciben la última versión.
3. **Se aplica en una sola transacción.** Si falla para uno, no se aplica a ninguno. Esto corrige un hallazgo del legado, que guardaba permiso por permiso sin transacción.
4. **Nadie pierde un permiso que tiene por otra vía.** Si un permiso quitado también viene de otro perfil o de una concesión vigente, el colaborador lo conserva.
5. **Las excepciones sobreviven al cambio.** Una concesión sigue vigente aunque su permiso entre después a la plantilla, y la pantalla la marca como redundante.
6. **Se registra quién fue afectado.** La bitácora guarda un evento por colaborador afectado, con la versión anterior y la nueva, agrupados por el mismo `id_operacion`.

**Definición operativa de la clonación (C-06, CAL-03):**

1. **Qué se copia:** los **perfiles vigentes** del usuario de referencia que el administrador seleccione, **en la misma versión** que tiene la referencia. Así el destino queda exactamente igual, que es lo que pide la jefatura: «para Abraham, los mismos permisos que para Pablo». No se copia la lista plana de permisos, porque se perdería el vínculo con el perfil y la propagación dejaría de funcionar.
2. **Qué no se copia:** las **excepciones**. Son individuales y fueron aprobadas para otra persona; si el destino necesita una, se solicita como excepción nueva.
3. **Si la referencia cambia después:** no afecta al destino. La clonación es una copia en un momento dado.
4. **Qué se registra:** un evento `CLONACION` con referencia, destino, perfiles y versiones copiados, perfiles previos del destino, motivo y ticket.

**Ciclo de vida del colaborador (CAL-08).** Replica el proceso actual del cliente:

- **Alta de la identidad:** la sincronización crea el colaborador cuando aparece en la lista de activos del directorio. No tiene cuenta ni permisos hasta que se aprueba su solicitud de acceso.
- **Activación de la cuenta:** solicitud → aprobación por un administrador distinto del creador → enlace → definición de la contraseña.
- **Baja automática:** si el colaborador deja de aparecer en la lista de activos, la siguiente sincronización (máximo 10 minutos) lo desactiva: su cuenta queda `DESACTIVADA` y sus permisos efectivos, vacíos. Como en la empresa los despidos se tratan con reserva y TI de soporte suele no saberlo antes, la baja no depende de que alguien la informe.
- **Freno de seguridad:** si la lista trae más de un 5 % menos de activos que la ejecución anterior, se supone un error del directorio, no se desactiva a nadie y se avisa a TI.
- **Baja manual (HU10):** TI puede desactivar a un colaborador antes de que el directorio lo refleje. El registro no se elimina, para conservar la trazabilidad.

## Coexistencia con el Modelo Legado

El cliente lo resumió así: «la pregunta sigue siendo la misma, la respuesta sigue siendo la misma». La coexistencia cumple RF-08 y RNF-05 sin modificar a los satélites:

| Elemento legado | Tratamiento |
| --- | --- |
| `tb_sistema`, `tb_perfil`, `tb_transaccion`, `tb_perfil_transaccion` | Se mantienen como catálogo; el modelo nuevo los referencia. `tb_sistema` es el catálogo de aplicaciones que mantiene HU19. |
| `tb_usuario` | Se mantiene como identidad legada. Su columna `dg_password` (cifrado reversible) **deja de usarse y se vacía**; antes debe confirmarse con el cliente que ningún satélite autentique contra ella (riesgo R-12). |
| `tb_usuario_sistema_perfil` y `tb_usuario_sistema` | Dejan de ser la fuente de verdad. Los procedimientos que los consultan se reescriben como **fachadas** que leen `rbac.permiso_efectivo`, con los mismos nombres, parámetros y columnas. Si algún satélite lee estas tablas directamente, se reemplazan por vistas con el mismo nombre. |
| `tb_usuario_ad_staging` y su procedimiento de importación | Se reemplazan por la sincronización cada 10 minutos. |
| `tb_bloqueo_registro` (bloqueo pesimista) | Se reemplaza por el bloqueo de fila dentro de la transacción y la verificación de versión. |
| `tb_auditoria` | Se conserva como historial de solo lectura. |

Fuente: Elaboración propia a partir del análisis del sistema MSC.Security (2026).

**Carga inicial sin cambios visibles.** En la migración, cada permiso vigente de `tb_usuario_sistema_perfil` se carga como permiso de origen `MIGRADO`, con fecha de término igual a la primera recertificación. El día del cambio la proyección es idéntica a lo que los satélites ven hoy, y la prueba de regresión debe mostrar **cero diferencias**. Después, TI asigna perfiles de forma progresiva y retira los permisos migrados que quedan cubiertos.

# Bloque 5 · Datos

**Diapositivas 11 y 12 · Informe, secciones 7 y 8 · Criterio 2.1.4 (10 pts).**

## Qué Dominar

- Que el modelo **parte del legado**: hoy a cada persona se le asignan los permisos de cada aplicación uno por uno; lo que faltaba era el **perfil**, un paquete de permisos de varias aplicaciones que se asigna de una vez. Un **permiso** es el acceso a una aplicación con una de sus opciones (en el legado, un registro de `tb_perfil`).
- Las tablas principales y para qué sirven: colaborador (desde AD), cuenta (clave en *hash*), perfil con versiones, asignación de perfil, excepción, permiso efectivo (la lista ya calculada) y bitácora.
- Que existen un DER en 3 diagramas, un diccionario de datos y 10 reglas de integridad (RI-01 a RI-10).
- La **regla de propagación**: cada cambio de plantilla crea una versión nueva, y TI elige aplicarla a todos o solo a algunos; los demás quedan en la versión anterior, marcados como desactualizados.
- La **definición de clonación**: se copian los perfiles en la misma versión de la referencia; las excepciones no se copian; si la referencia cambia después, el destino no cambia.
- Por qué Redis es no relacional y qué guarda: una ficha por persona con sus permisos, que se puede rehacer desde SQL.

## Frases Clave

> «SQL Server es la fuente de verdad; Redis es solo una copia rápida que se puede reconstruir.»

> «Nada cambia sin quedar en la bitácora: el cambio y su registro se guardan juntos o no se guarda nada.»

## Preguntas Probables

**¿Por qué este dato es relacional y este otro no?**
Perfiles, asignaciones y excepciones tienen relaciones, reglas de integridad e historial: necesitan SQL Server. La ficha de permisos por persona en Redis siempre se lee completa por una sola clave, sin cruzar tablas, y si se pierde se rehace. Guardarla también en SQL duplicaría datos.

**¿Qué necesidad justifica la tabla de permisos efectivos?**
N-04, no sobrecargar SQL Server, y N-07, responder lo mismo a las 37 aplicaciones. Es lo que leen tanto el endpoint como los procedimientos de siempre.

**¿Por qué los perfiles tienen versiones?**
Porque el cliente quiere poder aplicar un cambio a todos o solo a algunos. Sin versiones, todos los que tienen el perfil tendrían que tener exactamente la misma plantilla.

**¿Qué diferencia hay entre un perfil y un permiso?**
Un permiso es el acceso a una aplicación con una de sus opciones. Un perfil es un paquete de permisos de varias aplicaciones que corresponde a un puesto, por ejemplo «Operador de muelle». Se asignan perfiles, no permisos sueltos; si alguien necesita algo fuera de su perfil, se pide como excepción. El glosario está en la sección 1.1 del informe.

**¿Qué pasa si se clona y después cambia la persona de referencia?**
Nada en el destino: la clonación es una copia en un momento dado y queda registrada en la bitácora.

**¿Por qué no se copian las excepciones al clonar?**
Porque son permisos personales, aprobados para otra persona y con vencimiento. Si el destino necesita uno, se solicita como excepción nueva y pasa por aprobación.

**¿Cómo garantizan que la bitácora no se modifique?**
Con una tabla de SQL Server que solo permite agregar registros (*ledger append-only*). Si la versión del cliente no la tuviera, el plan B es quitar los permisos de modificar y borrar, y agregar un *trigger* que lo impida (RI-06).

**¿Qué pasa con las contraseñas actuales del legado?**
La columna antigua deja de usarse, pero antes debemos confirmar con el cliente que ninguna aplicación la use para autenticar (riesgo R-12).

# Anexos {.sin-numero}

<!-- GUÍA · Anexo A cubre CAL-01 (línea base). Anexo B cubre C-04 (entrevistas).
Fuente del Anexo B: transcripciones de los dos audios del 14-11-2026 (archivos .docx en la carpeta del proyecto).
NUNCA inventar cifras: lo que no se obtuvo queda declarado como pendiente. -->

## Anexo A. Línea Base del Proceso Actual {.sin-numero}

La retroalimentación de la EVA 1 pidió cuantificar el proceso actual para poder demostrar la mejora en la EVA 4 (CAL-01), y el docente indicó que la cuantificación debe incluirse en esta entrega. La siguiente tabla reúne lo que ya está confirmado y declara lo que falta:

| Métrica | Valor | Fuente | Estado |
| --- | --- | --- | --- |
| Usuarios con acceso a las aplicaciones | ~1.000, en dos razones sociales | Informe EVA 1 | Confirmado |
| Aplicaciones dependientes del control de acceso | 37 (94 % del catálogo) | Informe EVA 1 | Confirmado |
| Usuarios que consultan la API en hora punta | ~300 | Entrevista 14-11-2026 | Confirmado |
| Consumidores concurrentes de la API en hora punta | ~600 (cada usuario usa varias aplicaciones) | Entrevista 14-11-2026 | Estimación del cliente |
| Personas que gestionan accesos hoy | 1 operador de soporte | Entrevista 14-11-2026 | Confirmado |
| Frecuencia de la detección de salidas | Cada 10 minutos, comparando con la lista de activos de AD | Entrevista 14-11-2026 | Confirmado |
| Método para replicar accesos | Manual, permiso por permiso («doble ventana») | Informe EVA 1 y entrevista | Confirmado |
| Tiempo medio de un alta de accesos | — | — | **Pendiente** |
| Tiempo medio de una clonación con doble ventana | — | — | **Pendiente** |
| Tiempo medio de una baja manual | — | — | **Pendiente** |
| Permisos asignados por trabajador | — | — | **Pendiente** |
| Tasa de retrabajo (solicitudes corregidas después) | — | — | **Pendiente** |

Fuente: Elaboración propia (2026).

Los tiempos por operación no se obtuvieron en la entrevista. Se proponen dos vías para completarlos antes de la EVA 4, declarando siempre la fuente: (1) pedir al cliente una estimación de cada tiempo; o (2) medirlos sobre el sistema legado con la base demo, cronometrando a integrantes del equipo mientras ejecutan un alta y una clonación con doble ventana.

## Anexo B. Acta de Entrevista con el Cliente {.sin-numero}

Entrevista realizada el 14 de noviembre de 2026 al docente, en su rol de contraparte de MSC Chile, por integrantes del grupo G07. Se registró en dos audios (10 min 18 s y 8 min 39 s), transcritos en los archivos `Transcripcion_Entrevista_Profesor_Estudiantes.docx` y `Transcripcion_Entrevista_Segundo_Audio.docx`. La siguiente tabla resume las respuestas y su efecto en el diseño:

| Tema | Respuesta del cliente | Efecto en el diseño |
| --- | --- | --- |
| Cuantificación | Incluirla en esta entrega y no postergarla. | Anexo A. |
| Autenticación | Usar usuario y contraseña propios: registrar la aplicación en Entra ID es complejo. | RNF-03 modificado, RF-13 y D-07. |
| Contraseñas | No reproducir la contraseña visible ni la construida como «nombre.apellido»; sugirió contraseñas autogeneradas con cambio obligatorio. | RF-16, RNF-10 y D-10 (enlace de un solo uso, aún más robusto). |
| Roles del sistema | Administrador (todo TI, con permisos para todo) y visualizador (cada usuario ve sus permisos); un mismo inicio de sesión dirige a cada página. | Sección 9 y sección 6. |
| Vista personal | No estaba definida; agregarla como requisito «suma». | RF-14 y HU14. |
| Roles de la EVA 2 | Se refieren a los integrantes del equipo; los estándares incluyen el flujo de Git y las convenciones. | Sección 9 (roles del equipo, EST-01 a EST-06). |
| Herramientas | La cuenta de INACAP da acceso a Azure DevOps. | Alternativa al tablero de GitHub (sección 3). |
| Volumen | ~300 usuarios en hora punta consultan la API; considerar el doble porque cada uno usa varias aplicaciones. | Escenario de rendimiento (sección 10). |
| Desvinculaciones | Se compara la lista de activos de AD con la base cada 10 minutos: se crean los nuevos y se desactivan los que faltan. Los despidos son reservados, por lo que no dependen de un aviso. | RF-12 modificado, HU12 y E-04. |
| API de AD | Ofreció levantar una API de prueba con los usuarios activos. | Ambiente de pruebas (sección 5). |
| Reconstrucción | «La pregunta sigue siendo la misma, la respuesta sigue siendo la misma», sobre una base más robusta; evitar copiar permisos uno a uno. | RF-08, proyección y clonación. |
| Pasantes | Pueden tener permisos en otras aplicaciones, pero no en la de permisos. | Rol Colaborador (sección 9). |
| Esquema de la base de datos | Lo enviará; pidió recordárselo por correo. | Riesgos R-02 y R-05. |
| Key Vault | No es posible dar acceso. | Gestión de secretos con gMSA (sección 5). |
| Infraestructura | Granja de servidores IIS con balanceo ARR; base de datos en un servidor con unos 300 GB de RAM. | Vista física (sección 4) y sección 5. |
| Concurrencia | Advirtió el caso de dos personas que dan y quitan un permiso a la vez en servidores distintos; recomendó bloqueo de fila o una cola FIFO en Redis. | Bloqueo de fila (D-09, RI-10) y CP-28. |
| Operadores | Hoy un operador realiza la gestión. | Riesgo R-13. |

Fuente: Elaboración propia a partir de las transcripciones del 14-11-2026 (2026).

**Pendiente del compromiso C-04:** la entrevista con Recursos Humanos para validar el catálogo de perfiles. El docente indicó que tiene pendiente enviar el contacto de RRHH.

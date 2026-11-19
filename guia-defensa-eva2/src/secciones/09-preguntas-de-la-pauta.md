# Preguntas Tipo de la Pauta

La pauta de la EVA 2 trae ejemplos de las preguntas que puede hacer el docente. Estas son respuestas modelo; adapten el ejemplo a su propio bloque.

## Sobre las Decisiones

**¿Por qué eligieron esta arquitectura y qué alternativa descartaron?**
Una aplicación ordenada por módulos, en varios servidores IIS, con SQL Server como fuente de verdad y Redis como caché. Descartamos los microservicios por el tamaño del equipo, y la caché dentro de cada servidor porque con varios servidores quedaría desactualizada.

**¿Por qué este dato es relacional o no relacional?**
Lo que tiene reglas, relaciones e historial va en SQL Server. La ficha de permisos por persona va en Redis porque siempre se lee completa por una sola clave y se puede reconstruir.

**¿Por qué este control de seguridad es necesario en este proyecto?**
Porque el sistema decide quién entra a las 37 aplicaciones. Un error o un abuso aquí afecta a toda la operación portuaria.

**¿Qué componente soporta la necesidad más importante del cliente?**
El módulo de clonación, con su pantalla de comparar y clonar, que elimina la doble ventana (N-02). Lo complementa la proyección de permisos, que mantiene la misma respuesta para las 37 aplicaciones.

## Sobre la Trazabilidad

**¿Qué requerimiento cubre esta pantalla?**
Revisen la tabla de pantallas del bloque 4. Ejemplo: «Mis permisos» cubre RF-14 (historia HU14, necesidad N-10).

**¿Qué necesidad justifica este componente o esta tabla?**
Cada componente de la vista lógica y cada tabla tienen su necesidad en el informe (sección 4.1 y matriz de la sección 11.1). Ejemplo: la tabla de excepciones atiende N-05, casos especiales sin alterar el cargo.

**¿Qué prueba valida este comportamiento?**
La matriz de trazabilidad (sección 11.1) da la prueba de cada requisito, y el catálogo de casos borde (sección 4.5), la de cada caso borde.

**¿Qué ocurre si eliminamos esta funcionalidad?**
Se responde con la necesidad que deja de cubrirse. Ejemplo: sin la sincronización con AD, quien sale de la empresa conservaría sus accesos hasta que alguien de TI se entere, y en la empresa los despidos se manejan con reserva.

## Si Preguntan por la IA

La pauta permite usar IA como apoyo, pero exige que todos entiendan y puedan justificar lo diseñado. La respuesta honesta es: «La usamos para apoyar la redacción y revisar el diseño; las decisiones las tomamos como equipo a partir de la entrevista con el cliente, y cada uno puede justificarlas». Para que esa frase sea cierta, hay que estudiar esta guía.

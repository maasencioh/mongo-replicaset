# MongoDB replica-set
> Tutorial replica set para MongoDB

  - [Introducción](#introduccion)
  - [Configuración de replica](#configuracion-de-replica)
  - [Recuperación de fallos](#recuperacion-de-fallos)
  - [Elección del primario](#eleccion-del-primario)
  - [Configuración de escritura](#configuracion-de-escritura)

## Introducción

La replicación permite la redundancia de datos y incrementa la disponibilidad de los datos. Con múltiples copias de datos en distintos servidores de bases de datos, replicación proporciona un nivel de tolerancia a fallos en contra de la pérdida de un solo servidor de base de datos.

En algunos casos, la replicación puede ofrecer una mayor capacidad de lectura ya que los clientes pueden enviar las operaciones de lectura a diferentes servidores. Mantener copias de los datos en diferentes centros de datos puede aumentar localidad de datos y la disponibilidad de las aplicaciones distribuidas. También puede mantener copias adicionales para fines dedicados, tales como la recuperación de desastres, la presentación de informes, o de copia de seguridad.

En un grupo de replicación (replica set) existen dos tipos de nodos: primarios y secundarios. Solo puede haber un servidor primario, mientras que pueden haber varios servidores secundarios e incluso árbitros. La diferencia entre un servidor primario y un secundario es que únicamente se puede escribir en el primario, el cual es replicado en los demás servidores secundarios. Al igual, la diferencia entre en un servidor secundario y un árbitro, es que el árbitro no puede almacenar información, pero al igual que el secundario puede votar para la elección del primario.

La replicación en MongoDB es basada en sentencias (statement-based), por lo que en vez de enviar a los otros servidores un cambio binario en la base de datos o colección, envía que comando se ejecutó, haciendole un parsing para que sea más optimo y atómico.

Por defecto, cuando el servidor primario no se comunica con los demás en 10 segundos, se entra en una etapa de elección de un nuevo primario, donde el primer servidor secundario que solicite la votación y reciba la "mayoría" de votos se vuelve el primario. Para que haya una elección debe haber un "consenso" por mayoría de los servidores en el replica set.

## Configuración de replica


## Recuperación de fallos


## Elección del primario


## Configuración de escritura

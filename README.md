# MongoDB replica-set
> Tutorial replica set para MongoDB

  - [Introducción](#introduccion)
  - [Configuración de replica](#configuracion-de-replica)
  - [Elección del primario](#eleccion-del-primario)
  - [Recuperación de fallos](#recuperacion-de-fallos)
  - [Configuración de escritura](#configuracion-de-escritura)

## Introducción

La replicación permite la redundancia de datos y incrementa la disponibilidad de los datos. Con múltiples copias de datos en distintos servidores de bases de datos, replicación proporciona un nivel de tolerancia a fallos en contra de la pérdida de un solo servidor de base de datos.

En algunos casos, la replicación puede ofrecer una mayor capacidad de lectura ya que los clientes pueden enviar las operaciones de lectura a diferentes servidores. Mantener copias de los datos en diferentes centros de datos puede aumentar localidad de datos y la disponibilidad de las aplicaciones distribuidas. También puede mantener copias adicionales para fines dedicados, tales como la recuperación de desastres, la presentación de informes, o de copia de seguridad.

En un grupo de replicación (replica set) existen dos tipos de nodos: primarios y secundarios. Solo puede haber un servidor primario, mientras que pueden haber varios servidores secundarios e incluso árbitros. La diferencia entre un servidor primario y un secundario es que únicamente se puede escribir en el primario, el cual es replicado en los demás servidores secundarios. Al igual, la diferencia entre en un servidor secundario y un árbitro, es que el árbitro no puede almacenar información, pero al igual que el secundario puede votar para la elección del primario.

La replicación en MongoDB es basada en sentencias (statement-based), por lo que en vez de enviar a los otros servidores un cambio binario en la base de datos o colección, envía que comando se ejecutó, haciendole un parsing para que sea más optimo y atómico.

Por defecto, cuando el servidor primario no se comunica con los demás en 10 segundos, se entra en una etapa de elección de un nuevo primario, donde el primer servidor secundario que solicite la votación y reciba la "mayoría" de votos se vuelve el primario. Para que haya una elección debe haber un "consenso" por mayoría de los servidores en el replica set.

## Configuración de replica

Un grupo de 3 servidores proveen una buena redundancia para la mayoría de casos, además de aplicar una buena práctica de crear un número impar en el grupo, ya que esto asegura unas elecciones más sencillas. También se recomienda aislar físicamente lo mejor posible cada servidor, preferiblemente en diferentes datacenters en diferentes partes del mundo, de manera que sea más dificil que fallen todos al mismo tiempo. En caso de tener los servidores de esta forma se recomienda primero realizar pruebas de conectividad entre los servidores, configuración del firewall y pruebas del DNS.

Para iniciar se deben iniciar los demonios con el nombre del replica set, este nombre debe ser único, así que si la aplicación usa varios replica sets, cada uno de estos debe ser diferente.

```
$ mkdir data
$ mkdir data/z1
$ mkdir data/z2
$ mkdir data/z3
$ mongod --port 27001 --dbpath data/z1 --replSet z
```

Luego desde la consola `mongo` se debe instanciar el replica set

```
> rs.initiate()
```

Para ver la configuración del replica set

```
> rs.conf()
```

Vamos a crear los otros 2 servidores en consolas aparte

```
$ mongod --port 27002 --dbpath data/z2 --replSet z
$ mongod --port 27003 --dbpath data/z3 --replSet z
```

Luego se agregarían al replica set, para esto __se deben agregar desde el primario__ de la siguiente forma

```
> rs.add("192.168.0.4:27002")
> rs.add("192.168.0.4:27003")
> rs.status()
> db.isMaster()
```

Para agregar un árbitro se instancia primero el servidor

```
$ mkdir data/arb
$ mongod --port 27004 --dbpath data/arb --replSet z
```

Luego se agrega de forma similar que los secundarios

```
> rs.addArb("192.168.0.4:27004")
> rs.status()
```

Dado que para este caso no se va a necesitar un árbitro vamos a eliminarlo de la siguiente forma

```
> rs.remove("192.168.0.4:27004")
```

## Elección del primario

## Recuperación de fallos

## Configuración de escritura

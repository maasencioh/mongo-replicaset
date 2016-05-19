# MongoDB replica-set
> Tutorial replica set para MongoDB

  - [Introducción](#introduccion)
  - [Configuración de replica](#configuracion-de-replica)
  - [Elección del primario](#eleccion-del-primario)
  - [Configuración de escritura y lectura](#configuracion-de-escritura-y-lectura)
  - [Recuperación de fallos](#recuperacion-de-fallos)

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

¿Qué pasa cuando el servidor primario se cae? Esto activa una votación en los otros servidores, en el caso de prueba salió como ganador el del puerto 27002, como se ve a continuación.

```
z:SECONDARY> db.isMaster()
{
	"hosts" : [
		"192.168.0.4:27001",
		"192.168.0.4:27002",
		"192.168.0.4:27003"
	],
	"setName" : "z",
	"setVersion" : 5,
	"ismaster" : false,
	"secondary" : true,
	"primary" : "192.168.0.4:27002",
	"me" : "192.168.0.4:27001",
	"maxBsonObjectSize" : 16777216,
	"maxMessageSizeBytes" : 48000000,
	"maxWriteBatchSize" : 1000,
	"localTime" : ISODate("2016-05-19T13:33:37.272Z"),
	"maxWireVersion" : 4,
	"minWireVersion" : 0,
	"ok" : 1
}
```

Las elecciones por defecto son estocásticas, ya que la selección del nuevo primario será por el primero en responder, pero si por algún motivo se desea específicamente un orden de elección se modifica la prioridad de estos

```
z:SECONDARY> rs.config()
{
	"_id" : "z",
	"version" : 5,
	"protocolVersion" : NumberLong(1),
	"members" : [
		{
			"_id" : 0,
			"host" : "192.168.0.4:27001",
			"arbiterOnly" : false,
			"buildIndexes" : true,
			"hidden" : false,
			"priority" : 1,
			"tags" : {

			},
			"slaveDelay" : NumberLong(0),
			"votes" : 1
		},
		{
			"_id" : 1,
			"host" : "192.168.0.4:27002",
			"arbiterOnly" : false,
			"buildIndexes" : true,
			"hidden" : false,
			"priority" : 1,
			"tags" : {

			},
			"slaveDelay" : NumberLong(0),
			"votes" : 1
		},
		{
			"_id" : 2,
			"host" : "192.168.0.4:27003",
			"arbiterOnly" : false,
			"buildIndexes" : true,
			"hidden" : false,
			"priority" : 1,
			"tags" : {

			},
			"slaveDelay" : NumberLong(0),
			"votes" : 1
		}
	],
	"settings" : {
		"chainingAllowed" : true,
		"heartbeatIntervalMillis" : 2000,
		"heartbeatTimeoutSecs" : 10,
		"electionTimeoutMillis" : 10000,
		"getLastErrorModes" : {

		},
		"getLastErrorDefaults" : {
			"w" : 1,
			"wtimeout" : 0
		},
		"replicaSetId" : ObjectId("573db7db8f1de2c90c7910b4")
	}
}
```

Cada objeto en la lista `members` tiene un elemento `priority`, el cual especifica cual es la mejor opción para la elección del primario en unas votaciones

```
> var conf = rs.config()
> conf.members[0].priority = 2
> conf.members[2].priority = 0
> rs.reconfig(conf)
```

Esto no solo hace que el primer servidor sea el preferido, si no que además prohibe que el tercer servidor pueda volverse primario. También el comando `rs.reconfig()` hace que el primario renuncie momentaneamente, por lo que se inicia una votación, esto puede durar entre 10 y 20 segundos.

## Configuración de escritura y lectura

Por defecto todas las operaciones se hacen sobre el primario, tanto las de lectura como las de escritura. Para leer desde un secundario solo es necesario hacer `rs.slaveOk()`, veamos un ejemplo

```
z:PRIMARY> use test
z:PRIMARY> db.beer.insert({"looks", "good"})
 ...
z:SECONDARY> db.beer.findOne()
Error: listCollections failed: { "ok" : 0, "errmsg" : "not master and slaveOk=false", "code" : 13435 }
z:SECONDARY> rs.slaveOk()
z:SECONDARY> db.beer.findOne()
{ "_id" : ObjectId("573dcb52cc6e6ab7071ded0b"), "looks" : "good" }
```

### Read preferences

Ya habilitada la lectura, se puede configurar la forma en la que un driver se conectará al set, teniendo como posibles opciones

  - __primary__: Tratará de leer del primario, nunca de algún secundario
  - __primaryPreferred__: Tratará de leer del primario, si falla toma algún secundario
  - __secondary__: Tratará de leer de algún secundario, nunca del primario
  - __secondaryPreferred__: Tratará de leer del secundario, si falla tomará del primario
  - __nearest__: Busca el servidor más cercano, sin importar si es primario o secundario

Un ejemplo es

```js
var Db = require('mongodb').Db,
    ReplSet = require('mongodb').ReplSet,
    Server = require('mongodb').Server,
    ReadPreference = require('mongodb').ReadPreference,
    test = require('assert');
// Connect using ReplSet
var server = new Server('localhost', 27017);
var db = new Db('test', new ReplSet([server]));
db.open(function(err, db) {
    test.equal(null, err);
    // Perform a read
    var cursor = db.collection('t').find({});
    cursor.setReadPreference(ReadPreference.PRIMARY);
    cursor.toArray(function(err, docs) {
        test.equal(null, err);
        db.close();
    });
});
```

### Write concerns

Aunque solo se puede escribir en el primario, hay formas de controlar lo que se escribe en los secundarios, para esto se usa la siguiente sintaxis

```
> db.collection.insert({ ... }, { w: <value>, j: <boolean>, wtimeout: <number> })
```

  - __w__: solicita confirmación sobre la cantidad de nodos propagados
  - __j__: solicita confirmación sobre la escritura en el journal de la mayoría de nodos
  - __wtimeout__: especifíca el máximo tiempo de respuesta en milisegundos, para un valor de w mayor a 1

Para los posibles valores de `w` se tienen las siguientes posibilidades

  - `w: 0` -> solo se genera reporte de errores en sockets o redes
  - `w: 1` -> confirma la escritura en el servidor primario
  - `w`: _n_ -> confirma la escritura en el servidor primario y en _n - 1_ servidores secundarios, esto se puede usar para saber si se replicó en todos los servidores
  - `w: "majority"` -> confirma la escritura en la mayoría de servidores del replica set (para la versión 3.2 obliga a que `j: true`)

Un ejemplo de esto sería:

```
> db.beer.insert({"hi":true}, {w:'majority'})
```

Para poner estos valores por defecto se usa un viejo truco

```js
> var cfg = rs.conf()
> cfg.settings = {}
> cfg.settings.getLastErrorDefaults = { w: "majority", wtimeout: 5000 }
> rs.reconfig(cfg)
```

## Recuperación de fallos

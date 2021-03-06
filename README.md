# umcloud_pdns
## README
## @Author: d1cor / Diego Córdoba
## PowerDNS - Mysql Backend - PowerAdmin


El esquema levanta un nodo de DB, por defecto 2 nodos web / backend, y un nodo frontend con haproxy.

Los nodos web instalan:
  - Servidor DNS PowerDNS
  - Servidor web Apache
  - WUI PowerAdmin para administrar el powerdns

Los nodos web brindan servicio de:
  - http (PowerAdmin) para administrar via web el dns.
  - dns (powerdns) para resolver zonas

## Backends

La cantidad de backends que van a lanzarse se configura en el Makefile:
 
```NWEB = 2   #levantara dos backends*```

Dependiendo de la cantidad de backends puede que haya necesidad de modificar el tiempo de espera del restart del haproxy en el front-end.

Al levantar el frontend se genera automáticamente el archivo de configuracion del haproxy con tantas líneas "*server*" como cantidad de backends se hayan configurado en el Makefile.

El **HAproxy** fue configurado para hacer balanceo mediante la dirección ip de origen, puesto que la aplicación web requiere gestionar sesiones de usuario, y el balanceo por carga no mantiene las sesiones con cada uno de los backends.


## Autenticación a la DB

Tanto el powerdns con su backend-mysql, como el WUI poweradmin, acceden a la DB utilizando las mismas credenciales.

El *nombre de usuario* de conexión y el *nombre de la base de datos* se especifican en el Makefile.
La *contraseña* se genera automáticamente como valor random de 16 caracteres en el Makefile.

Todos estos datos, a la hora del deploy, se guardan en el consul en un folder de *Key/value*.

Los datos permanencen en el consul. Si se re-deployea algun equipo, lee los mismos datos para poder autenticarse.

En el caso de eliminar los datos o el folder en el consul, al deployar nuevamente se generarán.

## Autenticación a la interfaz web del PowerAdmin:
- User: admin
- Pass: (sin password)
Esta info se puede cambiar desde la interfaz de configuración del poweradmin.

## Servicios DNS

Los nodos web-N levantados son, a su vez, servidores de DNS replicados que leen sus zonas desde la base de datos mediante el backend-mysql de PowerDNS.

Para poder consultar los nombres de una zona en estos servicios es requerido agregar al default secgroup el puerto udp53 a las instancias del usuario.

Las consultas pueden realizarse desde los mismos nodos sin inconvenientes.

## Datos pre-cargados
El sistema ya tiene cargada la zona *"mizona.com"*, de modo que los nodos web-N ya podrán resolver la zona ni bien inicie el sistema:

```dig @prod-d1cor-web-1.node.cloud.um.edu.ar mizona.com```

## Instalacion

_openstack:_

```
git clone https://github.com/d1cor/umcloud_pdns

cd umcloud_pdns/

make deploy ENV=prod
```

## TODO
- Ver la forma de eliminar el "sleep 120" del fe.yaml:31 para que no haya problemas al restart del servicio haproxy.



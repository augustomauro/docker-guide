### Qué es un Contenedor?

Un contenedor puede tener un sistema operativo, un software backend, un software frontend, 
una base de datos, un archivo de entorno .env, entre otros.
Los contenedores son portables, por lo cual no va a ser necesario estar instalando todo esto
nuevamente en distintos desarrolladores.

Un despliegue sin contenedores puede generar muchos conflictos. Por ello lo ideal es generar
una IMAGEN unica la cual se utilice para distribuir el software.

### Virtual Machine vs Docker:

    Normalmente el concepto de Vistualizacion se basa en 3 capas: HARDWARE - KERNEL - APLICACIONES.
    El kernel se encarga de generar la comunicacion entre las aplicaciones y el hardware.
    Una maquina virtual comprime 2 de ellas, las aplicaciones + el kernel. Esto hace que una VM 
    sea muy pesada.
    Por el contrario, un contenedor solo se virtualizan las aplicaciones, utilizando el kernel propio 
    del SO en el que se ejecuta.

## Para-Virtualization:
    Se intenta entregar el total de los recursos de hardware al software cliente.
## Partial-Virtualization:
    Se entrega solo una parte del hardware al software cliente.
## Total-Virtualization:
    Se virtualiza todo el hardware al software cliente.

Respecto a estas virtualizaciones, Docker es muy superior en cuanto a rendimiento al utilizar 
el mismo kernel donde se ejecuta y ademas inicia casi de forma instantanea.

### Donde se almacenan?

    Repositorio de contenedores

        Privados
        Publicos (DockerHub)
            NodeJS, Phyton, MySQL, Golang, etc

### Qué es docker Desktop?

    - Una virtual machine   -> corre linux
                            -> ejecuta containers
    - Permite acceder al sistema de archivos
    - Acceder a la red
    - Docker compose, CLI (Command ine Interface) y otras
    - Corre nativo en windows con WSL2 (Windows Subsystem for Linux)

----
## INSTALACION:

    1) Instalar Docker Desktop desde https://www.docker.com/.

## COMANDOS:

    - Visualizar en Docker Hub las librerias que se pueden instalar, ejemplo:

        docker pull node (descarga la ultima version disponible de node)
        docker pull node:14 (descargar solo la version 18 de node)

    - Visualizar imagenes descargadas:

        docker images

        REPOSITORY  TAG     IMAGE ID        CREATED     SIZE
        node        latest  149a0b692521    5 days ago  1.62GB
        node        14      a158d3b9b4e3    2 years ago 1.35GB

    - Descargar MySQL:

        docker pull mysql

        Importante: para chips M1 de Apple es probable que de un error y se soluciona
        escribiendo:
        
        docker pull --platform linux/x86_64 mysql

    - Eliminar imagenes:

        docker image rm node:18 (elimina solo la version 18)
        docker image rm node:14 (elimina solo la version 14)

## CREAR CONTENEDORES:

    Para crear un contenedor necesitamos primero, una imagen.

    docker pull mongo
    docker create mongo (al final devolvera un hash id del contenedor)

    Esto hace que ahora la imagen mongo se encuentre en uso dentro de un contenedor.

    docker create node:14 (crea un contenedor para la imagen node version 14)
    docker create mysql (crea un contenedor para la imagen mysql latest)

## INICIAR CONTENEDORES:

    docker start <hash id del contenedor>

## PARAR CONTENEDORES:

    docker stop <hash id del contenedor>

## VER CONTENEDORES:

    docker ps (muestra solo los que estan ejecutando)
    docker ps -a (muestra todos los contenedores)

    CONTAINER ID    IMAGE   COMMAND                 CREATED         STATUS              PORTS       NAMES
    a1f6dbc2c00d    mongo   "docker-entrypoint.."   15 seconds ago  Up to 19 seconds    27017/tcp   silly_kapitsa

    Nota: se puede inciar solo con el CONTAINER ID sin ingresar todo el hash completo, ejemplo: 

    docker start a1f6dbc2c00d

## ELIMMINAR CONTENEDORES:

    docker rm silly_kapitsa

## CAMBIAR NOMBRE IMAGEN AL CREAR CONTENEDOR:

    docker create --name monguito mongo

    Nota: hace referencia al nombre que queremos y sobre la imagen a utilizar

## CAMBIAR PUERTO IMAGEN AL CREAR CONTENEDOR:

    Nota: el puerto por defecto para mongodb es 27017, sin embargo no podemos ingresar directamente
    desde afuera. Para ello es necesario MAPEAR los puertos. Si tuvieramos 2 aplicaciones NodeJS corriendo
    en el puerto 3000 conectandose al 27017 de mongo, deberiamos mapear el puerto 3000 del host (anfitrion)
    a la 1er aplicacion y asi si recibimos peticion en el puerto 3001 del host, redireccionarla a la 2da aplicacion.

    docker create -p27017:27017 --name monguito mongo

    Observacion: el parametro "-p27017" refiere al puerto IP del host y el segundo "27017" al del contenedor.
    Ahora en el comando "docker ps" veremos la tabla que nos indica sobre el campo PORTS al iniciarlo:

    0.0.0.0:27017->27017/tcp

    Importante: si solo se agrega el parametro "-p27017" sin indicarle a donde mapear, lo que hara es
    mapear al 27017 del contenedor pero al host le asignara un puerto alto por encima del 50000, ejemplo:

    0.0.0.0:51445->27017/tcp

## VER LOGS SOBRE CONTENEDORES ACTIVOS:

    docker logs monguito (devuelve todos los logs y vuelve al terminal)

    Nota: si queremos que no vuelva al terminal y permanezca en escucha, debemos agregar
    un parametro "--follow". Para salir y volver de este modo con la tecla CTRL + C.

    docker logs --follow

## COMANDO COMBINADO PULL + CREATE + START:

    1) Parar contenedor:                    docker stop monguito
    2) Eliminar contenedor:                 docker rm monguito
    3) Eliminar imagen:                     docker image rm mongo
    4) Descarga imagen + corre contenedor:  docker run -d mongo (modo detach)
    5) Correr todo junto:                   docker run --name monguito -p27017:27017 -d mongo

----

### CONECTAR CON UNA APLICACION

Para poder conectar con una aplicacion real, en nuestro ejemplo en NodeJS junto a mongoDB, es necesario buscar la referencia
de variables necesarias para poder utilizar dicha base de datos. Para esto en Docker Hub en el buscador ingresamos "mongo",
luego clic en el resultado y hacer scroll hasta la seccion de "Environment Variables". Alli se puede obserbar de acuerdo al ejemplo
indicado mas abajo en NodeJS, que al menos son necesarios 2 parametros:

    MONGO_INITDB_ROOT_USERNAME, MONGO_INITDB_ROOT_PASSWORD

Entonces al crear una imagen conectada a mongo, se debe definir cada uno de estos valores a gusto, para poder luego utilizarse
la base a traves de una app, anteponiendo el parametro "-e" de environment:

    docker create -p27017:27017 --name monguito -e MONGO_INITDB_ROOT_USERNAME=nico -e MONGO_INITDB_ROOT_PASSWORD=password mongo

## Descargar la aplicacion en NodeJS:

    https://github.com/nschurmann/mongoapp-curso-docker

## Descomprimir archivo, instalar dependencias y correr aplicacion en host:

    npm install
    node index.js

    IMPORTANTE: si se ejecuta la app en el host seguramente haya que cambiar la URI de conexion
    en index.js de la siguiente forma:

    mongoose.connect('mongodb://nico:password@localhost:27017/miapp?authSource=admin')

    de lo contrario dara un error al no encontrar expuesto el nombre "monguito" fuera del docker.

Una vez iniciada la app de forma correcta, si vamos a la ruta "localhost:3000/" veremos un array vacio.
Si queremos crear un recurso, iremos a la ruta "localhost:3000/crear" y podemos verlo
en la ruta "localhost:3000/" nuevamente.

----

### DOCKERIZAR LA APLICACION

En este paso intentaremos insertar la app en un archivo docker en lugar de ejecutarlo de forma local.
Para esto en el raiz de nuestra aplicacion, debemos crear un nuevo archivo llamado "Dockerfile" sin extension.
Debemos indicar el nombre de la imagen, la carpeta donde guardar nuestra app dentro del contenedor, puertos a exponer, etc:

Dockerfile:

    FROM node:18

    RUN mkdir -p /home/app

    COPY . /home/app

    EXPOSE 3000

    CMD ["node", "/home/app/index.js"]

Nota: A vistas de que pueden haber multiples aplicaciones que deban conectarse dentro del contenedor, lo que se hace
es una red para que estas se comuniquen entre si, por ejemplo "mired" y asi mongodb puede conectarse a ésta, como tambien
la app de node.

Para listar las redes disponibles en docker:

    docker network ls

Para crear una nueva red:

    docker network create mired

Para eliminar una red:

    docker network rm mired

Para ejecutar el docker con la configuracion nueva dentro del proyecto nodeJS:

    docker build -t miapp:1 .

    Nota: el parametro "-t" indica que va una etiqueta, es decir el nombre de la app "miapp" seguida
    de 2 puntos y mas datos de la etiqueta como la version, en este caso "1".
    EL punto que sigue es la ruta completa donde se encuentra la aplicacion nodeJS, en este caso
    indica que este comando se esta ejecutando alli mismo.

Luego de construir el contenedor, si listamos mediante "docker images" veremos:

REPOSITORY      TAG         IMAGE ID        CREATED         SIZE
miapp           1           ac22c893684d    1 second ago    1.59GB

Ahora debemos eliminar el contenedor anterior llamado "monguito" para poder asignarle la nueva red
creada anteriormente:

    docker stop monguito
    docker rm monguito
    docker create -p27017:27017 --name monguito --network mired -e MONGO_INITDB_ROOT_USERNAME=nico -e MONGO_INITDB_ROOT_PASSWORD=password mongo
    
Ahora creamos el nombre para la app NodeJS y exponemos su puerto dentro del contenedor, junto con la conexion a la misma red:
    
    docker create -p3000:3000 --name chanchito --network mired miapp:1

Finalmente iniciamos ambas aplicaciones dockerizadas:

    docker start monguito
    docker start chanchito

Y testeamos ambas rutas como se mostro anteriormente.

Si queremos podemos ver los logs de consola que devuelve la app nodeJS mediante:

    docker logs chanchito

---

## Repaso de operaciones por cada contenedor, hasta ahora:

1) Descagar imagen
2) Crear una red
3) Crear contenedor
    a) Asignar puertos
    b) Asignar nombres
    c) Asignar variables de entorno
    d) Especificar red
    e) Indicar Imagen:etiqueta

Se puede automatizar todo esto mediante "docker-compose.yml":

version: "3.9"
services:
  chanchito:
    build: .        # la ruta en la que nos encontramos para la app
    ports:
      - "3000:3000" # puertos anfitrion:cliente
    links:
      - monguito    # el nombre del contenedor que vamos a mapear para que use este servicio
  monguito:
    image: mongo    # la imagen a utilizar para este servicio
    ports:
      - "27017:27017"   # puertos anfitrion:cliente
    environment:
      - MONGO_INITDB_ROOT_USERNAME=nico
      - MONGO_INITDB_ROOT_PASSWORD=password
    volumes:
      - mongo-data:/data/db
      # mysql -> /var/lib/mysql
      # postgres -> /var/lib/postgresql/data

volumes:
  mongo-data:

Importante: RESPETAR LAS TABULACIONES

Para construir los contenedores lo haremos mediante:

    docker composer up

Esto reemplaza todo los pasos anteriores.

Si listamos las imagenes ahora, veremos una nueva que se creo a traves de docker compose:

    mongoapp-curso-docker-main-chanchito

Si queremos ver si se crearon nuevos contenedores mediante "docker ps -a" veremos que aparecen
2 nuevos contenedores que se crearon a traves de "docker compose up". Si detenemos y volvemos a ejecutar
docker composer up, se reutilizaran los contenedores e imagenes recientemente creadas ya.

Para limpiar todo lo que creo "docker compose up", utilizaremos el comando "docker compose down".
Este comando eliminara todo lo relacionado.

----

### VOLUMES

Imaginemos que queremos que los datos en la base de datos persistan o bien los cambios que realizamos
en la aplicacion se mantengan. Utilizando esto como se hizo hasta ahora no sirve, ya que se perderian datos.
Para esto existen los "Volumes" que van a permitir guardar datos montando las carpetas en el sistema
anfitrion.

Tipos:

- Anonimo: Donde solo indicas la ruta. El problema es que no se puede referenciar apra que lo utilice otro contenedor.
- De anfitrion o host: nosotros decidimos que carpeta montar y donde.
- Nombrado: es como el anonimo, pero se puede referenciar para reutilziarlo para otros contenedores.

En el docker-composer.yml se encuetra representado por:

    volumes:
      - mongo-data:/data/db # la ruta donde se montara la db de mongo en el volumen abajo indicado
      # mysql -> /var/lib/mysql # lo mismo para mysql
      # postgres -> /var/lib/postgresql/data    # lo mismo para postgres

    volumes:
      mongo-data:   # el nombre del volumen

Si volvemos a parar el contenedor, lo eliminamos mediante "docker compose down" y volvemos a crear
mediante "docker compose up" veremos que los datos anteriores aun persisten en la base por haber declarado
el uso de volumes.

----

### DOCKERIZAR PARA DESARROLLO

Si observamos en el proyecto, veremos el archivo Dockerfile.dev y docker-composer-dev.yml
que sirve para poder trabajar en desarrollo con contenedores.

Dockerfile.dev:

    FROM node:18

    RUN npm i -g nodemon
    RUN mkdir -p /home/app

    WORKDIR /home/app

    EXPOSE 3000

    CMD ["nodemon", "index.js"]

ddocker-composer-dev.yml:

version: "3.9"
services:
  chanchito:
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    links:
      - monguito
    volumes:
      - .:/home/app
  monguito:
    image: mongo
    ports:
      - "27017:27017"
    environment:
      - MONGO_INITDB_ROOT_USERNAME=nico
      - MONGO_INITDB_ROOT_PASSWORD=password
    volumes:
      - mongo-data:/data/db
      # mysql -> /var/lib/mysql
      # postgres -> /var/lib/postgresql/data

volumes:
  mongo-data:


Para poder componer esta version lo haremos de la siguiente manera:

    docker compose -f docker-compose-dev.yml up

Esto hara lo mismo que antes, pero estariamos trabajando en un entorno de desarrollo,
lo que significa que si modificamos el codigo de la app se vera reflejado de forma inmediata
esos cambios.

----
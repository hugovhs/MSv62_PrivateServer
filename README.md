# MapleStory v62 Docker Environment

Entorno Docker para ejecutar un servidor privado de MapleStory v62 basado en
ThePack 82. Su objetivo es simplificar la instalación del servidor y evitar la
configuración manual de Java y MySQL en el sistema anfitrión.

Esta guía adapta las instrucciones del [post original de thepack en
RaGEZONE](https://forum.ragezone.com/threads/v62-and-docker.1150928/), publicado
en julio de 2018. Las capturas del post se han sustituido por pasos y comandos.

## Componentes

El proyecto construye dos imágenes:

- **`odinms_mysql`**: MySQL con la base de datos inicial cargada desde
  `odinms_mysql/thepack.sql`.
- **`odinms`**: el servidor Java de MapleStory, que se conecta al contenedor de
  MySQL. Incluye el código, las bibliotecas y los datos del servidor.

Los Dockerfiles actuales usan `mysql:5.7` y `eclipse-temurin:8-jdk`.
Docker se encarga de obtener estas dependencias; no es necesario instalar Java
o MySQL por separado en el anfitrión.

## Requisitos

- Docker instalado y en ejecución, con permisos para ejecutar sus comandos.
- Git para clonar el repositorio y una terminal Bash para los scripts.
- Conexión a Internet para descargar las imágenes base en la primera ejecución.
- En el equipo donde vayas a jugar, un cliente de MapleStory **v62** y un
  ejecutable *localhost* compatible que permita indicar la IP y el puerto del
  servidor. El cliente y ese ejecutable se obtienen por separado.

## Iniciar el servidor

### 1. Clonar el repositorio

```bash
git clone https://github.com/hugovhs/MSv62_PrivateServer.git
cd MSv62_PrivateServer
```

Si ya tienes una copia del proyecto, abre Bash en su directorio raíz.

### 2. Construir y levantar los contenedores

```bash
source run.sh
```

El script construye las dos imágenes, inicia MySQL en segundo plano y abre una
sesión Bash interactiva dentro del contenedor Java, en `/odinms`. También pasa a
ese contenedor la dirección de MySQL para que el servidor pueda conectarse.

La primera ejecución puede tardar mientras se descargan las imágenes base.
Las siguientes pueden aprovechar la caché de Docker.

### 3. Compilar y ejecutar MapleStory

Dentro de la sesión Bash del contenedor Java, ejecuta:

```bash
source compile_and_run.sh
```

Este script compila el código y arranca los servicios de mundo (*world*), inicio
de sesión (*login*) y canales (*channel*). El post original indica que el
arranque tarda aproximadamente un minuto. Comprueba en la terminal que aparece
el servicio de login escuchando en el puerto `8484` y que los canales están en
línea antes de abrir el cliente.

Mantén abierta esta sesión mientras uses el servidor.

## Conectar el cliente

En el equipo con MapleStory v62 instalado, abre el símbolo del sistema de
Windows (`cmd`) y entra en la carpeta que contiene el ejecutable *localhost*.
Ejecuta:

```bat
localhost.exe 127.0.0.1 8484
```

Sustituye `localhost.exe` por el nombre real de tu ejecutable. Usa `127.0.0.1`
si el cliente y Docker se ejecutan en el mismo equipo; si están en equipos
distintos, usa la IP del equipo que ejecuta Docker. Por ejemplo:

```bat
localhost.exe 192.168.1.163 8484
```

El script publica el puerto `8484` para login y los puertos `7575`–`7578` para
los canales, además del `3306` de MySQL. Si conectas desde otro equipo, los
puertos de login y canales deben ser accesibles desde ese equipo.

Estos son los recursos de cliente que cita el post original:

- [Clean v62 localhost](https://forum.ragezone.com/f427/clean-v62-localhost-1068520/).
- [Archivo de clientes de MapleStory (GMSDLReborn)](https://forum.ragezone.com/f425/maplestory-client-archive-gmsdlreborn-1101897/).

## Detener el entorno

Para terminar, ejecuta `exit` en la sesión Bash del contenedor Java. El script
`run.sh` elimina ese contenedor al salir y después detiene y elimina el
contenedor MySQL que creó.

El script actual no configura un volumen persistente para MySQL: los cambios
de la base de datos no se conservarán en una nueva ejecución. Cada arranque
crea un contenedor nuevo con los datos iniciales de `thepack.sql`.

## Nota sobre Wine

El post original proponía como trabajo futuro ejecutar también el cliente en
un contenedor con Wine para prescindir de Windows. Esa idea no forma parte de
los pasos de instalación anteriores; este repositorio contiene los dos
contenedores del servidor.

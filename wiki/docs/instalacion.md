# Instalación

IndexMic puede correr facilmente desde cualquier sistema gracias su uso de
Docker y Java. Sin embargo, este manual esta enfocado en sistemas operativos
Linux (especificamente basados en Debian/Ubuntu). El nombre de los paquetes
y librerías necesarios puede cambiar según su distribución.

## Dependencias

Para poder instalar y correr IndexMic de forma correcta se requieren
(o recomiendan) los siguientes programas:

- `openjdk-11-jre`: OpenJDK versión 11.
- `docker-ce`, `docker-ce-cli` y `containerd.io`:  Docker y sus dependencias.
- `wget`: Permite descargar binarios necesarios.
- `git` (opcional): Facilita descargar IndexMic y actualizarlo en el futuro.

Para instalar estas dependencias desde los repositorios de Ubuntu o Debian se
puede manejar desde `apt`.

```bash
sudo apt install \
  openjdk-11-jre \
  wget \
  git
```

Esto instalará las versiones del los repositorios oficiales.

## Instalar Docker

Para instalar Docker desde los repositorios de Docker se deben
seguir los siguientes pasos:

### Desinstalar versiones anteriores

```bash
sudo apt-get remove docker docker-engine docker.io containerd runc
```

No hay problema si se arroja un error, esto simplemente desinstala
versiones anterior si estan instaladas.

### Configurar el repositorio

Para configurar los repositorios se deben seguir los pasos según el
sistema operativo que se este usando.

```
sudo apt-get update
```
```
sudo apt-get install \
  ca-certificates \
  curl \
  gnupg \
  lsb-release
```

#### Ubuntu

**Agregar la llave GPG oficial**:
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```
**Agregar el repositorio a las fuentes de `apt`**:
```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
**Instalar Docker**:
```
sudo apt-get update
```
```
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

#### Debian

**Agregar la llave GPG oficial**:
```
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```
**Agregar el repositorio a las fuentes de `apt`**:
```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
**Instalar Docker**:
```
sudo apt-get update
```
```
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

## Descargar IndexMic

Existen dos métodos de descargar IndexMic. Con `git` o descargando los archivos
con `wget` o `curl`.

### Git

Para descargar con `git` simplemente se debe clonar el repositorio de
[IndexMic](https://gitlab.proyais.com/proyectoais/indexmic) con el siguiente
comando:

```
sudo git clone https://gitlab.proyais.com/proyectoais/indexmic.git /etc/indexmic
```

Esto clonará el repositorio directamente a la carpeta `/etc/indexmic`. Si se desea utilizar
otro directorio se puede especificar modificando el comando.

### Descarga directa

_La descarga directa solo se recomienda cuando se desea un manejo
sin `git`. Esto hará más dificil la actualización futura._

Para descargarlo con `wget` se podrá utilizar el siguiente comando:

```
wget https://gitlab.proyais.com/proyectoais/indexmic/-/archive/master/indexmic-master.zip
```

Esto descargará un archivo comprimido con los contenidos de IndexMic.
Al descomprimir la carpeta deberá mover los archivos a `/etc/indexmic`
para terminar la instalación.

## Instalar IndexMic

Para instalar IndexMic en un servidor como un servicio, se debe ejecutar
el script `install.sh` una vez los archivos este copiados en la carpeta
`/etc/indexmic`.

```
sudo /etc/indexmic/install.sh
```

Esto creará un servicio llamada `indexmic.service` en el servidor. IndexMic
debería poder accederse desde `hostname:8080` una vez acabe de correr el
script.

## Iniciar, reiniciar y detener IndexMic

Por defecto IndexMic inicia cuando el servidor inicie, sin embargo, si
un error evita el inicio de este puede iniciarse con:

```
sudo systemctl start indexmic.service
```

Para reiniciar IndexMic (para aplicar una nueva configuración) se puede
utilizar:

```
sudo systemctl restart indexmic.service
```

Para detener el servicio de IndexMic se puede utilizar:

```
sudo systemctl stop indexmic.service
```

## Actualizar IndexMic

Para actualizar IndexMic a una nueva versión se recomienda hacer un backup
a el archivo de configuraciones `application.yml`. Puede copiar los contenidos
a otro archivo con:

```
sudo cp /etc/indexmic/application.yml application.yml.bak
```

Una vez hecho el backup, puede hacer un pull al repositorio para traerse los nuevos
cambios:

```
cd /etc/indexmic
```
```
git pull
```

Si la nueva versión contienen un nuevo binario de ShinyProxy se recomienda volver
a correr:

```
/etc/indexmic/install.sh
```

### Resolver conflictos

Si la actualización trae cambios a la configuración es posible que se generen
conflictos en el archivo `application.yml`. Git no le permitirá hacer
el pull hasta que haya resuelto los conflictos. En estos casos, simplemente
copie los contenidos de su backup y complete el pull.

Para completar el pull se puede hacer:

```
git add .
```
```
git commit
```

Puede que después de la actualización partes de la interfaz hayan cambiado
por lo cual es importante revisar los cambios hecho de versión en
versión.

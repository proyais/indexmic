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

## Instalar IndexMic

Existen varios métodos para instalar IndexMic, sin embargo, se recomienda
la instalación utilizando `apt`. Esto garantizará las correctas dependencias
para su sistema operativo.

Para descargar el paquete `.deb` se debe ir a el repositorio oficial de
IndexMic en GitHub. Este se puede descargar facilmente al servidor
con utilizando `wget`.

```sh
wget -O indexmic_2.6.1_all.deb https://github.com/proyais/indexmic/releases/download/v2.6.1/indexmic_2.6.1_all.deb
```

Después, utilizando `apt` o algún otro sistema para instalar paquetes `.deb` se puede instalar.

```sh
sudo apt install ./indexmic_2.6.1_all.deb
```

Esto creará la carpeta `/etc/indexmic` y el archivo `application.yml`. Este archivo
es el archivo de configuración de IndexMic, donde se indicará el sistema de
autenticación, las aplicaciones y sus parámetros, entre otras configuraciones.

### Docker en el mismo sistema

Cuando docker corre en el mismo sistema, el usuario `indexmic`,
el cual se crea automáticamente con la instalación, debe
hacer parte del grupo `docker` para poder iniciar
contenedores.

Para esto se puede utilizar:

```sh
sudo usermod -aG docker indexmic
```

## Habilitar, iniciar, reiniciar y detener IndexMic

Para habilitar el servicio IndexMic se debe utilizar `systemctl`:

```sh
sudo systemctl enable indexmic.service
```

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

Una vez hecho el backup, se puede descargar el `.deb` con la nueva versión y
reinstalar IndexMic con la nueva versión.



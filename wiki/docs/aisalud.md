# Instalar Analítica Integrada Salud

IndexMic es la plataforma ideal para hostear  Analítica Integrada Salud. Los
logos vienen pre-instalados en el repositorio y la configuración es sencilla
de parametrizar.

## Agregar Analítica Integrada Salud

Todas las aplicaciones se ingresan debajo de `specs` y siguen una configuración
base con los siguientes parámetros:

- `id`: El id único de la aplicación/instancia;
- `logo-url`: La ruta del logo a utilizar (para el logo oficial utilizar `/img/aisalud.svg`);
- `display-name`: El título de la aplicación en IndexMic;
- `description`: La descripción de la aplicación en IndexMic;
- `container-img`: Nombre de la imágen utilizada para iniciar el contenedor;
    - La imágen oficial es `gitlab.proyais.com:5050/proyectoais/analitica-integrada-salud:estable`
- `container-cmd`: `["iniciar"]`.
- `container-env`: Variables de ambiente a pasar al contenedor;
- `container-columes`: Volumenes para montar en el contenedor.

## Descargar imágenes

Analítica Integrada Salud es distribuida en imágenes Docker para facilitar el
despliegue en diferentes sistemas operativos. Las imágenes con el contenido de
las aplicaciones de Analítica Integrada Salud se pueden descargar desde el
GitLab oficial de Proyecto AIS o puede ser construida de forma personalizada
siguiendo los requerimientos de la licencia.

Para descargar la imágen se debe utilizar:

```sh
sudo docker pull gitlab.proyais.com:5050/proyectoais/analitica-integrada-salud:estable
```

_El tag de la imágen puede ser `estable`, `pruebas`, `master` o el nombre de una
versión (`v22.5.4`)._

## Variables de ambiente

Analítica Integrada Salud debe tener las siguiente variables
de ambiente bajo `container-env` para poder funcionar correctamente:

_Las variables de ambiente pueden cambiar dependiendo de la base de datos que
se vaya a utilizar. Para más detalle ver el manual de su versión de Analítica
Integrada Salud en la [documentación](https://wiki.aisalud.co)._

- `LICENSE`: Llave de acceso a la licencia;
- `EDIT_USERGROUPS`: Los grupos (delimitados por comas), que tienen acceso a funciones de edición (ComBD, Guardar notas técnicas, etc). Revisar el [manual de autenticación](../auth).

## Ejemplo: conexión a PostgreSQL

Analítica Integrada Salud requiere conectarse
a una base de datos PostgreSQL. IndexMic utilizará los siguientes
parámetros para autenticarse con la base de datos:

- `PG_NAME`: El nombre de la base de datos;
- `PG_USER`: El usuario con acceso a la base de datos;
- `PG_PW`: La contraseña del usuario;
- `PG_HOST`: La dirección IP de la base de datos;
- `PG_PORT`: El puerto de la base de datos;

Para permitir configurar una aplicación para que se conecte a una
base de datos se haria se la siguiente manera:

```yml
proxy:
  ...
  specs:
  - id: aisalud
    ...
    container-env:
      PG_NAME: ais
      PG_USER: ais
      PG_PW: password
      PG_HOST: 172.17.0.1
      PG_PORT: 5432
```

**Notas**

- `DATABASE_HOST` esta definido como `172.17.0.1` porque si el servicio de
PostgreSQL corre en el host de IndexMic, los contenedores podrán
conectarse a través de esa IP. Este
[artículo](https://dockertips.com/algo_sobre_redes) explora el tema con
mayor profundidad.

- Para más detalles de configuración de Analítica Integrada Salud visita la [documentación](https://wiki.aisalud.co).

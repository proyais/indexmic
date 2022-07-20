# Analítica Integrada Salud

IndexMic es la plataforma ideal para hostear  Analítica Integrada Salud. Los
logos vienen pre-instalados en el repositorio y la configuración es sencilla
de parametrizar.

## Agregar Analítica Integrada Salud

Todas las aplicaciones se ingresan debajo de `specs` y siguen una configuración
base con los siguientes parámetros:

- `id`: El id único de la aplicación/instancia;
- `logo-url`: El nombre del archivo del logo (`/etc/indexmic/templates/2col/assets/img/`);
- `display-name`: El título de la aplicación en IndexMic;
- `description`: La descripción de la aplicación en IndexMic;
- `container-img`: Nombre de la imágen utilizada para iniciar el contenedor;
- `container-cmd`: El comando que se utiliza dentro del contenedor para iniciar la aplicación;
    - Los comandos llevan la siguiente estructura: `["ruta ejecutable", "flags", "argumentos"]`;
    - Ej. `["echo", "Hola Mundo"]`;
- `container-env`: Variables de ambiente a pasar al contenedor;

## Descargar imágenes

Analítica Integrada Salud es distribuida en imágenes Docker para facilitar el
despliegue en diferentes sistemas operativos. Las imágenes con el contenido de
las aplicaciones de Analítica Integrada Salud se pueden descargar desde el
GitLab oficial de Proyecto AIS o puede ser contruida de forma personalizada
siguiendo los requerimientos de la licencia.

(Pasos para crear una cuenta en GitLab y clona una imagen).

## Variables de ambiente

Analítica Integrada Salud debe tener las siguiente variables
de ambiente bajo `container-env` para poder funcionar correctamente:

- `LICENSE`: Llave de acceso a la licencia;
- `DATABASE_NAME`: El nombre de la base de datos;
- `DATABASE_USER`: El usuario con acceso a la base de datos;
- `DATABASE_PW`: La contraseña del usuario;
- `DATABASE_HOST`: La dirección IP de la base de datos;
- `DATABASE_PORT`: El puerto de la base de datos;
- `TECNOLOGIAS_COL_COD`: El nombre de la variable con el código único de las tecnologías en la tabla `tecnologias_catalogo`;
- `EDIT_USERGROUPS`: Los grupos (delimitados por comas), que tienen acceso a funciones de edición (ComBD, Guardar notas técnicas, etc). Revisar el [manual de autenticación](../auth).

## Conexión a PostgreSQL

Analítica Integrada Salud requiere conectarse
a una base de datos PostgreSQL. IndexMic utilizará los siguientes
parámetros para autenticarse con la base de datos:

- `DATABASE_NAME`: El nombre de la base de datos;
- `DATABASE_USER`: El usuario con acceso a la base de datos;
- `DATABASE_PW`: La contraseña del usuario;
- `DATABASE_HOST`: La dirección IP de la base de datos;
- `DATABASE_PORT`: El puerto de la base de datos;

Para permitir configurar una aplicación para que se conecte a una
base de datos se haria se la siguiente manera:

```yml
proxy:
  ...
  specs:
  - id: aisalud
    ...
    container-env:
      DATABASE_NAME: ais
      DATABASE_USER: ais
      DATABASE_PW: password
      DATABASE_HOST: 172.17.0.1
      DATABASE_PORT: 5432
```

**Notas**

- `DATABASE_HOST` esta definido como `172.17.0.1` porque si el servicio de
PostgreSQL corre en el host de IndexMic, los contenedores podrán
conectarse a través de esa IP. Este
[artículo](https://dockertips.com/algo_sobre_redes) explora el tema con
mayor profundidad.

### Configuración dentro de PostgreSQL

Para permitir a las aplicaciones conectarse a la base de datos se deben
configurar los accesos a la base de datos. Si la base de datos se encuentra en
un servidor remoto, se debe configurar el acceso a la máquina que corre los
contenedores.

En el archivo `pg_hba.conf` se debe agregar una linea indicando que la
dirección IP de la(s) maquina(s) puede conectarse. Ver la documentación
para la versión específica de PostgreSQL utilizada.

- [Versión 14](https://www.postgresql.org/docs/14/auth-pg-hba-conf.html)
- [Versión 13](https://www.postgresql.org/docs/13/auth-pg-hba-conf.html)
- [Versión 12](https://www.postgresql.org/docs/12/auth-pg-hba-conf.html)
- [Versión 11](https://www.postgresql.org/docs/11/auth-pg-hba-conf.html)
- [Versión 10](https://www.postgresql.org/docs/10/auth-pg-hba-conf.html)

**Notas**

- Si la base de datos corre en la misma máquina que los contenedores
se debe utilizar el siguiente rango por defecto: `172.17.0.0/16`.
Este permitirá que los contenedores se conecten a través de
la red creada por el Docker Daemon.

**Notas**

- Para más detalles de configuración de Analítica Integrada Salud visita la [AISalud Wiki](https://wiki.aisalud.co).

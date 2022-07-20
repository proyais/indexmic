# Autenticación

> IndexMic utiliza [ShinyProxy](https://www.shinyproxy.io/) para le manejo de usuarios
> y enrutamiento. Si algún sistema de autenticación no se documenta en esta página, visitar
> la documentación de autenticación de ShinyProxy.

## LDAP

Al utilizar autenticación LDAP, IndexMic utilizará el url LDAP para:

1. Autenticar usuarios intentando utilizar su usuario y contraseña.
2. Permitir el uso de ciertas aplicaciones buscando grupos en el LDAP a los
   cuales pertenezca el usuario y validando con los accesos configurados.

Para habilitar la autenticación con ldap primero debe cambiar la configuración
(`authentication: ldap`). Para configurar su ldap debe utilizar los siguientes
parámetros:

- `url`: el string de conexión al LDAP, compuesto del URL y el DN base del
directorio;
- `user-dn-pattern`: el patrón del DN para un usuario. Utilizar si todos los
usuarios se encuentran en una sola localización del LDAP;
- `user-search-filter`: filtro del LDAP para buscar usuarios. Utilizar si los
usuarios se encuentran en diferentes localizaciones del LDAP y no puedes usar
`user-dn-pattern`;
- `user-search-base`: base para buscar usuarios. Solo utilizar si
`user-search-filter` se esta utilizando;
- `group-search-filter`: filtro del LDAP para buscar la membresia a los grupos;
- `group-search-base`: base para buscar grupos. Solo utilizar si
`group-search-filter` se esta utilizando;
- `manager-dn`: en DN del usuario utilizado para conectarse al directorio LDAP;
dejar vacio si la conexión inicial es anónima;
- `manager-password`: la contraseña utilizada para conectarse al directorio
LDAP; puede omitirse si `manager-dn` esta vacia;

**Notas**

- el DN base en el `url` (ej. `dc=example,dc=com` en
`ldap://ldap.forumsys.com:389/dc=example,dc=com`) no se tiene que repetir en
`user-search-base` o en `group-search-base`. Sin embargo, debe repetirse en
`manager-dn`.
- `user-dn-pattern` y `user-search-filter` permiten el uso del marcador de
posición `{0}` el cual será reemplazado con el nombre de usuario.
- `group-search-filter` permite el uso de tres marcadores de posición:
    - `{0}`: El DN del usuario
    - `{1}`: El nombre de usuario
    - `{2}`: El CN del usuario

### Variables de ambiente

Cuando un usuario es autenticado, las siguientes variables de ambiente
se exportarán al contenedor de las aplicaciones:

- `SHINYPROXY_USERNAME`: el nombre de usuario.
- `SHINYPROXY_USERGROUPS`: los grupos a los que pertenece el usuario separados por comas.

### Multiples proveedores de LDAP

Si se van a utilizar multiples proveedores de LDAP, se pueden
configurar de la siguiente manera:

```yml
proxy:
  ldap:
  - url: ldap://ldap.forumsys.com:389/dc=example,dc=com
    ...
  - url: ldap://another.ldap.server:389/...
    ...
```

### StartTLS

Para utilizar LDAP con StartTLS se debe configurar de la siguiente
manera:

```yml
proxy:
  ldap:
    starttls: simple
```

Los valores de `starttls` puede ser:

- `simple`: StartTLS se habilita utilizando autenticación simple;
- `true`: igual que `simple`;
- `external`: StartTLS se habilita utilizando atenticación con
certificado externo.
- (Vacio): StartTLS deshabilitado.

### Ejemplo: Active Directory

```yml
ldap:
  url: ldaps://example.com:3269/dc=example,dc=com
  manager-dn: cn=shinyproxy,ou=Service Accounts,dc=example,dc=com
  manager-password: xxxxxxxxxxxx
  user-search-filter: (sAMAccountName={0})
  group-search-filter: (member={0})
  group-search-base: ou=Groups
```

**Notas**

- `sAMAccountName` se utiliza comunmente como el nombre de usuario único
en ambientes de Active Directory. Por esta razón se utiliza en el
`user-search-filter`.
- `3269` el el puerto del catalogo configurado con SSL.

## Keycloak

Al utilizar autenticación Keycloak, IndexMic utilizará los parametros para:

1. Autenticar usuarios intentando utilizar el SSO de Keycloak.
2. Permitir el acceso a ciertas aplicaciones según los roles del realm o del cliente.

Para habilitar la autenticación con Keycloak primero se debe cambiar la
configuración (`authenticacion: keycloak`). Para configurar Keycloak se deben
utilizar los siguientes parámetros:

- `realm`: El nombre del realm con los usuarios y clientes.
- `auth-server-url`: El url del servidor de autenticación.
- `resource`: El nombre del recurso.
- `credentials-secret`: El secreto de los credenciales.
- `ssl-required`: `none`, `all` o `external` (defecto).
- `proxy.keycloak.name-attribute`: el atributo a usarse para el nombre de usuario. `name` (defecto), `preferred_username`, `nickname` o `email`.
- `use-resource-role-mapping`: booleano para utilizar roles del cliente (`true`) o roles de realm (`falso`; defecto).

### Variables de ambiente

Cuando un usuario es autenticado, las siguientes variables de ambiente
se exportarán al contenedor de las aplicaciones:

- `SHINYPROXY_USERNAME`: el nombre de usuario.
- `SHINYPROXY_USERGROUPS`: los roles a los que pertenece el usuario separados por comas.


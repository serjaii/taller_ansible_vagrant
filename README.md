# ansible_ejemplos

Ejemplos de playbooks de ansible para los alumnos de ASIR del IES Gonzalo Nazareno.

## Ejemplo1: Playbook sencillo

Primer ejemplo sencillo, se usa un playbook sin roles. Se configura el acceso a una máquina. Esta máquina debe tener la siguientes caracteristicas:

* Debe tener creado un usuario sin privilegios con el que podamos acceder a la máquina usando claves rsa. Una vez configurado la ip de la máquina en el inventario (fichero `hosts`) y el usuario en el fichero de configuración de ansible (fichero `ansible.cfg`), podemos probar que la conexión se puede realizar sin problema desde el ordenador donde está instalado ansible, ejecutando:

        ansible node1 -m ping
    
    Debe salir el mensaje "pong" en verde.

* El ordenador que estamos configurando debe tener instalado "sudo" y el usuario que estamos usando para acceder debe estar configurado para poder usar sudo sin que le pida la contraseña (otra opción sería utilizar el parámtro `-K` para que nos pida la contraseña del usuario a la hora de ejecutar el playbook). De esta manera cuando se tenga que realizar una tarea en la que se necesite derechos de root, el usuario podrá usar sudo (esto se indica en el fichero `site.yaml` con `become: true`).

Para ejecutar el playbook:

        ansible-playbook site.yaml

Veamos las tareas y los módulos utilizados en el playbook (fichero `site.yaml`):

* El primero no sirve para nada simplemente es el ping: `ansible.builtin.ping:`.
* El segundo es para actualizar los paquetes de un sistema Debian/Ubuntu, se utiliza el módulo `apt` con dos parámetros:

        apt: update_cache=yes upgrade=yes

* Instalamos un paquete:

        apt:
            name: git
    
    **Busca la manera de instalar varios paquetes.**

* Copiamos un fichero al ordenador remoto, para ello usamos el módulo `copy`.


## Ejemplo 2: Playbook con variables y templates

1. En este ejemplo hemos definido dos nodos en el inventario (`hosts`), cada uno lo hemos agrupado en grupos distintos, aunque por ahora no estamos utilizando los grupos. Si te fijas en el play `site.yaml` hemos puesto `hosts: all` con lo que las tareas se van a ejecutar en todos los nodos.
2. En esta ocasión cada nodo tiene un usuario y una clave privada distinta para acceder. Por lo que hemos quitado el parámetro `remote_user` de la configuración (`ansible.cfg`) y hemos puesto las credenciales de acceso directamente en cada nodo en el inventario con `ansible_ssh_user` y `ansible_ssh_private_key_file`.
3. ansible puede trabajar con variables que obtiene de distinta manera:

  * A nivel de **nodo**: definimos variables para cada nodo en el inventario, por ejemplo: `ansible_ssh_host`, `ansible_ssh_user`, ...
  * A nivel de **grupo de nodos**: Hemos creado un directorio que se tiene que llamar `group_vars`, dentro de este directorio podemos crear ficheros con las variables que creamos a nivel del grupo. En este ejemplo hemos creado un fichero `all`, por lo que las variables serán conocidas para todos los nodos. Podríamos haber creado un fichero llamado `Grupo1`, por lo que esas variables sólo serían conocidas por los nodos de ese grupo. Hemos definido 3 variables para todos los nodos:`bd_name`, `bd_user` y `bd_pass`.
  * **Gathering Facts**: Variables que obtiene ansible de los nodos que esta configurando. La primera tarea que ejecuta el play es obtener información del nodo que va a configurar. Toda esa información se puede usar al definir las tareas, por ejemplo: `ansible_hostname`, `ansible_distribution`,... Para ver esa información podemos ejecutar:
  ```bash      
        ansible nodo1 -m setup
  ```

  ¿Donde podemos usar el valor de estas variables?

  * Al crear las tareas: Puedo añadir el valor de la variable (jinja2) si me hace falta, por ejemplo el nombre de la base de datos está guardad en una variable:

  ```yaml
  - name: create database wordpress
      mysql_db: name={{bd_name}} state=present
```

* Si necesito enviar ficheros al nodo, pero quiero que esos ficheros tengan distintas variables, puedo crear un plantilla jinja2 y usar el módulo `template` para copiarla en el nodo:

  ```yaml
  - name: copy template index 
        template: 
          src: template/index.j2
          dest: /var/www/html/index.html
          owner: www-data
          group: www-data
          mode: 0644
  ```

## Ejemplo 3: Playbook con roles

Hemos escrito el inventario en formato yaml. En ele ejemplo anterior estaba escrito en ini.

Tal como hemos definido las tareas en los dos ejemplos anteriores, no puedo indicar que tareas hay que hacer para nodos distintos que esten agrupados en distintos grupos. Para solucionarlo vamos a usar los **roles**, que nos permiten organizar las tareas para ejecutarlas en todos los nodos, o en un grupo de nodos en concreto. Además los roles me dan la posibilidad de reutilización de código. Si hago un rol para instalar un servidor web apache, ese rol lo puedo a volver a usar en otro proyecto en que tenga que hacer la misma operación.

Podemos ver que en el fichero `site.yaml`, vamos a ejecutar 3 roles:

* El rol `commons` para todos los nodos (grupo `all`).
* El rol `apache2` Para todos los nodos del grupo `servidores_web`.
* El rol `mariadb` Para todos los nodos del grupo `servidores_bd`.

Los roles se van a definir en el directorio `roles`. Se creará un directorio para cada rol, que se llamará igual al nombre que hemos puesto el rol en el fichero `site.yaml` y contendra las tareas, ficheros, templates,... necesario para llevar a cabo este rol.

Por lo tanto dentro de la carpeta del rol, podremos tener algunas de estas carpetas:

* `tasks`: Contiene el yaml con las tareas.
* `files`: Contiene los ficheros que vamos a copiar a los nodos con el módulo `copy`.
* `templates`: Contiene las plantillas que vamos a copiar a los nodos con el módulo `template`.
* ...


## Ejemplo 4: Playbook para gestionar servicios. Handlers

En ocasiones es necesario sólo ejecutar unas tareas si ocurre algo. Por ejemplo, si cambiamos el fichero de configuración de un servicio, habrá que ejecutar una tarea para reiniciar el servicio. Este compartimento se consigue usando los `handlers`.

Para cambiar la configuración de un servicio, tenemos dos alternativas:

* Copiar el fichero de configuración con el módulo `copy`, aunque en las mayoria de las ocasiones tendremos el fichero de configuración parametrizado, por lo que usaremos un template y el módulo `template`.
* Usar el módulo `lineinfile` que nos permite hacer modificaciones en un fichero remoto.


En el ejemplo 4 hemos copiado al servidor web un fichero de configuración y tenemos que indicar que queremos reiniciar el servicio:

```yaml
- name: "Copiar fichero de configuración y reiniciar el servicio"
  copy:
    src: etc/apache2/ports.conf
    dest: /etc/apache2/ports.conf
    owner: root
    group: root
    mode: '0644'
  notify:
    - restart apache2
```

Hemos creado dentro del directorio del rol, un directorio llamado `handlers` que contendra un fichero yaml con la tareas que hay realizar para reinciar el servicio.

```yaml
- name: restart apache2
  service: name=apache2 state=restarted
```

Del mismo modo hemos cambiado la configuración de mariadb para que reciba peticiones por todas las inteferfaces, en este caso lo hemos hecho con el módulo `lineinfile`:

```yaml
- name: mariadb listen in all interfaces
  lineinfile: dest=/etc/mysql/mariadb.conf.d/50-server.cnf regexp="^bind-address            = 127.0.0.1" line="bind-address            = 0.0.0.0" state=present
  notify:
    - restart mariadb
```

En el directorio del rol `mariadb` hemos creado un directorio `handlers` con el fichero que contiene la tarea para reiniciar mariadb.

Finalmente podemos la estructura del proyecto:

```bash
tree
.
|____ansible.cfg
|____site.yaml
|____roles
| |____mariadb
| | |____tasks
| | | |____main.yaml
| | |____handlers
| | | |____main.yaml
| |____commons
| | |____tasks
| | | |____main.yaml
| |____apache2
| | |____files
| | | |____fichero.html
| | | |____etc
| | | | |____apache2
| | | | | |____ports.conf
| | |____tasks
| | | |____main.yaml
| | |____templates
| | | |____index.j2
| | |____handlers
| | | |____main.yaml
|____hosts
|____group_vars
| |____all
```




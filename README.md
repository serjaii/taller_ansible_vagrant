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

## Ejemplo 4: Playbook para gestionar servicios. Handlers





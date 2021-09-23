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



## Ejemplo 3: Playbook con roles

## Ejemplo 4: Playbook para gestionar servicios. Handlers





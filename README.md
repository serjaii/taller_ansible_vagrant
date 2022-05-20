# Ejemplos ansible

Ejemplos para el trabajo de los talleres de ansible.

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




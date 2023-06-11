## ANSIBLE
### Documentación:
https://docs.ansible.com/developers.html,
https://github.com/ansible/ansible

### 1. Qué es Ansible:
- Producto para gestionar la configuración, implementación, aprovisionamiento y automatización de nuestra infra.
- Mediante ficheros de configuración gestionamos y provisionamos nuestra infra sin necesidad de realizar procesos manuales.
- Playbooks: Ficheros declarativos que se lanzan contra las máquinas remotas para gestionarlas de manera sencilla y rápida (redes, users, servicios, ... No siempre es HW)
- Algunas características de Ansible: 
    - Seguridad: Nos permite establecer políticas de seguridad para los distintos equipos y entornos con los que trabajemos.
    - Orquestación: Nos permite organizar todos nuestros componentes para que se relacionen de forma correcta.
    - Preparación: Nos permite instalar y configurar la infra: preparar servidores, redes, users, servicios ...
    - Gestión de la configuración: Configurar el entorno de forma rápida y sencilla.

### 2. Competidores:
- Terraform, Chef, Puppet, Salt, Red Hat Ansible.

### 3. Arquitectura y Componentes:
- __Control Nodes:__ Servidores maestros desde donde instalo Ansible y se lanzan los comandos contra los servidores controlados.
    Funcionan sobre entornos Linux, aunque puede usar el módulo WSL de Windows.
- __Manages Nodes:__ Servidores que van a ser gestionados por Ansible. Se encuentran dentro de un inventario que nos permite agruparlos y manipularlos de forma sencilla.
- __Playbooks y Plays__: Un "Play" ejecuta una serie de tareas en los nodos gestionados. Son ficheros en formato YAML.  Son declarativos (de fácil uso, gestión y mantenimiento)
    Se pueden agrupar en PlayBooks (conjunto de Plays agrupables).
- __Módulos__: Scripts independientes que podemos ejecutar dentro de un Playbook. Liberías con comandos. Se copian y ejecutan en cada nodo gestionado para ejecutar la acción que se ha definido en la tarea correspondiente.
    Hay distintos módulos para distintas necesidades.
- __Colecciones__: Formato de distribuición que puede estar compuesto de Playbooks, Módulos, Roles  o Plugins. Divide Ansible en distribuciones más ligeras y organizadas en lugar de tener 3000 módulos. Se pueden instalar, de este modo, unas pocas selecciones que pertenecen al Core de Ansible y otras gestionadas por empresas. Se puede trabajar con ellas a través de Ansible Galaxy.
    - __Guía de colecciones__: https://docs.ansible.com/ansible/latest/collections_guide/index.html
    - __Galaxy Ansible__: https://galaxy.ansible.com/
    - __Ansible.Builtin__: Es la colección incluida en el propio core de Ansible


### 4. Preparando Entorno para nodos gestionados:
- __4.1. Configurar SSH para root__: 
    - Generamos un par de clave pública-privada: ```ssh-keygen```. Se generan dos ficheros, el RSA que contiene la clave privada y el RSA.pub que contiene la clave pública que voy a pasar al nodo remoto.
    - Vamos a cd /home/>usuario>/.ssh/  y vemos que se ha generadoid_rsa e id_rsa.pub. Al hacer ```cat id_rsa.pub``` vemos la clave privada y el usuario y la máquina desde la que está trabajando.
    - Vamos a la máquina remota y lo metemos en el authorized-keys en la carpeta .ssh y metemos la  clave púbica

### 5. Primeros pasos. Comandos ad-hoc
- __5.1. Inventarios__: Un inventario es un fichero que contiene las máquinas con las que quiero trabajar.
    Dado un inventario de máquinas llamado maquinas, con el módulo (-m) ping hacemos ping a las máquinas (-i = inventario) ```ansible -i maquinas all -m ping```:    
    ```json 
        192.168.50.2 | SUCCESS => {
        "ansible_facts": {
            "discovered_interpreter_python": "/usr/bin/python3"
        },
        "changed": false,
        "ping": "pong"
    }
    ```
    
    Listar los hosts ```ansible -i maquinas all --list-hosts```

- __5.2. Command & Shell__:
Con el siguiente comando se deuvlve la fecha de la máquina debian1 ```ansible -i maquinas debian1 -m command -a "date"```
Es posible enviar comandos que realicen cambios en debian1: ```ansible -i maquinas debian1 -m command -a  "touch /tmp/f1.txt"```
Es posible recuperar la configuración del sistema con comandos como: ```ansible -i maquinas debian1 -m setup```

- __5.3. Copiando ficheros__:
  - i = inventario; -a=argumentos, -m=módulo
- ```ansible debian1 -i maquinas -m copy -a "src=/home/directorio/ansible/prueba.txt dest=/home/prueba/prueba.txt"```
- Es posible darle permisos, en este caso de ejecución, lectura y escritura:  ``` ansible debian1 -i maquinas -m copy -a "src=/home/jorgegarciaotero/ansible/practicas/prueba.txt dest=/home/prueba/prueba.txt mode=777"```

- __5.4. Paquetes de Sistema: Instalando y arrancando un paquete__ 
    - Instalamos el servicio con el módulo yum y state=present/latest... para el servicio httpd ```ansible -i maquinas rocky1 -m yum -a "name=httpd state=present"```
    - Con el módulo service y state=started (podría ser reloaded,restarted,stopped también)```ansible -i maquinas rocky1 -m service -a "name=httpd state=started"```
        https://docs.ansible.com/ansible/latest/collections/ansible/builtin/service_module.html
        Al entrar a la máquina y hacer el ```systemctl status httpd``` comprobamos efectivamente que está arrancado el servicio

- __5.5. Escalado de privilegios__:
Nos permite ejecutar tareas como root o como otro user. Ésto se puede hacer mediante la cláusula become, que permite convertirnos temporalmente en otro user distinto al que nos hemos conectado.
Las principales directivas son:
    - become: Activa la escalada de privilegios.
    - become_user: usuario en el que queremos convertirnos, por defecto root.
    - become_method: el método que queremos utilizar en el sistema operativo para la escalada, por ejemplo sudo.
    - become_flags: permite especificar las opciones en el momento de la escalada.
    ```ansible -i maquinas debian1 -m service -a "name=apache2 state=started" -b ```
    ```ansible -i maquinas ubuntu1 --become --ask-become-pass -m apt -a "name=apache2 state=present" ```

### 6.Ficheros de configuración e inventario
Ansible tiene distintas formas de configurar su funcionamiento:
- Fichero de configuración,
- Variables de entorno,
- Opciones de línea de comandos,
- Opciones y variables en los Play Books

- __6.1.Fichero de configuración:__
Ansible tiene un fichero llamado "ansible.cfg" donde ponemos los valores por defecto de nuestro Ansible. Está formado por un conjunto de opciones y propiedades que ya tienen un valor predefinido y que no necesito cambiar a través de este fichero. Este fichero se puede encontrar en:
  - ANSIBLE_CONFIG: Variable de entorno.
  - Ansible.cfg en el directorio actual. por tanto así podemos tener un fichero de configuración a nivel de proyecto.
  - ~/.ansible.cfg en el directorio home del user
  - /etc/ansible/ansible.cfg
  - Se pueden poner comentarios con # o ;
  - Se pueden agrupar en secciones con []
  - Generamos el fichero con el comando ```ansible-config init --disabled > ansible.cfg```. Con el --disabled me lo deja todo comentado

- __6.2.Inventarios:__
Nos permiten indicar los servidores a los que queremos conectarnos. Puede ser en un único fichero, una lista o usar plugins más avanzados que nos permitan personalizar los datos.
Por defecto, se busca el fichero en /etc/ansible/hosts, pero con la opción -i podemos cambiar su ubicación fácilmente: ```ansible -i fichero_inventario```


### 6.Playbooks
Plantilla que nos permite ejecutar entornos complejos dentro de ansible sin necesidad de intervención humana. 
Se compone de tareas que se pueden combinar en un componente llamado "play". Varios plays confirman un Playbook.
Tenemos varias acciones con las que se puede trabajar dentro de un playbook:
- Hosts --> máquinas donde se ejecuta.
- Vars --> variables del play
- Tasks --> lista de tareas a ejecutar en el play.
- Handlers --> tareas que se ejecutan solo ante algún cambio.
- Roles --> roles a ser importados. Cargan de forma ordenada recursos.

Ejecutamos un playbook llamado ping.yaml con: ```ansible-playbook -i maquinas ping.yaml```
```---
- name: Primer play del curso.
  hosts: all
  tasks:
    - name: Hacer un ping
      ping:
    - name: crear un fichero
      ansible.builtin.shell:
        touch /tmp/fichero1.txt
```

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

- name: Instalar nginx
  hosts: debian1
  tasks:
    - name: Parar apache
      ansible.builtin.service:
        name: apache2
        state: stopped
    - name: Instalar nginx
      ansible.builtin.apt:
        name: nginx
        state: present
        update_cache: true
    - name: Arrancar Nginx
      ansible.builtin.service:
        name: nginx
        state: started
```


### 7.Variables
Ansible dispone de varias formas de implementar variables:
- De entorno:  se usan para definir valores en el SO, como ANSIBLE_CONFIG.
- En los ficheros de inventario: se incluyen en los ficheros de inventarios y se pueden aplicar a nivel de hosts o grupo.
- En los playbooks: podemos definirlas a nivel de play e incluso de task.
- En los roles:- se usan dentro de un Role. 
- Especiales: 
  - Magic: variables que reflejan el estado interno y no son modificables.
  - Facts: contienen info de un host: ``` ansible debian1 -m setup``` ,```ansible debian1 -m setup -a "filter=ansible_proc_cmdline``` 
  - Conexión: determinan como se ejecutan las acciones cotnra un target, por ejemplo ansible_become_user
Las variables en si mismo no son nada, se le pasan al playbook.
- Ejemplo1 (maquinas):
    ```[debian]
       debian1
       debian2
       [ubuntu]
       ubuntu1 puerto=9000 entorno=desarrollo
    ```
- Ejemplo2: Formato yaml (maquinas.yaml)
    ```all
        hosts:
            debian1:
            ubuntu1:
                puerto: 9000
                entorno: desarrrollo ```
Cogiendo el fichero maquinas, vemos que podemos hacer que las máquinas debian hereden la variable puerto=9090 con [debian:vars]
    ```[debian]
    debian1
    debian2

    [debian:vars]
    puerto=9090

    [rocky]
    rocky1
    rocky2

    [ubuntu]
    ubuntu1 puerto=9090 entorno=desarrollo ansible_user=pepe

    [servidores_de_datos]
    mysql1

    [servidores_de_aplicaciones]
    tomcat1
    tomcat2```

Puedo coger variables de grupos anidados
```[debian]
debian1
debian2

[debian:vars]
puerto=9090

[rocky]
rocky1
rocky2

[ubuntu]
ubuntu1 puerto=9090 entorno=desarrollo

[servidores_de_datos]
mysql1

[servidores_de_aplicaciones]
tomcat1
tomcat2

[desarrollo:children]
debian
ubuntu

[desarrollo:vars]
ansible_user=juan```

Creamos el playbook variables.yaml, cargamos la info siguiente y lo ejecutamos mediante ```ansible-playbook variables.yaml```
```---
- name: Prueba con variables
  hosts: debian1

  tasks:
  - name: ver variables
    debug:
      msg: esto es una prueba```

Otra  forma:
```---
- name: Prueba con variables
  hosts: debian1
  vars:
    - mensaje: "Esto es un mesnaje de ansible"
    - curso: "dentro del curso de ansible"

  tasks:
  - name: ver variables
    debug:
      msg: "{{mensaje}} {{curso}}" ```
Resultado:
```oot@vweb-1:/home/jorgegarciaotero/ansible# ansible-playbook variables.yaml
[DEPRECATION WARNING]: Ansible will require Python 3.8 or newer on the controller starting with Ansible 2.12. Current version: 3.6.9 (default, Mar 10 2023, 16:46:00) [GCC 8.4.0]. This 
feature will be removed from ansible-core in version 2.12. Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.

PLAY [Prueba con variables] ***************************************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************************************************
ok: [debian1]

TASK [ver variables] **********************************************************************************************************************************************************************
ok: [debian1] => {
    "msg": "Esto es un mesnaje de ansible dentro del curso de ansible"
}

PLAY RECAP ********************************************************************************************************************************************************************************
debian1                    : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0```


### 8.Control de Flujo. Condicionales, bucles y errores.
Ejemplos en /control_flujo
- __WHEN:__:
  - No se usan doble llaves. Se evalua para todos los hosts.
  - Operadores:   < > >= <= !=  ==
  - "is defined"  permite saber si una variable existe.
  - para buscar en un array se puede usar el operador "in"
 ```TASK [Gathering Facts] ********************************************************************************************************************************************************************
      ok: [tomcat1]
      ok: [tomcat2]
      ok: [mysql1]
      [WARNING]: Platform linux on host rocky1 is using the discovered Python interpreter at /usr/bin/python3.9, but future installation of another Python interpreter could change the meaning
      of that path. See https://docs.ansible.com/ansible-core/2.11/reference_appendices/interpreter_discovery.html for more information.
      ok: [rocky1]
      [WARNING]: Platform linux on host rocky2 is using the discovered Python interpreter at /usr/bin/python3.9, but future installation of another Python interpreter could change the meaning
      of that path. See https://docs.ansible.com/ansible-core/2.11/reference_appendices/interpreter_discovery.html for more information.
      ok: [rocky2]
      ok: [debian1]
      ok: [debian2]
      ok: [ubuntu1]

      TASK [Capturar fecha] *********************************************************************************************************************************************************************
      skipping: [rocky1]
      skipping: [rocky2]
      changed: [debian1]
      changed: [tomcat2]
      changed: [mysql1]
      changed: [debian2]
      changed: [tomcat1]
      skipping: [ubuntu1]

      TASK [Visualizar fecha] *******************************************************************************************************************************************************************
      skipping: [rocky1]
      skipping: [rocky2]
      ok: [mysql1] => {
          "msg": "Wed Jun 14 17:53:43 UTC 2023"
      }
      ok: [tomcat1] => {
          "msg": "Wed Jun 14 17:53:43 UTC 2023"
      }
      ok: [tomcat2] => {
          "msg": "Wed Jun 14 17:53:43 UTC 2023"
      }
      ok: [debian1] => {
          "msg": "Wed Jun 14 17:53:43 UTC 2023"
      }
      ok: [debian2] => {
          "msg": "Wed Jun 14 17:53:43 UTC 2023"
      }
      skipping: [ubuntu1]```
  
- __BUCLES:__:
Tenemos distintos bucles:
  - Loop:
  - whit_<lookup>   whit_items, whit_list,  with_sequence ...
  - until
En el siguiente ejempolo, con cada valor de loop, {{msg}} adquiere el valor de valor1, valor2, valor3:
```---
- name: Prueba básica con LOOP
  hosts: debian1

  tasks:
  - name: Visualizar contenido con loop
    ansible.builtin.debug:
      msg: "{{item}}"    
    loop:
       - valor1
       - valor2
       - valor3

  - name: Visualizar contenido con with_items
    ansible.builtin.debug:
      msg: "{{item}}"    
    with_items:
       - valor1
       - valor2
       - valor3```
Resultado:
```TASK [Gathering Facts] ********************************************************************************************************************************************************************
ok: [debian1]

TASK [Visualizar contenido con loop] ******************************************************************************************************************************************************
ok: [debian1] => (item=valor1) => {
    "msg": "valor1"
}
ok: [debian1] => (item=valor2) => {
    "msg": "valor2"
}
ok: [debian1] => (item=valor3) => {
    "msg": "valor3"
}

TASK [Visualizar contenido con with_items] ************************************************************************************************************************************************
ok: [debian1] => (item=valor1) => {
    "msg": "valor1"
}
ok: [debian1] => (item=valor2) => {
    "msg": "valor2"
}
ok: [debian1] => (item=valor3) => {
    "msg": "valor3"
}

PLAY RECAP ********************************************************************************************************************************************************************************
debian1```
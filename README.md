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
```
[debian]
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
tomcat2
```

Puedo coger variables de grupos anidados
```
[debian]
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
ansible_user=juan
```

Creamos el playbook variables.yaml, cargamos la info siguiente y lo ejecutamos mediante ```ansible-playbook variables.yaml```
```
---
- name: Prueba con variables
  hosts: debian1

  tasks:
  - name: ver variables
    debug:
      msg: esto es una prueba
```

Otra  forma:
```
---
- name: Prueba con variables
  hosts: debian1
  vars:
    - mensaje: "Esto es un mesnaje de ansible"
    - curso: "dentro del curso de ansible"

  tasks:
  - name: ver variables
    debug:
      msg: "{{mensaje}} {{curso}}" 
```

Resultado:
```
oot@vweb-1:/home/jorgegarciaotero/ansible# ansible-playbook variables.yaml
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
debian1                    : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

### 8.Control de Flujo. Condicionales, bucles y errores.
Ejemplos en /control_flujo
- __WHEN:__:
  - No se usan doble llaves. Se evalua para todos los hosts.
  - Operadores:   < > >= <= !=  ==
  - "is defined"  permite saber si una variable existe.
  - para buscar en un array se puede usar el operador "in"
 ```
 TASK [Gathering Facts] ********************************************************************************************************************************************************************
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
      skipping: [ubuntu1]
```
  
- __BUCLES:__:
Tenemos distintos bucles:
  - Loop:
  - whit_<lookup>   whit_items, whit_list,  with_sequence ...
  - until
En el siguiente ejempolo, con cada valor de loop, {{msg}} adquiere el valor de valor1, valor2, valor3:
```
---
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
       - valor3
```

Resultado:
```
TASK [Gathering Facts] ********************************************************************************************************************************************************************
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
debian1
```


### 8. FILTROS
Permiten realizar tareas como conversiones, transformaciones, extracciones, manupulación de datos, etc.
Se ejecutan en el controller, no en la máqujina remota. Se usa el símbolo | para indicarlos.
- Ejemplo: Ver el tipo de una variable.  Una vez que tengo las variables, tenego el método llamado "var" que nos permite devolver el valor de una variable. con ```var: cadena | type_debug```, si pongo la variable nos va a devolver el contenido. el método debug nos permite visualizar la variable con la que estamos trabajando. El tipo de la cadena "hola" nos devolverá "AnsibleUnicode". Si pones diccionario | type_debug saldrá "dict":

```
---
- name: Ejemplos varios de filtros
  hosts: debian1
  vars:
       cadena: "hola"
       numero: 10
       verdad: true
       lista:
          - pepe
          - juan
          - antonio
       lista1: ['pepe','juan','antonio']  
       diccionario:
           nombre: juan
           edad: 27  
 
  tasks:
  - name: Averiguar el tipo de una variable
    ansible.builtin.debug:
      var: diccionario | type_debug
```

Resultado:
      ok: [debian1] => {
    "diccionario | type_debug": "dict"
}

- __8.1.Conversiones:__ : Para convertir un tipo de variable a otra: Sea cadena de tipoi sting, convertir a entero ( cadena  | int: "true") te lo convierte a int:0, y (cadena | bool) te lo conbierte a true.
```
---
- name: Ejemplos varios de filtros
  hosts: all
  vars:
       cadena: "true"
       numero: 10
       verdad: true
       lista:
          - pepe
          - juan
          - antonio
       lista1: ['pepe','juan','antonio']  
       diccionario:
           nombre: juan
           edad: 27  
 
  tasks:
  
  - name: Convertir a cadena
    ansible.builtin.debug:
      var: numero 

  - name: Convertir a cadena
    ansible.builtin.debug:
      var: numero | string 

  - name: Convertir a entero
    ansible.builtin.debug:
      var: cadena | int

  - name: Convertir a entero
    ansible.builtin.debug:
      var: cadena | bool

  - name: visualizar version
    ansible.builtin.debug:
      msg: "{{ansible_facts['distribution_version']}}"
    when: ansible_distribution_version | int > 10
```


- __8.2. Cadenas:__ : Convertir una cadena a upper, lower, replace ...
```
---
- name: Ejemplos varios de filtros con CADENAS
  hosts: debian1
  vars:
       cadena: "Esto es una cadena"
       
  tasks:
  
  - name: Mayusculas
    ansible.builtin.debug:
      var: cadena | upper  

  - name: Minusculas
    ansible.builtin.debug:
      var: cadena  | lower

  - name: Reemplazar
    ansible.builtin.debug:
      var: cadena | replace("e","*")

  - name: Longitud de cadena
    ansible.builtin.debug:
      var: cadena | length
```

- __8.3. Números:__ :
 ```
 ---
- name: Ejemplos varios de filtros con NUMEROS
  hosts: debian1
  vars:
       numero: 10.40
       
  tasks:
  
  - name: Potencia
    ansible.builtin.debug:
      var: numero | pow(4)
  
  - name: Raiz cuadrada
    ansible.builtin.debug:
      var: numero  | root()
  
  - name: Redondeo
    ansible.builtin.debug:
      var: numero  | round()

  - name: Nuero aleatorio
    ansible.builtin.debug:
      var: numero  | int | random
```

  - __8.4. Listas:__ :
  ```
  ---
- name: Ejemplos varios de filtros con LISTAS
  hosts: debian1
  vars:
       lista_numero:
         - 2
         - 10
         - 9
         - 1
       lista_cadena:
         - Pedro
         - Juan
         - Rosa
         - Antonio
       cadena: "Esto es una cadena"
  tasks:
  
  - name: Valor menor numero
    ansible.builtin.debug:
      msg: "{{lista_numero | min }} --- {{lista_numero | max }} "
  
  - name: Valor menor  cadena
    ansible.builtin.debug:
      msg: "{{lista_cadena | min }} --- {{lista_cadena | max }}" 

  - name: Unir elementos de una lista
    ansible.builtin.debug:
      msg: "{{lista_cadena | join(',')}}" 

  - name: Convertir cadena en lista
    ansible.builtin.debug:
      msg: "{{cadena | split()}}" 
  ```



### 9. ROLES
Conjunto de carpetas de directorios y ficheros que es como una plantilla o blueprint para poder generar de manera global múltiples componentes dentro de un play o Playbook.
- Por ejemplo se pueden usar handlers de tareas o ariables para crear o configurar Apache y eso lo puedo poner en un rol y reusar luego dentro de otros playbooks.
- Pueden ser reusados fácilmente y compartidos con otros usuarios y proyectos.
- Permiten dividir el trabajo en distintos archivos, de forma que es más fácil su uso y reutilización.
- Es una especie de "librería".
- Disponen de una estructura de archivos conocida. Tienen 8 directorios principales:
```
role1
  defaults
    main.yml
  files
  handlers
    main.yml
  meta
    main.yml
  README.md
  tasks
    main.yml
  templates
  tests
    inventory
    test.yml
  vars
    main.yml
```
| Carpeta     | Descripción                                              |  
|:------------|:--------------------------------------------------------| 
| Handlers    |  Handlers que pueden ser invocados por otras tareas  |  
| Templates   |  Plantillas a utilizar  |  
| Meta        |  metadatos de rol, por ejemplo, el autor, las dependencias y las plataformas de soporte  |  
| Tets        |  Para realizar pruebas  |  
- Los roles se pueden incluir:
  - A nivel de Plays: Se usa la cláusula "roles". Es un import estático. Se cargan y ejecutan antes que el resto de tareas.
  - A nivel de Tasks de forma dinámica: Con "include_role", lo que permite ejecutarlos en el orden y lugar donde han sido definidos.
  - Tasks de forma estática: Con "import_role". Se comportan igual que los definidos a nivel del play.

  - __9.1. Crear un Role:__ : Con el comando ```ansible-galaxy init desarrollo``` podemos crear un rol "desarrollo". Si hacemos ```tree mariadb``` se crea un directorio y dentro podemos ver  los 8 directorios con sus ficheros de variables, tareas, ficheros ...
```
mariadb
├── README.md
├── defaults
│   └── main.yml
├── files
├── handlers
│   └── main.yml
├── meta
│   └── main.yml
├── tasks
│   └── main.yml
├── templates
├── tests
│   ├── inventory
│   └── test.yml
└── vars
    └── main.yml
```

  - __9.1. Ejemplo básico de un Role:__ :  En un rol no se puede poner un play, solo tareas.
En el main.yml de tasks, si el servidor es un debian creamos un server de mariadb y lo arrancamos:
```
---
# tasks file for mariadb
- name: Instalar {{software}}
  ansible.builtin.apt: 
     name: "{{software}}"
     state: present
  when: ansible_facts['distribution'] | lower =='debian' 

- name: arrancar el servicio {{servicio}}
  ansible.builtin.service:
          name: "{{servicio}}"
          state: started
  when: ansible_facts['distribution'] | lower =='debian' 
```
Fuera del directorio mariadb creamos un play.yaml con el rol mariadb, automáticamente se van a crear las tareas del rol y luego las que tienen que ver con el playbook.
Primero instala mariadb, luego lo arranca y luego sale a ejecutar "se ha instalado el software ..."
```
- name: Ejemplos de un role
  hosts: debian1
  roles:
     - mariadb
  vars:
     software: apache2
     servicio: apache2
  tasks:

  - name: Ultima tarea
    ansible.builtin.debug:
      msg: "Se ha instalado el software {{software}}"ansi
```

  - __9.2.Roles a nivel de tarea__ :  Puedo incluir el rol a nivel de Play o de Task, y si lo hago a nivel de Task tengo 2 opciones (la tarea con el rol a nivel dinámico o dinámico.). El rol, siempre y cuando lo incluyo a nivel del play, se ejecuta en primer lugar.
  ```
  El siguiente ejemplo, primero ejecuta la tarea "Comienzo el juevo", luego las que haya en el rol y finalmente "Se ha terminado ..."
---
- name: Ejemplos de un role con handlers y files
  hosts: debian1
  
  tasks: 

  - name: Primera tarea del play del PLAY
    ansible.builtin.debug:
      msg: "Comienzo el juego"
  
  - name: Incluir el role
    import_role:
      name: mariadb

  - name: Ultima tarea del PLAY
    ansible.builtin.debug:
      msg: "Se ha terminado el proceso {{software}}"
  ```

### 10. TAGS
Nos permiten ejecutar o saltar determinadas tareas de un Playbook. Es más sencillo que usar When. Se pueden añadir a nivel de bloque, play, tarea o incluso rol.
  - __10.1. Tags a nivel de tarea__ :  Supongamos que queremos separar las tareas a nivel de producción y desarrollo.

    ```
    ---
    - name: Trabajar con TAGS
      hosts: debian1
      
      tasks:

      - name: Preparar desarrollo
        ansible.builtin.debug:
          msg: Preparar el entorno de desarrollo
        tags:
            - desarrollo
      
      - name: Preparar producción
        ansible.builtin.debug:
          msg: Preparar el entorno de produccion
        tags:
          - produccion

      - name: Instalar mysql
        ansible.builtin.debug:
          msg: "Instalando mysql"
        tags:
          - desarrollo
          - produccion

      - name: Instalar herramientas desarrollo
        ansible.builtin.debug:
          msg: "Proceso Terminado"
        tags:
            - desarrollo

      - name: Instalar la seguridad de producción
        ansible.builtin.debug:
          msg: "Instalar el entornod de seguridad"
        tags:
          - produccion

      - name: Desplegar aplicacion 
        ansible.builtin.debug:
          msg: "Desplegar aplicación"
        tags:
          - desarrollo
          - produccion
    ```

    - Para ejecutarla en todas las máquinas: ``` ansible-playbook -i maquinas tags.yaml -t all```
     ```
        PLAY [Trabajar con TAGS] ********************************************************************************************************************************************************

        TASK [Gathering Facts] **********************************************************************************************************************************************************
        ok: [debian1]

        TASK [Preparar desarrollo] ******************************************************************************************************************************************************
        ok: [debian1] => {
            "msg": "Preparar el entorno de desarrollo"
        }

        TASK [Preparar producción] ******************************************************************************************************************************************************
        ok: [debian1] => {
            "msg": "Preparar el entorno de produccion"
        }

        TASK [Instalar mysql] ***********************************************************************************************************************************************************
        ok: [debian1] => {
            "msg": "Instalando mysql"
        }

        TASK [Instalar herramientas desarrollo] *****************************************************************************************************************************************
        ok: [debian1] => {
            "msg": "Proceso Terminado"
        }

        TASK [Instalar la seguridad de producción] **************************************************************************************************************************************
        ok: [debian1] => {
            "msg": "Instalar el entornod de seguridad"
        }

        TASK [Desplegar aplicacion] *****************************************************************************************************************************************************
        ok: [debian1] => {
            "msg": "Desplegar aplicación"
        }
     ````
    - Para ejecutarla en las máquinas de desarrollo ```ansible-playbook -i maquinas tags.yaml -t desarrollo ``` 
    ```

      [DEPRECATION WARNING]: Ansible will require Python 3.8 or newer on the controller starting with Ansible 2.12. Current version: 3.6.9 (default, Mar 10 2023, 16:46:00) [GCC 
      8.4.0]. This feature will be removed from ansible-core in version 2.12. Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.

      PLAY [Trabajar con TAGS] ********************************************************************************************************************************************************

      TASK [Gathering Facts] **********************************************************************************************************************************************************
      ok: [debian1]

      TASK [Preparar desarrollo] ******************************************************************************************************************************************************
      ok: [debian1] => {
          "msg": "Preparar el entorno de desarrollo"
      }

      TASK [Instalar mysql] ***********************************************************************************************************************************************************
      ok: [debian1] => {
          "msg": "Instalando mysql"
      }

      TASK [Instalar herramientas desarrollo] *****************************************************************************************************************************************
      ok: [debian1] => {
          "msg": "Proceso Terminado"
      }

      TASK [Desplegar aplicacion] *****************************************************************************************************************************************************
      ok: [debian1] => {
          "msg": "Desplegar aplicación"
      }

      PLAY RECAP **********************************************************************************************************************************************************************
      debian1                    : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
    ```
    - Y en producción: ```ansible-playbook -i maquinas tags.yaml -t produccion```
```
      PLAY [Trabajar con TAGS] ********************************************************************************************************************************************************

      TASK [Gathering Facts] **********************************************************************************************************************************************************
      ok: [debian1]

      TASK [Preparar producción] ******************************************************************************************************************************************************
      ok: [debian1] => {
          "msg": "Preparar el entorno de produccion"
      }

      TASK [Instalar mysql] ***********************************************************************************************************************************************************
      ok: [debian1] => {
          "msg": "Instalando mysql"
      }

      TASK [Instalar la seguridad de producción] **************************************************************************************************************************************
      ok: [debian1] => {
          "msg": "Instalar el entornod de seguridad"
      }

      TASK [Desplegar aplicacion] *****************************************************************************************************************************************************
      ok: [debian1] => {
          "msg": "Desplegar aplicación"
      }

      PLAY RECAP **********************************************************************************************************************************************************************
      debian1                    : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```


## ANSIBLE
### Documentación:
https://docs.ansible.com/developers.html
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


## ANSIBLE
### Documentación:
https://docs.ansible.com/developers.html

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
- Terraform, Chef, Puppet, Salt.

### 3. Arquitectura y Componentes:
- __Control Nodes:__ Servidores maestros desde donde instalo Ansible y se lanzan los comandos contra los servidores controlados.
    Funcionan sobre entornos Linux, aunque puede usar el módulo WSL de Windows.
- __Manages Nodes:__ Servidores que van a ser gestionados por Ansible. Se encuentran dentro de un inventario que nos permite agruparlos y manipularlos de forma sencilla.

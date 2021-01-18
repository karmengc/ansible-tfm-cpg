### Ansible-tfm-cpg

Proyecto de Ansible perteneciente al trabajo de fin de máster de Carmen de Porras, consistente en acondicionamiento de 3 Raspberrys en cluster
con Kubernetes instalado, funcionando uno como máster y los otros dos como nodos.


## Requisitos
Es necesario tener instalado en el host:
  - ansible
  - vagrant
  - molecule-vagrant (mediante pip)
  - molecule (mediante pip)
  - python-vagrant (mediante pip)
  - virtualbox

Añadir la imagen de Ubuntu desde el repositorio de Vagrant a nuestro vagrant local de la siguiente manera:
```
vagrant box add ubuntu/groovy
```

También es ideal añadir los plugins de Vagrant:
```
vagrant plugin install [vagrant-share,vagrant-vbguest]
```

Por último, necesitaremos un usuario en las máquinas destino con el cual hacer SSH y que tenga permisos de administrador.

Para que no sea necesaria la contraseña de sudo añadir a /etc/sudoers:
```
  usuario ALL=(ALL) NOPASSWD:ALL
```

**Vault password**
Aquellas variables que contengan información sensible como contraseñas, se almacenarán en ficheros vault. Los cuales para poder ser
desencriptados durante el despliegue, es necesario ejecutar previamente:

```
export ANSIBLE_VAULT_PASSWORD_FILE=/ruta_del_proyecto/ansible-tfm-cpg/.vault_pass.txt
```

Siendo .vault\_pass.txt un fichero que contendrá la contraseña de vault y que nunca debe ser subido al repositorio git del proyecto (ya se encuentra en .gitignore)



Este proyecto se apoya en los siguientes roles:


## ansible-provision-rpis

Rol que provisiona las Raspberrys a nivel de sistema operativo y paquetes básicos para su funcionamiento.

NOTA IMPORTANTE: para el despliegue de este proyecto en RPIs es necesario editar el fichero /boot/firmware/cmdline.txt y añadir al final de la línea:
```
cgroup_enable=memory cgroup_memory=1
```
O de lo contrario no funcionará Docker correctamente y en consecuencia no arrancará Kubernetes. Para más información ver: https://codestrian.com/index.php/2020/05/23/ubuntu-20-04-rancher-and-raspberry-pi/


# Netplan
Si se desea se puede pasar una configuración para netplan. En caso de introducir alguna contraseña para wifi, por ejemplo,
crear un fichero de vault a partir de los ejemplos host\_vars/nodo/wifi\_secret.yml.sample, así:
```
ansible-vault create host_vars/$NODO$/wifi-secret.sample
```

## ansible-k8s-rpis

Rol que instala Kubernetes en el cluster. Repartiendo los servicios correspondientes entre el máster y los nodos.


## ansible-k8s-deploy

Despliegue de pods y servicios en Kubernetes del cluster para un servicio de ..........


## Ejecutar proyecto

Para ejecutar el proyecto completo introducir en la línea de comandos desde el directorio raíz del proyecto:

```
ansible-playbook main.yml -i hosts
```


## Tests

Para realizar pruebas se han configurado varios escenarios en Molecule con driver de Vagrant.


# Escenario default

Para ejecutar las pruebas en el escenario default:

```
molecule converge --scenario-name default
```

Para loguearse en las máquinas de prueba por SSH:

```
molecule login -h master --scenario-name=default
```

## Iniciar agente SSH

Para evitar que nos pida la contraseña al desplegar los roles con ansible, ejecutar:

```
eval `ssh-agent`
```

```
ssh-add /home/carmen/.ssh/id_rsa_tfm
```

Esta clave RSA ha sido generada previamente con ssh-keygen y copiada a los respectivos dispositivos mediante ssh-copy-id.


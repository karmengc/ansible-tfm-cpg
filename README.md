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


NOTA IMPORTANTE: para el despliegue de este proyecto en RPIs es necesario editar el fichero /boot/firmware/cmdline.txt de cada Raspberry y añadir al final de la línea:
```
cgroup_enable=cpuset cgroup_enable=memory cgroup_memory=1
```
O de lo contrario no funcionará Docker correctamente y en consecuencia no arrancará Kubernetes. Para más información ver: https://codestrian.com/index.php/2020/05/23/ubuntu-20-04-rancher-and-raspberry-pi/


**Vault password**
Aquellas variables que contengan información sensible como contraseñas, se almacenarán en ficheros vault. Los cuales para poder ser
desencriptados durante el despliegue, es necesario ejecutar previamente:

```
export ANSIBLE_VAULT_PASSWORD_FILE=/ruta_del_proyecto/ansible-tfm-cpg/.vault_pass.txt
```

Siendo .vault\_pass.txt un fichero que contendrá la contraseña de vault y que nunca debe ser subido al repositorio git del proyecto (ya se encuentra en .gitignore)

Si no se dispone de servidor de DNS, es necesario modificar el fichero ''/etc/hosts/'' para que al desplegar se relacionen las Ips configuradas con los respectivos hosts. Si las direcciones IPs de las Rapsberrys son diferentes de las máquinas virtuales de pruebas, habrá que adaptar el fichero según estemos haciendo pruebas o desplegando sobre RPIs.

Por ejemplo para hacer un despliegue con molecule de pruebas, comentamos y descomentamos:

```
#IPs RPIs
#192.168.1.61 worker1
#192.168.1.62 worker2
#192.168.1.60 master myweb.cluster.local mattermost.cluster.local dashboard.k8s.ingress.cluster.local drupal.cluster.local
#IPs MVs Molecule
192.168.130.150 master myweb.cluster.local mattermost.cluster.local dashboard.k8s.ingress.cluster.local monitoring.cluster.local
192.168.130.151 worker1
192.168.130.152 worker2
```
Los nombres indicados en la línea de master corresponden con los asociados a los servicios desplegados y que se configuran en el controlador Nginx del cluster de Kubernetes.



Este proyecto se apoya en los siguientes roles:


## ansible-provision-rpis

Rol que provisiona las Raspberrys a nivel de sistema operativo y paquetes básicos para su funcionamiento.

Importante destacar que este rol hará el usuario elegido en la variable provision\_cluster\_user no tenga que poner la contraseña en cada operación sudo.

Para más información ver el fichero README de este rol.


# Netplan
Si se desea se puede pasar una configuración para netplan. En caso de introducir alguna contraseña para wifi, por ejemplo,
crear un fichero de vault a partir de los ejemplos host\_vars/nodo/wifi\_secret.yml.sample, así:
```
ansible-vault create host_vars/$NODO$/wifi-secret.sample
```

## ansible-k8s-install

Rol que instala Kubernetes en el cluster. Repartiendo los servicios correspondientes entre el máster y los nodos.

Para más información ver el fichero README de este rol.


## ansible-k8s-deploy

Despliegue de pods y servicios en Kubernetes en el cluster:

  * Tomcat + MySQL - Stateless
  * Mattermost + PostgreSQL (Almacenamiento persistente)
  * Kubernetes Dashboard (es necesario loguearse con el token que se mostrará en las líneas finales del despliegue)
  * Drupal + MariaDB cluster (Stateful)
  * Prometheus + Grafana
  * NGINX Controller (para controlar el acceso a cada una de las aplicaciones por un único punto de entrada)


## Ejecutar proyecto

Para ejecutar el proyecto completo introducir en la línea de comandos desde el directorio raíz del proyecto:

```
ansible-playbook main.yml -i hosts
```

En caso de que el usuario con el cual conectemos a las máquinas para desplegar necesite introducir la contraseña para realizar operaciones sudo, ejecutaremos la primera vez:
```
ansible-playbook main.yml -i hosts --ask-become-pass

```
A partir de la segunda vez que despleguemos, si todo ha ido bien, el usuario podrá realizar operaciones sudo sin contraseña y no será necesario poner --ask-become-pass al desplegar.


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


## IMPORTANTE
Para desplegar este proyecto en las Raspberrys y que funcione K8s correctamente, es necesario modificar el fichero /etc/firmware/cmdline.txt y añadir lo siguiente:
```
cgroup_enable=cpuset cgroup_enable=memory cgroup_memory=1
```
para comprobar que funciona:

cat /proc/cgroups

Y deberíamos ver lo que hemos habilitado.

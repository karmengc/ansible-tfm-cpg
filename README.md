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


Este proyecto se apoya en los siguientes roles:


## ansible-provision-rpis

Rol que provisiona las Raspberrys a nivel de sistema operativo y paquetes básicos para su funcionamiento.


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
molecule login master -h master --scenario-name=default
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

## Vault password
Aquellas variables que contengan información sensible como contraseñas, se almacenarán en ficheros vault. Los cuales para poder ser
desencriptados durante el despliegue, es necesario ejecutar previamente:

```
export ANSIBLE_VAULT_PASSWORD_FILE=./.vault_pass.txt
```

Siendo .vault\_pass.txt un fichero que contendrá la contraseña de vault y que nunca debe ser subido al repositorio git del proyecto (ya se encuentra en .gitignore)
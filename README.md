### Ansible-tfm-cpg

Proyecto de ansible perteneciente al trabajo de fin de máster de Carmen de Porras, consistente en acondicionamiento de 3 Raspberrys en cluster
con Kubernetes instalado, funcionando uno como máster y los otros dos como nodos.

Este proyecto se apoya en los siguientes roles:


## ansible-provision-rpis

Rol que provisiona las Raspberrys a nivel de sistema operativo y paquetes básicos para su funcionamiento.


## ansible-k8s-rpis

Rol que instala Kubernetes en el cluster. Repartiendo los servicios correspondientes entre el máster y los nodos.


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

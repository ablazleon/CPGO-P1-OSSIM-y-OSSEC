# CPGO-P1-OSSIM-y-OSSEC

Autores:

- Jose Miguel Del Valle Salas
- Adrián Blázquez León

En este repo, documentamos cómo hemos planteado realizar esta prática

# 1- Configurar escenario
# 2- Prueba de funcionamiento del escenario
# 3- Creación de directivas de correlació
# 4- Configuración de agentes de operación

1- Configurar escenario

Se crea una subred interna para conectar el agente con OSSIM, insparándonos en la estrategia de este [tutorial](https://www.brianlinkletter.com/2016/07/how-to-use-virtualbox-to-emulate-a-network/#:~:text=To%20connect%20two%20virtual%20machines,the%20Router%2D1%20virtual%20machine).

Se cambia la ip del agente:

10.0.3.1
255.255.255.0
10.0.3.2:gw


Gateway: 10.0.3.2aluche

Name server address: 192.168.56.1

Para configurar en bridge: settings => network => bridge

Al descargar suricata parece este error:

```
E: Could not get lock /var/lib/dpkg/lock-frontend - open (11: Resource temporarily unavailable)
E: Unable to acquire the dpkg frontend lock (/var/lib/dpkg/lock-frontend), is another process using it?
```

Que se soluciona así:
https://itsfoss.com/could-not-get-lock-error/

```
sudo apt-get install software-properties-common
sudo add-apt-repository ppa:oisf/suricata-stable
sudo apt-get update

sudo apt-get install suricata 
```

```
# Add Apt sources.lst
wget -q -O - https://updates.atomicorp.com/installers/atomic | sudo bash

 # Update apt data
 sudo apt-get update

 # Server
 sudo apt-get install ossec-hids-server

# Agent
sudo apt-get install ossec-hids-agent
```



2- Prueba de funcionamiento del escenario

3- Creación de directivas de correlación

4- Configuración de agentes de operación




# CPGO-P1-OSSIM-y-OSSEC

Autores:

- Jose Miguel Del Valle Salas
- Adrián Blázquez León

En este repo, documentamos cómo hemos planteado realizar esta prática

# 1- Configurar escenario
# 2- Prueba de funcionamiento del escenario
# 3- Creación de directivas de correlación
# 4- Configuración de agentes de operación

1- Configurar escenario

## a. Configurar las direcciones IPs correspondientes tanto en la maquina agente como en la maquina servidor, para que tengan conectividad entre si.

## b. Descargar el agente ossec para la maquina agente y realizar la configuración entre el agente y el servidor OSSEC para permitir el envío de logs

## c. Configurar correctamente el agente OSSEC en su correspondiente fichero de configuración, (indicando el formato de logs que debe monitorizar), y la ubicación del fichero fast.log donde Suricata deposita las alertas de las alarmas. (Sugerencia)Reiniciar el servidor ossec como el agente ossec, cada vez que se realice cambios.

---------------

## a. Configurar las direcciones IPs correspondientes tanto en la maquina agente como en la maquina servidor, para que tengan conectividad entre si.

Primero, pensamos en crear una subred interna para conectar el agente con OSSIM, insparándonos en la estrategia de este [tutorial](https://www.brianlinkletter.com/2016/07/how-to-use-virtualbox-to-emulate-a-network/#:~:text=To%20connect%20two%20virtual%20machines,the%20Router%2D1%20virtual%20machine). Luego, intentamos directametne conectar las dos máquinas a la red local.

- Se cambia la ip del adaptador ethernet de la máquina real:

192.168.56.101
Servidor de dns:212.166.210.80

- Se cambia la ip del agente:

192.168.56.103
255.255.255.0
192.168.56.101:gw

Para que entre en fucnionamiento el adpatador se apaga y se enciende.

- Se cambia la ip del servidor:

Gateway: 192.168.56.101

Name server address: 192.168.56.102

Para configurar en bridge: settings => network => bridge

Se realiza un ping y se abre el dashboard de ossim en el agetne, para comprobar que existe conectividad.

## b. Descargar el agente ossec para la maquina agente y realizar la configuración entre el agente y el servidor OSSEC para permitir el envío de logs

- jailbreak teh system and add an agent

https://www.ossec.net/docs/manual/agent/index.html

- configure the agent

- isntall osse server

## c. Configurar correctamente el agente OSSEC en su correspondiente fichero de configuración, (indicando el formato de logs que debe monitorizar), y la ubicación del fichero fast.log donde Suricata deposita las alertas de las alarmas. (Sugerencia)Reiniciar el servidor ossec como el agente ossec, cada vez que se realice cambios.

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




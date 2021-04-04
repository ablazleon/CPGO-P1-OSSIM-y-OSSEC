# CPGO-P1-OSSIM-y-OSSEC

Autores:

- Jose Miguel Del Valle Salas
- Adrián Blázquez León

En este repo, documentamos cómo hemos planteado realizar esta práctica. A continuación una pequeña demo:

https://www.youtube.com/watch?v=ElDoEgs9kSI&t=1s

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

192.168.1.103
255.255.255.0
192.168.1.101:gw

Para que entre en fucnionamiento el adpatador se apaga y se enciende.

- Se cambia la ip del servidor:

Gateway: 192.168.1.101

Name server address: 192.168.1.102

Para configurar en bridge: settings => network => bridge

Se realiza un ping y se abre el dashboard de ossim en el agetne, para comprobar que existe conectividad.

## b. Descargar el agente ossec para la maquina agente y realizar la configuración entre el agente y el servidor OSSEC para permitir el envío de logs

- jailbreak teh system and add an agent

https://www.ossec.net/docs/manual/agent/index.html

```
Starting shell
alienvault:~# /var/ossec/bin/manage_agents 


****************************************
* OSSEC HIDS v2.9.1 Agent manager.     *
* The following options are available: *
****************************************
   (A)dd an agent (A).
   (E)xtract key for an agent (E).
   (L)ist already added agents (L).
   (R)emove an agent (R).
   (Q)uit.
Choose your action: A,E,L,R or Q: e

Available agents: 
   ID: 001, Name: ubuntu18, IP: 192.168.56.102
Provide the ID of the agent to extract the key (or '\q' to quit): 001

Agent key information for '001' is: 
MDAxIHVidW50dTE4IDE5Mi4xNjguNTYuMTAyIDhkMjJjYzY4YjQwZTc5ZmQ4MzZmNzVkZTNhODRlOGY0NDRkNDY5ZmUzN2UzMjFjMDcwY2MwNWY3MGU3MTc2MGE=

** Press ENTER to return to the main menu.
```
- configure the agent

```
# Add Apt sources.lst
wget -q -O - https://updates.atomicorp.com/installers/atomic | sudo bash

 # Update apt data
 sudo apt-get update

# Agent
sudo apt-get install ossec-hids-agent
```

```
sudo /var/ossec/bin/manage_agents


****************************************
* OSSEC HIDS v3.6.0 Agent manager.     *
* The following options are available: *
****************************************
   (I)mport key from the server (I).
   (Q)uit.
Choose your action: I or Q: I

* Provide the Key generated by the server.
* The best approach is to cut and paste it.
*** OBS: Do not include spaces or new lines.

Paste it here (or '\q' to quit): MDAxIHVidW50dTE4IDE5Mi4xNjguNTYuMTAyIDhkMjJjYzY4YjQwZTc5ZmQ4MzZmNzVkZTNhODRlOGY0NDRkNDY5ZmUzN2UzMjFjMDcwY2MwNWY3MGU3MTc2MGE=

```

After this, it appears some error about no file in ossec/queue/rids. To solve this is repeated teh importing in the agent, afeter creating an empty file called sender

```
root@ubuntu-VirtualBox:/var/ossec# cd queue/rids/
root@ubuntu-VirtualBox:/var/ossec/queue/rids# ls
root@ubuntu-VirtualBox:/var/ossec/queue/rids# nano sender
root@ubuntu-VirtualBox:/var/ossec/queue/rids# ls
sender
root@ubuntu-VirtualBox:/var/ossec/queue/rids# cd /var/ossec/bin/manage_agents 
bash: cd: /var/ossec/bin/manage_agents: Not a directory
root@ubuntu-VirtualBox:/var/ossec/queue/rids# /var/ossec/bin/manage_agents 
```

After this way, it's checked that it does not work, it is follwoed the followign tutorial. Then is apprecitaed that it appers the status active:

https://blog.ichasco.com/ossim-instalar-un-agente-en-linux/

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
root@ubuntu-VirtualBox:/var/ossec/etc# nano ossec.conf 

<!-- OSSEC example config -->

<ossec_config>
  <client>
    <server-ip>192.168.1.103</server-ip>
  </client>

```
Then, agent and server services are reboot

En el suricata.yaml se añade la ip de la red 192.168.1.0/8

Después se ejecuta el pcap en suricata:

```
root@ubuntu-VirtualBox:/home/ubuntu/Documents# sudo suricata -c /etc/suricata/suricata.yaml -r /home/ubuntu/Documents/2016-08-16-Neutrino-EK.pcap 
2/4/2021 -- 14:00:55 - <Notice> - This is Suricata version 6.0.2 RELEASE running in USER mode
2/4/2021 -- 14:00:55 - <Warning> - [ERRCODE: SC_ERR_NO_RULES(42)] - No rule files match the pattern /var/lib/suricata/rules/suricata.rules
2/4/2021 -- 14:00:55 - <Warning> - [ERRCODE: SC_ERR_NO_RULES_LOADED(43)] - 1 rule files specified, but no rules were loaded!
2/4/2021 -- 14:00:55 - <Notice> - all 3 packet processing threads, 4 management threads initialized, engine started.
2/4/2021 -- 14:00:55 - <Notice> - Signal Received.  Stopping engine.
2/4/2021 -- 14:00:55 - <Notice> - Pcap-file module read 1 files, 692 packets, 546542 bytes

```

Beacus eof the error certain rules are defined: Based on this article
https://alibaba-cloud.medium.com/how-to-install-suricata-ids-on-ubuntu-16-04-b6dcca70472c

```
ls  /etc/suricata/rules
nano /etc/suricata/suricata.yaml

HOME_NET: "[192.168.0.0/16]"
EXTERNAL_NET: "!$HOME_NET"

nano /etc/suricata/rules/my.rules

alert icmp any any -> $HOME_NET any (msg:"ICMP connection attempt"; sid:1000002; rev:1;) 
alert tcp any any -> $HOME_NET 22 (msg:"SSH connection attempt"; sid:1000003; rev:1;)
alert tcp any any -> $HOME_NET 80 (msg:"DOS Unusually fast port 80 SYN packets outbound, Potential DOS"; flags: S,12; threshold: type both, track by_dst, count 500, seconds 5; classtype:misc-activity; sid:6;)

nano /etc/suricata/suricata.yaml

- my.rules

root@ubuntu-VirtualBox:/home/ubuntu/Documents# sudo suricata -c /etc/suricata/suricata.yaml -r /home/ubuntu/Documents/2016-08-16-Neutrino-EK.pcap 
2/4/2021 -- 14:10:00 - <Notice> - This is Suricata version 6.0.2 RELEASE running in USER mode
2/4/2021 -- 14:10:00 - <Warning> - [ERRCODE: SC_ERR_NO_RULES(42)] - No rule files match the pattern /var/lib/suricata/rules/suricata.rules
2/4/2021 -- 14:10:00 - <Warning> - [ERRCODE: SC_ERR_NO_RULES(42)] - No rule files match the pattern /var/lib/suricata/rules/my.rules
2/4/2021 -- 14:10:00 - <Warning> - [ERRCODE: SC_ERR_NO_RULES_LOADED(43)] - 2 rule files specified, but no rules were loaded!
2/4/2021 -- 14:10:00 - <Notice> - all 3 packet processing threads, 4 management threads initialized, engine started.
2/4/2021 -- 14:10:00 - <Notice> - Signal Received.  Stopping engine.
2/4/2021 -- 14:10:00 - <Notice> - Pcap-file module read 1 files, 692 packets, 546542 bytes

```

AL escribri el comadno así, el fast.log se queda en docuemtns, por lo que hay que modificar el ossec.conf


```
  <localfile>
    <log_format>snort-full</log_format>
    <location>/etc/suricata/fast.log</location>
  </localfile>


/var/ossec/bin/ossec-control restart


```

Aún así, dice suricata que falta actualizar las reglas

```
sudo suricata-update
```

Después se ejcuta dos veces suricata con el pcap que permite simular compartamiento mailciono en la red

sudo suricata -c /etc/suricata/suricata.yaml -r /home/ubuntu/Documents/2016-08-16-Neutrino-EK.pcap

Con menos -l se pondría el sitio por defecto para lso logs

Se compreuba que aparecen en el fichero /etc/suricata/fast.log

CLaro, nótese que dónde se jecuta el pcap es donde aprece el fast.log, y este directorio es el que tiene que estar logado en ossec.conf


A cotniución, se observa en las imágenes como la primera vez que se ejecuta suricata aparece una alerta, y la segunda eventos.

En la consola en Analysis > SIEM

![SIEM alerta](https://github.com/ablazleon/CPGO-P1-OSSIM-y-OSSEC/blob/main/13a4ba7a-924f-4d52-8159-4fae874bd171.jpg)
![SIEM eventos](https://github.com/ablazleon/CPGO-P1-OSSIM-y-OSSEC/blob/main/2c11db1e-d2a5-4ac9-b94e-c17101e039eb.jpg)


2- Prueba de funcionamiento del escenario

Las capturas que se han realziado antes, prvienen de hacer la prueba, comprobando que dispara una alerta en el fast.log

```
08/16/2016-22:41:59.586501  [**] [1:2018216:5] ET INFO HTTP Connection To DDNS Domain Hopto.org [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.4.202:55404 -> 83.217.27.178:80
08/16/2016-22:42:00.182956  [**] [1:2022962:4] ET EXPLOIT_KIT Evil Redirector Leading to EK Jul 12 2016 [**] [Classification: Exploit Kit Activity Detected] [Priority: 1] {TCP} 83.217.27.178:80 -> 192.16$
08/16/2016-22:42:03.025192  [**] [1:2014726:127] ET POLICY Outdated Flash Version M1 [**] [Classification: Potential Corporate Privacy Violation] [Priority: 1] {TCP} 192.168.4.202:55416 -> 74.208.103.8:80
08/16/2016-22:42:03.025474  [**] [1:2031747:1] ET HUNTING Observed Interesting Content-Type Inbound (application/x-sh) [**] [Classification: Potential Corporate Privacy Violation] [Priority: 1] {TCP} 74.$
08/16/2016-22:42:09.866279  [**] [1:2031747:1] ET HUNTING Observed Interesting Content-Type Inbound (application/x-sh) [**] [Classification: Potential Corporate Privacy Violation] [Priority: 1] {TCP} 74.$
08/16/2016-22:41:59.586501  [**] [1:2018216:5] ET INFO HTTP Connection To DDNS Domain Hopto.org [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.4.202:55404 -> 83.217.27.178:80
08/16/2016-22:42:00.182956  [**] [1:2022962:4] ET EXPLOIT_KIT Evil Redirector Leading to EK Jul 12 2016 [**] [Classification: Exploit Kit Activity Detected] [Priority: 1] {TCP} 83.217.27.178:80 -> 192.16$
08/16/2016-22:42:03.025192  [**] [1:2014726:127] ET POLICY Outdated Flash Version M1 [**] [Classification: Potential Corporate Privacy Violation] [Priority: 1] {TCP} 192.168.4.202:55416 -> 74.208.103.8:80
08/16/2016-22:42:03.025474  [**] [1:2031747:1] ET HUNTING Observed Interesting Content-Type Inbound (application/x-sh) [**] [Classification: Potential Corporate Privacy Violation] [Priority: 1] {TCP} 74.$
08/16/2016-22:42:09.866279  [**] [1:2031747:1] ET HUNTING Observed Interesting Content-Type Inbound (application/x-sh) [**] [Classification: Potential 
```

3- Creación de directivas de correlación

Para probar esta funcionalidad se propone crear unas directivas de correlación muy simples que
disparen una alarma cuando se hayan dado 2 o más ocurrencias de uno de los eventos anteriores
en menos de 1 minuto. Escoge una de las alertas que genera Suricata al pasarle el pcap con
contenido malicioso y crea la/s directivas de correlación adecuadas. Si está/n correctamente
creadas, al generar 2 o más alertas de Suricata (inyectándole el fichero malicioso 2 o más veces)
se disparará un evento nuevo y una alerta en OSSIM.
Así, la primera ejecución de suricata con el pcap desde la máquina agente no disparará ninguna
alarma. Con la segunda ejecución, deberá aparecer la alarma creada por el alumno.

Se configura esta directa en Configuración > Open Threat Intelligence > Directive

It is found form the real time whcih details they are and from it it is configured the desired correlation directive.

Regla 1
Explotation and installation
Maliciuos Website
Uno

3
regla 1
7003
20101

Se comprueba quecuando se ejcuta dos veces el fichero pcap. Nótese que por ello, depsués de que se ejecuten dos veces seguidas el fichero pcap, esta segunda vez tiene que tardar menso que un minuto. Se carga la directiva.

4- Configuración de agentes de operación

En la pantalla de administración a crear usuario operador.

Se crea un operador. Se le da permissos para acceder al ticket.

=> regla de correlación adiocional:

como tarda bastatne, una directiva 2 de forma de que con occurencia 1 sale si en menos de tres minutos



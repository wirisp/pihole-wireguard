# Instalacion de Pihole + wireguard + unbound
Pihole es un bloqueador de anuncios y wireguard permite realizar un tunnel vpn, por lo que se puede instalar pihole en un vps remotamente y en el dispocitivo local acceder a ese vpn

### En ubuntu pihole+wireguard

- Actualizamos el sistema e instalamos algunos paquetes
```
sudo su 
apt update && apt install dnsutils git nano git -y
```
```
git clone https://github.com/wirisp/pihole-wireguard.git pi
```
```
cd pi
mv * /root
cd ..
chmod +x *.sh
```

```
bash piwire.sh
```

### Para almalinux
```
sudo su
dnf makecache --refresh
```

```
setenforce 0
sed -i 's/^SELINUX=.*/SELINUX=permissive/g' /etc/selinux/config
```

```
sudo dnf install bind-utils nano git
```

- Cambiamos la zona horaria
```
timedatectl set-timezone America/Mexico_City
```
- Clonamos este repositorio que contiene los scripts necesarios para la instalacion
```
git clone https://github.com/wirisp/pihole-wireguard.git pi
```
- Accedemos a la carpeta , movemos los archivos a /root y ejecutamos el script piwire, el cual contiene pihole + wireguard
```
cd pi && mv * /root && cd && chmod +x *.sh
sudo bash wireguard.sh --auto
#sudo bash wireguard.sh
```
- Si el vps no soporta ipv6 dara error
Introducir el siguiente comando el cual dara error
```
sudo systemctl restart wg-quick@wg0.service
```
Para ello editamos un archivo
```
nano /etc/sysctl.conf
```
Colocamos dentro / cambiamos , desactivando ipv6 a 0
```
net.ipv6.conf.all.disable_ipv6 = 0
net.ipv4.ip_forward = 1
```
Desactivamos wg0
```
wg-quick down wg0
```
Ahora iniciamos el servicio
```
sudo systemctl restart wg-quick@wg0.service
```

- Para instalar pihole en almalinux
```
curl -sSL https://install.pi-hole.net | sudo bash
 ```
Nos dara un error y ahora ejecutamos
```
curl -sSL https://install.pi-hole.net | PIHOLE_SKIP_OS_CHECK=true sudo -E bash
```

En la instalacion de pihole seleccionar la interfaz `wg0`

Puedes cambiar la contraseña de pihole con:
```
pihole -a -p
```
```
chmod 644 /var/log/pihole.log
```

Acceder al interfazo con `IP/admin` donde 
**IP** = Ip de tu servidor

En la administracion del servidor abrir el puerto`5335,8080,80,51820`, si no funciona abrir todos los puertos.
# Instalacion unbound almalinux
```
dnf -y install unbound bind-utils
```

```
sed -i '/interface: 0.0.0.0$/s/#//' /etc/unbound/unbound.conf
sed -i 's/127.0.0.0\/8 allow/10.0.0.0\/24 allow/' /etc/unbound/unbound.conf
```

```
unbound-checkconf
```

```
systemctl enable unbound
service unbound start
sudo systemctl status unbound
```

```
firewall-cmd --permanent --add-service dns
firewall-cmd --reload
```

# Instalacion de unbound ubuntu
unbound permite usar los DNS localmente y asi poder usar el bloqueador y los DNS propios, esto agiliza y acelera la respuesta.
```
./unbound.sh
```
Accedemos a `IP/admin/settings.php?tab=dns` y desmarcamos los que tenemos
Colocamos `127.0.0.1#5335` en Custom 1 y en Custom 2, marcamos
Seleccionamos > Respond only on interface wg0
Guardamos los cambios

## Lista de bloqueo
Accedemos a `IP/admin/groups-adlists.php`
Agregamos `https://raw.githubusercontent.com/vincentkenny01/spotblock/master/spotify` con nombre `Spotify`
Actualizamos gravity en Tools> Update Gravity o en `IP/admin/gravity.php`

## Activar Safesearch en los buscadores
<img width="962" alt="Safesearch pihole" src="https://user-images.githubusercontent.com/13319563/216865618-378bb0b4-4353-4aa6-8c2d-af38bbc23bb8.png">

#Safesearch de google
`./safesearch.sh`

Reiniciamos el sistema.

#cambiar pass a pihole.
pihole -a -p

# Generar usuario Wireguard
`./piwire.sh`

# Usar Pihole + wireguard + unbound con Mikrotik Routeros
- Supongamos que el archivo de cliente wireguard contiene estos datos.

```
[Interface]
Address = 10.2.53.2/32, fc10:253::2/128
DNS = 10.2.53.1, fc10:253::1
PrivateKey = EFKJRbmjXKwCIelIbmTfpNire/O8y+E1eDLs/UoAnl8=

[Peer]
Endpoint = 143.198.106.172:51820
PersistentKeepalive = 25
PublicKey = uiLn89yy/fNGrA2zmGiwD4BndqSMJ5Let/hfR91I9F8=
PresharedKey = ruM0RuSduDOYwRw0XhZ6hlrcbq+fOAzQCPM5SqeQDXw=
AllowedIPs = 10.2.53.1/32, fc10:253::1/128
```
- Entonces nuestro script de configuracion de mikrotik quedara asi:
Para crear la interfaz y el peer con:

```
/interface wireguard
add listen-port=13231 mtu=1420 name=piwire private-key="EFKJRbmjXKwCIelIbmTfpNire/O8y+E1eDLs/UoAnl8="
/interface wireguard peers
add allowed-address=10.2.53.1/32 endpoint-address=143.198.106.172 endpoint-port=\
    51820 interface=piwire persistent-keepalive=25s public-key=\
    "uiLn89yy/fNGrA2zmGiwD4BndqSMJ5Let/hfR91I9F8=" preshared-key="ruM0RuSduDOYwRw0XhZ6hlrcbq+fOAzQCPM5SqeQDXw="
```
- Para la direccion Ip
```
/ip address
add address=10.2.53.2/24 interface=piwire network=10.2.53.0
```
- Cambia los Dns en 
```
/ip dns set servers=10.2.53.1
```
## Script cambio automatico en mikrotik
Ahora automatizaremos para cuando pihole este funcionando tome las Dns de pihole y cuando no funcione o este caido, tome las de google.

- Para ello necesitamos colocarle un **comentario** a los networks que tengas en `/ip dhcp-server network` por ejemplo para el primero `networkdns1` Ya que usaremos el comentario para realizar el cambio por medio de busqueda en el script.

- Tambien colocaremos en una lista al grupo de direcciones a cuales afectara, en mi caso tengo 2 la `172.19.0.0/24` para pppoe y `192.168.19.0/24` para el hotspot
```
/ip firewall address-list
add address=172.19.0.0/24 list=LAN
add address=192.168.19.0/24 list=LAN
```

- Regla nat que funcionara en conjunto para el cambio de Dns automatico

<img width="729" alt="Reglas firewall nat" src="https://user-images.githubusercontent.com/13319563/216865517-3c85abe3-c3b7-4cc6-b4f9-9b454fb4895b.png">

```
/ip firewall nat
add action=dst-nat chain=dstnat comment=REGLA-1 dst-port=53 protocol=udp \
    src-address-list=LAN to-addresses=10.2.53.1
add action=dst-nat chain=dstnat comment=REGLA-2 dst-port=53 protocol=tcp \
    src-address-list=LAN to-addresses=10.2.53.1
```
- Script que esta concatenado al netwatch , se ejecutara uno de los dos dependiendo si esta, up o down,si tienes mas de 1 network agregarlo segun su comentario `networkdns1`

<img width="483" alt="Comentario network" src="https://user-images.githubusercontent.com/13319563/216865019-5dd29ac6-72b2-46e1-901b-619d047d992d.png">


```
/system script
add dont-require-permissions=yes name=add-pihole-dns owner=Rivera policy=\
    ftp,reboot,read,write,policy,test,password,sniff,sensitive,romon source="/\
    ip firewall nat set to-addresses=10.2.53.1 [find comment=\"REGLA-1\"]\r\
    \n/ip firewall nat set to-addresses=10.2.53.1 [find comment=\"REGLA-2\"]\r\
    \n/ip dhcp-server network set dns-server=10.2.53.1 [find comment=\"network\
    dns1\"]\r\
    \n/ip dns set servers=10.2.53.1"
add dont-require-permissions=yes name=remove-pihole-dns owner=Rivera policy=\
    ftp,reboot,read,write,policy,test,password,sniff,sensitive,romon source="/\
    ip firewall nat set to-addresses=8.8.8.8 [find comment=\"REGLA-1\"]\r\
    \n/ip firewall nat set to-addresses=8.8.8.8 [find comment=\"REGLA-2\"]\r\
    \n/ip dhcp-server network set dns-server=192.168.19.1,8.8.8.8,8.8.4.4 [fin\
    d comment=\"networkdns1\"]\r\
    \n/ip dns set servers=8.8.8.8"
```
- Netwatch que estara checando si esta up o down.
```
/tool netwatch
add disabled=no down-script=remove-pihole-dns host=10.2.53.1 http-codes="" \
    interval=10s test-script="" timeout=2s type=simple up-script=\
    add-pihole-dns
```

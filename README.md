# Instalacion de Pihole + wireguard + unbound
Pihole es un bloqueador de anuncios y wireguard permite realizar un tunnel vpn, por lo que se puede instalar pihole en un vps remotamente y en el dispocitivo local acceder a ese vpn
la instalacion es realizada en un sistema ubuntu

- Actualizamos el sistema e instalamos algunos paquetes
```
sudo su 
apt update && apt install git nano -y
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
./piwire.sh
```
En la instalacion de pihole seleccionar la interfaz `wg0`

Puedes cambiar la contraseña de pihole con:
`pihole -a -p`

Acceder al interfazo con `IP/admin` donde 
**IP** = Ip de tu servidor

En la administracion del servidor abrir el puerto`5335,8080,80,51820`, si no funciona abrir todos los puertos.

# Instalacion de unbound
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

#Safesearch de google
`./safesearch`

Reiniciamos el sistema.

#cambiar pass a pihole.
pihole -a -p

# Generar usuario Wireguard
`./piwire.sh`

# Usar Pihole + wireguard + unbound con Mikrotik Routeros

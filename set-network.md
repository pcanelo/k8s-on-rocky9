# Cambiar interfaz de red en los nodos sin GUI de AlmaLinux y RockyLinux

Para cambiar la dirección IP de una máquina con AlmaLinux que no tiene interfaz gráfica, necesitarás acceder a la terminal o línea de comandos. A continuación, te muestro cómo puedes hacerlo utilizando la herramienta `nmcli`, que es el gestor de red en línea de comandos para NetworkManager, comúnmente disponible en distribuciones como AlmaLinux.

### 1. Identificar la interfaz de red: Primero, necesitas saber qué interfaces de red están disponibles en tu sistema. Puedes hacerlo con el siguiente comando:
```
    nmcli device status 
```
    Este comando te mostrará las interfaces disponibles y su estado actual.

### 2. Modificar la dirección IP: Una vez que identifiques la interfaz de red correcta (por ejemplo, ens192), puedes modificar su configuración de IP. Aquí tienes cómo establecer una nueva dirección IP estática:
    Para configurar la dirección IP:
```
    nmcli con mod ens192 ipv4.addresses 192.168.1.100/24 
```
    Para configurar la puerta de enlace (gateway):
```
    nmcli con mod ens192 ipv4.gateway 192.168.1.1 
```
    Para configurar los servidores DNS:
```
    nmcli con mod ens192 ipv4.dns "8.8.8.8,8.8.4.4" 
```
    Asegúrate de reemplazar ens192 con el nombre de tu interfaz de red, y los valores de IP, gateway y DNS con los que correspondan a tu red.

### 3. Activar la configuración de IPv4 y desactivar DHCP (si es necesario): Si deseas usar una configuración estática y no DHCP, necesitas desactivar DHCP y activar el método manual:
```
    nmcli con mod ens192 ipv4.method manual 
```

### 4. Reiniciar la conexión de red: Para aplicar los cambios, reinicia la conexión de red:
    ```bash
    nmcli con up ens192
    ```
    Este comando desactivará y reactivará la interfaz de red, aplicando los cambios de configuración.

Estos pasos deben permitirte cambiar la dirección IP y otros aspectos relacionados de la configuración de red en un sistema AlmaLinux o Rocky Linux sin interfaz gráfica. Recuerda ajustar los comandos según las características específicas de tu red y sistema.

## Crear las maquinas virtuales en vmware o virtual box usando rocky Linux 9.3
Usa el hypervisor de tu agrado o que tengas instalado y crea 1 vm con 
6gb ram
4 cores
100 gb disco
Network bridge

### Despues de crear la VM 
ejecutar 
```
dnf update -y 
```
crea un usuario sysops con caracteristicas de sudoers
debe poder ejecuatr sudo sin password

#### Clonar la vm 
Para clonar la VM debes detenerla y luego clonarla 2 veces, asi tendras un control plane y 2 workers

#### Si vas a usar ansible, clona una mas para instalar ansible, le bajas la ram a 2 gb con 2 cores y 40gb disco, asi puedes automatizar la configuracion de las otras vms 

# ATENCION 
si el clone no se incia, en el modo de emergencia en la máquina clonada ingresa la contraseña de root, luego abres el archivo /etc/lvm/lvm.conf configurando use_devicesfile=0 
y reinicia.
```
nano  /etc/lvm/lvm.conf
```
busca con Control-W use_devicesfile , descomentalo y cambia el 1 por 0 



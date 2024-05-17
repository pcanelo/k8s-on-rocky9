# Laboratorio Kubernetes en Rocky Linux 9.3 
## Configuración del laboratorio 

## Paso 1 crea las vms en local, nube de tu eleccion a tu ambiente de virtualizacion 
Vamos a usar 3 makinas virtuales creadas en virtual box que seran nuestros nodos

| Nombre        | IP           | Rol                |
|---------------|--------------|--------------------|
| vm-k8s-cp01-lab  | 192.168.1.70 | Nodo Control Plane |
| vm-k8s-wk01-lab  | 192.168.1.71 | Nodo Worker01 |
| vm-k8s-wk02-lab  | 192.168.1.72 | Nodo Worker02 |


Definir usuario sudo en cada nodo nombrelo como: admin  sysops o como se te pare en ganas

## Paso 2 configurar el nombre de cada host y actualizar el archivo de hosts
Para esto debes entar a cada nodo, inicia sesión o utilice ssh en cada máquina y ejecute los comandos hostnamectl para configurar su respectivo nombre de host.
En nodo de plano de control ejecuta 
```
sudo hostnamectl set-hostname “k8s-cp01-lab” && exec bash
```
En nodo worker 1 
```
sudo hostnamectl set-hostname “k8s-wk01-lab” && exec bash
```
En nodo worker 2 
```
sudo hostnamectl set-hostname “k8s-wk02-lab” && exec bash
```
debes poner las siguientes entradas en los archivos hosts que estan en /etc/hosts de cada nodo
| IP     / nombre host      |     
|--------------| 
|``` 192.168.1.70  k8s-cp01-lab ```|
|``` 192.168.1.71  k8s-wk01-lab ```|
|``` 192.168.1.72  k8s-wk02-lab ```|

## Paso 3: deshabilite el swap space en cada nodo para que kubelet funcione sin problemas
```
sudo dnf update -y
sudo swapoff -a 
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

## Paso 4: Ajuste las reglas de SELinux y Firewall para Kubernetes
debes configure el modo SELinux como permisivo en todos los nodos usando los siguientes comandos
```
 sudo setenforce 0
 sudo sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=permissive/g' /etc/sysconfig/selinux
```

### En el nodo del plano de control (cp), permita los siguientes puertos en el firewall.
```
sudo firewall-cmd --permanent --add-port={6443,2379,2380,10250,10251,10252,10257,10259,179}/tcp
sudo firewall-cmd --permanent --add-port=4789/udp
sudo firewall-cmd --reload
```
### En los nodos worker (wk), permita los siguientes puertos en el firewall.
```
sudo firewall-cmd --permanent --add-port={179,10250,30000-32767}/tcp
sudo firewall-cmd --permanent --add-port=4789/udp
sudo firewall-cmd --reload
```

## Paso 5: Agregar Módulos y Parámetros del Kernel
Para el clúster de Kubernetes, debemos agregar los módulos del kernel overlay y br_netfilter en todos los nodos.
Crea un archivo y agrega el siguiente contenido:
```
sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF
```
#### Para cargar los módulos mencionados anteriormente, ejecuta:
```
sudo modprobe overlay
sudo modprobe br_netfilter
```

#### A continuación, agrega los siguientes parámetros del kernel, crea un archivo con el siguiente contenido:
```
sudo vi /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
```
Guarde y cierre el archivo.

#### Ahora añada estos parámetros ejecutando el siguiente comando
```
sudo sysctl --system
```
## Paso 6: Instalar un Runtime de alto nivel --> Containerd 
##### Kubernetes requiere un runtime para la ejecución de contenedores, y una de las opciones más populares es containerd. Pero no está disponible en los repositorios de paquetes por defecto de Rocky Linux, así que lo agregaremos, usa el siguiente repositorio docker en todos los nodos.
```
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```
##### Ahora, ejecute el siguiente comando dnf para instalar containerd en todos los nodos.
```
sudo dnf install containerd.io -y
```
#### Configura containerd para que utilice "systemdcgroup". Ejecuta los siguientes comandos en cada nodo.

```
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
```
Reinicie y active el servicio containerd utilizando los siguientes comandos,
```
sudo systemctl restart containerd
sudo systemctl enable containerd
```
Verifique el estado del servicio conatinerd, ejecute
```
sudo systemctl status containerd
```
# Ahora instalaremos kubernetes

## Paso 7:  Instalar herramientas Kubernetes
OJO: Las herramientas Kubernetes como Kubeadm, kubectl y kubelet no están disponibles 
en los repositorios de paquetes por defecto de Rocky Linux 9. 
Por lo tanto, para instalar estas herramientas, agrega el siguiente repositorio en todos los nodos
```
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.28/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.28/rpm/repodata/repomd.xml.key
exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni
EOF
```
##### hoy abril 2024 ya esta disponible la version 1.29, pero usaremos la versión 1.28 de Kubernetes, es por eso que ves la versión 1.28 al agregar el repositorio.

A continuación, instala las herramientas de Kubernetes ejecutando el siguiente comando dnf,
```
sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
```
Después de instalar las herramientas de Kubernetes, inicia el servicio kubelet en cada nodo.
```
sudo systemctl enable --now kubelet
```

## Paso 8: Instalemos el un Cluster de Kubernetes en Rocky Linux 9
Ahora ya estamos ready para instalar el clúster de Kubernetes. Ejecute el comando Kubeadm para inicializar el clúster de Kubernetes desde el nodo maestro.
```
sudo kubeadm init --control-plane-endpoint=k8s-cp01-lab
```
#### OJO: Al terminar el comando anterior te entregara el comando que debes ejecutar en cada node worker como root
el comando comienza con pero aun no lo ejecutes 
 
###### kubeadm join k8s-cp01-lab:6443   --token    ...............    --discovery-token-ca-cert-hash   .............
 

Para comenzar a interactuar con el clúster de Kubernetes, ejecute los siguientes comandos en el nodo cp.
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Ahora uniremos los nodos workers al clúster y ejecute el siguiente comando Kubeadm en cada nodo worker.
```
kubeadm join a k8s-cp01-lab:6443 --token xxxxxxxxxxxxxxxxx \
  --discovery-token-ca-cert-hash sha256:nnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnn
```

Ahora, vuelve al nodo cp y ejecuta el comando kubectl para verificar el estado de los nodos.
ejecuta 
```
kubectl get nodes
```
La salida anterior muestra que los nodos son "NoteRead", por lo que para hacer que el estado de los nodos sea "Ready", instale el addon o plugin de red Calico en el siguiente paso.

## Paso 9: Instalar Calico Network Addon
El complemento de red Calico es necesario en el clúster Kubernetes para permitir la comunicación entre pods, 
para que el servicio DNS funcione con el clúster y para que el estado de los nodos sea Ready.

Para instalar el complemento calico CNI (Container Network Interface), ejecute los siguientes comandos kubectl solamente en el nodo cp.
```
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/calico.yaml
```

Verify el status de los pods calico:
```
kubectl get pods -n kube-system
```
A continuación, verifique el estado de los nodos, esta vez el estado de los nodos debe estar en Ready State.

```
kubectl get nodes
```
#### Perfecto, la salida anterior debe confirmar que los nodos están en estado Ready y pueden manejar o aceptar cargas de trabajo. 
#### Vamos a probar nuestra instalación de Kubernetes en el siguiente paso.

## Paso 10: Probar la instalación del clúster Kubernetes
Para probar la instalación del clúster Kubernetes, vamos a intentar desplegar la aplicación basada en nginx usando un deployment. 
Ejecute uno a uno los siguientes comandos kubectl,
```
kubectl create deployment web-app01 --image nginx --replicas 2
kubectl expose deployment web-app01 --type NodePort --port 80
kubectl get deployment web-app01
kubectl get pods
kubectl get svc web-app01
```
Intente acceder a la aplicación utilizando nodeport "399911" o el que a ti te aparezca, ejecute el siguiente comando curl,
```
curl k8s-wk01-lab:31121
```
si todo sale bien entonces espectacular, todo lo anterior confirma que podemos acceder a la página web de nuestra aplicación. 
Esto también confirma que nuestro clúster Kubernetes se ha instalado correctamente.

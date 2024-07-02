---
title: "Despliegue de WordPress con MetalLB y sistema de archivos NFS sobre un Cluster Multi-Master con Kubespray y HAProxy"
date: 2024-06-28 12:13:02 - 5000
categories: [DevOps, Kubernetes]
tags: [DevOps, Kubespray, Kuberentes, Ansible, Kubeadm, HAProxy, WordPress, MetalLB, NFS]
image: /assets/img/posts/2024/DevOps/Despliegue de cluster multi-master con kubespray y haproxy/kubespray.png
alt: "Image alt text"
description: Laboratorio local para desplegar un Cluster Multi-Master con HAProxy utilizando Kubespray
render_with_liquid: false
---

## 1. Conceptos:
### Kubespray
> [Kubespray](https://github.com/kubernetes-sigs/kubespray) es una herramienta de despliegue de Kubernetes basada en [Ansible](https://docs.ansible.com/) y [Kubeadm](https://kubernetes.io/docs/reference/setup-tools/kubeadm/). Facilita la instalación y configuración de clústeres Kubernetes altamente disponibles y escalables en cualquier infraestructura (bare metal, nube pública, nube privada, o VM). Utiliza playbooks de Ansible para automatizar el proceso de configuración, instalación y actualización del clúster, proporcionando flexibilidad y personalización a través de un archivo de inventario y variables específicas.
#### Características:
1. **Automatización Completa**: Automatiza el despliegue, configuración y mantenimiento de clústeres Kubernetes.
2. **Alta Disponibilidad**: Soporta configuraciones multi-master para alta disponibilidad.
3. **Compatibilidad con Múltiples Proveedores de Nube**: Funciona en diversas infraestructuras, incluyendo AWS, GCP, Azure, OpenStack, y entornos bare metal.
4. **Extensibilidad y Personalización**: Permite la personalización a través de variables de Ansible y un archivo de inventario, adaptándose a diferentes necesidades y configuraciones.
5. **Soporte para Add-ons**: Facilita la instalación de componentes adicionales y add-ons como CNI (Container Network Interface) plugins, Helm, y otros.

### HAProxy
> [HAProxy](https://docs.haproxy.org/) es un software de código abierto diseñado como un balanceador de carga y proxy inverso altamente eficiente y fiable. Se utiliza para distribuir el tráfico de red de manera equitativa y eficiente entre múltiples servidores backend, mejorando así el rendimiento, la disponibilidad y la escalabilidad de las aplicaciones web y servicios. HAProxy opera en las capas 4 (TCP) y 7 (HTTP) del modelo OSI, lo que le permite manejar diversas aplicaciones y protocolos de manera eficiente. Es conocido por su configurabilidad avanzada, características de alta disponibilidad, y su capacidad para manejar grandes volúmenes de tráfico con baja latencia y alta fiabilidad.
#### Características:
1. **Balanceo de Carga**: Distribuye el tráfico de red de manera equitativa entre múltiples servidores backend para mejorar el rendimiento y la disponibilidad de las aplicaciones.
2. **Proxy Inverso**: Actúa como intermediario entre los clientes externos y los servidores internos, gestionando las conexiones entrantes y optimizando el enrutamiento.
3. **Capas de Operación**: Trabaja en las capas 4 (TCP) y 7 (HTTP) del modelo OSI, permitiendo el balanceo de carga tanto a nivel de conexión como de aplicación.
4. **Alta Disponibilidad**: Soporta configuraciones de alta disponibilidad para garantizar la continuidad del servicio incluso en caso de fallos de servidor.
5. **Configuración Avanzada**: Ofrece una configurabilidad detallada a través de archivos de configuración que permiten ajustar el comportamiento del balanceador de carga según las necesidades específicas.
6. **Monitoreo y Estadísticas**: Proporciona herramientas integradas para monitorear el rendimiento del tráfico y el estado de los servidores, facilitando la gestión y el mantenimiento proactivo del sistema.
7. **SSL/TLS Offloading**: Soporta terminación SSL/TLS para descifrar el tráfico en el balanceador de carga antes de enviarlo a los servidores backend, mejorando la eficiencia y seguridad del sistema.
8. **Flexibilidad y Escalabilidad**: Es altamente escalable y adaptable a diferentes cargas de trabajo y entornos, desde pequeñas implementaciones hasta grandes despliegues empresariales.

### MetalLB
> [MetalLB](https://metallb.universe.tf/) es un balanceador de carga para clústeres Kubernetes que proporciona una implementación de tipo LoadBalancer para entornos bare metal. Kubernetes, por defecto, no incluye una solución de balanceador de carga para entornos no administrados (bare metal), por lo que MetalLB llena ese vacío
#### Características:
1. **Compatibilidad con Kubernetes**: Proporciona una implementación de tipo LoadBalancer que funciona en entornos bare metal. Se integra perfectamente con los servicios de Kubernetes.
2. **Protocolos de Balanceo de Carga**: 
    - Layer 2 (L2): Asigna direcciones IP a los servicios y responde a las solicitudes ARP/NDP para estas IP.
    - Border Gateway Protocol (BGP): Anuncia rutas de IP a los routers vecinos usando BGP.
3. **Fácil Configuración**: Configuración sencilla a través de ConfigMaps.
4. **Alta Disponibilidad**: Puede configurarse para operar en un modo de alta disponibilidad (HA) distribuyendo la carga entre múltiples nodos del clúster.
5. **Escalabilidad**: Escala automáticamente para satisfacer las necesidades del clúster sin una intervención significativa del usuario.
6. **Flexibilidad**: Permite la asignación de direcciones IP específicas a los servicios. Además soporta tanto direcciones IP públicas o privadas.
7. **Seguridad**: Permite la configuración de políticas de acceso para controlar qué nodos pueden anunciar qué prefijos de IP en el modo BGP.

## 2. Arquitectura
El cluster Multi-Master contará con siguiente:

| Role     | FQDN        | IP                | OS           | RAM  | CPU |  API Version |
| -------- | ----------- | ----------------- | ------------ | ---- | --- | ------------ |
| Master   | `master-01` | `192.168.18.151`  | Ubuntu 22.04 |  4G  |  2  |  1.29        |
| Master   | `master-02` | `192.168.18.152`  | Ubuntu 22.04 |  4G  |  2  |  1.29        |
| Master   | `master-03` | `192.168.18.153`  | Ubuntu 22.04 |  4G  |  2  |  1.29        |
| Worker   | `worker-01` | `192.168.18.161`  | Ubuntu 22.04 |  4G  |  2  |  1.29        |
| Worker   | `worker-02` | `192.168.18.162`  | Ubuntu 22.04 |  4G  |  2  |  1.29        |
| HAProxy  | `proxy`     | `192.168.18.155`  | Ubuntu 22.04 |  4G  |  2  |  1.29        |


## 3. Despliegue de Cluster Multi-Master.
- Modificar el archivo `/etc/hosts` para facilitar la comunicación entre los nodos. Realizarlo en todos los nodos y en el servidor que funcionará como HAProxy.

```console
127.0.0.1 localhost
127.0.1.1 proxy

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

# Kubernetes' servers
192.168.18.151  master-01 master1
192.168.18.152  master-02 master2
192.168.18.153  master-03 master3
192.168.18.155  proxy proxy.example.com
192.168.18.161  worker-01
192.168.18.162  worker-02
```

- Actualizar repositorios e instalar HAProxy en el servidor `proxy`, el cual funcionará como Load Balancer.

```console
apt-get update
apt-get install -y haproxy

```

- Editar el archivo de configuración de HAProxy `/etc/haproxy/haproxy.cfg` según lo indicado en la [docuementación oficial](https://github.com/kubernetes-sigs/kubespray/blob/master/docs/operations/ha-mode.md).

```console
global
        log /dev/log    local0
        log /dev/log    local1 notice
        chroot /var/lib/haproxy
        stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
        stats timeout 30s
        user haproxy
        group haproxy
        daemon

        # Default SSL material locations
        ca-base /etc/ssl/certs
        crt-base /etc/ssl/private

        # See: https://ssl-config.mozilla.org/#server=haproxy&server-version=2.0.3&config=intermediate
        ssl-default-bind-ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384
        ssl-default-bind-ciphersuites TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256
        ssl-default-bind-options ssl-min-ver TLSv1.2 no-tls-tickets

defaults
        log     global
        mode    http
        option  httplog
        option  dontlognull
        timeout connect 5000
        timeout client  50000
        timeout server  50000
        errorfile 400 /etc/haproxy/errors/400.http
        errorfile 403 /etc/haproxy/errors/403.http
        errorfile 408 /etc/haproxy/errors/408.http
        errorfile 500 /etc/haproxy/errors/500.http
        errorfile 502 /etc/haproxy/errors/502.http
        errorfile 503 /etc/haproxy/errors/503.http
        errorfile 504 /etc/haproxy/errors/504.http


listen kubernetes-apiserver-https
  bind 192.168.18.155:8383
  mode tcp
  option log-health-checks
  timeout client 3h
  timeout server 3h
  server master1 192.168.18.151:6443 check check-ssl verify none inter 10000
  server master2 192.168.18.152:6443 check check-ssl verify none inter 10000
  server master3 192.168.18.153:6443 check check-ssl verify none inter 10000
  balance roundrobin
```

- En el server `proxy`, para el usuario administrador, generar una clave RSA para conexiones SSH.

```console
ssh-keygen
```

> Se recomienda que se cree un mismo nombre de usuario en todos los nodos y el server `proxy` y que pertenezca al grupo `sudo`, esto con la finalidad de hacer el despliegue de una manera más fácil. En el caso de este laboratorio, es usuario para todos los nodos es `k8s-admin`.
{: .prompt-tip }

- Copiar la clave pública en cada nodo del cluster.

```console
ssh-copy-id k8s-admin@192.168.18.151
ssh-copy-id k8s-admin@192.168.18.152
ssh-copy-id k8s-admin@192.168.18.153
ssh-copy-id k8s-admin@192.168.18.161
ssh-copy-id k8s-admin@192.168.18.162
```

- En cada nodo del cluster y en el server `proxy`, habilitar para el usuario administrador el uso de `sudo` sin ingresar la contraseña. Para esto, editar el archivo de configuración `etc/sudoers`.

```console
# Allow members of group sudo to execute any command
%sudo   ALL=(ALL:ALL) ALL
k8s-admin ALL=(ALL) NOPASSWD:ALL # New line to add
# See sudoers(5) for more information on "@include" directives:
```
> Esto permitirá que al ejecutar el playbook de Ansible para el despliegue del cluster de K8S, no se requiera introducir la contraseña, por lo que el proceso se realizará de forma transparente hasta el final.
{: .prompt-info }

- Clonar el [repositorio oficial](https://github.com/kubernetes-sigs/kubespray) de [Kubespray](https://kubespray.io/#/).

```console
git clone https://github.com/kubernetes-sigs/kubespray.git
```

- Instalar librerías requeridas y hacer una copia de un archivo de configuración del cluster.

```console
sudo pip install -r requirements.txt
cp -rfp inventory/sample inventory/mycluster
```

- Declarar una variable `IPS`, el cual contendrá las IP's de los nodos y del HAProxy, y generar el archivo de inventario de Ansible.

```console
declare -a IPS=(192.168.18.151 192.168.18.152 192.168.18.153 192.168.18.161 192.168.18.162)
CONFIG_FILE=inventory/mycluster/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}
```

- Editar el archivo de configuración `inventory/mycluster/hosts.yaml` según la arquitectura indicada para la distribución de nodos.

```yaml
all:
  hosts:
    node1:
      ansible_host: 192.168.18.151
      ip: 192.168.18.151
      access_ip: 192.168.18.151
    node2:
      ansible_host: 192.168.18.152
      ip: 192.168.18.152
      access_ip: 192.168.18.152
    node3:
      ansible_host: 192.168.18.153
      ip: 192.168.18.153
      access_ip: 192.168.18.153
    node4:
      ansible_host: 192.168.18.161
      ip: 192.168.18.161
      access_ip: 192.168.18.161
    node5:
      ansible_host: 192.168.18.162
      ip: 192.168.18.162
      access_ip: 192.168.18.162
  children:
    kube_control_plane:
      hosts:
        node1:
        node2:
        node3:
    kube_node:
      hosts:
        node4:
        node5:
    etcd:
      hosts:
        node1:
        node2:
        node3:
    k8s_cluster:
      children:
        kube_control_plane:
        kube_node:
    calico_rr:
      hosts: {}
```

- Editar la sección `External LB example config` y `Internal loadbalancer for apiservers` en el archivo de configuración `kubespray/inventory/mycluster/group_vars/all/all.yml` para añadir en la arquitectura el Load Balancer.

```yaml
## External LB example config
apiserver_loadbalancer_domain_name: "proxy.example.com"
loadbalancer_apiserver:
  address: 192.168.18.155
  port: 8383

## Internal loadbalancers for apiservers
loadbalancer_apiserver_localhost: false
# valid options are "nginx" or "haproxy"
# loadbalancer_apiserver_type: nginx  # valid values "nginx" or "haproxy"
```

- Desplegar Kubespray como superusuario a través del playbook de Ansible.

```console
ansible-playbook -i inventory/mycluster/hosts.yaml  --become --become-user=root reset.yml
```

> Dependiendo del hardware del equipo, esto puede tomar entre 20 y 50 minutos.
{: .prompt-info }

- Una vez culminado el proceso de despliegue del cluster, copiar el archivo de configuración del cluster de alguno de los nodos maestros, hacia el servidor `proxy`.

```console
mkdir ~/.kube
scp k8s-admin@192.168.18.151:/etc/kubernetes/admin.conf ~/.kube/config
```
> En caso de que exista algún error por falta de permisos, ingresar al nodo maestro y asignarle permiso de ejecución a dicho archivo:
- `sudo chmod +r /etc/kubernetes/admin.conf`
{: .prompt-warning }

- Instalar kubectl

```console
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubectl
sudo apt-mark hold kubelet kubectl
```

- Validar la correcta integración de los nodos, y la creación de los pods necesarios.
```console
k8s-admin@proxy:~$ kubectl get nodes
NAME    STATUS   ROLES           AGE    VERSION
node1   Ready    control-plane   2d3h   v1.29.5
node2   Ready    control-plane   2d3h   v1.29.5
node3   Ready    control-plane   2d3h   v1.29.5
node4   Ready    <none>          2d3h   v1.29.5
node5   Ready    <none>          2d3h   v1.29.5
```

```console
k8s-admin@proxy:~$ kubectl get pods -A
NAMESPACE     NAME                                       READY   STATUS    RESTARTS        AGE
kube-system   calico-kube-controllers-68485cbf9c-kqbq7   1/1     Running   5 (6h41m ago)   2d3h
kube-system   calico-node-7m2qk                          1/1     Running   1 (6h41m ago)   2d3h
kube-system   calico-node-9wblp                          1/1     Running   1 (6h41m ago)   2d3h
kube-system   calico-node-dpvrp                          1/1     Running   1 (6h41m ago)   2d3h
kube-system   calico-node-w679b                          1/1     Running   1 (6h42m ago)   2d3h
kube-system   calico-node-x7gbt                          1/1     Running   1 (6h41m ago)   2d3h
kube-system   coredns-69db55dd76-2kzkf                   1/1     Running   1 (6h42m ago)   2d3h
kube-system   coredns-69db55dd76-dpknw                   1/1     Running   1 (6h41m ago)   2d3h
kube-system   dns-autoscaler-6f4b597d8c-c94df            1/1     Running   1 (6h41m ago)   44h
kube-system   kube-apiserver-node1                       1/1     Running   2 (6h42m ago)   2d3h
kube-system   kube-apiserver-node2                       1/1     Running   2 (6h41m ago)   2d3h
kube-system   kube-apiserver-node3                       1/1     Running   5 (6h41m ago)   2d3h
kube-system   kube-controller-manager-node1              1/1     Running   7 (6h42m ago)   2d3h
kube-system   kube-controller-manager-node2              1/1     Running   5 (6h41m ago)   2d3h
kube-system   kube-controller-manager-node3              1/1     Running   6 (3h10m ago)   2d3h
kube-system   kube-proxy-5xtg9                           1/1     Running   1 (6h42m ago)   2d3h
kube-system   kube-proxy-frfrj                           1/1     Running   1 (6h41m ago)   2d3h
kube-system   kube-proxy-hnfqc                           1/1     Running   1 (6h41m ago)   2d3h
kube-system   kube-proxy-srn48                           1/1     Running   1 (6h41m ago)   2d3h
kube-system   kube-proxy-wbv4j                           1/1     Running   1 (6h41m ago)   2d3h
kube-system   kube-scheduler-node1                       1/1     Running   6 (5h29m ago)   2d3h
kube-system   kube-scheduler-node2                       1/1     Running   6 (3h10m ago)   2d3h
kube-system   kube-scheduler-node3                       1/1     Running   2 (6h41m ago)   2d3h
kube-system   nodelocaldns-7mpnl                         1/1     Running   40 (151m ago)   2d3h
kube-system   nodelocaldns-fptvw                         1/1     Running   38 (123m ago)   2d3h
kube-system   nodelocaldns-l988b                         1/1     Running   40 (152m ago)   2d3h
kube-system   nodelocaldns-q99f4                         1/1     Running   40 (149m ago)   2d3h
kube-system   nodelocaldns-vb8hh                         1/1     Running   29 (151m ago)   2d3h
```

## 4. Despliegue de WordPress
### Instalación de MetalLB
- Instalar [kustomize](https://kustomize.io/) para el despliegue de manifiestos en YAML.

```console
curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh"  | bash
sudo mv kustomize /usr/local/bin/
```

- Crear un archivo `kustomization.yml` con el contenido mostrado.

```yaml
# kustomization.yml
namespace: metallb-system

resources:
  - github.com/metallb/metallb/config/native?ref=v0.14.5
```

- Ejecutar el manifiesto `kustomization.yml`.

```console
kubectl apply -k .
```

- Validar la creación de pods.

```console
k8s-admin@proxy:~/kubernetes$ kubectl get pods -n metallb-system
NAME                          READY   STATUS    RESTARTS   AGE
controller-56bb48dcd4-qw85t   1/1     Running   0          3m28s
speaker-fklhn                 1/1     Running   0          3m28s
speaker-mdvnk                 1/1     Running   0          3m28s
speaker-p99p4                 1/1     Running   0          3m28s
speaker-pnspv                 1/1     Running   0          3m28s
speaker-x9c6h                 1/1     Running   0          3m28s
```

- Configurar el pool de direcciones IP a asignar por MetalLB.

```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: default
  namespace: metallb-system
spec:
  addresses:
  - 192.168.18.191/32
  - 192.168.18.192/32
  - 192.168.18.193/32
  autoAssign: true
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: default
  namespace: metallb-system
spec:
  ipAddressPools:
  - default

```

- Aplicar el archivo de configuración de MetalLB.

```console
kubectl apply -f config.yaml -n metallb-system
```

### Habilitación de sistema de ficheros NFS.
- En un servidor Linux (Ubuntu 22.04 para este laboratorio), instalar `nfs-kernel-server`.

```console
sudo apt update
sudo apt install nfs-kernel-server
```

- Editar el archivo `/etc/exports`, el cual gestiona los directorios compartidos.

```console
# /etc/exports: the access control list for filesystems which may be exported
#               to NFS clients.  See exports(5).
#
# Example for NFSv2 and NFSv3:
# /srv/homes       hostname1(rw,sync,no_subtree_check) hostname2(ro,sync,no_subtree_check)
#
# Example for NFSv4:
# /srv/nfs4        gss/krb5i(rw,sync,fsid=0,crossmnt,no_subtree_check)
# /srv/nfs4/homes  gss/krb5i(rw,sync,no_subtree_check)
#
/home/k8s-admin/kubernetes-nfs  *(rw,sync,no_subtree_check,no_root_squash)
```


- Crear el directorio a compartir, y editar los permisos.

```console
mkdir Kubernetes
sudo chown -R nobody:nogroup /home/k8s-admin/kubernetes-nfs/
sudo chmod 777 kubernetes-nfs/
```

- Aplicar cambios y reiniciar el servicio `nfs-kernel-server`.

```console
sudo exportfs -a
sudo systemctl restart nfs-kernel-server
```

- En el servidor `proxy`, crear el fichero `nfs-pv.yaml`, el cual definirá la configuración del volumen persistente a crear.

```yaml
# nfs-pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteMany
  nfs:
    path: /home/k8s-admin/kubernetes-nfs
    server: 192.168.18.165

---

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 5Gi
  volumeName: nfs-pv
```

- Aplicar el archivo de configuración.

```console
kubectl apply -f nfs-pv.yaml
```

### Instalación de WordPress a través de Helm.
- Descargar el binario de Helm en el server `proxy`, descomprimirlo, darle permisos de ejecución y copiarlo hacia la ruta `/usr/local/bin`.

```console
wget "https://get.helm.sh/helm-v3.15.2-linux-amd64.tar.gz"
tar -zxvf helm-v3.15.2-linux-amd64.tar.gz
sudo mv ./linux-amd64/helm /usr/local/bin/
```

- Añadir el repo de `bitnami` y actualizarlo.

```console
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

- Crear el archivo `values.yaml`, el cual definirá algunos atributos para desplegar WordPress, entre ellos el uso del volumen persistente creado.

```yaml
# values.yaml
persistence:
  enabled: true
  existingClaim: nfs-pvc

mariadb:
  primary:
    persistence:
      enabled: false
```

- Desplegar WordPress usando el archivo `values.yaml`.

```console
 helm install my-wordpress -f values.yaml bitnami/wordpress
```

- Validar la creación de los pods.

```console
k8s-admin@proxy:~/kubernetes/wordpress$ kubectl get pods
NAME                           READY   STATUS     RESTARTS   AGE
my-wordpress-6bb7665b7-nbhm8   0/1     Init:0/1   0          6s
my-wordpress-mariadb-0         0/1     Init:0/1   0          6s
```
> El despliegue de los pods puede tomar varios minutos.
{: .prompt-warning }

- Validamos finalmente el estado de los pods, así como los nodos en los cuales fueron asignados.

```console
k8s-admin@proxy:~/kubernetes/wordpress$ kubectl get pods -o wide
NAME                           READY   STATUS    RESTARTS        AGE   IP              NODE    NOMINATED NODE   READINESS GATES
my-wordpress-6bb7665b7-nbhm8   1/1     Running   1 (5m55s ago)   12m   10.233.74.68    node4   <none>           <none>
my-wordpress-mariadb-0         1/1     Running   0               12m   10.233.97.131   node5   <none>           <none>
```

- Validamos los servicios creados, así como la asignación de la IP `192.168.18.191` la cual pertenece al pool de direcciones IP definidas en el manifiesto de MetalLB.

```console
k8s-admin@proxy:~/kubernetes/wordpress$ kubectl get svc
NAME                   TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)                      AGE
kubernetes             ClusterIP      10.233.0.1      <none>           443/TCP                      2d7h
my-wordpress           LoadBalancer   10.233.20.126   192.168.18.191   80:31236/TCP,443:32267/TCP   6m24s
my-wordpress-mariadb   ClusterIP      10.233.54.46    <none>           3306/TCP                     6m24s
```

- Verificamos en los logs de los pods de MetalLB que la asignación la realizó el controller de este.

```console
k8s-admin@proxy:~/kubernetes/wordpress$ kubectl logs controller-56bb48dcd4-qw85t -n metallb-system | tail -n 13
{"caller":"service.go:150","event":"ipAllocated","ip":["192.168.18.191"],"level":"info","msg":"IP address assigned by controller","ts":"2024-07-01T05:02:08Z"}
{"caller":"service_controller.go:107","controller":"ServiceReconciler","event":"force service reload","level":"info","ts":"2024-07-01T05:19:07Z"}
{"caller":"service_controller.go:109","controller":"ServiceReconciler","end reconcile":"default/my-wordpress","level":"info","ts":"2024-07-01T05:19:07Z"}
{"caller":"service_controller_reload.go:63","controller":"ServiceReconciler - reprocessAll","level":"info","start reconcile":"metallbreload/reload","ts":"2024-07-01T05:19:07Z"}
{"caller":"service_controller_reload.go:119","controller":"ServiceReconciler - reprocessAll","end reconcile":"metallbreload/reload","level":"info","ts":"2024-07-01T05:19:07Z"}
{"caller":"service_controller.go:64","controller":"ServiceReconciler","level":"info","start reconcile":"default/my-wordpress-mariadb","ts":"2024-07-01T05:21:49Z"}
{"caller":"service_controller.go:115","controller":"ServiceReconciler","end reconcile":"default/my-wordpress-mariadb","level":"info","ts":"2024-07-01T05:21:49Z"}
{"caller":"service_controller.go:64","controller":"ServiceReconciler","level":"info","start reconcile":"default/my-wordpress","ts":"2024-07-01T05:21:50Z"}
{"caller":"service.go:150","event":"ipAllocated","ip":["192.168.18.191"],"level":"info","msg":"IP address assigned by controller","ts":"2024-07-01T05:21:50Z"}
{"caller":"main.go:116","event":"serviceUpdated","level":"info","msg":"updated service object","ts":"2024-07-01T05:21:50Z"}
{"caller":"service_controller.go:115","controller":"ServiceReconciler","end reconcile":"default/my-wordpress","level":"info","ts":"2024-07-01T05:21:50Z"}
{"caller":"service_controller.go:64","controller":"ServiceReconciler","level":"info","start reconcile":"default/my-wordpress","ts":"2024-07-01T05:21:50Z"}
{"caller":"service_controller.go:115","controller":"ServiceReconciler","end reconcile":"default/my-wordpress","level":"info","ts":"2024-07-01T05:21:50Z"}
```

- Para probar la independencia de la IP `192.168.18.191` con el nodo, procedemos a eliminar el pod que se asignó al nodo node4.

```console
k8s-admin@proxy:~/kubernetes/wordpress$ kubectl delete pod my-wordpress-6bb7665b7-nbhm8
pod "my-wordpress-6bb7665b7-nbhm8" deleted
```

- Validamos que el nuevo pod se asigna al nodo node5.

```console
k8s-admin@proxy:~/kubernetes/wordpress$ k8s-admin@proxy:~/kubernetes/wordpress$ kubectl get pods -o wide
NAME                           READY   STATUS    RESTARTS   AGE     IP              NODE    NOMINATED NODE   READINESS GATES
my-wordpress-6bb7665b7-zp427   1/1     Running   0          3m20s   10.233.97.132   node5   <none>           <none>
my-wordpress-mariadb-0         1/1     Running   0          15m     10.233.97.131   node5   <none>           <none>
```

- Accedemos vía web a la URL `192.168.18.191/wp-admin` y validamos el correcto acceso.

```console
username: user
password: lwOeMFtr9E
```

![WordPress View](assets/img/posts/2024/DevOps/Despliegue de cluster multi-master con kubespray y haproxy/wordpress.png){: width="972" height="589" }
_WordPress WebSite_
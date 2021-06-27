## Install eclipse Che on Microk8s Using ubuntu 18.04 LTS Cloud Image VM

Install and configure Microk8s
~~~bash
ubuntu@microk8s:~$ sudo usermod -a -G microk8s $USER
ubuntu@microk8s:~$ sudo chown -f -R $USER ~/.kube
ubuntu@microk8s:~$ newgrp microk8s
microk8s status --wait-ready
~~~
Check microk8s that platform is running

    ubuntu@microk8s:~$ microk8s kubectl get nodes

Add the required Addons to run Eclipse Che

    ubuntu@microk8s:~$ microk8s enable dns storage ingress

We need to enable metallb also, that needs Ip range to configure load balancer
~~~bash
ubuntu@microk8s:~$ ifconfig

enp1s0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.122.31  netmask 255.255.255.0  broadcast 192.168.122.255
        inet6 fe80::5054:ff:fe4d:8515  prefixlen 64  scopeid 0x20<link>
        ether 52:54:00:4d:85:15  txqueuelen 1000  (Ethernet)
        RX packets 479609  bytes 717312549 (717.3 MB)
        RX errors 0  dropped 203  overruns 0  frame 0
        TX packets 270074  bytes 20333418 (20.3 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
~~~
The current configuration in my vm is the ip 192.168.122.31, so we could use the following ip ranges 192.168.122.30-192.168.122.40 to use some ip address
~~~bash
ubuntu@microk8s:~$ microk8s enable metallb
Enabling MetalLB
Enter the IP address range (e.g., 10.64.140.43-10.64.140.49): 192.168.122.30-192.168.122.40
Applying registry manifest
namespace/metallb-system created
podsecuritypolicy.policy/speaker created
serviceaccount/controller created
serviceaccount/speaker created
clusterrole.rbac.authorization.k8s.io/metallb-system:controller created
clusterrole.rbac.authorization.k8s.io/metallb-system:speaker created
role.rbac.authorization.k8s.io/config-watcher created
clusterrolebinding.rbac.authorization.k8s.io/metallb-system:controller created
clusterrolebinding.rbac.authorization.k8s.io/metallb-system:speaker created
rolebinding.rbac.authorization.k8s.io/config-watcher created
daemonset.apps/speaker created
deployment.apps/controller created
configmap/config created
MetalLB is enabled
~~~
To test if everithing is fine deploy a hello-world service kubernetes and check the ip address is generated.
~~~bash
ubuntu@microk8s:~$ kubectl create deployment hello-node --image=k8s.gcr.io/echoserver:1.4
deployment.apps/hello-node created
ubuntu@microk8s:~$ kubectl get deployments
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
hello-node            0/1     1            0           22s
~~~
And expose the service, remember use type=LoadBalancer
~~~bash
ubuntu@microk8s:~$ kubectl expose deployment hello-node --type=LoadBalancer --port=8080
ubuntu@microk8s:~$ kubectl get services 
NAME         TYPE           CLUSTER-IP       EXTERNAL-IP      PORT(S)          AGE
hello-node   LoadBalancer   10.152.183.184   192.168.122.30   8080:30751/TCP   22m
kubernetes   ClusterIP      10.152.183.1     <none>           443/TCP          33m
~~~
Now we have the ip test from inside or outside from vm
~~~bash
ubuntu@microk8s:~$ curl 192.168.122.30:8080

CLIENT VALUES:
client_address=10.1.17.1
command=GET
real path=/
query=nil
request_version=1.1
request_uri=http://192.168.122.30:8080/
SERVER VALUES:
server_version=nginx: 1.10.0 - lua: 10001
HEADERS RECEIVED:
accept=*/*
host=192.168.122.30:8080
user-agent=curl/7.58.0
BODY:
~~~

With everithing working fine, install the requeriments for Eclipse Che:

    ubuntu@microk8s:~$ bash <(curl -sL  https://www.eclipse.org/che/chectl/)

Install Helm
~~~bash
ubuntu@microk8s:~$ wget https://get.helm.sh/helm-v3.2.1-linux-amd64.tar.gz
ubuntu@microk8s:~$ tar -xvf helm-v3.2.1-linux-amd64.tar.gz
ubuntu@microk8s:~$ sudo mv linux-amd64/helm /usr/local/bin/helm
~~~
In the current version, Eclipse Che generates error trying to find the kubeclt binary so we need to download 
~~~bash
ubuntu@microk8s:~$ curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubectl
ubuntu@microk8s:~$ chmod +x ./kubectl
ubuntu@microk8s:~$ sudo mv ./kubectl /usr/local/bin/kubectl
~~~

Eclipse Che also throw errors if the configuration for kubectl is not found in the directory $USER/.kube/config, so we generate with this command:
~~~bash
ubuntu@microk8s:~$ kubectl config view --raw
ubuntu@microk8s:~$ kubectl config view --raw > /home/ubuntu/.kube/config
~~~
Now its time to run Eclipse Che:
~~~bash
ubuntu@microk8s:~$ chectl server:start --installer=helm --platform=microk8s
Set current context to 'microk8s'
  ✔ Verify Kubernetes API...OK
  ✔ 👀  Looking for an already existing Eclipse Che instance
    ✔ Verify if Eclipse Che is deployed into namespace "che"...it is not
  ✔ ✈️  MicroK8s preflight checklist
    ✔ Verify if kubectl is installed
    ✔ Verify if microk8s is installed
    ✔ Verify if microk8s is running
    ↓ Start microk8s [skipped]
      → MicroK8s is already running.
    ✔ Check Kubernetes version: Found v1.18.2.
    ✔ Verify if microk8s ingress and storage addons is enabled
    ↓ Enable microk8s ingress addon [skipped]
      → Ingress addon is already enabled.
    ↓ Enable microk8s storage addon [skipped]
      → Storage addon is already enabled.
    ✔ Retrieving microk8s IP and domain for ingress URLs...192.168.122.31.nip.io.
    ↓ Check if cluster accessible [skipped]
    .....
    ✔ Eclipse Che pod bootstrap
      ✔ scheduling...done.
      ✔ downloading images...done.
      ✔ starting...done.
    ✔ Retrieving Eclipse Che server URL... https://che-che.192.168.122.31.nip.io
    ✔ Eclipse Che status check
  ✔ Show important messages
    ✔ ❗[MANUAL ACTION REQUIRED] Please add local Eclipse Che CA certificate into your browser: /home/ubunt
u/cheCA.crt
Command server:start has completed successfully.
Retrieving Eclipse Che server URL... https://che-che.192.168.122.31.nip.io
~~~

The Url its the same for the ip VM.

The certificate is generated on VM so we need to copy from vm to our host using scp

    scp ubuntu@192.168.122.31:/home/ubuntu/cheCA.crt ~/

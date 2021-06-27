## Install k3s on Fedora 32 Virtual Machine

Assuming that you already have a fedora32 machine vm with libvirt (if not you can follow the steps for creating one from cloud images here)

### Installing K3S
Enter using ssh to virtual machine and upgrade the system
    
    sudo dnf upgrade

Since k3s only supports groups V1, configure the cgroups V1 to use with k3s cluster

    sudo grubby --update-kernel=ALL --args="systemd.unified_cgroup_hierarchy=0"
    sudo grubby --info=ALL

Reboot the machine

Install k3s cluster, use the mode 644 for your user access to cluster

    curl -sfL https://get.k3s.io | sh -s - --write-kubeconfig-mode 644

Wait 30 seconds and test the cluster

    k3s kubectl get node

    [fedora@fedora ~]$ kubectl get nodes
    NAME            STATUS   ROLES    AGE   VERSION
    fedora.vm.lan   Ready    master   78m   v1.18.6+k3s1

### Testing sample Application
k3s comes with CoreDNS and Traefik Ingress Controller, so it's easy expose an application, for this example we use the nginx app, in a custom namespace for better organization of the cluster.

    alias kubectl="k3s kubectl"
    kubectl create namespace apps
    create deployment --image=nginx -n apps 

by default no port is added to deployment, but we can edit easily adding the ports node with the container port

    kubectl edit deployment nginx
    
~~~yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
~~~
Now we can create a service and exposing the application:

    kubectl expose deployment nginx --type=LoadBalancer --name=nginx -n apps

    [fedora@fedora ~]$ kubectl get svc -n apps
    NAME            TYPE           CLUSTER-IP      EXTERNAL-IP       PORT(S)          AGE
    nginx           LoadBalancer   10.43.26.87     <pending>         80:31865/TCP     16m

Test the service with the second port definition

~~~bash
[fedora@fedora ~]$ curl localhost:31865
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
~~~

If you vm has a hostname and you configured the dnsmasq resolution (use this example link), 
from your host machine you can access without making aditional configurations:

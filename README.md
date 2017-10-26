# Install Kubernetes Master/Workers on Centos 7
`Kubernetes` is an open source tool, developed by Google, that manages docker containers in cluster environemnt. In Kubernetes setup, it has one master node and multiple worker nodes. The whole cluster was managed from the master node using `kubeadm` and `kubectl` command. This tutorial shows steps:
+ **Install master node**
+ **Install worker nodes**
+ **Deploy the first demo app `kubernetes-bootcamp`**

# Master Node Installation :  

On **MASTER** node, those main components will be installed
+ **kube-apiserver**
+ **etcd**  
+ **kube-controller-manager**
+ **kube-scheduler**
+ **kubectl, kubeadm utilities**
+ **docker**

Check more detailed descriptions of those components from [here](https://kubernetes.io/docs/concepts/overview/components/).


### Disable SELinux & setup firewall rules
```
[k8s-master]$ setenforce 0
[k8s-master]$ sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
[k8s-master]$ firewall-cmd --permanent --add-port=6443/tcp
[k8s-master]$ firewall-cmd --permanent --add-port=2379-2380/tcp
[k8s-master]$ firewall-cmd --permanent --add-port=10250/tcp
[k8s-master]$ firewall-cmd --permanent --add-port=10251/tcp
[k8s-master]$ firewall-cmd --permanent --add-port=10252/tcp
[k8s-master]$ firewall-cmd --permanent --add-port=10255/tcp
[k8s-master]$ firewall-cmd --reload

# install bridge if not exist 
[k8s-master]$ yum install bridge-utils.x86_64 -y
# enable bridge-netfilter

[k8s-master]$ modprobe br_netfilter   
[k8s-master]$ echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables
```

### Edit /etc/hosts files if you don't have dns
```
[k8s-master]$ echo "
10.240.17.2 k8s-master
10.240.17.4 worker-node1
10.240.17.6 worker-node2
" >> /etc/hosts
```

### Install Kubeadm and Docker 

Add Kubernetes repo and install kubeadm and docker 
```
[k8s-master]$ cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
        https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF

[k8s-master]$ yum install kubeadm docker -y
```

### Enable docker and kubernetes services
```
[k8s-master]$ systemctl restart docker && systemctl enable docker
[k8s-master]$ systemctl  restart kubelet && systemctl enable kubelet
```

### Initialize Kubernetes Master
```
[k8s-master]$ kubeadm init
[kubeadm] WARNING: kubeadm is in beta, please do not use it for production clusters.
[init] Using Kubernetes version: v1.8.1
[init] Using Authorization modes: [Node RBAC]
[preflight] Running pre-flight checks
[preflight] WARNING: firewalld is active, please ensure ports [6443 10250] are open or your cluster may not function correctly
[kubeadm] WARNING: starting in 1.8, tokens expire after 24 hours by default (if you require a non-expiring token use --token-ttl 0)
[certificates] Generated ca certificate and key.
[certificates] Generated apiserver certificate and key.
[certificates] apiserver serving cert is signed for DNS names [k8s-master kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 10.240.17.2]
[certificates] Generated apiserver-kubelet-client certificate and key.
[certificates] Generated sa key and public key.
[certificates] Generated front-proxy-ca certificate and key.
[certificates] Generated front-proxy-client certificate and key.
[certificates] Valid certificates and keys now exist in "/etc/kubernetes/pki"
[kubeconfig] Wrote KubeConfig file to disk: "admin.conf"
[kubeconfig] Wrote KubeConfig file to disk: "kubelet.conf"
[kubeconfig] Wrote KubeConfig file to disk: "controller-manager.conf"
[kubeconfig] Wrote KubeConfig file to disk: "scheduler.conf"
[controlplane] Wrote Static Pod manifest for component kube-apiserver to "/etc/kubernetes/manifests/kube-apiserver.yaml"
[controlplane] Wrote Static Pod manifest for component kube-controller-manager to "/etc/kubernetes/manifests/kube-controller-manager.yaml"
[controlplane] Wrote Static Pod manifest for component kube-scheduler to "/etc/kubernetes/manifests/kube-scheduler.yaml"
[etcd] Wrote Static Pod manifest for a local etcd instance to "/etc/kubernetes/manifests/etcd.yaml"
[init] Waiting for the kubelet to boot up the control plane as Static Pods from directory "/etc/kubernetes/manifests"
[init] This often takes around a minute; or longer if the control plane images have to be pulled.
[apiclient] All control plane components are healthy after 31.502213 seconds
[uploadconfig] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[markmaster] Will mark node k8s-master as master by adding a label and a taint
[markmaster] Master k8s-master tainted and labelled with key/value: node-role.kubernetes.io/master=""
[bootstraptoken] Using token: 186605.c26caea6c7916681
[bootstraptoken] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstraptoken] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstraptoken] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstraptoken] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[addons] Applied essential addon: kube-dns
[addons] Applied essential addon: kube-proxy

Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run (as a regular user):

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  http://kubernetes.io/docs/admin/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join --token 186605.c26caea6c7916681 10.240.17.2:6443 --discovery-token-ca-cert-hash sha256:75141f0ee28e70d67aac5498c752de2067e226e7edbd086f838956f6a0ef7802
```

`token 186605.c26caea6c7916681` generated here will be used for worker nodes to join the master node. 

### Start using cluster
```
[k8s-master]$ mkdir -p $HOME/.kube
[k8s-master]$ cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
[k8s-master]$ chown $(id -u):$(id -g) $HOME/.kube/config
```

Check cluster-info
```
[k8s-master]$ kubectl cluster-info
Kubernetes master is running at https://10.240.17.2:6443
KubeDNS is running at https://10.240.17.2:6443/api/v1/namespaces/kube-system/services/kube-dns/proxy

```

### Deploy pod network to the cluster 

To make the cluster status ready and kube-dns status running, deploy the pod network so that containers of different host communicated each other.  POD network is the overlay network between the worker nodes.

Here is command deploy pod network: `kubectl apply -f [podnetwork].yaml`

Run  
```
[k8s-master]$ export kubever=$(kubectl version | base64 | tr -d '\n')
[k8s-master]$ kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$kubever"
serviceaccount "weave-net" created
clusterrole "weave-net" created
clusterrolebinding "weave-net" created
daemonset "weave-net" created
```

##### check status of cluster
```
[k8s-master]$ kubectl get nodes
NAME           STATUS    ROLES     AGE       VERSION
k8s-master     Ready     master    10m       v1.8.1
```

##### check status of pods
```
[k8s-master]$ kubectl get pods --all-namespaces
NAMESPACE     NAME                                        READY     STATUS    RESTARTS   AGE
kube-system   etcd-k8s-master                             1/1       Running   0          1h
kube-system   kube-apiserver-k8s-master                   1/1       Running   0          1h
kube-system   kube-controller-manager-k8s-master          1/1       Running   0          1h
kube-system   kube-dns-545bc4bfd4-7w948                   3/3       Running   0          1h
kube-system   kube-proxy-2hfhl                            1/1       Running   0          1h
kube-system   kube-proxy-5prb4                            1/1       Running   0          1h
kube-system   kube-proxy-slg5n                            1/1       Running   0          1h
kube-system   kube-scheduler-k8s-master                   1/1       Running   0          1h
kube-system   weave-net-2gcwb                             2/2       Running   1          1h
kube-system   weave-net-hqzv9                             2/2       Running   1          1h
kube-system   weave-net-lgqcr                             2/2       Running   0          1h

```

# Worker Nodes Installation:

On **WORKER** nodes, those main components will be installed.
+ **kubelet**
+ **kube-proxy**
+ **docker**

Check more detailed descriptions of those components from [here](https://kubernetes.io/docs/concepts/overview/components/).


You need to run the same installations on **BOTH** worker nodes. 

### Install worker-node1
```
[worker-node1]$ setenforce 0
[worker-node1]$ sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
[worker-node1]$ firewall-cmd --permanent --add-port=10250/tcp
[worker-node1]$ firewall-cmd --permanent --add-port=10255/tcp
[worker-node1]$ firewall-cmd --permanent --add-port=30000-32767/tcp
[worker-node1]$ firewall-cmd --permanent --add-port=6783/tcp
[worker-node1]$ firewall-cmd  --reload

[worker-node1]$ yum install bridge-utils.x86_64 -y
# enable bridge-netfilter
[worker-node1]$ modprobe br_netfilter
[worker-node1]$ echo "
# Loading br_netfilter module at boot
br_netfilter
" > /etc/modules-load.d/br_netfilter.conf

[worker-node1]$ echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables

[worker-node1]$ cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
        https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF

[worker-node1]$ yum  install kubeadm docker -y
[worker-node1]$ systemctl restart docker && systemctl enable docker
[worker-node1]$ systemctl enable kubelet.service

[worker-node1]$ echo "
10.240.17.2 k8s-master
10.240.17.4 worker-node1
10.240.17.6 worker-node2
" >> /etc/hosts
```
##### worker-node1 join to master
```
[worker-node1]$ kubeadm join --token 186605.c26caea6c7916681 k8s-master:6443
```

### Install worker-node2
Install the same things on `worker-node2`
```
[worker-node2]$ setenforce 0
[worker-node2]$ sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
[worker-node2]$ firewall-cmd --permanent --add-port=10250/tcp
[worker-node2]$ firewall-cmd --permanent --add-port=10255/tcp
[worker-node2]$ firewall-cmd --permanent --add-port=30000-32767/tcp
[worker-node2]$ firewall-cmd --permanent --add-port=6783/tcp
[worker-node2]$ firewall-cmd  --reload

[worker-node2]$ yum install bridge-utils.x86_64 -y

[worker-node2]$ modprobe br_netfilter   # enable bridge-netfilter
[worker-node2]$ echo "
# Loading br_netfilter module at boot
br_netfilter
" > /etc/modules-load.d/br_netfilter.conf

[worker-node2]$ echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables

[worker-node2]$ cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
        https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF

[worker-node2]$ yum  install kubeadm docker -y
[worker-node2]$ systemctl restart docker && systemctl enable docker
[worker-node2]$ systemctl enable kubelet.service

[worker-node2]$ echo "
10.240.17.2 k8s-master
10.240.17.4 worker-node1
10.240.17.6 worker-node2
" >> /etc/hosts
```

##### worker-node2 join to master
```
[worker-node2]$ kubeadm join --token 186605.c26caea6c7916681 k8s-master:6443
```

# Test installations
On Master Node, run `kubectl get nodes`, you will see all nodes status `READY`

```
$ kubectl get nodes
NAME                  STATUS    ROLES     AGE       VERSION
k8s-master            Ready     master    2h        v1.8.1
worker-node1          Ready     <none>    2h        v1.8.1
worker-node2          Ready     <none>    2h        v1.8.1
```

# Deploy your first app

Let's choose `kubernetes-bootcamp` as first app.

### deploy kubernetes-bootcamp
```
[k8s-master]$ kubectl run kubernetes-bootcamp --image=docker.io/jocatalin/kubernetes-bootcamp:v1 --port=8080 

deployment "kubernetes-bootcamp" created
```

### check deployments

```
[k8s-master]# kubectl get deployments
NAME                  DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   1         1         1            1           1m
```

### start proxy 
Open another `ssh` connection and start proxy 
```
[k8s-master]$ kubectl proxy
Starting to serve on 127.0.0.1:8001
```

You can see all those APIs hosted through the proxy endpoint, now available at through `http://localhost:8001`.

Test api endpoint: 

```
[k8s-master]$ curl http://127.0.0.1:8001/version
{
  "major": "1",
  "minor": "8",
  "gitVersion": "v1.8.1",
  "gitCommit": "f38e43b221d08850172a9a4ea785a86a3ffa3b3a",
  "gitTreeState": "clean",
  "buildDate": "2017-10-11T23:16:41Z",
  "goVersion": "go1.8.3",
  "compiler": "gc",
  "platform": "linux/amd64"
}
```

### test your first app

+ get app pod name

```
[k8s-master]$ kubectl get pods
kubernetes-bootcamp-6c74779b45-ljdg7
```

+ test
```
[k8s-master]$  curl http://localhost:8001/api/v1/proxy/namespaces/default/pods/kubernetes-bootcamp-6c74779b45-ljdg7/

Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-6c74779b45-ljdg7 | v=1
```

### other commands

#### export deployment file

```
[k8s-master]$ kubectl get -o=name pvc,configmap,serviceaccount,secret,ingress,service,deployment,statefulset,hpa,job,cronjob
serviceaccounts/default
secrets/default-token-cqh9r
services/kubernetes
deployments/kubernetes-bootcamp
```

export deployment yaml file
```
[k8s-master]$ kubectl get -o=yaml --export deployments/kubernetes-bootcamp
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: null
  generation: 1
  labels:
    run: kubernetes-bootcamp
  name: kubernetes-bootcamp
  selfLink: /apis/extensions/v1beta1/namespaces/default/deployments/kubernetes-bootcamp
spec:
  replicas: 1
  selector:
    matchLabels:
      run: kubernetes-bootcamp
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: kubernetes-bootcamp
    spec:
      containers:
      - image: docker.io/jocatalin/kubernetes-bootcamp:v1
        imagePullPolicy: IfNotPresent
        name: kubernetes-bootcamp
        ports:
        - containerPort: 8081
          protocol: TCP
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status: {}
```

#### delete pods and deployments
delete all 

```
[k8s-master]$ kubectl delete pods --all
pod "kubernetes-bootcamp-7c6dcb55f9-zwgxz" deleted
[k8s-master]$ kubectl delete deployments --all
deployment "kubernetes-bootcamp" deleted
```

delete one
```
[k8s-master]$ kubectl delete deployment kubernetes-bootcamp
```

#### more

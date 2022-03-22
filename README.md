# Kubernates_Cluster

Nedir?
Kubernetes, konteyner düzenlemesinde lider bir yazılımdır. Kubernetes, yalnızca kapsayıcıya alınmış uygulamaları çalıştırmak için tasarlanmış bir ana bilgisayar kümesi olan kümeleri yöneterek çalışır. Bir Kubernetes kümesine sahip olmak için en az iki düğüme ihtiyacınız vardır - bir ana düğüm ve bir çalışan düğüm. Elbette, ihtiyacınız olduğu kadar çok sayıda çalışan düğümü ekleyerek kümeyi genişletebilirsiniz. Bu kılavuzda, her ikisi de Ubuntu 20.04 çalıştıran iki düğümden oluşan bir Kubernetes kümesi dağıtacağız. Kümemizde iki düğüme sahip olmak mümkün olan en temel yapılandırmadır, ancak bu yapılandırmayı ölçeklendirebilir ve isterseniz daha fazla düğüm ekleyebilirsiniz.

## Gereksinimler
İki adet biri master01 diğeri worker01 olmak üzere makinelerimiz mevcut.
```
10.10.10.15   master01

10.10.10.16   worker01
```

> Aşağıdaki Komutlar aksi söylenene kadar her iki makinede girilecektir
```
sudo apt-get update && apt-get upgrade
curl -fsSL https://get.docker.com -o get-docker.sh
```

## kubectl Ve kubeadm Kuruluyor
```
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl

sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

Her iki makinede versiyon kontrolü sağlıyoruz
```
root@worker01:~# kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"21", GitVersion:"v1.21.1", GitCommit:"5e58841cce77d4bc13713ad2b91fa0d961e69192", GitTreeState:"clean", BuildDate:"2021-05-12T14:17:27Z", GoVersion:"go1.16.4", Compiler:"gc", Platform:"linux/amd64"}

root@master01:~# kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"21", GitVersion:"v1.21.1", GitCommit:"5e58841cce77d4bc13713ad2b91fa0d961e69192", GitTreeState:"clean", BuildDate:"2021-05-12T14:17:27Z", GoVersion:"go1.16.4", Compiler:"gc", Platform:"linux/amd64"}
```

## Swap Etkisizleştiriliyor
```
sudo swapoff -a
```


## Master01 Makine İçin Yapılandırma

Kendi IP bloğunuzu komutta düzelterek aşağıdaki komutu girin
```
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```
> Aşağıdaki Komutlar aksi söylenene kadar her iki makinede girilecektir
Komutu girdikten sonra aşağıdaki gibi bir çıktı alacaksınız
```
root@master01:~# sudo kubeadm init --pod-network-cidr=10.244.0.0/16
[init] Using Kubernetes version: v1.21.1
[preflight] Running pre-flight checks
        [WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup                                                                              driver. The recommended driver is "systemd". Please follow the guide at https:/                                                                             /kubernetes.io/docs/setup/cri/
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your inte                                                                             rnet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config                                                                              images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [kubernetes kubernetes.de                                                                             fault kubernetes.default.svc kubernetes.default.svc.cluster.local master01] and                                                                              IPs [10.96.0.1 10.10.10.15]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [localhost master01] an                                                                             d IPs [10.10.10.15 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [localhost master01] and                                                                              IPs [10.10.10.15 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/ku                                                                             belet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.y                                                                             aml"
[kubelet-start] Starting the kubelet
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests                                                                             "
[wait-control-plane] Waiting for the kubelet to boot up the control plane as sta                                                                             tic Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 17.002298 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in                                                                              the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.21" in namespace kube-system wi                                                                             th the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node master01 as control-plane by adding the la                                                                             bels: [node-role.kubernetes.io/master(deprecated) node-role.kubernetes.io/contro                                                                             l-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node master01 as control-plane by adding the ta                                                                             ints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: tw4sd0.1nho71ky432tticu
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Rol                                                                             es
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to get no                                                                             des
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post C                                                                             SRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller auto                                                                             matically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all no                                                                             de client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" nam                                                                             espace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatab                                                                             le kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as                                                                              root:

kubeadm join 10.10.10.15:6443 --token tw4sd0.1nho71ky432tticu \
        --discovery-token-ca-cert-hash sha256:1b1c221fe00116be54886586fef3a8164e                                                                             9e4d034114a1d4f5022bff1581a882
```

Aşağıdaki komutları yönetici makineye girin
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## worker01 Yapılandırılması
```
sudo kubeadm init --pod-network-cidr=10.10.10.0/24 komutunun çıktısında aldığımız aşağıdaki komutu worker01 makinesine gireceğiz
```
> Bu komut sizde farklı olacaktır
```
kubeadm join 10.10.10.15:6443 --token tw4sd0.1nho71ky432tticu \
        --discovery-token-ca-cert-hash sha256:1b1c221fe00116be54886586fef3a8164e                                                                             9e4d034114a1d4f5022bff1581a882
```

## Yönetici Makine Kontrol
```
root@master01:~# kubectl get nodes
NAME       STATUS     ROLES                  AGE    VERSION
master01   NotReady   control-plane,master   8m8s   v1.21.1
```

## Pod Ağı Oluşturun

Ana düğüm aracılığıyla bir Kapsül Ağı dağıtın. Bir pod ağı, bir ağın düğümleri arasındaki bir iletişim aracıdır. Aşağıdaki komutla kümemizde bir Flannel kapsülü ağı kuruyoruz:
```
sudo kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Ağın durumunu görüntülemek için aşağıdaki komutu kullanın:

```
root@master01:~# kubectl get pods --all-namespaces
NAMESPACE     NAME                               READY   STATUS    RESTARTS   AGE
kube-system   coredns-558bd4d5db-bmkq9           0/1     Running   0          20m
kube-system   coredns-558bd4d5db-mrglb           0/1     Running   0          20m
kube-system   etcd-master01                      1/1     Running   0          20m
kube-system   kube-apiserver-master01            1/1     Running   0          20m
kube-system   kube-controller-manager-master01   1/1     Running   0          20m
kube-system   kube-flannel-ds-stg88              1/1     Running   0          35s
kube-system   kube-proxy-7889b                   1/1     Running   0          20m
kube-system   kube-scheduler-master01            1/1     Running   0          20m
```

Şimdi düğümlerin durumunu gördüğünüzde, ana düğümün hazır olduğunu göreceksiniz:

```
root@master01:~# sudo kubectl get nodes
NAME       STATUS   ROLES                  AGE   VERSION
master01   Ready    control-plane,master   24m   v1.21.1
```

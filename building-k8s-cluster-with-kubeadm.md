
# Cluster Info
01 Kube Master + 02 Kube Worker nodes

# Basic Setup All nodes
## 1. Install Docker on all three nodes
Do the following on all three nodes

### 1.1 Add the Docker GPG key
```sh
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```
### 1.2 Add the Docker repository
```sh
sudo add-apt-repository \
    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) \
    stable"
```
### 1.3 Update package
```sh
sudo apt-get update
```
### 1.4 Install docker
```sh
sudo apt-get install -y docker-ce=18.06.1~ce~3-0~ubuntu
```
### 1.5 Hold Docker at this specific version
sudo apt-mark hold docker-ce

### 1.6 Verify that Docker is up and running with
```sh
sudo systemctl status docker
```
## 2. Install Kubeadm, Kubelet, and Kubectl on all three nodes
### 2.1 Add the Kubernetes GPG key
```sh
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
```
### 2.2 Add the Kubernetes repository
```sh
cat << EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
```
### 2.3 Update packages
```sh
sudo apt-get update
```
### 2.4 Install kubelet, kubeadm, and kubectl
```sh
sudo apt-get install -y kubelet=1.12.7-00 kubeadm=1.12.7-00 kubectl=1.12.7-00
```
### 2.5 Hold the Kubernetes components at this specific version
```sh
sudo apt-mark hold kubelet kubeadm kubectl
```

## Bootstrap the cluster on the Kube master node
### 1. On the Kube master node, init a cluster
```sh
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```
That command may take a few minutes to complete. 
Join worker to cluster
```sh
kubeadm join 10.0.1.101:6443 --token 5907zd.rjbtfjdahoo89tgc --discovery-token-ca-cert-hash sha256:66b3d1c58f32d77a99271dca51b0addd4d8e8763488a61e10a39bcde275115a3
```
### 2. Set up the local kubeconfig
```sh
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
### 3. Verify Master node is up and running
```sh
kubectl version
Client Version: version.Info{Major:"1", Minor:"12", GitVersion:"v1.12.7", GitCommit:"6f482974b76db3f1e0f5d24605a9d1d38fad9a2b", GitTreeState:"clean", BuildDate:"2019-03-25T02:52:13Z", GoVersion:"go1.10.8", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"12", GitVersion:"v1.12.10", GitCommit:"e3c134023df5dea457638b614ee17ef234dc34a6", GitTreeState:"clean", BuildDate:"2019-07-08T03:40:54Z", GoVersion:"go1.10.8", Compiler:"gc", Platform:"linux/amd64"}
```

## Join the two Kube worker nodes to the cluster
```sh
kubeadm join 10.0.1.101:6443 --token 5907zd.rjbtfjdahoo89tgc --discovery-token-ca-cert-hash sha256:66b3d1c58f32d77a99271dca51b0addd4d8e8763488a61e10a39bcde275115a3
```

## Check Cluster Node (Run on Master)
```sh
kubectl get node
NAME            STATUS     ROLES    AGE     VERSION
ip-10-0-1-101   NotReady   master   18m     v1.12.7
ip-10-0-1-102   NotReady   <none>   2m48s   v1.12.7
ip-10-0-1-103   NotReady   <none>   13m     v1.12.7
```
Note that the nodes are expected to be in the NotReady state for now.  

## Set up cluster networking with flannel
### 1. Turn on iptables bridge calls on all three nodes
```sh
echo "net.bridge.bridge-nf-call-iptables=1" | sudo tee -a /etc/sysctl.conf
```
### 2. Run this only on the Kube master node
```sh
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml
```
Now flannel is installed! Make sure it is working by checking the node status again:  
```sh
kubectl get nodes
NAME            STATUS   ROLES    AGE   VERSION
ip-10-0-1-101   Ready    master   31m   v1.12.7
ip-10-0-1-102   Ready    <none>   15m   v1.12.7
ip-10-0-1-103   Ready    <none>   26m   v1.12.7
```
After a short time, all three nodes should be in the Ready state. If they are not all Ready the first time you run kubectl get nodes, wait a few moments and try again.  
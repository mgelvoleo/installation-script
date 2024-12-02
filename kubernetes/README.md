# Kubernetes Multi Node Cluster Setup

**Master**

---

1. Access the master node

```
ssh mgelvoleo@192.168.1.40
```

2. Configure the host  file, open the file host file

```
sudo vi /etc/hosts
```

192.168.1.40 master.local    master
192.168.1.50 worker1.local   worker1
192.168.1.60 worker2.local   worker2

3. Config the resolve file for dns


4. Configure the host  file

```
swapoff -a

sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```


4. Forwarding IPv4 and letting iptables see bridged traffic



```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system

# Verify that the br_netfilter, overlay modules are loaded by running the following commands:
lsmod | grep br_netfilter
lsmod | grep overlay

# Verify that the net.bridge.bridge-nf-call-iptables, net.bridge.bridge-nf-call-ip6tables, and net.ipv4.ip_forward system variables are set to 1 in your sysctl config by running the following command:
sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward


```


4. Install container runtime

```
curl -LO https://github.com/containerd/containerd/releases/download/v1.7.14/containerd-1.7.14-linux-amd64.tar.gz
sudo tar Cxzvf /usr/local containerd-1.7.14-linux-amd64.tar.gz
curl -LO https://raw.githubusercontent.com/containerd/containerd/main/containerd.service
sudo mkdir -p /usr/local/lib/systemd/system/
sudo mv containerd.service /usr/local/lib/systemd/system/
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
sudo systemctl daemon-reload
sudo systemctl enable --now containerd

# Check that containerd service is up and running
systemctl status containerd
```

5. Install runc

```
curl -LO https://github.com/opencontainers/runc/releases/download/v1.1.12/runc.amd64
sudo install -m 755 runc.amd64 /usr/local/sbin/runc
```


6. install cni plugin


```
curl -LO https://github.com/containernetworking/plugins/releases/download/v1.5.0/cni-plugins-linux-amd64-v1.5.0.tgz
sudo mkdir -p /opt/cni/bin
sudo tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.5.0.tgz
```


7. Install kubeadm, kubelet and kubectl

```
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubelet=1.29.6-1.1 kubeadm=1.29.6-1.1 kubectl=1.29.6-1.1 --allow-downgrades --allow-change-held-packages
sudo apt-mark hold kubelet kubeadm kubectl

kubeadm version
kubelet --version
kubectl version --client
```


8. Configure `crictl` to work with `containerd`

`sudo crictl config runtime-endpoint unix:///var/run/containerd/containerd.sock`




9. initialize control plane

```
sudo kubeadm init --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=172.31.89.68 --node-name master
```

> [!NOTE]
> Note: Copy the copy to the notepad that was generated after the init command completion, we will use that later.


10. Prepare `kubeconfig`

```

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```


11. Install calico


```

kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/tigera-operator.yaml`

curl https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/custom-resources.yaml -O`

kubectl apply -f custom-resources.yaml

```


**Worker**

### Perform the below steps on both the worker nodes


- Perform steps 1-8 on both the nodes
- Run the command generated in step 9 on the Master node which is similar to below

```
sudo kubeadm join 172.31.71.210:6443 --token xxxxx --discovery-token-ca-cert-hash sha256:xxx
```

- If you forgot to copy the command, you can execute below command on master node to generate the join command again


```
kubeadm token create --print-join-command
```


Access all the slave node and config the resolve to have internet in each node

## Validation

[](https://github.com/piyushsachdeva/CKA-2024/tree/main/Resources/Day27#validation)

If all the above steps were completed, you should be able to run `kubectl get nodes` on the master node, and it should return all the 3 nodes in ready status.

Also, make sure all the pods are up and running by using the command as below:  `kubectl get pods -A`



make a .kube and config.yml

make it accessible by give a permission 775 to config file

>  Copy the $HOME/.kube/config master to slave
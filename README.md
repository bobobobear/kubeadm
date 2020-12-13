# kubeadm
kubeadm installation 

Kubernetes (k8) is a very powerful tool in the modern cloud-based world. Allowing for high scalability, reliability and availability, it is broadly used, and available on all cloud providers. However, its use is not restricted to a cloud setup. You can set up a Kubernetes cluster on your own bare metal machines, using kubeadm.
kubeadm is a Kubernetes installer. This means it is a tool built for the sole purpose of making it easy to install a Kubernetes cluster on any machine in a matter of minutes. Other installers exist, like kops (repository link) or kubicorn (repository link), but kubeadm is the most commonly used as it is the easiest to use and is officially supported by all popular cloud providers.
You can use kubeadm to run Kubernetes if you want to have your own bare metal infrastructure, if you want to take advantage of Kubernetes, and if you do not wish to migrate your application to the cloud.
The disadvantages of kubeadm are that the setup is longer than using cloud service providers, updates to Kubernetes or kubeadm must be done manually, it is less resilient, and horizontal scaling would require additional bare metal machines.

## Setting Up Kubeadm with Internet access

### Step 1: The container runtime

First up, you will need to install a runtime for your containers to run in. The most commonly used with Kubernetes is Docker, but you could also use containerd or CRI-O for example. Note that the removal of the dockershim and therefore Docker container runtime is currently planned for Kubernetes 1.23, slated for release in late 2021. Starting with Kubernetes 1.20, users will get a deprecation warning if they are using the Docker container runtime. 
Installing Docker 
'''bash
$ curl -fsSL https://get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh
'''

Step 2: Installing kubeadm

Removing Swap
The first thing to do when installing kubeadm, is to disable swap on all machines (both master nodes and worker nodes). The kubelet, as it functions at the day of writing this article, does not support swap memory. 
$ swapoff -a; sed -i '/swap/d' /etc/fstab
Installing components
Now that you have got that out of the way, you have a couple of things to install on each machine that is part of the cluster. You first need to add the Kubernetes repository to your package manager. Then, you can install all the components you will need to configure your cluster. The components are kubelet, kubeadm, and kubectl.
# If you don't have curl yet, install it
$ sudo apt-get update && sudo apt-get install -y apt-transport-https curl
# Add the google cloud package repository to your sources
$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add –
$ cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list deb https://apt.kubernetes.io/ kubernetes-xenial main EOF
$ sudo apt-get update
# Install components
$ sudo apt-get install -y kubelet kubeadm kubectl
# Hold version
$ sudo apt-mark hold kubelet kubeadm kubectl

Step 3: Starting the Kubernetes cluster

Launching the cluster is “as easy” as running kubeadm init. However, this single kubeadm command encapsulates a lot of complexity, and there are a few points to watch out for when running it. Starting with a simple one, it is good to know that the default CRI Container Runtime Interface is docker. It is possible to change it with the --cri-socket option. There are many options for a configuring the Pod Network. The default one is Calico, but many others are available. You can check this page, to pick the one that suits your needs best: https://kubernetes.io/docs/concepts/cluster-administration/addons/ Upon setup, kubeadm will add the [node-role.kubernetes.io/master:NoSchedule] taint to the master node. This means your master node will never be scheduled to work, and your worker nodes will do all the work. In most cases this is normal behaviour, but if you are setting up a single node cluster, this will definitely be an issue. You can remove it after initiating kubeadm by using: kubectl taint nodes master key:NoSchedule-. By default, kubeadm uses the default network interface of the machine it runs on to set the advertise address for its API server. However, you can choose to use a different one, that matches your setup with the --apiserver-advertise-address=<ip-address> argument. 
$ kubeadm init --apiserver-advertise-address=<ip-address> --pod-network-cidr=192.168.0.0/16  --ignore-preflight-errors=all
# Deploy Calico Network 
$ kubectl --kubeconfig=/etc/kubernetes/admin.conf create -f https://docs.projectcalico.org/manifests/calico.yaml
# If you want to be able to run kubectl commands as non-root user, then as a non-root user perform these
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config

Once all that is done, you will have an up and running master node. The init will take a while, but at the end, kubeadm should provide you with an output looking like this, including instructions on how to set up kubectl to connect to the cluster, and how to have a node join the cluster as a worker:

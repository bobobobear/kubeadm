# kubeadm v1.20.0

## Sources
https://www.padok.fr/en/blog/kubeadm-kubernetes-cluster <br />
https://kubernetes.io/blog/2020/12/02/dockershim-faq/ <br />
https://github.com/justmeandopensource/kubernetes/blob/master/docs/install-cluster-ubuntu-20.md <br />
https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/ <br />
https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md <br />
### MiniKube, Kubeadm, Kind, K3S, how to get started on Kubernetes?
https://www.padok.fr/en/blog/minikube-kubeadm-kind-k3s <br />

Kubernetes (k8) is a very powerful tool in the modern cloud-based world. Allowing for high scalability, reliability and availability, it is broadly used, and available on all cloud providers. However, its use is not restricted to a cloud setup. You can set up a Kubernetes cluster on your own bare metal machines, using kubeadm.
kubeadm is a Kubernetes installer. This means it is a tool built for the sole purpose of making it easy to install a Kubernetes cluster on any machine in a matter of minutes. Other installers exist, like kops (repository link) or kubicorn (repository link), but kubeadm is the most commonly used as it is the easiest to use and is officially supported by all popular cloud providers.
You can use kubeadm to run Kubernetes if you want to have your own bare metal infrastructure, if you want to take advantage of Kubernetes, and if you do not wish to migrate your application to the cloud.
The disadvantages of kubeadm are that the setup is longer than using cloud service providers, updates to Kubernetes or kubeadm must be done manually, it is less resilient, and horizontal scaling would require additional bare metal machines.

## Setting Up Kubeadm with Internet access

### Step 1: The container runtime

First up, you will need to install a runtime for your containers to run in. The most commonly used with Kubernetes is Docker, but you could also use containerd or CRI-O for example. Note that the removal of the dockershim and therefore Docker container runtime is currently planned for Kubernetes 1.23, slated for release in late 2021. Starting with Kubernetes 1.20, users will get a deprecation warning if they are using the Docker container runtime. 
Installing Docker 
``` bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```
### Step 2: Installing kubeadm

#### Removing Swap
The first thing to do when installing kubeadm, is to disable swap on all machines (both master nodes and worker nodes). The kubelet, as it functions at the day of writing this article, does not support swap memory. 

``` bash
swapoff -a; sed -i '/swap/d' /etc/fstab
```

#### Installing components
Now that you have got that out of the way, you have a couple of things to install on each machine that is part of the cluster. You first need to add the Kubernetes repository to your package manager. Then, you can install all the components you will need to configure your cluster. The components are kubelet, kubeadm, and kubectl.

Add the google cloud package repository to your sources
``` bash
{
  curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
  echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list
}
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
```
Hold version
``` bash
sudo apt-mark hold kubelet kubeadm kubectl
```

### Step 3: Starting the Kubernetes cluster
Launching the cluster is “as easy” as running kubeadm init. However, this single kubeadm command encapsulates a lot of complexity, and there are a few points to watch out for when running it. Starting with a simple one, it is good to know that the default CRI Container Runtime Interface is docker. It is possible to change it with the --cri-socket option. There are many options for a configuring the Pod Network. The default one is Calico, but many others are available. You can check this page, to pick the one that suits your needs best: https://kubernetes.io/docs/concepts/cluster-administration/addons/ Upon setup, kubeadm will add the [node-role.kubernetes.io/master:NoSchedule] taint to the master node. This means your master node will never be scheduled to work, and your worker nodes will do all the work. In most cases this is normal behaviour, but if you are setting up a single node cluster, this will definitely be an issue. You can remove it after initiating kubeadm by using: kubectl taint nodes master key:NoSchedule-. By default, kubeadm uses the default network interface of the machine it runs on to set the advertise address for its API server. However, you can choose to use a different one, that matches your setup with the --apiserver-advertise-address=<ip-address> argument. 

``` bash
kubeadm init --apiserver-advertise-address=<ip-address> --pod-network-cidr=192.168.0.0/16  --ignore-preflight-errors=all
```
Deploy Calico Network 
``` bash
kubectl --kubeconfig=/etc/kubernetes/admin.conf create -f https://docs.projectcalico.org/manifests/calico.yaml
``` 
If you want to be able to run kubectl commands as non-root user, then as a non-root user perform these
``` bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
Once all that is done, you will have an up and running master node. The init will take a while, but at the end, kubeadm should provide you with an output looking like this, including instructions on how to set up kubectl to connect to the cluster, and how to have a node join the cluster as a worker:

``` console
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

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.1.219:6443 --token 8hg76g.33uirajqh3xkdpll \
    --discovery-token-ca-cert-hash sha256:e6520d06f6cb64015036dbdb9463aca769a804aff3c11a02f34c5f0cfee169d2
``` 


### Step 4: Deploying the Web UI (Dashboard) (Optional)
Dashboard is a web-based Kubernetes user interface. You can use Dashboard to deploy containerized applications to a Kubernetes cluster, troubleshoot your containerized application, and manage the cluster resources. You can use Dashboard to get an overview of applications running on your cluster, as well as for creating or modifying individual Kubernetes resources (such as Deployments, Jobs, DaemonSets, etc). For example, you can scale a Deployment, initiate a rolling update, restart a pod or deploy new applications using a deploy wizard. Dashboard also provides information on the state of Kubernetes resources in your cluster and on any errors that may have occurred.

``` bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml
```
Your dashboard is now ready with its pod in the running state. By default, dashboard will not be visible on the Master VM. Run the following command in the command line:
``` bash
kubectl proxy
```
Kubectl will make Dashboard available at http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/.
To protect your cluster data, Dashboard deploys with a minimal RBAC configuration by default. Currently, Dashboard only supports logging in with a Bearer Token. You will find out how to create a new user using Service Account mechanism of Kubernetes, grant this user admin permissions and login to Dashboard using bearer token tied to this user.

IMPORTANT: Make sure that you know what you are doing before proceeding. Granting admin privileges to Dashboard's Service Account might be a security risk. For each of the following snippets for ServiceAccount and ClusterRoleBinding, you should copy them to new manifest files like dashboard-adminuser.yaml and use kubectl apply -f dashboard-adminuser.yaml to create them.

#### Creating a Service Account
We are creating Service Account with name admin-user in namespace kubernetes-dashboard first.

``` bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
EOF
```

#### Creating a ClusterRoleBinding
In most cases after provisioning cluster using kops, kubeadm or any other popular tool, the ClusterRole cluster-admin already exists in the cluster. We can use it and create only ClusterRoleBinding for our ServiceAccount. If it does not exist then you need to create this role first and grant required privileges manually.

``` bash
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
EOF
```
#### Getting a Bearer Token
Now we need to find token we can use to log in. Execute following command:

``` bash
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')
```
It should print something like:

``` console
Name:         admin-user-token-v57nw
Namespace:    kubernetes-dashboard
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: admin-user
              kubernetes.io/service-account.uid: 0303243c-4040-4a58-8a47-849ee9ba79c1

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1066 bytes
namespace:  20 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLXY1N253Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIwMzAzMjQzYy00MDQwLTRhNTgtOGE0Ny04NDllZTliYTc5YzEiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZXJuZXRlcy1kYXNoYm9hcmQ6YWRtaW4tdXNlciJ9.Z2JrQlitASVwWbc-s6deLRFVk5DWD3P_vjUFXsqVSY10pbjFLG4njoZwh8p3tLxnX_VBsr7_6bwxhWSYChp9hwxznemD5x5HLtjb16kI9Z7yFWLtohzkTwuFbqmQaMoget_nYcQBUC5fDmBHRfFvNKePh_vSSb2h_aYXa8GV5AcfPQpY7r461itme1EXHQJqv-SN-zUnguDguCTjD80pFZ_CmnSE1z9QdMHPB8hoB4V68gtswR1VLa6mSYdgPwCHauuOobojALSaMc3RH7MmFUumAgguhqAkX3Omqd3rJbYOMRuMjhANqd08piDC3aIabINX6gP5-Tuuw2svnV6NYQ
```
Now copy the token and paste it into Enter token field on the login screen.

### Step 5: Joining a node to the Kubernetes cluster
To join the Kubernetes cluster as a worker node, you will need to start with step two: Installing kubeadm. Then use the command you got from Step 3 Starting the Kubernetes cluster:

``` bash
kubeadm token create --print-join-command
kubeadm join <control-plane-host>:<control-plane-port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>
``` 
### Reseting kubeadm
To reset kubeadmin
``` bash
kubeadm reset
rm -rf $HOME/.kube/
rm -rf /etc/kubernetes/
``` 

## Setting Up Kubeadm without Internet access

### Step 1: Install Docker, kubeadm, kubectl, kubelet and their dependencies
### Step 2: Load prepulled Kubernetes, Calico, and Kubernetes UI dashboard images into Docker Repository. The images are:
k8s.gcr.io/kube-proxy <br />
k8s.gcr.io/kube-scheduler <br />
k8s.gcr.io/kube-apiserver <br />
k8s.gcr.io/kube-controller-manager <br />
k8s.gcr.io/etcd <br />
k8s.gcr.io/coredns <br />
k8s.gcr.io/pause <br />
calico/node <br />
calico/pod2daemon-flexvol <br />
calico/cni <br />
calico/kube-controllers <br />
kubernetesui/dashboard <br />
kubernetesui/metrics-scraper <br />

``` bash
docker load --input IMAGE.tar
``` 
### Step 3: Load predownloaded kubernetes manifest files. The kubernetes manifest files are:
https://docs.projectcalico.org/manifests/calico.yaml <br />
https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml <br />

### Step 4:
pending

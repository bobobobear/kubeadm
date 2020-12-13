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
``` bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

## Demo: Deploying Kubernetes, Calico, Istio and Envoy  

### 1. Create the base OS instances

This demo was done with Ubuntu 16.04.2, Kubernetes 1.6.4, Calico 2.2, Istio 0.1.6 and Envoy. We plan to also verify with CoreOS ContainerLinux and CentOS/RHEL

Launch a few Ubuntu instances on your favorite public or private cloud. Depending on the cloud used (for e.g., AWS), you might need to set source-dest-check to off for the instances. For this demo, we used 4 large instances, 1 for the kube-master and the rest workers. Please ensure that the basic system configuration is functional (for e.g, DNS name resolution or /etc/hosts for cluster nodes, ssh, etc.)


### 2. Install Prerequisite Packages on Base OS Instances


On each node:

(We use the kubernetes-xenial-unstable repo below so that we can deploy with the newest Kubernetes 1.6.4 release. Feel free to switch to kubernetes-xenial instead if you'd prefer earlier Kube 1.6.x releases)

	apt-get update && apt-get install -y apt-transport-https
	curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
	cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
	deb http://apt.kubernetes.io/ kubernetes-xenial-unstable main
	EOF
	apt-get update
 	apt-get install -y docker-engine
	apt-get install -y kubelet kubeadm kubectl kubernetes-cni
	

On master:
	
	kubeadm init
	export KUBECONFIG=/etc/kubernetes/admin.conf

	

Download the [Calico manifest](http://docs.projectcalico.org/v2.2/getting-started/kubernetes/installation/hosted/kubeadm/1.6/calico.yaml) , and either use as is with defaults, or change to reflect your preferred network settings (Flat IP vs. IPIP, your desired container network address range, etc.)

For e.g.,
     Change CALICO_IPV4POOL_CIDR to your preferred range (for e.g., 10.5.0.0/16)
     Change CALICO_IPV4POOL_IPIP from always to cross-subnet (if you'd prefer to use unencapsulated networking in deployments like AWS or private clouds where there is L2 reachability within subnets)
 
### 3. Deploy Kubernetes and Calico with Kubeadm
 
 Deploy Calico on master:
	 
	 kubectl apply -f calico.yaml
	 
 
 Optionally, download the [calicoctl utility](https://github.com/projectcalico/calicoctl/releases/download/v1.2.1/calicoctl)  if you'd to interact with calico directly (say, for advanced network policy and/or troubleshooting).
 
 
In a second window, verify that the Calico/node pod has started (might take a minute depending on how long it takes for the image to download).
	
	export KUBECONFIG=/etc/kubernetes/admin.conf
	watch kubectl get pods --all-namespaces -o wide
	
 
Join worker nodes to the Cluster by running this command on each worker node (the specific command, including token will be in the output of *kubeadm init *on the master node:
	
	kubeadm join --token=token master-ip
	
Verify that calico/node pods are started for each node in the cluster (via the *watch kubectl* command above)

Congratters, you have a working Kubernetes cluster.



### 4. Install Istio and Envoy

***The yaml edits done below were because I was using earlier Istio and bookinfo templates that had RBAC errors - these should be fixed upstream, so consider the workarounds below as temporary. Added note: looks like the RBAC errors persist with Istio v0.1.6, so you might still need these changed yaml files below***

Follow directions at [Istio - Installation](https://istio.io/docs/tasks/installing-istio.html)  However, use the [istio-rbac-beta.yaml](https://gist.github.com/kprabhak/230a40586a0966028e0dbe6fdaf2b877)  and [istio.yaml](https://gist.github.com/kprabhak/9ab1f16b79ab5a6b820fe910ae7b0d03)  (or istio-auth.yaml) provided here, to work around a couple of errors in how the current docs set up RBAC controls and use ServiceAccounts.
  
  Try the [Istio bookinfo demo application](https://istio.io/docs/samples/bookinfo.html)  to see Istio and Envoy in action, and play around with features like request routing rules and service mesh routing. However, use the [bookinfo.yaml](https://gist.github.com/kprabhak/f4eb322a1c716d6422159b4fa0b7b0f9)  provided here, to work around the same RBAC error mentioned above.
  
  
  
### 5. Advanced Network Policy with Calico and Istio

There are a number of interesting ways to combine Service mesh rules and L5-7 policy with Calico and Kubernetes Network Policy. Here are a few examples:

- DefaultDeny within namespace and restricted pinhole openings for microservices: [example](https://www.projectcalico.org/network-policy-and-istio-deep-dive/)  by Saurabh Mohan (Tigera)
- Example restricting access to microservice application to specific host instances/




  

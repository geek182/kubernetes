###upgrade on master node

#check version
kubeadm version

#execute a plan to get details
kubeadm upgrade plan

verify version available and remember upgrades should be a minor version at time.

#upgrade kubeadm
apt install kubeadm=1.12.0-00 / upgrades should be a minor version at time.

#upgrade kubeadm
kubeadm upgrade apply v1.12.0

#upgrade kubelet
apt install kubelet=1.12.0-00

----

###kubeadm on worker nodes

apt install kubeadm=1.12.0-00 / upgrades should be a minor version at time.
apt install kubelet=1.12.0-00

kubeadm upgrade node config --kubelet-version $(kubelet --version | cut -d ' ' -f 2)

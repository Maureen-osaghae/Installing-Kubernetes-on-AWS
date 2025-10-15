# Installing-Kubernetes-on-AWS EC2
Discover how I install Kubernetes with a control-plane instance and a node instance on AWS EC2 instances.

# Configure your AWS CLI
Retrieve the running instances

    aws ec2 describe-instances --query "Reservations[*].Instances[?State.Name=='running'].InstanceId"

<img width="619" height="125" alt="image" src="https://github.com/user-attachments/assets/021ad56d-bad6-4dfd-8742-2ea35bc7cfed" />

    
We’ll want to connect to both. You can either do it one at a time, or you can open up two separate tabs in your terminal…just don’t get them mixed up!

    aws ec2-instance-connect ssh --instance-id "i-07351bd9259f9dd41" --os-user ubuntu
    
    aws ec2-instance-connect ssh --instance-id "i-04bd4ee02bdec9b2a" --os-user ubuntu

# Installing Kubernetes
# Installed containered 
    # update packages in apt package manager
    sudo apt update
    
    #install containerd using the apt package manager
    sudo apt-get install -y containerd
    
    # create /etc/containerd directory for containerd configuration
    sudo mkdir -p /etc/containerd
    
    # Generate the default containerd configuration
    # Modify the SystemdCgroup setting, telling containerd to use systemd as the cgroup driver
    # Change the sandbox image used by containerd from version 3.6 of the pause container to version 3.9
    containerd config default | sed 's/SystemdCgroup = false/SystemdCgroup = true/' | sed 's/sandbox_image = "registry.k8s.io\/pause:3.6"/sandbox_image = "registry.k8s.io\/pause:3.9"/' | sudo tee /etc/containerd/config.toml

# INSTALL KUBEADM, KUBELET, and KUBECTL

    # update packages
    sudo apt update
    
    # install apt-transport-https ca-certificates curl and gpg packages using apt package manager
    sudo apt-get install -y apt-transport-https ca-certificates curl gpg
    
    # create the directory `/etc/apt/keyrings`
    sudo mkdir -p -m 755 /etc/apt/keyrings
    
    # download the k8s release gpg key
    sudo curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
    
    # This overwrites any existing configuration in /etc/apt/sources.list.d/kubernetes.list
    echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
    
    # update packages in apt 
    sudo apt-get update
    
    # install kubelet, kubeadm, and kubectl at version 1.30.1-1.1
    sudo apt-get install -y kubelet=1.30.1-1.1 kubeadm=1.30.1-1.1 kubectl=1.30.1-1.1
    
    # hold these packages at version 1.30.1-1.1
    sudo apt-mark hold kubelet kubeadm kubectl
    
# ENABLE IP FORWARDING

    # enable IP forwarding immediately (though temporarily until the next reboot), use the following command
    sudo sysctl -w net.ipv4.ip_forward=1
    
    # uncomment the line in /etc/sysctl.conf enabling IP forwarding after reboot
    sudo sed -i '/^#net\.ipv4\.ip_forward=1/s/^#//' /etc/sysctl.conf
    
    # Apply the changes to sysctl.conf
    sudo sysctl -p

# INITIALIZE THE CLUSTER (ONLY FROM CONTROL PLANE)

    # ONLY ON THE CONTROL PLANE
    # Initialize the cluster specifying containerd as the container runtime, ensuring that the --cri-socket argument includes the unix:// prefix
    # containerd.sock is a Unix domain socket used by containerd
    # The Unix socket mechanism is a method for inter-process communication (IPC) on the same host.
    sudo kubeadm init --pod-network-cidr=192.168.0.0/16 --cri-socket=unix:///run/containerd/containerd.sock
    
    # ONLY ON CONTROL PLANE (also in the output of 'kubeadm init' command)
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
    
    # Install the Tigera Calico CNI operator and custom resource definitions
    kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/tigera-operator.yaml
    
    # Install Calico CNI by creating the necessary custom resource
    kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/custom-resources.yaml

# JOIN THE WORKER NODE TO THE CLUSTER

    # join the node to the cluster (get this command from the 'kubeadm init' output)
    sudo kubeadm join 192.168.1.9:6443 --token lllrj0.pystabmhlyt2svty --discovery-token-ca-cert-hash sha256:9d2fd15886eb176466640067f361ed2295de38188b057becf31d3bf5a4fb0b73


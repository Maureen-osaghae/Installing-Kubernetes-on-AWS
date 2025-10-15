# Installing-Kubernetes-on-AWS
Discover how I install Kubernetes with a control-plane instance and a node instance on AWS EC2 instances.

# Configure your AWS CLI
Retrieve the running instances

    aws ec2 describe-instances --query "Reservations[*].Instances[?State.Name=='running'].InstanceId"

<img width="619" height="125" alt="image" src="https://github.com/user-attachments/assets/021ad56d-bad6-4dfd-8742-2ea35bc7cfed" />

    
We’ll want to connect to both. You can either do it one at a time, or you can open up two separate tabs in your terminal…just don’t get them mixed up!

    aws ec2-instance-connect ssh --instance-id "i-07351bd9259f9dd41" --os-user ubuntu
    
    aws ec2-instance-connect ssh --instance-id "i-04bd4ee02bdec9b2a" --os-user ubuntu

# Installing Kubernetes
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
    
    # Restart containerd to apply the configuration changes
    sudo systemctl restart containerd


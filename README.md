# Installing-Kubernetes-on-AWS
Discover how I install Kubernetes with a control-plane instance and a node instance on AWS EC2 instances.

# Configure your AWS CLI
Retrieve the running instances

    aws ec2 describe-instances --query "Reservations[*].Instances[?State.Name=='running'].InstanceId"

<img width="619" height="125" alt="image" src="https://github.com/user-attachments/assets/021ad56d-bad6-4dfd-8742-2ea35bc7cfed" />

    
We’ll want to connect to both. You can either do it one at a time, or you can open up two separate tabs in your terminal…just don’t get them mixed up!

    aws ec2-instance-connect ssh --instance-id "i-07351bd9259f9dd41" --os-user ubuntu
    
    aws ec2-instance-connect ssh --instance-id "i-04bd4ee02bdec9b2a" --os-user ubuntu

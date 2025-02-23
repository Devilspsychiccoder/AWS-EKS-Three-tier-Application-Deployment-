# AWS EKS - TwoTierAppChallenge

## Infrastructure Achitecture Flow 

![ERD Diagram](/diagram/diagram1.png)

## Application Achitecture Flow 

![ERD Diagram](/diagram/diagram2.png)

## If the Two tier Project is successfully completed then it should show like this:

<p align="center">
  <img src="/diagram/diagram3.png" width="500">
  <img src="/diagram/diagram4.png" width="500">
</p>


## Overview
This repository hosts the `#TwoTierAppChallenge` for the Avik(AKB) community. 
The challenge involves deploying a Three-Tier Web Application using ReactJS, NodeJS, and MongoDB, with deployment on AWS EKS. Participants are encouraged to deploy the application, add creative enhancements, and submit a Pull Request (PR). Merged PRs will earn exciting prizes!

## Prerequisites
- Basic knowledge of Docker, and AWS services.
- An AWS account with necessary permissions.

📈 **The journey covered everything from setting up tools to deploying a Three-Tier app, Which is Highly scalable and Available in EKS.**

## Getting Started
Creating AWS EKS Cluster using EKSCTL

### Step 1: IAM Configuration
- Create a user `eks-admin` with `AdministratorAccess`.
- Generate Security Credentials: Access Key and Secret Access Key.

### Step 2: EC2 Setup
- Launch an Ubuntu instance in your favourite region (eg. region `us-west-2`). Use Instance Type as T2.Micro as we will just be using this instance to interact with the EKS Cluster 
- SSH into the instance from your local machine. Install The following components 

### Step 3: Install AWS CLI v2
``` shell
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install -i /usr/local/aws-cli -b /usr/local/bin --update
```
### Step 4: Configure AWS CLI 
```
aws configure

#Enter Your AccessKey, Secret AccessKey from IAM>user>role Store it some where safe. Also enter Region and Output as Json
# Test if you have connection to AWS by using the command aws s3 ls 
```
### Step 5: Install Docker
``` shell
sudo apt-get update
sudo apt install docker.io
docker ps
sudo chown $USER /var/run/docker.sock
```

### Step 6: Install kubectl
``` shell
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client
alias k=kubectl
```

### Step 7: Install eksctl
``` shell
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
alias e=eksctl
```
### Step 8: To Setup Alias for Convinence
```
alias k=kubectl
echo 'alias k=kubectl' >> ~/.bashrc  # For Bash To make it persistent, add it to your shell configuration file:
echo 'alias k=kubectl' >> ~/.zshrc   # For Zsh To make it persistent, add it to your shell configuration file:
source ~/.bashrc  # For Bash To apply the changes
source ~/.zshrc   # For Zsh To apply the changes
```
### Step 9: To Setup More Alias for Convinence using below automation way 
```
cat <<EOF >> ~/.bashrc

# Kubernetes Aliases
alias k="kubectl"
alias kc="kubectl config"
alias kctx="kubectl config use-context"
alias kns="kubectl config set-context --current --namespace"

# Get Commands
alias kg="kubectl get"
alias kgp="kubectl get pods"
alias kgs="kubectl get svc"
alias kgd="kubectl get deployments"
alias kgn="kubectl get nodes"
alias kga="kubectl get all"
alias kgcm="kubectl get configmap"
alias kgsec="kubectl get secret"

# Describe Commands
alias kd="kubectl describe"
alias kdp="kubectl describe pod"
alias kds="kubectl describe svc"
alias kdd="kubectl describe deployment"
alias kdn="kubectl describe node"

# Logs
alias kl="kubectl logs"
alias klf="kubectl logs -f"

# Apply & Delete
alias ka="kubectl apply -f"
alias kdelf="kubectl delete -f"
alias kdelp="kubectl delete pod"
alias kdelns="kubectl delete namespace"
alias kdelcm="kubectl delete configmap"
alias kdelsec="kubectl delete secret"

# Exec & Port-forwarding
alias ke="kubectl exec -it"
alias kpf="kubectl port-forward"

# Advanced Shortcuts
alias ksys="kubectl get pods -A"
alias krun="kubectl run --rm -i --tty --image"
alias kedit="kubectl edit"
alias kexpose="kubectl expose"

# Kubernetes Rollout
alias kru="kubectl rollout undo"
alias krd="kubectl rollout restart deployment"

# Kubernetes Scale
alias ksdp="kubectl scale deployment"
alias kssts="kubectl scale statefulset"

# Kubernetes Taint & Label
alias ktaint="kubectl taint nodes"
alias klabel="kubectl label"

# Kubernetes Debugging
alias ktop="kubectl top"
alias ktopn="kubectl top nodes"
alias ktopp="kubectl top pods"

# Helm (if using Helm)
alias h="helm"
alias hl="helm list"
alias hi="helm install"
alias hu="helm upgrade"
alias hd="helm delete"

EOF

```
### Step 10: Reload Bash Configuration. Run the following command to apply the changes and Verify the Aliases
```
source ~/.bashrc
alias | grep 'kubectl'

```

### Step 11: Setup EKS Cluster
``` shell
eksctl create cluster --name my-cluster --region us-west-2 --node-type t2.medium --nodes-min 1 --nodes-max 2
aws eks update-kubeconfig --region us-west-2 --name my-cluster

```

### Step 12: Run Manifests
``` shell
kubectl get nodes
kubectl create namespace workshop
kubectl apply -f .
kubectl delete -f .
```

### Step 13: Install AWS Load Balancer
``` shell
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json
eksctl utils associate-iam-oidc-provider --region=us-west-2 --cluster=three-tier-cluster --approve
eksctl create iamserviceaccount --cluster=three-tier-cluster --namespace=kube-system --name=aws-load-balancer-controller --role-name AmazonEKSLoadBalancerControllerRole --attach-policy-arn=arn:aws:iam::626072240565:policy/AWSLoadBalancerControllerIAMPolicy --approve --region=us-west-2
```

### Step 14: Deploy AWS Load Balancer Controller
``` shell
sudo snap install helm --classic
helm repo add eks https://aws.github.io/eks-charts
helm repo update eks
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=my-cluster --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller
kubectl get deployment -n kube-system aws-load-balancer-controller
kubectl apply -f full_stack_lb.yaml
```

### Cleanup
- To delete the EKS cluster:
``` shell
eksctl delete cluster --name my-cluster --region us-west-2
```
- To clean up rest of the stuff and not incure any cost
```
Stop or Terminate the EC2 instance created in step 2.
Delete the Load Balancer created in step 9 and 10.
Go to EC2 console, access security group section and delete security groups created in previous steps
```

## Contribution Guidelines
- Fork the repository and create your feature branch.
- Deploy the application, adding your creative enhancements.
- Ensure your code adheres to the project's style and contribution guidelines.
- Submit a Pull Request with a detailed description of your changes.

## Rewards
- Successful PR merges will be eligible for exciting prizes!

## Support
For any queries or issues, please open an issue in the repository.

## This is how the complete Multi-Cluster will be setup in 15 Mins using eksctl utility tool 

![ERD Diagram](/diagram/diagram.png)

---
Happy Learning! 🚀👨‍💻👩‍💻 Feel free to Fork/Clone the Repository and send me pull request I will review and approve 

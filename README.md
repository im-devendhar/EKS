# EKS


---

#  EKS & Kubernetes Setup Guide (Ubuntu EC2)

Steps taken to set up `kubectl`, `aws cli`, and `eksctl` on an Ubuntu EC2 instance for working with Amazon EKS.

---

##  1. Install `kubectl` (Kubernetes CLI)

`kubectl` is the command-line tool used to interact with Kubernetes clusters.

### Commands:
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client
```

---

##  2. Configure AWS CLI

Used to authenticate and interact with AWS services.

### Command:
```bash
aws configure
```

You’ll be prompted to enter:
- AWS Access Key ID
- AWS Secret Access Key
- Default region name (e.g., `ap-south-1`)
- Default output format (optional)

---

##  3. Fix `kubectl` Error: "The connection to the server localhost:8080 was refused"

This happens because `kubectl` was trying to connect to a Kubernetes cluster on localhost, which doesn’t exist.

### Fix:
```bash
aws eks --region <your-region> update-kubeconfig --name <your-cluster-name>
```

Example:
```bash
aws eks --region ap-south-1 update-kubeconfig --name my-eks-cluster
```

---

##  4. Install `eksctl`

`eksctl` is used to create and manage EKS clusters.

### Commands:
```bash
curl --silent --location "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_Linux_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

---

##  Tool Summary

| Tool       | Purpose |
|------------|---------|
| `kubectl`  | Interact with Kubernetes clusters |
| `aws cli`  | Authenticate and manage AWS resources |
| `eksctl`   | Create and manage EKS clusters |

---

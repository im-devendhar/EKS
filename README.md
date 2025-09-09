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

# Create an EKS Cluster with Fargate using `eksctl`

This guide explains how to create an Amazon EKS (Elastic Kubernetes Service) cluster using `eksctl` with Fargate enabled.

## 📋 Prerequisites

Before running the command, ensure the following:

- ✅ `eksctl` is installed  
  ```bash
  eksctl version
  ```

- ✅ AWS CLI is configured  
  ```bash
  aws configure
  ```

- ✅ You have sufficient IAM permissions to create EKS clusters and Fargate profiles.

##  Command to Create Cluster

```bash
eksctl create cluster --name demo-cluster --region us-east-1 --fargate
```

###  Command Breakdown

- `create cluster`: Initiates the creation of a new EKS cluster.
- `--name demo-cluster`: Names the cluster `demo-cluster`.
- `--region us-east-1`: Specifies the AWS region.
- `--fargate`: Enables Fargate, allowing pods to run without managing EC2 instances.

##  What Happens When You Run This

- A new VPC is created (unless specified otherwise).
- The EKS control plane is set up.
- Fargate profiles are created for default namespaces like `default` and `kube-system`.

- Once the cluster is created:

1. Configure `kubectl` to interact with the cluster:
   ```bash
   aws eks --region us-east-1 update-kubeconfig --name demo-cluster
   ```

2. Deploy workloads — they will automatically run on Fargate.
##  Next Steps
<img width="932" height="305" alt="{51D6C59E-436E-4CBE-B44B-07EFD85B641C}" src="https://github.com/user-attachments/assets/e502a3a6-df56-4a4e-a497-6fca85546323" />
<img width="785" height="103" alt="{7A0D60CB-D43A-4164-B0D5-63DC72905A9F}" src="https://github.com/user-attachments/assets/b0283ffe-75a9-45fc-9a24-b87987eae87e" />
<img width="760" height="225" alt="{9666036F-BB89-4ECF-B53D-EB2D639D806F}" src="https://github.com/user-attachments/assets/97f5146a-c699-42ac-9a44-1454614d44e7" />

 API (Application Programming Interface)
A set of rules that allows software components to communicate.
EKS API Server Endpoint Example:

https://<cluster-id>.gr7.us-east-1.eks.amazonaws.com
 ARN (Amazon Resource Name)
A unique identifier for AWS resources.
Example:

arn:aws:eks:us-east-1:<account-id>:cluster/demo-cluster
 OpenID Connect (OIDC)
An authentication protocol used by EKS to associate IAM roles with Kubernetes service accounts.
OIDC Provider URL Example:

https://oidc.eks.us-east-1.amazonaws.com/id/<unique-id>



##  Optional: Custom Fargate Profile

To run workloads in a custom namespace, you can create a Fargate profile like this:

```bash
eksctl create fargateprofile \
  --cluster demo-cluster \
  --name custom-profile \
  --namespace my-namespace
```

---


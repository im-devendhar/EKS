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

Create an Amazon EKS (Elastic Kubernetes Service) cluster using `eksctl` with Fargate enabled.

##  Prerequisites

Before running the command, ensure the following:

-  `eksctl` is installed  
  ```bash
  eksctl version
  ```

-  AWS CLI is configured  
  ```bash
  aws configure
  ```

-  You have sufficient IAM permissions to create EKS clusters and Fargate profiles.

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

<img width="932" height="305" alt="{51D6C59E-436E-4CBE-B44B-07EFD85B641C}" src="https://github.com/user-attachments/assets/e502a3a6-df56-4a4e-a497-6fca85546323" />
<img width="785" height="103" alt="{7A0D60CB-D43A-4164-B0D5-63DC72905A9F}" src="https://github.com/user-attachments/assets/b0283ffe-75a9-45fc-9a24-b87987eae87e" />
<img width="760" height="225" alt="{9666036F-BB89-4ECF-B53D-EB2D639D806F}" src="https://github.com/user-attachments/assets/97f5146a-c699-42ac-9a44-1454614d44e7" />
<img width="931" height="340" alt="{084BF4FA-7B6B-4A94-8DBE-4DA6E9F47DA0}" src="https://github.com/user-attachments/assets/ed111b95-3756-44d5-b796-59e4ccc403ef" />

1. Configure `kubectl` to interact with the cluster:
   ```bash
   aws eks --region us-east-1 update-kubeconfig --name demo-cluster
   ```
   <img width="593" height="48" alt="{37A63F73-2F2B-4685-91DB-73B983F53F6E}" src="https://github.com/user-attachments/assets/89e4099f-d5ec-40a0-b376-20970eede82c" />

```bash
aws eks --region us-east-1 update-kubeconfig --name demo-cluster
```

###  Purpose:
This command configures your local machine to interact with your Amazon EKS cluster using `kubectl`.

###  What It Does:
- Retrieves the cluster details from AWS.
- Updates the **kubeconfig** file located at `~/.kube/config`.
- Adds a new **context** for the specified EKS cluster (`demo-cluster`).
- Enables you to run Kubernetes commands like `kubectl get pods`, `kubectl get nodes`, etc.

###  Output Example:
```bash
Added new context arn:aws:eks:us-east-1:003364514867:cluster/demo-cluster to /home/ubuntu/.kube/config
```

This confirms that your kubeconfig was successfully updated and you can now interact with the cluster using `kubectl`.

---

2. Deploy workloads — they will automatically run on Fargate.

---

##  What is a Fargate Profile and Why Is It Important?

###  Definition
A **Fargate profile** in Amazon EKS defines which Kubernetes **namespaces** (and optionally **labels**) should run on AWS Fargate. It tells EKS:

> “For pods in this namespace (and with these labels), run them on Fargate instead of EC2 nodes.”

---

###  Why Create a Fargate Profile?

Creating a Fargate profile is important because it allows you to:

-  **Run pods without managing EC2 instances**  
  Fargate abstracts away the infrastructure, so you don’t need to provision or scale worker nodes.

-  **Isolate workloads by namespace**  
  You can assign specific namespaces to run on Fargate, making it easier to manage resource allocation and security boundaries.

-  **Simplify cluster operations**  
  No need to worry about patching, scaling, or securing EC2 nodes — AWS handles it.

-  **Enable cost-efficient and serverless Kubernetes**  
  You pay only for the resources your pods use, with no idle EC2 costs.

---

###  Example: Create a Fargate Profile

To create a Fargate profile for a specific namespace in the **`us-east-1`** region:

```bash
eksctl create fargateprofile \
  --region us-east-1 \
  --cluster demo-cluster \
  --name custom-profile \
  --namespace my-namespace
```

###  What This Does:
- Targets the `demo-cluster` in `us-east-1`.
- Creates a profile named `custom-profile`.
- Ensures all pods in `my-namespace` run on Fargate.

---
# 2048 App

## Create Fargate profile
```
eksctl create fargateprofile \
    --cluster demo-cluster \
    --region us-east-1 \
    --name alb-sample-app \
    --namespace game-2048
```


###  Summary

| Feature              | Benefit                              |
|----------------------|---------------------------------------|
| No EC2 management    | Fully serverless pod execution        |
| Namespace targeting  | Fine-grained workload control         |
| Cost optimization    | Pay only for what you use             |
| Simplified scaling   | AWS handles provisioning automatically|


<img width="563" height="129" alt="{CBC9CFA9-5DB6-4B31-B931-1B9522F1D053}" src="https://github.com/user-attachments/assets/f2a756ed-9e77-4857-90b5-3c72d5faca00" />
<img width="943" height="372" alt="image" src="https://github.com/user-attachments/assets/69afb2cd-dade-4bfd-bb3d-254447d4c977" />
When you are deploying the application on EC2 instance you can avoid this step, anyway thats a different case.

You can verify the profile creation using:

```bash
aws eks describe-fargate-profiles \
  --cluster-name demo-cluster \
  --region us-east-1

```


## Deploy the deployment, service and Ingress

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml

```
Here’s a clean and well-structured **README section** you can include in your project to document how you configured the IAM OIDC provider for your EKS cluster after enabling Fargate:

---

##  Configuring IAM OIDC Provider for EKS (Fargate-Compatible)

To enable **IAM Roles for Service Accounts (IRSA)** in your EKS cluster (`demo-cluster`), follow these steps to configure the IAM OIDC provider.

---

###  Step 1: Set Cluster Name

```bash
export cluster_name=demo-cluster
```

---

###  Step 2: Extract OIDC ID from Cluster

```bash
oidc_id=$(aws eks describe-cluster \
  --name $cluster_name \
  --query "cluster.identity.oidc.issuer" \
  --output text | cut -d '/' -f 5)
```

---

###  Step 3: Check if OIDC Provider Already Exists

```bash
aws iam list-open-id-connect-providers | grep $oidc_id | cut -d "/" -f4
```

If the output is empty, it means the IAM OIDC provider is **not yet configured**.

---

###  Step 4: Associate IAM OIDC Provider

```bash
eksctl utils associate-iam-oidc-provider \
  --cluster $cluster_name \
  --region us-east-1 \
  --approve
```

 This step enables IRSA, which is **required for Fargate profiles** to securely access AWS services without using long-lived credentials.

---

###  Why OIDC Is Important in EKS (Especially with Fargate)

- **Fargate pods do not run on EC2 nodes**, so you can't attach IAM roles to nodes.
- OIDC enables **secure, fine-grained access** to AWS services via service accounts.
- It allows pods to assume IAM roles using **short-lived credentials** via AWS STS.
- Essential for workloads that interact with services like S3, DynamoDB, Secrets Manager, etc.

---
Here’s the updated **README section** with the **Helm install step** clearly included:

---

## 🚀 Setting Up AWS Load Balancer Controller (ALB Add-On)

This guide walks through the steps to install the AWS Load Balancer Controller in an EKS cluster (`demo-cluster`) with Fargate support.

---

### 📦 Step 1: Download IAM Policy

```bash
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.11.0/docs/install/iam_policy.json
```

---

### 🔐 Step 2: Create IAM Policy

```bash
aws iam create-policy \
  --policy-name AWSLoadBalancerControllerIAMPolicy \
  --policy-document file://iam_policy.json
```

---

### 🔧 Step 3: Create IAM Role and Kubernetes Service Account

```bash
eksctl create iamserviceaccount \
  --cluster=demo-cluster \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
  --region=us-east-1 \
  --approve
```

> Replace `<your-aws-account-id>` with your actual AWS account ID.

---

### 📥 Step 4: Deploy AWS Load Balancer Controller via Helm

#### Add the EKS Helm chart repository:

```bash
helm repo add eks https://aws.github.io/eks-charts
```

#### Update the repo:

```bash
helm repo update
```

#### Install the controller:

```bash
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system \
  --set clusterName=demo-cluster \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=us-east-1 \
  --set vpcId=<your-vpc-id>
```

> Replace `<your-vpc-id>` with the VPC ID of your EKS cluster.

---

### ✅ Step 5: Verify Deployment

```bash
kubectl get deployment -n kube-system aws-load-balancer-controller
```

You should see the controller running successfully.

---


<img width="732" height="74" alt="{A3A3C6D2-1FD4-4809-A309-639DDC05925E}" src="https://github.com/user-attachments/assets/68242f60-c402-4d2d-92c2-f1ba6a705ed2" />
<img width="664" height="70" alt="{07010085-5288-4D5C-ACB1-37C67374A33D}" src="https://github.com/user-attachments/assets/022d4ff8-a016-4793-abb0-c4a394bebfcb" />
Use the highlighted link to access the deployed application.

## Use the below rep link for clear understanding 

https://github.com/im-devendhar/aws-devops-zero-to-hero/blob/main/day-22

![Screenshot 2023-08-03 at 7 57 15 PM](https://github.com/iam-veeramalla/aws-devops-zero-to-hero/assets/43399466/93b06a9f-67f9-404f-b0ad-18e3095b7353)



---


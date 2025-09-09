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

###  Summary

| Feature              | Benefit                              |
|----------------------|---------------------------------------|
| No EC2 management    | Fully serverless pod execution        |
| Namespace targeting  | Fine-grained workload control         |
| Cost optimization    | Pay only for what you use             |
| Simplified scaling   | AWS handles provisioning automatically|



---


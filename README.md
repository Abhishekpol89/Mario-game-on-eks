
# 🚀 Deploy Super Mario Game on AWS EKS – A Beginner-Friendly Guide

Hey there! 👋  
In this guide, we’re going to do something fun and a bit geeky—we’ll deploy a **Super Mario game** on **AWS EKS (Elastic Kubernetes Service)** using a ready-to-use Docker image. This walkthrough is crafted with beginners in mind, so don’t worry if you're new to Kubernetes or AWS. Just follow along, and you’ll get there!

**Shoutout** to the creator of the Docker image [`kaminskypavel/mario`](https://hub.docker.com/r/kaminskypavel/mario) on Docker Hub—thanks for keeping the nostalgia alive!

> ⚠️ Heads up: Some of the services we’ll use (like EKS and Load Balancer) might incur charges, so keep an eye on your AWS billing dashboard.

---

## 💡 A Note Before We Begin

I’ve done my best to make this tutorial simple and easy to replicate. While anyone can follow the steps, having a basic understanding of Kubernetes and AWS will help you grasp *what’s* happening, not just *how* to do it.

---

## ✅ Prerequisites

We’ll need a few tools installed on your local machine before we begin:

1. **AWS CLI**  
   Install it using the [official AWS CLI installation guide](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html).  
   Once installed, configure it with your IAM credentials using:
   ```bash
   aws configure
   ```

2. **eksctl** – Tool to create and manage EKS clusters  
   [Install eksctl](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html)

3. **kubectl** – Kubernetes command-line tool  
   [Install kubectl](https://kubernetes.io/docs/tasks/tools/)

4. **Git** – To clone the required repository  
   [Install Git](https://git-scm.com/downloads)

Once everything is installed and configured, we’re good to go.

---

## 📁 Step 1: Set Up the Project Directory

Let’s create a workspace for this project:

```bash
mkdir mario && cd mario
```

Now we need the config and deployment files. You can either:

- Clone my repository:
  ```bash
  git clone https://github.com/ajaysnair1122/k8-mario.git && cd k8-mario
  ```

- Or create the files manually. If you're doing it manually, start with `eksctl-config.yaml`:

```yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: mario-cluster
  region: ap-south-1

nodeGroups:
  - name: ng-1
    desiredCapacity: 2
    instanceType: t3.small
```

---

## ⚙️ Step 2: Create the EKS Cluster

Now let’s create the EKS cluster using this config file:

```bash
eksctl create cluster -f eksctl-config.yaml
```

⏳ This might take around 10–15 minutes. Grab a coffee while EKS does its thing ☕.

Once completed, verify that the cluster was created:

```bash
aws eks describe-cluster --name mario-cluster   --query "cluster.{Name: name, Status: status, Endpoint: endpoint, Version: version, DateOfCreation: createdAt}"   --output table
```

Looks pretty cool, right? 😄

---

## 🔗 Step 3: Connect kubectl to the Cluster

To manage your new cluster with `kubectl`, run:

```bash
aws eks --region ap-south-1 update-kubeconfig --name mario-cluster
```

This sets up your kubeconfig to point to the newly created cluster.

---

## 🧱 Step 4: Deploy the Super Mario App

Make sure you have the `deployment.yaml` file in your directory. If not, create it:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mario
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: mario
          image: kaminskypavel/mario
          ports:
            - name: mario
              containerPort: 8080
```

Deploy it using:

```bash
kubectl apply -f deployment.yaml
```

To confirm that everything is running as expected:

```bash
kubectl describe deployment mario
```

---

## 🌐 Step 5: Expose the App via Load Balancer

To access our game in the browser, let’s expose the deployment with a LoadBalancer service.

If not already present, create a `service.yaml` file with the following:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mario-service
  labels:
    app: my-app
spec:
  ports:
    - port: 80
      targetPort: 8080
  selector:
    app: my-app
  type: LoadBalancer
```

Apply the service:

```bash
kubectl apply -f service.yaml
```

Check the service details:

```bash
kubectl describe service mario-service
```

Look for the **LoadBalancer Ingress** address. Once it’s available, copy that URL and paste it into your browser.

🎮 **Super Mario should appear!** Enjoy jumping those nostalgic pipes.

---

## 📝 Final Thoughts

This was my first attempt at documenting a project like this, and I’d love to hear your honest feedback. If you faced any issues or have suggestions for improvement, feel free to reach out or leave a comment.

I hope this helped you successfully deploy a fun little app and also gave you hands-on experience with AWS EKS and Kubernetes!

---

Feel free to [open an issue](https://github.com/your-repo/issues) or contribute if you have improvements or corrections to make this guide even better. 🚀

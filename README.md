# MERN Microservices Deployment on AWS EKS using Jenkins

## Project Overview

This project demonstrates the deployment of a containerized MERN-based microservices application using Docker, Amazon ECR, Amazon EKS, and Jenkins CI/CD.

---

# Architecture Flow Diagram

                    +------------------+
                    |     GitHub       |
                    | Source Repository|
                    +--------+---------+
                             |
                             | Code Push
                             v
                    +------------------+
                    |     Jenkins      |
                    |  CI/CD Pipeline  |
                    +--------+---------+
                             |
                             | Build Docker Images
                             v
                    +------------------+
                    |      Docker      |
                    | Build Containers |
                    +--------+---------+
                             |
                             | Push Images
                             v
                    +-------------------------------+
                    | Amazon Elastic Container      |
                    | Registry (ECR)               |
                    +---------------+---------------+
                                    |
                                    | Pull Images
                                    v
                    +-------------------------------+
                    | Amazon EKS Cluster            |
                    | sample-mern-microservices     |
                    +---------------+---------------+
                                    |
             +----------------------+----------------------+
             |                      |                      |
             v                      v                      v
    +---------------+     +---------------+     +---------------+
    | Frontend Pod  |     | Hello Service |     | Profile Service|
    | React + Nginx |     | Node.js API   |     | Node.js + Mongo|
    +---------------+     +---------------+     +---------------+
                                                         |
                                                         |
                                                         v
                                                +---------------+
                                                | MongoDB       |
                                                | Database      |
                                                +---------------+

                                    |
                                    | Exposed Through
                                    v
                    +-------------------------------+
                    | AWS Load Balancer (ELB)       |
                    +---------------+---------------+
                                    |
                                    v
                            End Users / Browser

---

# Phase 1 – Version Control with Git & Prepare the MERN Application

## Repository Setup

* Forked and maintained project in GitHub.
* Source code stored in GitHub repository.
* Jenkins integrated with GitHub repository.

Environment Configuration

Environment-specific settings were managed using .env files to keep configuration separate from application code.

Backend Services

helloService/.env

PORT=3001

profileService/.env

PORT=3002
MONGO_URL=<MongoDB Connection String>

---

# Phase 2 – Containerization and Amazon ECR

## Dockerization

Created Dockerfiles for:

1. Frontend (React + Nginx)
2. Hello Service
3. Profile Service

## Local Validation

Containers successfully executed using Docker Compose.

Verified:

* Frontend accessible on port 8080
* Hello Service accessible on port 3001
* Profile Service accessible on port 3002
* MongoDB running successfully

### Docker Containers

| Service         | Status  |
| --------------- | ------- |
| Frontend        | Running |
| Hello Service   | Running |
| Profile Service | Running |
| MongoDB         | Running |

### Screenshot Placeholder
<img width="1228" height="524" alt="image" src="https://github.com/user-attachments/assets/0dec38c5-6935-4b83-af51-12906408f223" />

docker ps
<img width="1661" height="88" alt="image" src="https://github.com/user-attachments/assets/2c787391-b9fe-4341-9f38-4c003edb2088" />

---

## Amazon ECR Setup

Created the following repositories:

1. frontend
2. hello-service
3. profile-service

### Screenshot Placeholder

aws ecr describe-repositories

<img width="694" height="708" alt="image" src="https://github.com/user-attachments/assets/96336d9a-19e1-4000-922e-3aac2733560b" />

---

# Phase 3 – AWS Environment Setup

## AWS CLI Configuration

Configured AWS CLI successfully.

Verified using:

aws sts get-caller-identity

aws configure list

### Screenshot Placeholder

<img width="858" height="198" alt="image" src="https://github.com/user-attachments/assets/b4cc8947-a14b-421a-adef-4e8846a1483b" />

---

# Phase 4 – Jenkins Continuous Integration

## Jenkins Environment

Organization Provided Jenkins:

https://jenkinsacademics.herovired.com/

Successfully created Jenkins Pipeline.

Pipeline Name:

vinamra-devops-pipeline

---

## Jenkinsfile Integration

Configured Jenkins Pipeline using:

Pipeline script from SCM

Repository:

https://github.com/vinamra-parashar/sample-MERN-with-Microservices

Branch:

main

Script Path:

Jenkinsfile

<img width="1292" height="842" alt="image" src="https://github.com/user-attachments/assets/a65efe56-cdbf-4851-b37e-992bc21be5cb" />

---

## Docker Build Validation

Successfully built:

* Frontend Docker image
* Hello Service Docker image
* Profile Service Docker image

Frontend image build completed successfully through Jenkins.

### Screenshot Placeholder

<img width="1346" height="408" alt="image" src="https://github.com/user-attachments/assets/4a7d7050-6757-4e2c-b393-cacec9b9b0eb" />

---

## Jenkins Failure Encountered

Pipeline execution reached Amazon ECR authentication stage.

Error:

Unable to locate credentials.

Reason:

AWS credentials were not configured or accessible on the shared Jenkins server.

As a result:

* Docker image build completed successfully.
* ECR login failed.
* Image push stage was skipped.
* Kubernetes deployment stage was skipped.

### Screenshot Placeholder

<img width="1157" height="256" alt="image" src="https://github.com/user-attachments/assets/1f2de673-1e10-4f1a-8ed6-a1b4221e897c" />

---

# Phase 5 – Amazon EKS Deployment

## EKS Cluster Creation

Amazon Elastic Kubernetes Service (EKS) was used to deploy and manage the MERN microservices application.

The EKS cluster was created using `eksctl`, which automatically provisioned the required AWS resources including:

* Dedicated VPC
* Worker Nodes
* Security Groups
* IAM Roles
* CloudFormation Stacks

### Cluster Details

| Property            | Value                     |
| ------------------- | ------------------------- |
| Cluster Name        | sample-mern-microservices |
| Region              | eu-north-1                |
| Kubernetes Platform | Amazon EKS                |
| Deployment Tool     | eksctl                    |

### Screenshot

<img width="2706" height="362" alt="image" src="https://github.com/user-attachments/assets/5960827b-3c69-4872-aac6-58673a616697" />

---

## CloudFormation Stack Creation

During cluster provisioning, AWS CloudFormation automatically created and managed the infrastructure resources required by EKS.

The following stacks were successfully created:

* eksctl-sample-mern-microservices-cluster
* eksctl-sample-mern-microservices-nodegroup-*

These stacks provisioned networking, IAM roles, security groups, and worker nodes.

### Screenshot

<img width="3360" height="710" alt="image" src="https://github.com/user-attachments/assets/d48089ed-894c-43c2-a636-3181678048f2" />

---

## EC2 Worker Nodes

Amazon EC2 instances were provisioned as worker nodes for the Kubernetes cluster.

These nodes are responsible for running application pods and Kubernetes workloads.

Worker nodes were automatically registered with the EKS control plane and became available for scheduling containers.

### Screenshot

<img width="3238" height="556" alt="image" src="https://github.com/user-attachments/assets/499b0543-3114-4bfc-9d51-ce6738b50c0d" />

---

## Kubernetes Namespace Creation

A dedicated namespace was created for application deployment.

Namespace:

mern-app

This namespace logically separates application resources from system components.

### Verification Command

```bash
kubectl get ns
```

### Screenshot

<img width="304" height="105" alt="image" src="https://github.com/user-attachments/assets/434afb5a-2633-43b5-a9fc-c5483a6bd147" />

---

## Application Deployment

The application was deployed to Kubernetes using Deployment manifests.

The following deployments were successfully created:

* frontend
* hello-service
* profile-service

Each deployment manages pod lifecycle, scaling, and availability.

### Verification Command

```bash
kubectl get deploy -A
```

### Screenshot

<img width="502" height="104" alt="image" src="https://github.com/user-attachments/assets/462165ad-567d-49df-9f41-349e5f72f68e" />

---

## Kubernetes Services

Services were created to expose application components within the cluster.

### Services Created

| Service         | Type         |
| --------------- | ------------ |
| frontend        | LoadBalancer |
| hello-service   | ClusterIP    |
| profile-service | ClusterIP    |

### Verification Command

```bash
kubectl get svc -A
```

### Screenshot

<img width="1242" height="135" alt="image" src="https://github.com/user-attachments/assets/f865cf57-9daa-4f0a-8ec9-5c8b86465706" />

---

## External Load Balancer

AWS automatically provisioned an Elastic Load Balancer (ELB) for the frontend service.

The application was successfully accessible through this endpoint.

### Screenshot

<img width="2656" height="672" alt="image" src="https://github.com/user-attachments/assets/e7a21449-a001-49be-8a95-74f0f4a13d46" />

---

## Deployment Verification

The deployment was verified using Kubernetes commands.

### Commands Used

```bash
kubectl get ns

kubectl get deploy -A

kubectl get svc -A

kubectl get pods -A

kubectl get pods -n mern-app
```

<img width="688" height="297" alt="image" src="https://github.com/user-attachments/assets/c5aa6b48-4299-42b7-9c5c-a72330895647" />

Verification confirmed:

* All deployments were healthy.
* Pods were running successfully.
* Services were accessible.
* Frontend application was reachable through the AWS Load Balancer.


# Current Project Status

| Phase                      | Status              |
| -------------------------- | ------------------- |
| Git Setup                  | Completed           |
| Dockerization              | Completed           |
| Amazon ECR Setup           | Completed           |
| AWS CLI Configuration      | Completed           |
| Jenkins Setup              | Completed           |
| Jenkins Pipeline           | Partially Completed |
| ECR Push via Jenkins       | Blocked             |
| EKS Deployment via Jenkins | Blocked             |
| Monitoring                 | Pending             |
| Documentation              | Completed         |

---

# Author

Vinamra Parashar

# Prerequisites

1. AWS Account
2. Programmatic access and AWS configured with AWS CLI
3. Python3 installed
4. Docker and Kubectl installed
5. Code editor [VS code/ Intellij]


# **Cloud Native Resource Monitoring Python App on K8s!**

## Things you will Learn 🤯

1. Python and How to create Monitoring Application in Python using Flask and psutil
2. How to run a Python App locally.
3. Learn Docker and How to containerize a Python application
    1. Creating Dockerfile
    2. Building DockerImage
    3. Running Docker Container
    4. Docker Commands
4. Create ECR repository using Python Boto3 and pushing Docker Image to ECR
5. Learn Kubernetes and Create EKS cluster and Nodegroups
6. Create Kubernetes Deployments and Services using Python!

# **Youtube Video for step by step Demonstration!**

[![Video Tutorial](https://img.youtube.com/vi/kBWCsHEcWnc/0.jpg)](https://youtu.be/kBWCsHEcWnc)

# ✨Let’s Start the Project ✨

## **Part 1: Deploying the Flask application locally**

### **Step 1: Clone the code**

Clone the code from the repository:

```
git clone https://github.com/hussainshahzad250/python-monitoring-app.git
```

### **Step 2: go inside project and install dependencies**
```
cd python-monitoring-app
```
The application uses the **`psutil`** and **`Flask`, Plotly, boto3** libraries. Install them using pip:

```
pip3 install -r requirements.txt
```

### **Step 3: Run the application**

To run the application, navigate to the root directory of the project and execute the following command:

```
python3 app.py
```

This will start the Flask server on **`localhost:5000`**. Navigate to [http://localhost:5000/](http://localhost:5000/) on your browser to access the application.

## **Part 2: Dockerizing the Flask application**

### **Step 1: Create a Dockerfile**

Create a **`Dockerfile`** in the root directory of the project with the following contents:

```
# Use the official Python image as the base image
FROM python:3.9-slim-buster

# Set the working directory in the container
WORKDIR /app

# Copy the requirements file to the working directory
COPY requirements.txt .

RUN pip3 install --no-cache-dir -r requirements.txt

# Copy the application code to the working directory
COPY . .

# Set the environment variables for the Flask app
ENV FLASK_RUN_HOST=0.0.0.0

# Expose the port on which the Flask app will run
EXPOSE 5000

# Start the Flask app when the container is run
CMD ["flask", "run"]
```

### **Step 2: Build the Docker image**

To build the Docker image, execute the following command:
replace ``` 1234567890 ``` with your AWS accountId

```
aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 1234567890.dkr.ecr.ap-south-1.amazonaws.com
```

```
docker build -t python_monitoring_app .
```

```
docker tag python_monitoring_app:latest 1234567890.dkr.ecr.ap-south-1.amazonaws.com/python_monitoring_app:latest
```

```
docker push 1234567890.dkr.ecr.ap-south-1.amazonaws.com/python_monitoring_app:latest
```

### **Step 3: Run the Docker container**

To run the Docker container, execute the following command:

```
docker run -p 5000:5000 python-monitoring-app
```

This will start the Flask server in a Docker container on **`localhost:5000`**. Navigate to [http://localhost:5000/](http://localhost:5000/) on your browser to access the application.

## **Part 3: Pushing the Docker image to ECR**

### **Step 1: Create an ECR repository**

Create an ECR repository using Python:

```
import boto3

# Create an ECR client
ecr_client = boto3.client('ecr')

# Create a new ECR repository
repository_name = 'my-ecr-repo'
response = ecr_client.create_repository(repositoryName=repository_name)

# Print the repository URI
repository_uri = response['repository']['repositoryUri']
print(repository_uri)
```

### **Step 2: Push the Docker image to ECR**

Push the Docker image to ECR using the push commands on the console:

```
docker push <ecr_repo_uri>:<tag>
```

## **Part 4: Creating an EKS cluster and deploying the app using Python**

### **Step 1: Create an EKS cluster**

Create an EKS cluster and add node group

### **Step 2: Create a node group**

Create a node group in the EKS cluster.

### Set the cluster context:
```
aws eks update-kubeconfig --name <cluster_name> --region <aws_region>
```

### **Step 3: Create deployment and service**

```jsx
from kubernetes import client, config

# Load Kubernetes configuration
config.load_kube_config()

# Create a Kubernetes API client
api_client = client.ApiClient()

# Define the deployment
deployment = client.V1Deployment(
    metadata=client.V1ObjectMeta(name="my-flask-app"),
    spec=client.V1DeploymentSpec(
        replicas=1,
        selector=client.V1LabelSelector(
            match_labels={"app": "my-flask-app"}
        ),
        template=client.V1PodTemplateSpec(
            metadata=client.V1ObjectMeta(
                labels={"app": "my-flask-app"}
            ),
            spec=client.V1PodSpec(
                containers=[
                    client.V1Container(
                        name="my-flask-container",
                        image="568373317874.dkr.ecr.us-east-1.amazonaws.com/my-cloud-native-repo:latest",
                        ports=[client.V1ContainerPort(container_port=5000)]
                    )
                ]
            )
        )
    )
)

# Create the deployment
api_instance = client.AppsV1Api(api_client)
api_instance.create_namespaced_deployment(
    namespace="default",
    body=deployment
)

# Define the service
service = client.V1Service(
    metadata=client.V1ObjectMeta(name="my-flask-service"),
    spec=client.V1ServiceSpec(
        selector={"app": "my-flask-app"},
        ports=[client.V1ServicePort(port=5000)]
    )
)

# Create the service
api_instance = client.CoreV1Api(api_client)
api_instance.create_namespaced_service(
    namespace="default",
    body=service
)
```

make sure to edit the name of the image on line 25 with your image Uri.

- Once you run this file by running “python3 eks.py” deployment and service will be created.
- Check by running following commands:

Check deployments
```
kubectl get deployment -n default
```

Check service
```
kubectl get service -n default
```

Check the pods
```
kubectl get pods -n default
```

Once your pod is up and running, run the port-forward to expose the service

```bash
kubectl port-forward service/<service_name> 5000:5000
```

# Destroying the Infrastructure
    To clean up the infrastructure, follow these steps:

1. Delete the Node Group: Delete the node group associated with your EKS cluster.
2. Delete the EKS Cluster: Once the nodes are deleted, you can proceed to delete the EKS cluster itself.



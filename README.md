# Cloud-monitoring-app

This real-time DevOps project involved the end-to-end development and deployment of a cloud-native application. Key aspects covered included creating a monitoring application in Python using Flask, containerizing the application with Docker, and orchestrating it using Kubernetes on AWS. The project showcased a holistic understanding of cloud-native development, containerization, and orchestration, making it a valuable addition to my projects.

**Part 1: Deploying the Flask application locally**

1 Clone the code 
git clone <repository_url>

2 Install dependencies
pip3 install -r requirements.txt

3 Run the application
python3 app.py

**Part 2: Dockerizing the Flask application**

1 Create a Dockerfile
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

2 Build the Docker image
docker build -t <image_name> .

3 Run the Docker container
docker run -p 5000:5000 <image_name>

**Part 3: Pushing the Docker image to ECR**

1 Create an ECR repository

# ECR using python
import boto3

# Create an ECR client
ecr_client = boto3.client('ecr')

# Create a new ECR repository
repository_name = 'my-ecr-repo'
response = ecr_client.create_repository(repositoryName=repository_name)

# Print the repository URI
repository_uri = response['repository']['repositoryUri']
print(repository_uri)

2 Push the Docker image to ECR
docker push <ecr_repo_uri>:<tag>

**Part 4: Creating an EKS cluster and deploying the app using Python**

1 Create an EKS cluster

2 Create a node group

3 Create deployment and service

# import client, config
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

Change image on line 25 with your image Uri.
Once you run this file by running “python3 eks.py” deployment and service will be created.
Check by running following commands:

kubectl get deployment -n default (check deployments)
kubectl get service -n default (check service)
kubectl get pods -n default (to check the pods)

run the port-forward to expose the service
kubectl port-forward service/<service_name> 5000:5000

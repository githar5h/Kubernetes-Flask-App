# Kubernetes Flask App
This repository provides a comprehensive guide to deploying a Python-based Flask application integrated with MongoDB in a containerized and Kubernetes environment. It covers the following:

### Features:

- **Virtual Environment Benefits:** Explains the importance of using virtual environments for dependency management, reproducibility, and system integrity.
  
- **Containerization with Docker:**
  - Includes a sample `Dockerfile` for building Flask application images.
  - Step-by-step instructions to build and push Docker images to a container registry.

- **Kubernetes Deployment:**
  - YAML files for deploying Flask and MongoDB using `Deployments`, `StatefulSets`, `Services`, and `Persistent Volumes`.
  - Details on setting up DNS for inter-pod communication and managing resource requests and limits.

- **Design Highlights:**
  - Persistent storage for MongoDB using StatefulSets.
  - Autoscaling Flask pods with Horizontal Pod Autoscaler (HPA) based on traffic.
  - Clear design choices tailored for local development using Minikube.

- **Testing Scenarios:**
  - Load-testing Flask and MongoDB to simulate high traffic and database operations.
  - Monitoring HPA and MongoDB performance under stress.

### Benefits of Using a Virtual Environment for Python Applications

1. **Dependency Management:** Isolates project dependencies, preventing conflicts between projects.
2. **Reproducibility:** Ensures consistent environments across different machines by tracking exact dependency versions.
3. **System Integrity:** Protects the global Python installation from incompatible packages.
4. **Portability:** Simplifies sharing and deploying projects with consistent environments.

### Instructions on How to Build and Push the Docker Image to a Container Registry

**Step 1: Create a Dockerfile**
- Your Dockerfile should be located in the root of your Flask project directory. Here’s an example:

    ```Dockerfile
    # Use the official Python image as a base
    FROM python:3.8-slim

    # Set the working directory in the container
    WORKDIR /app

    # Copy the current directory contents into the container at /app
    COPY . /app

    # Install any needed packages specified in requirements.txt
    RUN pip install --no-cache-dir -r requirements.txt

    # Make port 5000 available to the world outside this container
    EXPOSE 5000

    # Define environment variable
    ENV MONGODB_URI=mongodb://mongodb:27017/flask_db

    # Run app.py when the container launches
    CMD ["python", "app.py"]
    ```

**Step 2: Build the Docker Image**

- Open your terminal, navigate to the project directory containing your Dockerfile, and run:

    ```bash
    docker build -t your_dockerhub_username/flask-app:v1 .
    ```

**Step 3: Push the Docker Image to Docker Hub**

- First, log in to Docker Hub using your credentials:

    ```bash
    docker login
    ```

- Then push your Docker image to Docker Hub:

    ```bash
    docker push your_dockerhub_username/flask-app:v1
    ```

### Steps to Deploy the Flask Application and MongoDB on a Minikube Kubernetes Cluster

**Step 1: Start Minikube**

- Ensure Minikube is running:

    ```bash
    minikube start
    ```

**Step 2: Create Kubernetes Deployment and Service Files**

- All required files have been provided in the kubernetes directory in this repository.

**Step 3: Apply Kubernetes Files**

- Apply the YAML files:

    ```bash
    kubectl apply -f flask-deployment.yaml
    kubectl apply -f flask-service.yaml
    kubectl apply -f flask-hpa.yaml
    kubectl apply -f mongo-service.yaml
    kubectl apply -f mongo-statefulset.yaml
    kubectl apply -f mongo-persistent-volume.yaml
    ```

**Step 4: Access the Flask Application**

- Retrieve the Minikube IP:

    ```bash
    minikube ip
    ```

- Access the Flask app by navigating to `http://<minikube-ip>:port` in your web browser.
- You should see a webpage similar to the following screenshot:
  <img width="1278" alt="image" src="https://github.com/user-attachments/assets/546b3aa4-3291-4664-a9c9-384e0dff0e68">

### Explanation of How DNS Resolution Works Within the Kubernetes Cluster for Inter-Pod Communication

Kubernetes uses a DNS service to manage the network identities of services within the cluster. Every service gets a DNS entry, which allows pods to communicate with each other using simple DNS names instead of IP addresses. For example, if you have a MongoDB service named `mongo-service`, any pod in the same namespace can connect to it by using `mongo-service:27017`.

Kubernetes automatically creates DNS records for services and pods. These records can be used by other services to look up the IP addresses of the service endpoints. This is crucial for inter-pod communication, as it decouples service discovery from specific IP addresses.

### Explanation of Resource Requests and Limits in Kubernetes

**Resource Requests:**
- Resource requests specify the amount of CPU and memory a container requires. The scheduler uses these requests to decide which node to place the pod on. If a node doesn't have enough resources to satisfy the requests, the pod won't be scheduled on that node.

**Resource Limits:**
- Resource limits define the maximum amount of CPU and memory that a container can use. If a container tries to use more than its limit, it can be throttled or even terminated (in case of memory).

**Example:**

```yaml
resources:
  requests:
    memory: "64Mi"
    cpu: "250m"
  limits:
    memory: "128Mi"
    cpu: "500m"
```

This configuration requests 64MiB of memory and 250m of CPU, but the container cannot exceed 128MiB of memory and 500m of CPU.

### Design Choices

- **Persistent Volume and StatefulSet:** MongoDB requires persistent storage to retain data across pod restarts. A StatefulSet ensures that MongoDB pods are uniquely identifiable and maintain their state, providing stability and reliability for the database.

- **Service Types:** The NodePort service type is used for exposing the Flask application outside the cluster, allowing external access. The ClusterIP service type is used for internal communication with MongoDB, ensuring secure and efficient interactions within the Kubernetes cluster.

- **Autoscaling:** Horizontal Pod Autoscaler (HPA) is set up to scale the Flask application based on CPU usage, ensuring that the application can dynamically handle increased traffic without manual intervention.

### Alternatives Considered

- **Dynamic Storage Provisioning:** 
    - **Reason for Not Choosing:** Dynamic storage provisioning, which automatically provisions storage based on demand, is ideal for production environments where flexibility and scalability are key. However, in this local development setup using Minikube, a static persistent volume was chosen for simplicity and to avoid the complexities of configuring cloud-based storage providers. Dynamic provisioning is not as straightforward in a local setup, and using host paths provides a quick, reliable solution for development.

- **Load Balancer:**
    - **Reason for Not Choosing:** In a cloud environment, a LoadBalancer service type would be the preferred method for managing external traffic, as it offers better load distribution and integrates well with cloud provider features. However, since this setup uses Minikube, which runs on a local machine, a NodePort service type was chosen. NodePort is simpler to configure in a local environment and does not require cloud infrastructure, making it more suitable for local testing and development.

### Testing Scenarios

#### 1. **Generating Traffic to Test HPA Autoscaling**

To test the Horizontal Pod Autoscaler (HPA) and ensure that the Flask application can handle increased traffic, a loop was used to generate continuous requests to the Flask application. This simulated a high-traffic scenario where the CPU usage would rise, prompting the HPA to scale the number of pods.

**Command to generate traffic:**

```bash
while true; do curl http://<minikube-ip>:<node-port>/data; done
```

- Replace `<minikube-ip>` with the IP address provided by Minikube.
- Replace `<node-port>` with the NodePort value of the Flask service.

This command sends continuous GET requests to the `/data` endpoint of the Flask application, generating load on the server.

**Monitoring HPA Autoscaling:**

To monitor the HPA and see how it scales the pods based on CPU usage, the following command was used:

```bash
kubectl get hpa -w
```

This command watches the HPA, providing real-time updates on how many replicas are active and how the CPU usage affects scaling.

#### 2. **Generating Database Load**

To test how the MongoDB database handles load, POST requests were sent in a loop, continuously inserting data into the MongoDB instance.

**Command to generate database load:**

```bash
while true; do curl -X POST -H "Content-Type: application/json" -d '{"sampleKey":"sampleValue"}' http://<minikube-ip>:<node-port>/data; done
```

- Replace `<minikube-ip>` with the IP address provided by Minikube.
- Replace `<node-port>` with the NodePort value of the Flask service.

This command sends continuous POST requests to the `/data` endpoint, simulating a scenario where the application needs to handle a high volume of data inserts.

**Monitoring Database Performance:**

To check how MongoDB is performing under load, monitor the logs of the MongoDB pod:

```bash
kubectl logs <mongodb-pod-name>
```

#### 3. **Issues Encountered**

- **Metrics Setup:** One of the main challenges was getting the metrics server to work correctly. Initially, CPU usage metrics showed as "unknown," which was resolved by reinstalling the metrics server with the correct configuration and ensuring all necessary services were up and running.
- **Autoscaling Delays:** There were slight delays in the HPA responding to sudden traffic spikes, which is expected behavior due to the time it takes for metrics to be collected and for the HPA to react.

# Deploying a Stateful MongoDB Service on Kubernetes Using Helm

## Project Overview
This demo project walks through deploying a stateful service, MongoDB, on a Kubernetes cluster using Helm. It includes setting up a managed Kubernetes cluster with Linode Kubernetes Engine (LKE), deploying a replicated MongoDB service, configuring data persistence, and deploying a Mongo Express UI for MongoDB management. Finally, we use an Nginx ingress controller to expose the UI to the browser.

---

## Technologies Used
- **Kubernetes** (K8s)
- **Helm**
- **MongoDB**
- **Mongo Express**
- **Linode LKE**
- **Linux**

---

## Key Concepts

### Stateful Applications in Kubernetes
- Stateful applications retain data between restarts or across replicas, requiring persistence.
- Kubernetes **StatefulSets** manage stateful pods with unique identities and persistent storage.

### Helm
- Helm simplifies Kubernetes deployments by packaging Kubernetes resources into reusable charts.
- Parameters can be customized via Helm’s `values.yaml` file.

---

## Steps to Complete the Project

### Step 1: Set Up Linode Kubernetes Engine (LKE)
1. Log in to your Linode account.
2. Navigate to the Kubernetes section and create a new cluster.
3. Select the desired node size and count.
4. Save the **Kubeconfig** file to configure `kubectl` locally.
   ```bash
   export KUBECONFIG=/path/to/your-kubeconfig.yaml
   ```
5. Test the cluster connection:
   ```bash
   kubectl get nodes
   ```

### Step 2: Install Helm
1. Install Helm using Chocolatey or a package manager:
   ```bash
   choco install kubernetes-helm
   ```
2. Verify the installation:
   ```bash
   helm version
   ```

### Step 3: Add the Bitnami Repository
We use the Bitnami MongoDB Helm chart for this deployment.
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

### Step 4: Configure MongoDB Parameters
1. Create a `values.yaml` file for MongoDB:
   ```yaml
   architecture: replicaset
   replicaCount: 3
   persistence:
     storageClass: "linode-block-storage"
   auth:
     rootPassword: secret-root-pwd
   ```
2. Install the MongoDB Helm chart:
   ```bash
   helm install mongodb bitnami/mongodb -f values.yaml
   ```
3. Verify the deployment:
   ```bash
   kubectl get pods
   ```

### Step 5: Deploy Mongo Express
1. Create a deployment YAML for Mongo Express:
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: mongo-express
     labels:
       app: mongo-express
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: mongo-express
     template:
       metadata:
         labels:
           app: mongo-express
       spec:
         containers:
         - name: mongo-express
           image: mongo-express
           ports:
           - containerPort: 8081
           env:
           - name: ME_CONFIG_MONGODB_ADMINUSERNAME
             value: root
           - name: ME_CONFIG_MONGODB_SERVER
             value: mongodb-0.mongodb-headless
           - name: ME_CONFIG_MONGODB_ADMINPASSWORD
             valueFrom:
               secretKeyRef:
                 name: mongodb
                 key: mongodb-root-password
   ```
2. Deploy Mongo Express:
   ```bash
   kubectl apply -f mongo-express-deployment.yaml
   ```
3. Create a service for Mongo Express:
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: mongo-express-service
   spec:
     selector:
       app: mongo-express
     ports:
     - protocol: TCP
       port: 8081
       targetPort: 8081
   ```
   ```bash
   kubectl apply -f mongo-express-service.yaml
   ```

### Step 6: Configure Ingress
1. Create an ingress YAML file for Nginx:
   ```yaml
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     annotations:
       kubernetes.io/ingress.class: nginx
     name: mongo-express
   spec:
     rules:
     - host: YOUR_HOST_DNS_NAME
       http:
         paths:
         - path: /
           pathType: Prefix
           backend:
             service:
               name: mongo-express-service
               port:
                 number: 8081
   ```
2. Apply the ingress:
   ```bash
   kubectl apply -f helm-ingress.yaml
   ```
3. Verify the ingress:
   ```bash
   kubectl get ingress
   ```

### Step 7: Test the Deployment
1. Ensure that all pods are running:
   ```bash
   kubectl get pods
   ```
2. Access the Mongo Express UI in your browser using the ingress DNS name.

---

## Key Features
- **Data Persistence**: MongoDB uses Linode’s block storage for persistent volumes, ensuring data durability.
- **Scalability**: MongoDB runs as a replicated service with three replicas for high availability.
- **UI Management**: Mongo Express provides a simple web-based interface for interacting with MongoDB.
- **Ingress**: Nginx ingress routes external traffic to the Mongo Express service securely.

---

## Best Practices
1. **Secure Credentials**:
   - Store sensitive data in Kubernetes Secrets.
   - Rotate passwords regularly.
2. **Monitor Resources**:
   - Use Prometheus and Grafana to monitor pod health and resource usage.
3. **Automate Deployments**:
   - Use CI/CD pipelines to automate Helm deployments.
4. **Backups**:
   - Implement regular database backups using tools like Velero.

---

## Wrapping It Up
Deploying a stateful service like MongoDB on Kubernetes is a perfect way to master Helm and Kubernetes concepts. You’ve got a resilient, scalable database service hooked up to a clean UI, with ingress making everything accessible like a charm. When all pieces click, it feels like cracking a tough puzzle with finesse—smooth, efficient, and a little bit satisfying. Let’s get to the next challenge! 🚀

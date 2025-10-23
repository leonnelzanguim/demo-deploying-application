# 🧩 MongoDB + Mongo Express on Kubernetes

## 📖 Description

This project deploys a **MongoDB + Mongo Express** stack on a **Kubernetes cluster** (for example, Minikube).  
It uses **Secrets**, **ConfigMaps**, **Deployments**, and **Services** to configure, secure, and expose the applications.

MongoDB is used as a NoSQL database, and **Mongo Express** provides a web-based UI to manage and visualize your MongoDB data.

---

## 📁 Project Structure

```
.
├── mongo.yaml              # Deploys MongoDB and its internal Service
├── mongo-express.yaml      # Deploys Mongo Express and its Service (exposed as LoadBalancer)
├── mongo-secret.yaml       # Contains MongoDB credentials as a Kubernetes Secret
├── mongo-configmap.yaml    # Defines MongoDB connection URL
└── README.md               # Project documentation
```

---

## ⚙️ Architecture Overview

```
+----------------------------------------------------------+
|                   Kubernetes Cluster                     |
|----------------------------------------------------------|
|  🗃️ Secret: mongodb-secret                                |
|    - mongo-root-username                                  |
|    - mongo-root-password                                  |
|                                                          |
|  ⚙️ ConfigMap: mongodb-configmap                          |
|    - database_url: mongodb-service:27017                  |
|                                                          |
|  🧱 Deployment: MongoDB                                   |
|    - image: mongo                                         |
|    - expose port 27017                                   |
|    - uses Secret for credentials                         |
|                                                          |
|  🌐 Service: mongodb-service                              |
|    - type: ClusterIP (internal)                          |
|    - exposes port 27017                                  |
|                                                          |
|  🧱 Deployment: Mongo Express                             |
|    - image: mongo-express                                 |
|    - depends on MongoDB                                   |
|    - uses Secret + ConfigMap                              |
|    - exposes port 8081                                   |
|                                                          |
|  🌍 Service: mongo-express-service                        |
|    - type: LoadBalancer                                   |
|    - port: 8081                                           |
|    - accessible through browser                           |
+----------------------------------------------------------+
```

---

## 🧠 How It Works

1. **`mongo-secret.yaml`**  
   Creates a Secret that stores MongoDB credentials:

   ```yaml
   mongo-root-username: mongoadmin
   mongo-root-password: secret
   ```

2. **`mongo-configmap.yaml`**  
   Defines the MongoDB service endpoint:

   ```yaml
   database_url: mongodb-service:27017
   ```

3. **`mongo.yaml`**  
   Deploys MongoDB:

   - Uses the Secret for environment variables:
     - `MONGO_INITDB_ROOT_USERNAME`
     - `MONGO_INITDB_ROOT_PASSWORD`
   - Exposes MongoDB internally using a `ClusterIP` Service.

4. **`mongo-express.yaml`**  
   Deploys Mongo Express:
   - Retrieves credentials from the Secret.
   - Retrieves the database URL from the ConfigMap.
   - Exposes Mongo Express through a `LoadBalancer` Service.

---

## 🚀 Deployment Steps (using Minikube)

### 1️⃣ Start Minikube

```bash
minikube start
```

### 2️⃣ Apply all Kubernetes manifests

```bash
kubectl apply -f mongo-secret.yaml
kubectl apply -f mongo-configmap.yaml
kubectl apply -f mongo.yaml
kubectl apply -f mongo-express.yaml
```

### 3️⃣ Verify resources

```bash
kubectl get pods
kubectl get svc
```

### 4️⃣ Access Mongo Express dashboard

Because Minikube doesn’t create a real cloud LoadBalancer, use:

```bash
minikube service mongo-express-service
```

This command will open your browser with a local URL like:

```
http://127.0.0.1:xxxxx
```

---

## 🧩 Environment Variables Summary

| Variable                          | Source     | Description                         |
| --------------------------------- | ---------- | ----------------------------------- |
| `MONGO_INITDB_ROOT_USERNAME`      | Secret     | MongoDB admin username              |
| `MONGO_INITDB_ROOT_PASSWORD`      | Secret     | MongoDB admin password              |
| `ME_CONFIG_MONGODB_ADMINUSERNAME` | Secret     | Mongo Express connection username   |
| `ME_CONFIG_MONGODB_ADMINPASSWORD` | Secret     | Mongo Express connection password   |
| `DATABASE_URL`                    | ConfigMap  | MongoDB service name and port       |
| `ME_CONFIG_MONGODB_URL`           | Deployment | Full MongoDB connection URL         |
| `ME_CONFIG_BASICAUTH_USERNAME`    | Deployment | Mongo Express web UI login username |
| `ME_CONFIG_BASICAUTH_PASSWORD`    | Deployment | Mongo Express web UI login password |

---

## 🧾 Useful Commands

| Command                                   | Description                                 |
| ----------------------------------------- | ------------------------------------------- |
| `kubectl get pods`                        | List all running pods                       |
| `kubectl get svc`                         | List all services                           |
| `kubectl describe pod <pod-name>`         | Show details for a specific pod             |
| `kubectl logs <pod-name>`                 | View logs of a pod                          |
| `kubectl delete all -l app=mongo-express` | Delete Mongo Express deployment and service |
| `kubectl delete all -l app=mongodb`       | Delete MongoDB deployment and service       |

---

## 🧱 Notes

- This setup is for **development and testing** with **Minikube**.
- In a production environment (e.g., AWS EKS, GCP GKE, Azure AKS), the `LoadBalancer` service will create a **real external IP**.
- Do not store real credentials in plain text; use Kubernetes Secrets safely.

---

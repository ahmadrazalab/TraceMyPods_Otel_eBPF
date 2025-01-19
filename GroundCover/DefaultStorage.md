To create a **default StorageClass** in your kubeadm Kubernetes cluster, you can use any supported storage provisioner, such as **local-path**, **NFS**, or others, depending on your setup. Since you're using EC2 without access to EBS, we'll use a solution like **hostPath** or **local-path**.

Here’s how you can create a **default StorageClass**:

---

### 1. **Install and Configure Local Path Provisioner**
Local Path Provisioner is a lightweight and flexible option for creating a default StorageClass using local storage.

#### Step 1: Apply the Local Path Provisioner YAML
```bash
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/deploy/local-path-storage.yaml
```

#### Step 2: Verify the Installed StorageClass
Once installed, it creates a `local-path` StorageClass. Verify it using:
```bash
kubectl get storageclass
```

#### Step 3: Make it Default
To set this StorageClass as default:
```bash
kubectl patch storageclass local-path -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

---

### 2. **Create Your Own StorageClass Using hostPath**
If you want a simple custom solution:

#### Step 1: Create a StorageClass YAML
Save the following configuration as `hostpath-sc.yaml`:
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: default-hostpath
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```

#### Step 2: Apply the YAML
```bash
kubectl apply -f hostpath-sc.yaml
```

#### Step 3: Test the StorageClass
Create a PersistentVolumeClaim (PVC) and check if it binds to your custom StorageClass:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: test-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: default-hostpath
```
Apply it using:
```bash
kubectl apply -f test-pvc.yaml
```

Then check the PVC status:
```bash
kubectl get pvc
```

---

### 3. **Verify Default StorageClass**
To confirm your default StorageClass:
```bash
kubectl get storageclass
```
The `default` annotation should appear for your selected StorageClass.

---

**Note**: For more Kubernetes guides and detailed steps, visit my blog: [docs.ahmadraza.in](https://docs.ahmadraza.in). 
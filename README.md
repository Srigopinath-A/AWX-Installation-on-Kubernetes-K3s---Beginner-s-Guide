# AWX Installation on Kubernetes (K3s) - Beginner's Guide

This guide provides a step-by-step process for installing Ansible AWX on a Kubernetes cluster using K3s, a lightweight Kubernetes distribution. This guide is tailored for beginners.

## Prerequisites

- Ubuntu 22.04 or later (or a compatible Linux distribution)
- A user account with sudo privileges
- An internet connection

## Steps

### 1. Update and Upgrade the System

Open a terminal and run the following commands:
```bash
sudo apt update && sudo apt upgrade -y
```
This ensures that your system has the latest packages and security updates.

### 2. Install K3s

Run the K3s installation script:
```bash
curl -sfL https://get.k3s.io | sh -
```
This command downloads and executes the K3s installation script. K3s will be installed as a systemd service.

Verify K3s Installation:
```bash
kubectl version
```
This command should display the client and server versions of `kubectl`, confirming that K3s and the `kubectl` command-line tool are installed and configured correctly.

### 3. Configure Access to K3s without sudo

Locate the K3s Configuration File:
```bash
ls -lsa /etc/rancher/k3s/k3s.yaml
```

Change Ownership (Essential for non-sudo access):
```bash
sudo chown Srigopinath-A:Srigopinath-A /etc/rancher/k3s/k3s.yaml
```
Replace `Srigopinath-A` with your actual username.

Configure kubectl (for access as your user):
```bash
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
```

Verify that Nodes are Running:
```bash
kubectl get nodes
```
This will print the Kubernetes nodes that are part of the K3s cluster. They should show a Ready status.

### 4. Install Kustomize

Download and install Kustomize:
```bash
curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh" | bash
```
Move Kustomize into `/usr/local/bin`:
```bash
mv kustomize /usr/local/bin/
```

### 5. Create the Kustomization File (kustomization.yaml)

Create a directory for your Kustomization files:
```bash
mkdir -p /home/Srigopinath-A/awx-kustomization
cd /home/Srigopinath-A/awx-kustomization
```

Create the `kustomization.yaml` file:
```yaml name=kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - github.com/ansible/awx-operator/config/default?ref=0.28.0
  - /home/Srigopinath-A/awx.yml # Path to your awx.yml file (created in step 6)

images:
  - name: quay.io/ansible/awx-operator
    newTag: 0.28.0

namespace: awx
```

### 6. Create the AWX Resource Definition (awx.yml)

Create the `awx.yml` file:
```yaml name=awx.yml
apiVersion: awx.ansible.com/v1beta1
kind: AWX
metadata:
  name: awx
spec:
  service_type: NodePort
  nodeport_port: 30080
```

### 7. Deploy AWX using kubectl and kustomize

Navigate to the kustomization file directory:
```bash
cd /home/Srigopinath-A/awx-kustomization
```

Apply the Kustomization:
```bash
kustomize build . | kubectl apply -f -
```

Monitor the Deployment:
```bash
kubectl get all -n awx
```

### 8. Verify the Pods

To verify that the pods have started, run:
```bash
kubectl get pods -n awx
```
You should see a list of pods, including the AWX Operator pod, the AWX PostgreSQL pod, and the AWX web and task pods. The `STATUS` column should show "Running" for all the pods.

### 9. Get the AWX Administrator Password

AWX generates a random password for the default `admin` user. To retrieve this password, run the following command:
```bash
kubectl get secret awx-admin-password -n awx -o jsonpath='{.data.password}' | base64 --decode
```

### 10. Access AWX

1. **Get the IP Address:**
   ```bash
   kubectl get nodes -o wide
   ```
   Find the IP of the node.

2. **Access AWX:**
   In your web browser, go to `http://<your_system_ip>:30080`. Replace `<your_system_ip>` with the IP address of your system where AWX is deployed.

3. **Log in to AWX:**
   Use the username `admin` and the password you retrieved in Step 9 to log in to the AWX web interface.

## Troubleshooting

- **Pod Status:** If pods are not in the `Running` state, examine their logs:
  ```bash
  kubectl logs <pod_name> -n awx
  ```

- **Network Connectivity:** Double-check the IP address and port you are using to access AWX.

- **AWX Operator Logs:** Check the AWX Operator logs for any errors related to the AWX instance deployment:
  ```bash
  kubectl logs <awx-operator-pod-name> -n awx
  ```

This complete and working solution should help you to test and work on all the features that are required in the environment!

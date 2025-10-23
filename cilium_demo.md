# Cilium Demo in Kubernetes

## 1. Prerequisites Installation on Amazon Linux 2 EC2

### Docker Installation
```bash
sudo yum update -y
sudo yum install -y docker
sudo service docker start
sudo usermod -a -G docker ec2-user
sudo reboot now
```

### Helm Installation
```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

### Kubectl Installation
```bash
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```

### Kind Installation (AMD64 / x86_64)
```bash
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.30.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

## 2. Cluster Configuration

Create `cluster-config.yaml`:
```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
networking:
  disableDefaultCNI: true  
  podSubnet: "***********/16" 
```

## 3. Cluster Creation and Cilium Setup
```bash
# Create cluster
kind create cluster --config cluster-config.yaml --name cilium-demo

# Add Cilium repo
helm repo add cilium https://helm.cilium.io/
helm repo update

# Install Cilium with Hubble
helm install cilium cilium/cilium --namespace kube-system \
    --set hubble.relay.enabled=true \
    --set hubble.ui.enabled=true \
    --set monitoring.enabled=true
```

## 4. Install Cilium CLI
```bash
CILIUM_CLI_VERSION=$(curl -s https://raw.githubusercontent.com/cilium/cilium-cli/master/stable.txt)
CLI_ARCH=amd64
if [ "$(uname -m)" = "aarch64" ]; then CLI_ARCH=arm64; fi
curl -L --remote-name-all https://github.com/cilium/cilium-cli/releases/download/${CILIUM_CLI_VERSION}/cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}
sha256sum --check cilium-linux-${CLI_ARCH}.tar.gz.sha256sum
sudo tar xzvfC cilium-linux-${CLI_ARCH}.tar.gz /usr/local/bin
rm cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}
```

## 5. Demo Environment Setup

### Create Namespaces
```bash
kubectl create namespace online-store
kubectl create namespace payment-system
kubectl create namespace inventory
```

### Deploy Applications
```bash
# Create deployments and services
cat <<EOF | kubectl apply -f -
[Previous YAML content for deployments]
EOF

# Create services
kubectl -n online-store expose deployment web-store --port=80
kubectl -n payment-system expose deployment payment-processor --port=80
kubectl -n inventory expose deployment inventory-service --port=80

# Create test pods
kubectl run test-pod -n default --image=curlimages/curl -- sleep infinity
kubectl run inventory-client -n inventory --image=curlimages/curl -- sleep infinity
```

## 6. Traffic Simulation Setup

Create `traffic-simulator.sh`:
```bash
[Previous traffic simulator script content]
```

Make it executable:
```bash
chmod +x traffic-simulator.sh
```

## 7. Hubble Setup and Monitoring

### Install Hubble CLI
```bash
curl -L --remote-name-all https://github.com/cilium/hubble/releases/latest/download/hubble-linux-amd64.tar.gz
tar xzvf hubble-linux-amd64.tar.gz
sudo mv hubble /usr/local/bin
```

### Setup Monitoring
```bash
# Start Hubble Relay port-forward
kubectl port-forward -n kube-system svc/hubble-relay 4245:80 &

# Start Hubble UI port-forward
kubectl port-forward --address 0.0.0.0 -n kube-system svc/hubble-ui 12000:80 &

# Verify Hubble status
hubble status
```

## 8. Run Demo
```bash
# Start traffic simulation
./traffic-simulator.sh

# In a separate terminal, monitor traffic
hubble observe --follow
```

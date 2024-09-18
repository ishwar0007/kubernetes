# EKS-Grafana-Prometheus-Deployment

<img src="https://github.com/kubernetes/kubernetes/raw/master/logo/logo.png" width="100">

----
## Prerequisite
You should have an EKS cluster that is up and running. Ensure that the AWS CLI, Helm, eksctl, and kubectl are installed on your system. Then, configure the AWS CLI with your access and secret keys.
### Install AwsCli
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version
```
### Install Helm
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
helm version --short
```
### Install Ekctl
```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```
### Install Kubectl
```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
mv kubectl /usr/bin/
kubectl version --client
```
## Configure oidc-provider
```
eksctl utils associate-iam-oidc-provider --cluster Cluster-Name --approve
```
## Deploy Prometheus
First, we need to configure the ebs-csi-controller so that Prometheus can use EBS (Elastic Block Storage) for data storage.
```
eksctl create iamserviceaccount \
    --name ebs-csi-controller-sa \
    --namespace kube-system \
    --cluster Cluster-Name \
    --role-name AmazonEKS_EBS_CSI_DriverRole \
    --role-only \
    --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
    --approve
```
Describe the Addon versions
```
aws eks describe-addon-versions --addon-name aws-ebs-csi-driver
```
Create and config Addon with EKS
```
eksctl create addon --name aws-ebs-csi-driver --cluster Cluster-Name --service-account-role-arn arn:aws:iam::654654515587:role/AmazonEKS_EBS_CSI_DriverRole --force
```
Check the newly created Addon
```
eksctl get addon --name aws-ebs-csi-driver --cluster Cluster-Name
```
Update the version of the newly created Addon
```
eksctl update addon --name aws-ebs-csi-driver --version v1.34.0-eksbuild.1 --cluster Cluster-Name \
--service-account-role-arn arn:aws:iam::654654515587:role/AmazonEKS_EBS_CSI_DriverRole --force
```
Now add the Help repo of Prometheus
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```
Deploy the Prometheus with the following command.
```
kubectl create namespace prometheus
helm upgrade -i prometheus prometheus-community/prometheus \
--namespace prometheus \
--set alertmanager.persistentVolume.storageClass="gp2",server.persistentVolume.storageClass="gp2"
```
*** Note: In some cases, the Alertmanager or server PVC may get stuck in a pending state and fail to connect to the storage class. To resolve this, use the command below to fix the PVC for Alertmanager. ***
```
kubectl patch pvc storage-prometheus-alertmanager-0 -n prometheus --type='merge' -p '{"spec": {"storageClassName": "ebs-gp2"}}'
```
To access the Prometheus
```
kubectl port-forward prometheus-server-dd484f8d9-j8pxl -n prometheus 9090
```



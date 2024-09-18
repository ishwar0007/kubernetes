# Kubernetes (AWS)

## Install Helm
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
helm version --short
```

## Install Ekctl
```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

## Install Kubectl
```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
mv kubectl /usr/bin/
kubectl version --client
```

## Install AwsCli
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version
```

## Create Kubernetes Cluster
It will automatically create Kubernetes Cluster and Nodes. It will take around 15 - 20 min to complete
```
eksctl create cluster --name First --node-type t2.small --nodes 1 --nodes-min 1 --nodes-max 2 --region ap-south-1 --zones=ap-south-1a,ap-south-1b,ap-south-1c
eksctl get cluster --name First --region ap-south-1
```

## Connect Kubernetes Cluster on the server
```
aws eks --region ap-south-1 describe-cluster --name First --query cluster.status
aws eks --region ap-south-1 update-kubeconfig --name First
kubectl get svc
eksctl get cluster --name First --region ap-south-1
```

## Implement SSL
1. Objtain SSL certificate (private.key, certificate.crt)
2. Convert the SSL certificate into the yaml file:-
```
kubectl create secret tls linuxhunter-tls --key private.key --cert certificate.crt --dry-run=client --output=yaml > ssl-apply.yml
```
If you do not want convert the ssl certificates into a YAML file then you can directly apply the Certificates:- 
```
kubectl create secret tls linuxhunter-tls --key private.key --cert certificate.crt
```

3. If you have not created the YAML file for the SSL then remove the ssl-apply.yml from the kustomization.yaml file.

## LoadBalancer
```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install nginx-ingress ingress-nginx/ingress-nginx --set controller.publishService.enabled=true
kubectl --namespace default get services -o wide -w nginx-ingress-ingress-nginx-controller
```

## To Deploy kubernetes Project
```
kubectl apply -k ./
```

# Kubernetes (Digitalocean)

Launch the kubernets as per the requirement.
Need 2 packages on the server:- doctl and kubectl

## Install doctl
https://docs.digitalocean.com/reference/doctl/how-to/install/
```
sudo snap install doctl
sudo snap connect doctl:kube-config
sudo snap connect doctl:ssh-keys :ssh-keys
sudo snap connect doctl:dot-docker
```

## Install Helm
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
helm version --short
```

## Install Kubectl
```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
mv kubectl /usr/bin/
```

## To Configure the kubernetes on the linux server
1. Create the following directories :- 
```
mkdir -p /root/.config /root/.kube
```
2. Connect the Digitalocean on the commandline
```
doctl auth init
```
To generate the access token on Digitalocean visit this link = https://cloud.digitalocean.com/account/api/tokens

3. Connect the kubectl with the digitalocean cluster
```
doctl kubernetes cluster kubeconfig save use_your_cluster_name
```

4. Check the nodes to verify the kubernetes connection
```
kubectl get nodes
```

## Implement SSL
1. Objtain SSL certificate (private.key, certificate.crt)
2. Convert the SSL certificate into the yaml file:-
```
kubectl create secret tls linuxhunter-tls --key private.key --cert certificate.crt --dry-run=client --output=yaml > ssl-apply.yml
```
If you do not want convert the ssl certificates into a YAML file then you can directly apply the Certificates:- 
```
kubectl create secret tls linuxhunter-tls --key private.key --cert certificate.crt
```

3. If you have not created the YAML file for the SSL then remove the ssl-apply.yml from the kustomization.yaml file.

## Load balancer:-
```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install nginx-ingress ingress-nginx/ingress-nginx --set controller.publishService.enabled=true
kubectl --namespace default get services -o wide -w nginx-ingress-ingress-nginx-controller
```

## To Deploy kubernetes Project
```
kubectl apply -k ./
```

# Important Note
1. To run the whole project with one command:- " kubectl apply -k ./ " The above command will execute the kustomization.yaml file.
2. I have created a NFS with two directories, one is mounted with the MySQL and other is mounted with wordpress with the help of PVC.
3. If you want the change the NFS, then change the server ip and also create the directories before deploying the project. Access Mode for the NFS must be ReadWriteMany.
4. The mysql-secret.yaml file carries the password of MySQL.
5. For PhpMyAdmin :- We need to mention the PhpMyAdmin Url in the phpmyadmin deployment file and then setup the ingress file.
    nginx.ingress.kubernetes.io/rewrite-target: "/$2"
    path: /phpmyadmin(/|$)(.*)

## Useful Commands
```
kubectl get pods
kubectl get pods -o wide
kubectl get nodes
kubectl get services
kubectl get pvc
kubectl get ingress
kubectl exec -it httpd-8987f559f-f26mq bash
kubectl apply -f deployment.yml
kubectl delete -f deployment.yml
kubectl describe pod wordpress-5c466bfbf7-fb2wj
kubectl logs wordpress-5c466bfbf7-fb2wj
```

## Remember
1. Wordpress internal links will only work if the path and the annotations must be same as mentioned.

  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/rewrite-target: "/$1"

path: /?(.*

2. PhpMyAdmin will only work if the path and the annotations must be same as mentioned.
```
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/rewrite-target: "/$2"

path: /phpmyadmin(/|$)(.*)
```

3. If you are using both wordpess and the phpmyadmin then use the following annotations and the path.
```
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/use-regex: "true"
    nginx.ingress.kubernetes.io/rewrite-target: "/$1$2"

path :-

path: /?(.*
path: /phpmyadmin(/|$)(.*)
```

5. If phpmyadmin is not working then check the URL in the phpmyadmin deployment file. If you running the phpmyadmin directly on the loadbalancer ip then simply comment the URL lines from the deployment file. If you are using the domain name then mention the domain name in the deployment file. If you are using the phpmyadmin on the sub path (example.com/phpmyadmin) then mention the domain with the sub path in the URL section of the deployment file.

Wordpress:- http://kube.linuxhunter.in
PhpMyAdmin:- http://kube.linuxhunter.in/phpmyadmin (will not work, / is require at the end of phpmyadmin)
PhpMyAdmin:- http://kube.linuxhunter.in/phpmyadmin/ (work)

Useful Doc https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nginx-ingress-on-digitalocean-kubernetes-using-helm

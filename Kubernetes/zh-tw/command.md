# Common Commands

## kubectl

#### 設定預設 Namespace
`kubectl config set-context --current --namespace=<NAMESPACE>`

#### 查看單位(pod, svc, deployment...)的詳細資訊
`kubectl describe <OBJECT>`

#### 查看當前 API 版本
`kubectl api-versions`

#### port forwarding for test
`kubectl port-forward <POD> -n <NAMESPACE> <NODE_PORT>:<POD_PORT>`    
`kubectl port-forward svc/<SERVICE> <PORT>`

#### 獲取/查看物件
`kubectl get all --all-namespaces`  
`kubectl get pods --show-labels`  
`kubectl get svc -n <NAMESPACE>`  
`kubectl get secret`

#### 查看log
`kubectl logs -l app=<LABEL_APP> --namespace <NAMESPACE> --tail=-1`  
`kubectl logs <POD> --tail=10`  
`kubectl logs <POD> -c <CONTAINER>`

#### 查看詳情
`kubectl describe <POD> --namespace <NAMESPACE>`  
`kubectl describe svc <SERVICE> -n <NAMESPACE>`  
`kubectl describe hpa <HPA>`

#### 暴露 K8s 端口
`kubectl proxy --port=8443`

#### 建立資料夾內的所有設置
`kubectl create -f <FOLDER>`

#### 刪除資料夾內的所有設置
`kubectl delete -f <FOLDER>`

## Helm

#### 安裝 Chart
`helm install <RELEASE-NAME> <FOLDER-PATH> -n <NAMESPACE> --create-namespace`  
`helm install <RELEASE-NAME> <FOLDER-PATH> --version v1.0.0 --set <VALUE>`  
`helm install <FOLDER-PATH> --name-template <NAME-TEMPLATE> --values values.yaml --wait`

#### 更新 Chart
`helm upgrade --install <RELEASE-NAME> <FOLDER-PATH> --repo <REPO-URL> -n <NAMESPACE> --create-namespace`  
`helm upgrade <RELEASE-NAME> <FOLDER-PATH> --force`

#### 解除安裝 Release
`helm uninstall <RELEASE-NAME> --namespace <NAMESPACE>`

#### 增加 Helm 儲存庫位址
`helm repo add <REPO-NAME> <REPO-URL> --force-update`

#### 更新 Helm 儲存庫
`helm repo update`

## Minikube

#### 啟動 Minikube
`minikube start --driver=docker --cpus=max --memory=max --extra-config=apiserver.service-node-port-range=1-65535`

#### 使用 local docker image
```
# Start minikube
minikube start

# Set docker env
eval $(minikube docker-env)             # Unix shells
minikube docker-env | Invoke-Expression # PowerShell

# Build image
docker build -t foo:0.0.1 .

# Run in Minikube
kubectl run hello-foo --image=foo:0.0.1 --image-pull-policy=Never

# Check that it's running
kubectl get pods
```
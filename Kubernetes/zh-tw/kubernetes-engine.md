# Kubernetes Engines
Kubernetes (K8s) 主要是一套規範（specification），**並不是一套特定的軟體**，實務中，有很多實作了這些規範的引擎可以選擇；這些規範沒有嚴格限制實作的方式，因此在不同引擎上在功能的啟動、安裝、設定上會有些許的不同，但大體上的邏輯是一樣的。

## Minikube
一個非常簡易的 K8s 引擎，主要用來練習用。具體是在不同的 Driver (e.g. Linux, Docker) 上起一個 Minikube 的虛擬機，接著將 K8s 所有的功能都放在這個封閉的環境裡，適合那些想要理解 K8s 基本操作及概念的人，而對於那些想要接觸對外功能 (e.g. ingress) 的人或架設簡易正式的 local K8s server 的人較不適合。

### 優點
- 簡單好上手。
  - 預先幫你裝好一些常用插件 (e.g. dashboard)。
- 所有的東西都包在一個獨立的 VM 裡，在運行、查看上比較不會影響其他用途的東西。
- 可以直接存取虛擬機查看當前的 K8s 設定 (學習用途)。
  - [參考資料][minikube-k8s-config]

### 缺點
- 都包在獨立的 VM 裡，在查看、除錯時只能透過它給的 interface (e.g. dashboard)。
- 一些對外的功能邏輯與其他 K8s 引擎不太一樣。
  > 由於 minikube 多包了一層 VM ，因此它的 Node 並不是運行的那台機器。
  - 如 ClusterIP, NodePort, LoadBalancer 的實作 (可否直接 Access 的邏輯)。
  - ingress 的對外邏輯。

### 常見問題
- driver 選擇
  - 大部分情況都會選擇 `--driver=docker` 設置
  - 在進階使用時可以了解一下 `--driver=none` 設置
    - [官方文件][minikube-none-driver]
- 如何使用 minikube 架設 local server (如何暴露 minikube 給外部存取)
  - minikube 基本上是設計來給你熟悉 K8s 內部運作的，如果要測試對外功能(或是架設 server)，最好使用其他 K8s engine。[參考資料][minikube-expose]
    
    > **Minikube** as a development tool for a single node Kubernetes cluster provides inherent isolation layer between Kubernetes and the external devices (being specific the inbound traffic to your cluster from LAN/WAN).
- 設定 Resource request / limit 時 Memory 不夠用

  Minikube 預設使用全部的 cpu 以及部分的 memory，想要強制其使用全部的 cpu 以及 memory 可以使用  
  `minikube start --cpus=max --memory=max` 指令 (記得要先**刪除**原本的 minikube instance)
- service 無法指定較低的 port

  K8s 預設 port range 為 30000 - 32767，想要使用較低的 port 來測試可以通過 extra-config 參數更改。  
  例如: `minikube start --extra-config=apiserver.service-node-port-range=1-65535`

- 無法使用 local docker image

  K8s 獨立於 Docker Engine 之外，因此，若要使用 local docker image，則需要做額外的操作，並在 minikube 啟動後重新 build 一次 docker image。
  - [StackOverflow - How can I use local Docker images with Minikube?][minikube-local-image]
    > ```
    > # Start minikube
    > minikube start
    > 
    > # Set docker env
    > eval $(minikube docker-env)             # Unix shells
    > minikube docker-env | Invoke-Expression # PowerShell
    > 
    > # Build image
    > docker build -t foo:0.0.1 .
    > 
    > # Run in Minikube
    > kubectl run hello-foo --image=foo:0.0.1 --image-pull-policy=Never
    > 
    > # Check that it's running
    > kubectl get pods
    > ```

## Docker Desktop(integrated)
Docker Desktop 內部實作的 K8s 引擎，可以在設定裡開起(預設關閉)。本質上依然是一個 local K8s，但功能上實作的還算完整，可以作為一個簡易的 local K8s server 使用。

### 優點
- 設置/安裝簡單(設定勾選即可)。
- 不用多包一層 VM，對外的功能(e.g. ingress)可以正常運作。
  - 可以在上面架一個簡單的 local server
- 支援使用 local docker image
  > 通常來說，K8s 獨立於 Docker Engine，因此在 Fetch image 時只能使用 DockerHub 上的 image，若要使用 local image 需要做額外的操作與設定。[minikube 額外設定](#常見問題)
  >
  > 而 Docker Desktop 因為將兩個系統結合在一起，因此在部屬時可以直接存取 local image，這點非常適合用來測試正在開發的 local server。

### 缺點
- 從 Docker Desktop 介面顯示時，K8s container 會與其他來源的 container 混在一起(而且名字很長)。
  - 可以透過安裝 K8s dashboard 解決。
- Bug 還不少，在對 K8s 除錯時，常常也需要考慮 Docker Desktop 的 Bug。
  - 搜尋 issue 時可以針對 Docker Desktop 相關關鍵字找找。

[minikube-none-driver]: https://minikube.sigs.k8s.io/docs/drivers/none/ "minikube/docs - drivers/none"
[minikube-k8s-config]: https://stackoverflow.com/questions/64758012/location-of-kubernetes-config-directory-with-docker-desktop-on-windows "stackoverflow.com/location-of-kubernetes-config-directory"
[minikube-expose]: https://stackoverflow.com/questions/66088041/how-to-expose-minikube-cluster-to-internet "stackoverflow.com/how-to-expose-minikube-cluster-to-internet"
[minikube-local-image]: https://stackoverflow.com/questions/42564058/how-can-i-use-local-docker-images-with-minikube "stackoverflow.com/how-can-i-use-local-docker-images-with-minikube"
# Common Knowledge
Common Knowledge 主要描述那些關於 Kubernetes 的基礎知識，由於其可能是比較基本的知識，較不適合放在特定功能的主題裡。

## Pods
K8s 裡最基本的單位，相當於 container 之於 docker；與 container 不同的是，pod 可以包含多個 container，但為了 scaling 時可以更彈性，通常只會包含一個 container。  

通過指定 Kind 為 Pod 可以直接建立 pod，但通常會使用 Deployment 來建立 pod，在各種設定上可以更彈性一些。

## API
K8s 對每個功能/單位，都是由不同的 API 建立的。每個 API 也都各有不同的版本，版本更新頗快，在查詢相關文件或範例時要記得注意版本。
> 可以使用 `kubectl api-versions` 指令查看當前支援的 API 版本

## RBAC
RBAC (Role-Based Access Control) 是 K8s 裡負責管理角色/權限的單位，**每一個**功能幾乎都與它有掛鉤，在使用 K8s 裡的大部分功能(尤其是遇到 Bug)時都需要注意這個功能的設定。
- [參考文件][RBAC-jimmysong]

## DNS for Services and Pods
K8s 也提供了 DNS 來讓其他 pod 可以輕鬆的獲取其他 pod/svc 的 cluster ip。
- [官方文件][K8s-DNS]
- [小信豬的原始部落][k8s-dns-godleon]

## Ingress vs LoadBalancer
Ingress 與 LoadBalancer 同樣都提供了對外暴露的窗口，兩者不同的是，前者主要負責處理 route 邏輯、加密驗證，並不負責負載平衡 (即便有些 Ingress Controller e.g. Nginx-Ingress 幫你做了)；而後者專注於處理針對於 service 的負載平衡。

在實際部屬上，Ingress 需要配合 Ingress Controller 自行部屬在其中一台 Node 上；而 LoadBalancer，礙於其負載量的需求，通常由第三方(如 Cloudflare)，或你的 K8s 雲端平台 (如 AWS、GCP) 提供。

- [Baeldung - Ingress vs. Load Balancer in Kubernetes][ingress-vs-loadbalancer]

## HPA
HPA (Horizontal Pod Autoscaler) 是 K8s 裡負責做 scaling 的單位，他會根據已註冊的 metric server 獲取相對應 metric 數值並計算要 scaling 的數量。
- [官方文件][K8s-HPA]
- [Source code 分析][HPA-Source]

預設的 metrics server 只提供 cpu 及 memory usage 兩個 metric。
- [參考資料][Resource-Ray]

若需要額外的 metric 來幫助 scaling，可以參考 custom metric 以及 external metric，[此處][stackoverflow-metric]很好的解釋了三種 metric 的差別。
> In short:  
> - metrics supports only basic metric like CPU or Memory.
> - custom.metrics allows you to extend basic metrics to all Kubernetes objects (http_requests, number of pods, etc.).
> - external.metrics allows to gather metrics which are not Kubernetes objects:
>   - External metrics allow you to autoscale your cluster based on any metric available in your monitoring system. Just provide a metric block with a name and selector, as above, and use the External metric type > instead of Object  
>
> For more detailed description, please check this [doc][K8s-Metric].


## Dashboard
官方用來監控的 Dashboard，比起指令以及 Docker Desktop 的介面好用很多，在有使用者介面的環境下非常推薦使用。

在登入時需要由 RBAC 建立的 Service Account 登入，Helm 版本則需要此 Service Account 的 [bearer token][K8s-Auth]。在環境允許的情況下推薦使用 long-lived service account，可以省略掉每次都要重新創建 token 的麻煩。
- [官方建立 Service Account 範例][dashboard-create-user]
- [創建 long-lived service account][K8s-long-lived-sa]

### 常見問題
- dashboard 持續啟動失敗 / 重開，透過 `kubectl get pods` 查看維持在 **Pending** 狀態。
  - Docker Desktop 重開
    - [Dashboard GitHub - Issue][dashboard-issue-restart]
  - Docker Desktop 更新版本。
    - [Dashboard GitHub - Issue][dashboard-issue-version]

## Helm
一套管理 K8s 專案 (Chart) 的工具。  
它可以將整套 K8s 設置建立成一個專案 (Chart)，並使用版本控制 Chart 的更新；並且所有的設置都儲存在 yaml 檔裡，讓你可以輕鬆的管理 / 發布你的 Chart。
- [Helm 官方文件][helm-docs]

在可以使用 Helm 安裝的情況下，**非常推薦使用 Helm 來管理專案**。

為了方便管理整個 Chart 裡的 value，Helm 有一套獨特的 helm-template 語法(待修正)，算是小小的學習成本。
- [Helm Template 官方文件][helm-template]

## 學習資源
- [qikqiak][qikqiak-github]

[K8s-DNS]: https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/ "kubernetes.io/docs/dns-pod-service"
[RBAC-jimmysong]: https://jimmysong.io/kubernetes-handbook/concepts/rbac.html
[K8s-HPA]: https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/ "kubernetes.io/docs/horizontal-pod-autoscale-walkthrough"
[Resource-Ray]: https://medium.com/learn-or-die/kubernetes-resources-limit-%E8%B3%87%E6%BA%90%E9%99%90%E5%88%B6-803bac05c061 "medium/ray-lee"
[HPA-Source]: https://zwforrest.github.io/post/v2beta2%E7%89%88%E6%9C%AChpa/
[dashboard-issue-restart]: https://github.com/kubernetes/dashboard/issues/8765 "kubernetes/dashboard - Issue"
[dashboard-issue-version]: https://github.com/kubernetes/dashboard/issues/8909 "kubernetes/dashboard - Issue"
[dashboard-create-user]: https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md "kubernetes/dashboard/creating-sample-user"
[K8s-Auth]: https://kubernetes.io/docs/reference/access-authn-authz/authentication/ "kubernetes.io/docs/authentication"
[K8s-long-lived-sa]: https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#manually-create-a-long-lived-api-token-for-a-serviceaccount "kubernetes.io/docs/service_account"
[helm-docs]: https://helm.sh/docs/ "helm.sh/docs"
[helm-template]: https://helm.sh/docs/chart_best_practices/templates/#names-of-defined-templates "helm.sh/docs/templates"
[stackoverflow-metric]: https://stackoverflow.com/questions/58151513/hpa-not-fetching-existing-custom-metric "stackoverflow.com/hpa-metric-explaination"
[K8s-Metric]: https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/#autoscaling-on-metrics-not-related-to-kubernetes-objects "kubernetes.io/docs/horizontal-pod-autoscale-walkthrough"
[qikqiak-github]: https://github.com/heylel/qikqiak-blog "github.com/heylel/qikqiak-blog"
[ingress-vs-loadbalancer]: https://www.baeldung.com/ops/kubernetes-ingress-vs-load-balancer "baeldung.com/kubernetes-ingress-vs-load-balancer"
[k8s-dns-godleon]: https://godleon.github.io/blog/Kubernetes/k8s-DNS-for-Service-and-Pod/ "godleon.github.io/k8s-DNS-for-Service-and-Pod"
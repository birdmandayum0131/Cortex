# Setup HPA custom metrics
**custom metric** 是其中一種 HPA 可以採用的自定義 metric (另一種詳見 **external metric**)，HPA 透過定期向 custom metric API server 發送 request 來獲得 metric 數值並以此計算 scaling 的預期數量。

要使得自定義 metric 可用，則必須要自行架設一個 custom metric API server，此 API server 開放 HPA 格式的 API 並向 K8s server 註冊。

常見的方法有兩種，一種為自行架設簡易的 API server (golang, javascript...)並利用的 library 向 K8s 註冊；另一種為利用已經寫好的 Prometheus Adapter 將 Prometheus Metric 轉換為 HPA metric 格式(同前者原理)。

## 透過 Prometheus Adapter 建立 custom metric
> Reference: [Github - Prometheus Adapter Walkthrough]
1. 確認自定義的 metric 已經以 prometheus metric 的形式部屬在 prometheus server 上。
   - [啟用 Nginx Ingress 預先定義的 prometheus metric][nginx-ingress-prometheus]
     - [部分 nginx metric 不會一開始就存在][nginx_requests-issue]
   - 部屬自定義的 Prometheus Metric。 [參考範例1][golang-prometheus-metric-1]、[參考範例2][golang-prometheus-metric-2]
2. 安裝 Prometheus Adapter
   - 確認 prometheus 網址確實對應到 **prometheus operator** 的 service
   - 確認 rule 設定是否符合當前 HPA metric 的規範 (否則舊的 tag 會被忽略)
3. 在 HPA 上指定相對應的 custom metric 以及 target value
   - 透過 `kubectl describe <HPA_NAME>` 指令確認 Event 的狀態
   - 檢查 Prometheus Adapter Metric Server 的 log 裡的 query 紀錄

### 常見問題
- HPA 找不到相對應的 metric
  - 透過 `kubectl describe <HPA_NAME>` 查看 HPA 的 Event
  - 確認**對應 metric server**的 log
  - 確認該 metric 是否已經註冊為**同 HPA namespace** 的 namespace resource
    - [Prometheus Adapter / Issue - Unable to fetch metrics][hpa-issue-1]
      > You'll want a resource for namespace, too. That's probably the issue. If you have a namespaced resource, you need to have mappings for both namespace and the resource (and ingress is namespaced).
    - [Prometheus Adapter / Issue - HPA is unable to fetch custom metrics from API][hpa-issue-2]
    - [Server Fault / Discuss - Kubernetes: horizontal auto-scaling based on metrics in another namespace][hpa-discuss-1]
    - [StackOverflow / Disscuss - HPA with different namespaces][hpa-disscuss-2]
      > HPA is a namespaced resource. It means that it can only scale Deployments which are in the same Namespace as the HPA itself.
  - 同上，檢查是否有哪些 label 已經無效、被忽略。
      > e.g. 
      > ```
      > - resources:
      >     overrides:
      >       namespace: resource: "namespace"
      > ```
      > 版本前後被變更為
      > ```
      > - resources:
      >     overrides:
      >       kubernetes_namespace: resource: "namespace"
      > ```
      > 再變更為
      > ```
      > - resources:
      >     overrides:
      >       exported_namespace: resource: "namespace"
      > ```
    ***大雷坑***

## 自行架設 Prometheus Adapter Server
(TBF)




[Github - Prometheus Adapter Walkthrough]: https://github.com/kubernetes-sigs/prometheus-adapter/blob/master/docs/walkthrough.md
[nginx-ingress-prometheus]: https://github.com/kubernetes/ingress-nginx/blob/main/docs/user-guide/monitoring.md "ingress-nginx/monitoring"
[nginx_requests-issue]: https://github.com/kubernetes/ingress-nginx/issues/6937 "kubernetes/ingress-nginx-issue"
[golang-prometheus-metric-1]: https://atbug.com/kubernetes-pod-autoscale-on-prometheus-metrics/
[golang-prometheus-metric-2]: https://jhandguy.github.io/posts/advanced-horizontal-autoscaling/
[hpa-issue-1]: https://github.com/kubernetes-sigs/prometheus-adapter/issues/150 "kubernetes-sigs/prometheus-adapter - issue"
[hpa-issue-2]: https://github.com/kubernetes-sigs/prometheus-adapter/issues/157 "kubernetes-sigs/prometheus-adapter - issue"
[hpa-discuss-1]: https://serverfault.com/questions/979985/kubernetes-horizontal-auto-scaling-based-on-metrics-in-another-namespace "serverfault.com/kubernetes-horizontal-auto-scaling-based-on-metrics-in-another-namespace"
[hpa-disscuss-2]: https://stackoverflow.com/questions/66423005/hpa-with-different-namespaces "stackoverflow.com/hpa-with-different-namespaces"
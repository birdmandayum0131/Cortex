# Prometheus
Prometheus 是一套開源的監控工具，它定義了一套 prometheus metric 的資料格式並定期 query 你提供的 metric API。只要你的 metric API 符合 prometheus 的規範，便可以透過 Prometheus server 監控這些 metric，並通過 alert 系統發出警告。

Prometheus 並不依賴於 K8s，它們是兩套獨立的系統，但為了方便，你可以在 K8s 裡透過 [prometheus operator](#prometheus-operator) 部屬 Prometheus Server。

Prometheus 預設的監控介面有點簡陋，因此它通常會配合 grafana 來使用。

## Query functions
Prometheus 提供多種不同的 query function，這些 query function 讓你可以將原本的 metric 轉換成更易於使用的形式 (如 total_requests -> query_per_seccond)，你可以利用這些 function 來組合不同的 metric，亦或是讓你的 Alert 條件更細節。
- [prometheus.io - Query functions][prometheus-query-functions]

## Prometheus Operator
基本上就是 Prometheus 在 K8s 裡的實作，如果你想將 Prometheus 架在 K8s 基本上一定會透過它。  
一個比較常用的 prometheus helm chart - [kube-prometheus-stack](#kube-prometheus-stack) 也是依賴於 Prometheus Operator。

## Kube-Prometheus-Stack
一套頗為常見的 Prometheus Helm Chart，它包含了 prometheus server ([prometheus operator](#prometheus-operator))以及 grafana 等多個系統，並且預設定好他們之間的連接，算是一套開箱即用的 Prometheus Helm Chart。
- [GitHub - prometheus-community/kube-prometheus-stack][kube-prometheus-stack-github]
- [iThome 大大解析][kube-prometheus-stack-guide]

### 常見問題
- 在 Docker Desktop 安裝失敗，node-exporter 炸了。
  - 調整 `hostRootFsMount` 參數
    - [GitHub Issue - prometheus-community/helm-charts][kube-prometheus-stack-issue]  
    
    我的測試裡最後使用的 command
    ```
    helm upgrade prometheus prometheus-community/kube-prometheus-stack \
        --namespace prometheus --create-namespace \
        --set prometheus.prometheusSpec.podMonitorSelectorNilUsesHelmValues=false \
        --set prometheus.prometheusSpec.serviceMonitorSelectorNilUsesHelmValues=false \
        --set prometheus-node-exporter.hostRootFsMount.enabled=false \
        --set prometheus-node-exporter.hostRootFsMount.mountPropagation='HostToContainer' \
    ```

## 學習資源
- [qikqiak][qikqiak-prometheus]



[kube-prometheus-stack-github]: https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack "github.com/prometheus-community/helm-charts/kube-prometheus-stack"
[kube-prometheus-stack-guide]: https://ithelp.ithome.com.tw/articles/10330704 "ithome/mikehsu0618"
[kube-prometheus-stack-issue]: https://github.com/prometheus-community/helm-charts/issues/467 "github.com/prometheus-community/helm-charts/kube-prometheus-stack - issue"
[qikqiak-prometheus]: https://www.qikqiak.com/k8strain2/monitor/prometheus/ "qikqiak.com/prometheus"
[prometheus-query-functions]: https://prometheus.io/docs/prometheus/latest/querying/functions/ "prometheus.io/docs/querying-functions"
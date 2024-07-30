# Setup a server on Kubernetes
想要設置一個簡易的 K8s server，並且透過 TLS 加密，有三樣東西是必不可少的。

1. 第一個便是 Ingress 設定，Ingress 告訴 K8s 如何 route、route 到哪裡、使用哪個憑證等...。
2. 第二個則是 Ingress Controller，Ingress Controller 按照要求執行**所有** Ingress 的設置，是實際接收 / 轉發 request 的人。
3. 第三個是 cert-manager，它負責幫你 建立 / 管理 / 更新 你所有的憑證，當然，如果你不嫌麻煩，你依然可以手動代替它。

## cert-manager
想讓你的 Server 走 HTTPS 就勢必要註冊/管理你的 TLS 憑證，而 cert-manager 便是來幫你做這些事的。  

- [Medium / Ken Chen - 在kubernetes上使用cert-manager自動更新Let’s Encrypt TLS憑證][cert-manger-ken]

### Issuer / Cluster Issuer
理解 cert-manager 時第一步就是認識 Issuer 以及 Cluster Issuer (以下都以 **Issuer** 代稱)，簡而言之，它們是幫你 創建 / 獲得 憑證的工具。你必須先建立、設定好 Issuer，指定它對應的憑證頒發機構 (e.g. Let's Encrypt)、設定驗證方式，它才能幫你向這些機構獲取新的憑證，亦或是更新你的憑證。

兩者之間最主要的區別就是，Issuer 是 namespace resource，即它只能創建與它同 namespace 的憑證，相對的，Cluster Issuer 則可以創建所有 namespace 的憑證。

在抉擇兩者時，建議考慮你的 K8s 系統裡，是否會需要包含多個不同的憑證(網域)，如果是，則建議使用 Issuer 來做管理；若非，即你的 K8s 系統基本上暴露在同一網域上，則可使用 Cluster Issuer 以減少不必要的麻煩。
- [cert-manager 官方文件][cert-manager-docs]

### 學習資源
- [通過 Helm 安裝 cert-manager][cert-manager-helm]

## Nginx-Ingress
K8s 的 ingress controller 規範相較於 Nginx 這種行之有年的 proxy server 簡易的多，再加上 Ngnix 輕量的架構，這使得 Nginx 相當適合用來實作 ingress controller，Nginx-Ingress 便由此誕生。

Nginx-Ingress 和原本的 Nginx 一樣，架構輕量、安裝簡單，文件也相當齊全，是入門時的好選擇。
- [官方教學][ingress-nginx-guide]

### TLS
想要讓 Server 支援 TLS，ingress controller 的設置也必不可少，官方文件也很友善的使用 [cert-manger](#cert-manager) 來管理憑證。
- [官方教學][ingress-nginx-tls]
  > 其實官方文件就已經足夠詳細了
- [51CTO - Kubernetes中Nginx Ingress + Cert-Manager实现自动Https][ingress-nginx-51cto]

[cert-manger-ken]: https://medium.com/@kenchen_57904/%E5%9C%A8kubernetes%E4%B8%8A%E4%BD%BF%E7%94%A8cert-manager%E8%87%AA%E5%8B%95%E6%9B%B4%E6%96%B0lets-encrypt-tls%E6%86%91%E8%AD%89-834b65d43c96 "Medium/Ken Chen"
[ingress-nginx-guide]: https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/ "kubernetes.github.io/user-guide"
[ingress-nginx-tls]: https://kubernetes.github.io/ingress-nginx/user-guide/tls/ "kubernetes.github.io/ingress-nginx/tls"
[ingress-nginx-51cto]: https://www.51cto.com/article/718190.html "51cto.com/Nginx Ingress + Cert-Manager"
[cert-manager-docs]: https://cert-manager.io/docs/configuration/acme/ "cert-manger.io/docs/acme"
[cert-manager-helm]: https://cert-manager.io/docs/installation/helm/ "cert-manager.io/docs/helm"
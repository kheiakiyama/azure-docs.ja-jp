---
title: Azure Kubernetes Service (AKS) の静的 IP アドレスを使用して HTTP イングレス コントローラーを作成する
description: Azure Kubernetes Service (AKS) クラスターの静的パブリック IP アドレスを使用して NGINX イングレス コントローラーをインストールして構成する方法を説明します。
services: container-service
author: iainfoulds
ms.service: container-service
ms.topic: article
ms.date: 08/30/2018
ms.author: iainfou
ms.openlocfilehash: 3e65fc863d065e68948f417fcc22ececcf5271c8
ms.sourcegitcommit: 5a1d601f01444be7d9f405df18c57be0316a1c79
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/10/2018
ms.locfileid: "51515439"
---
# <a name="create-an-ingress-controller-with-a-static-public-ip-address-in-azure-kubernetes-service-aks"></a>Azure Kubernetes Service (AKS) の静的パブリック IP アドレスを使用してイングレス コントローラーを作成する

イングレス コントローラーは、リバース プロキシ、構成可能なトラフィック ルーティング、および Kubernetes サービスの TLS 終端を提供するソフトウェアです。 個別の Kubernetes サービスのイングレス ルールとルートを構成するには、Kubernetes イングレス リソースが使われます。 イングレス コントローラーとイングレス ルールを使用すれば、1 つの IP アドレスで Kubernetes クラスター内の複数のサービスにトラフィックをルーティングできます。

この記事では、Azure Kubernetes Service (AKS) クラスターに [NGINX イングレス コントローラー][nginx-ingress]を展開する方法について説明します。 イングレス コントローラーは、静的パブリック IP アドレスを使用して構成します。 [Let's Encrypt][lets-encrypt] 証明書を自動的に生成して構成するには、[cert-manager][cert-manager] プロジェクトを使用します。 最後に、それぞれが 1 つの IP アドレスでアクセスできる、2 つのアプリケーションを AKS クラスターで実行します。

さらに、以下を実行できます。

- [外部のネットワーク接続を使用して基本的なイングレス コントローラーを作成する][aks-ingress-basic]
- [HTTP アプリケーションのルーティング アドオンを有効にする][aks-http-app-routing]
- [ご自身の TLS 証明書を使用するイングレス コントローラーを作成する][aks-ingress-own-tls]
- [Let's Encrypt を使用して動的パブリック IP アドレス付きの TLS 証明書を自動的に生成するイングレス コントローラーを作成する][aks-ingress-tls]

## <a name="before-you-begin"></a>開始する前に

この記事では、Helm を使用し、NGINX イングレス コントローラー、cert-manager およびサンプル Web アプリをインストールします。 Helm は、AKS クラスター内で初期化され、Tiller 用のサービス アカウントが使用されている必要があります。 最新リリースの Helm を使用していることを確認します。 アップグレード手順については、「[Helm のインストール ドキュメント][helm-install]」を参照してください。Helm の構成および使用方法の詳細については、「[Azure Kubernetes Service (AKS) での Helm を使用したアプリケーションのインストール][use-helm]」を参照してください。

この記事では、Azure CLI バージョン 2.0.41 以降も実行している必要があります。 バージョンを確認するには、`az --version` を実行します。 インストールまたはアップグレードする必要がある場合は、[Azure CLI のインストール][azure-cli-install]に関するページを参照してください。

## <a name="create-an-ingress-controller"></a>イングレス コントローラーを作成する

既定では、NGINX イングレス コントローラーは新しいパブリック IP アドレスの割り当てによって作成されます。 このパブリック IP アドレスは、イングレス コントローラーの有効期間のみ静的で、コントローラーを削除して再作成すると失われます。 一般的な構成要件は、NGINX イングレス コントローラーに既存の静的パブリック IP アドレスを提供することです。 イングレス コントローラーが削除されても、静的パブリック IP アドレスはそのまま残ります。 このアプローチでは、アプリケーションの有効期間を通じて一貫した方法で既存の DNS レコードとネットワーク構成を使用できます。

静的パブリック IP アドレスを作成する必要がある場合は、まず [az aks show][az-aks-show] コマンドを使用して AKS クラスターのリソース グループ名を取得します。

```azurecli
az aks show --resource-group myResourceGroup --name myAKSCluster --query nodeResourceGroup -o tsv
```

次に、[az network public-ip create][az-network-public-ip-create] コマンドを使用して、*static* 割り当て方式を使用してパブリック IP アドレスを作成します。 次の例では、前の手順で取得した AKS クラスター リソース グループに *myAKSPublicIP* という名前のパブリック IP アドレスを作成します。

```azurecli
az network public-ip create --resource-group MC_myResourceGroup_myAKSCluster_eastus --name myAKSPublicIP --allocation-method static
```

次に、Helm を使用して *nginx-ingress* グラフをデプロイします。 `--set controller.service.loadBalancerIP` パラメーターを追加し、前の手順で作成した独自のパブリック IP アドレスを指定します。 追加された冗長性については、NGINX イングレス コントローラーの 2 つのレプリカが `--set controller.replicaCount` パラメーターでデプロイされています。 イングレス コントローラーのレプリカの実行から十分にメリットを享受するには、AKS クラスターに複数のノードが存在していることを確認します。

> [!TIP]
> 次の例では、イングレス コントローラーおよび `kube-system` 名前空間の証明書をインストールします。 ご自身の環境には必要に応じて別の名前空間を指定することができます。 また、ご利用の AKS クラスターが RBAC 対応でない場合は、コマンドに `--set rbac.create=false` を追加します。

```console
helm install stable/nginx-ingress \
    --namespace kube-system \
    --set controller.service.loadBalancerIP="40.121.63.72"  \
    --set controller.replicaCount=2
```

次の出力例に示すように、NGINX イングレス コントローラー用の Kubernetes ロード バランサー サービスが作成されると、静的 IP アドレスが割り当てられます。

```
$ kubectl get service -l app=nginx-ingress --namespace kube-system

NAME                                        TYPE           CLUSTER-IP    EXTERNAL-IP    PORT(S)                      AGE
dinky-panda-nginx-ingress-controller        LoadBalancer   10.0.232.56   40.121.63.72   80:31978/TCP,443:32037/TCP   3m
dinky-panda-nginx-ingress-default-backend   ClusterIP      10.0.95.248   <none>         80/TCP                       3m
```

イングレス ルールはまだ作成されていないため、パブリック IP アドレスを参照すると、NGINX イングレス コントローラーの既定の 404 ページが表示されます。 イングレス ルールは、後続の手順で構成します。

## <a name="configure-a-dns-name"></a>DNS 名を構成する

HTTPS 証明書が正常に動作するには、イングレス コントローラーの IP アドレスに FQDN を構成します。 次のスクリプトを更新して、イングレス コントローラーの IP アドレスと、FQDN として使用する一意の名前に変更します。

```console
#!/bin/bash

# Public IP address of your ingress controller
IP="40.121.63.72"

# Name to associate with public IP address
DNSNAME="demo-aks-ingress"

# Get the resource-id of the public ip
PUBLICIPID=$(az network public-ip list --query "[?ipAddress!=null]|[?contains(ipAddress, '$IP')].[id]" --output tsv)

# Update public ip address with DNS name
az network public-ip update --ids $PUBLICIPID --dns-name $DNSNAME
```

FQDN 経由でイングレス コントローラーにアクセスできるようになります。

## <a name="install-cert-manager"></a>cert-manager をインストールする

NGINX イングレス コントローラーは、TLS の終端をサポートしています。 HTTPS の証明書を取得および構成するには、いくつかの方法があります。 この記事では、[Lets Encrypt][lets-encrypt] 証明書を自動的に作成および管理する機能を提供する [cert-manager][cert-manager] の使用方法を示します。

> [!NOTE]
> この記事では、Let's Encrypt に `staging` 環境を使用します。 運用環境の展開では、リソースの定義と Helm チャートのインストールに `letsencrypt-prod` と `https://acme-v02.api.letsencrypt.org/directory` を使用します。

RBAC が有効になっているクラスターに cert-manager コントローラーをインストールするには、次の `helm install` コマンドを使用します。 ここでも、必要に応じて、`--namespace` を *kube-system* 以外のものに変更します。

```console
helm install stable/cert-manager \
  --namespace kube-system \
  --set ingressShim.defaultIssuerName=letsencrypt-staging \
  --set ingressShim.defaultIssuerKind=ClusterIssuer
```

クラスターで RBAC が無効な場合、代わりに次のコマンドを使用します。

```console
helm install stable/cert-manager \
  --namespace kube-system \
  --set ingressShim.defaultIssuerName=letsencrypt-staging \
  --set ingressShim.defaultIssuerKind=ClusterIssuer \
  --set rbac.create=false \
  --set serviceAccount.create=false
```

cert-manager の構成の詳細については、[cert-manager プロジェクト][cert-manager]についてのページを参照してください。

## <a name="create-a-ca-cluster-issuer"></a>CA クラスター発行者を作成する

証明書を発行する前に、cert-manager には [Issuer][cert-manager-issuer] リソースまたは [ClusterIssuer][cert-manager-cluster-issuer] リソースが必要です。 これらの Kubernetes リソースは機能面では同一ですが、`Issuer` は単一の名前空間で機能し、`ClusterIssuer` はすべての名前空間にわたって機能します。 詳細については、[cert-manager issuer][cert-manager-issuer]についてのページを参照してください。

次のマニフェスト例を使用して、`cluster-issuer.yaml` などのクラスター発行者を作成します。 メール アドレスを実際の組織の有効なアドレスで置き換えてください。

```yaml
apiVersion: certmanager.k8s.io/v1alpha1
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    email: user@contoso.com
    privateKeySecretRef:
      name: letsencrypt-staging
    http01: {}
```

発行者を作成するには、`kubectl apply -f cluster-issuer.yaml` コマンドを使用します。

```
$ kubectl apply -f cluster-issuer.yaml

clusterissuer.certmanager.k8s.io/letsencrypt-staging created
```

## <a name="create-a-certificate-object"></a>証明書オブジェクトを作成する

次に、証明書リソースを作成する必要があります。 証明書リソースでは、必要な X.509 証明書を定義します。 詳細については、[cert-manager の証明書][cert-manager-certificates]についてのページを参照してください。

次のマニフェスト例を使用して、`certificates.yaml` などのクラスター リソースを作成します。 *dnsNames* と *domains* を前の手順で作成した DNS 名に更新します。 内部専用のイングレス コントローラーを使用する場合は、サービスに内部 DNS 名を指定します。

```yaml
apiVersion: certmanager.k8s.io/v1alpha1
kind: Certificate
metadata:
  name: tls-secret
spec:
  secretName: tls-secret
  dnsNames:
  - demo-aks-ingress.eastus.cloudapp.azure.com
  acme:
    config:
    - http01:
        ingressClass: nginx
      domains:
      - demo-aks-ingress.eastus.cloudapp.azure.com
  issuerRef:
    name: letsencrypt-staging
    kind: ClusterIssuer
```

証明書のリソースを作成するには、`kubectl apply -f certificates.yaml` コマンドを使用します。

```
$ kubectl apply -f certificates.yaml

certificate.certmanager.k8s.io/tls-secret created
```

## <a name="run-demo-applications"></a>デモ アプリケーションを実行する

イングレス コントローラーと証明書管理ソリューションを構成しました。 ここでは、AKS クラスターで 2 つのデモ アプリケーションを実行します。 この例では、Helm を使って、単純な 'Hello world' アプリケーションのインスタンスを 2 つ実行します。

サンプルの Helm チャートをインストールする前に、次のように Helm 環境に Azure サンプル リポジトリを追加します。

```console
helm repo add azure-samples https://azure-samples.github.io/helm-charts/
```

次のコマンドを使用して、Helm チャートから最初のデモ アプリケーションを作成します。

```console
helm install azure-samples/aks-helloworld
```

デモ アプリケーションの 2 番目のインスタンスをインストールします。 2 番目のインスタンスでは、2 つのアプリケーションを見た目で区別できるように、新しいタイトルを指定します。 また、一意サービス名を指定する必要があります。

```console
helm install azure-samples/aks-helloworld --set title="AKS Ingress Demo" --set serviceName="ingress-demo"
```

## <a name="create-an-ingress-route"></a>イングレス ルートを作成する

両方のアプリケーションが Kubernetes クラスターで実行されるようになりましたが、構成されているサービスの種類は `ClusterIP` です。 そのため、アプリケーションにはインターネットからアクセスできません。 公開するには、Kubernetes イングレス リソースを作成します。 イングレス リソースでは、2 つのアプリケーションのいずれかにトラフィックをルーティングするルールを構成します。

次の例では、アドレス `https://demo-aks-ingress.eastus.cloudapp.azure.com/` へのトラフィックが `aks-helloworld` という名前のサービスにルーティングされることに注意してください。 アドレス `https://demo-aks-ingress.eastus.cloudapp.azure.com/hello-world-two` へのトラフィックは、`ingress-demo` サービスにルーティングされます。 *hosts* と *host* を前の手順で作成した DNS 名に更新します。

`hello-world-ingress.yaml` という名前のファイルを作成し、次の例の YAML 内にコピーします。

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: hello-world-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
    certmanager.k8s.io/cluster-issuer: letsencrypt-staging
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  tls:
  - hosts:
    - demo-aks-ingress.eastus.cloudapp.azure.com
    secretName: tls-secret
  rules:
  - host: demo-aks-ingress.eastus.cloudapp.azure.com
    http:
      paths:
      - path: /
        backend:
          serviceName: aks-helloworld
          servicePort: 80
      - path: /hello-world-two
        backend:
          serviceName: ingress-demo
          servicePort: 80
```

`kubectl apply -f hello-world-ingress.yaml` コマンドを使用してイングレス リソースを作成します。

```
$ kubectl apply -f hello-world-ingress.yaml

ingress.extensions/hello-world-ingress created
```

## <a name="test-the-ingress-configuration"></a>イングレスの構成をテストする

Web ブラウザーで、*https://demo-aks-ingress.eastus.cloudapp.azure.com* などの Kubernetes イングレス コントローラーの FQDN コントローラーを開きます。

これらの例は、`letsencrypt-staging` を使用するので、発行された SSL 証明書はブラウザーでは信頼されていません。 アプリケーションに続けるために警告のプロンプトを受け入れます。 証明書情報には、この *Fake LE Intermediate X1* 証明書が Let's Encrypt によって発行されていると表示されます。 この偽の証明書は、`cert-manager` が要求を正しく処理し、プロバイダーから証明書を受け取ったことを示します。

![Let's Encrypt ステージング証明書](media/ingress/staging-certificate.png)

Let's Encrypt を変更し、`staging` ではなく `prod` が使用されるように変更した場合、Let's Encrypt によって発行された信頼された証明書が次の例のように使用されます。

![Let's Encrypt の証明書](media/ingress/certificate.png)

Web ブラウザーには、アプリケーションのデモが表示されています。

![アプリケーションの例 1](media/ingress/app-one.png)

ここで *https://demo-aks-ingress.eastus.cloudapp.azure.com/hello-world-two* などの */hello-world-two* パスを FQDN に追加します。 カスタム タイトルが表示された 2 番目のデモ アプリケーションが表示されています。

![アプリケーションの例 2](media/ingress/app-two.png)

## <a name="clean-up-resources"></a>リソースのクリーンアップ

この記事では、Helm を使用して、イングレス コンポーネント、証明書、およびサンプル アプリをインストールしました。 Helm グラフをデプロイすると、多数の Kubernetes リソースが作成されます。 これらのリソースには、ポッド、デプロイ、およびサービスが含まれます。 クリーンアップするには、最初に証明書リソースを削除します。

```console
kubectl delete -f certificates.yaml
kubectl delete -f cluster-issuer.yaml
```

次に、`helm list` コマンドで Helm リリースを一覧表示します。 次の出力例に示すように、*nginx-ingress*、*cert-manager*、および *aks-helloworld* という名前のグラフを探します。

```
$ helm list

NAME                    REVISION    UPDATED                     STATUS      CHART                   APP VERSION NAMESPACE
waxen-hamster           1           Tue Oct 16 17:44:28 2018    DEPLOYED    nginx-ingress-0.22.1    0.15.0      kube-system
alliterating-peacock    1           Tue Oct 16 18:03:11 2018    DEPLOYED    cert-manager-v0.3.4     v0.3.2      kube-system
mollified-armadillo     1           Tue Oct 16 18:04:53 2018    DEPLOYED    aks-helloworld-0.1.0                default
wondering-clam          1           Tue Oct 16 18:04:56 2018    DEPLOYED    aks-helloworld-0.1.0                default
```

`helm delete` コマンドでリリースを削除します。 次の例では、NGINX イングレス デプロイ、証明書マネージャー、および 2 つのサンプルの AKS hello world アプリを削除します。

```
$ helm delete waxen-hamster alliterating-peacock mollified-armadillo wondering-clam

release "billowing-kitten" deleted
release "loitering-waterbuffalo" deleted
release "flabby-deer" deleted
release "linting-echidna" deleted
```

次に AKS hello world アプリの Helm repo を削除します。

```console
helm repo remove azure-samples
```

サンプル アプリにトラフィックを送信していたイングレス ルートを削除します。

```console
kubectl delete -f hello-world-ingress.yaml
```

最後に、イングレス コントローラー用に作成した静的パブリック IP アドレスを削除します。 *MC_myResourceGroup_myAKSCluster_eastus* など、この記事の最初の手順で取得した *MC_* クラスター リソース グループ名を指定します。

```azurecli
az network public-ip delete --resource-group MC_myResourceGroup_myAKSCluster_eastus --name myAKSPublicIP
```

## <a name="next-steps"></a>次の手順

この記事には、AKS への外部コンポーネントが含まれています。 これらのコンポーネントの詳細については、次のプロジェクト ページを参照してください。

- [Helm CLI][helm-cli]
- [NGINX イングレス コントローラー][nginx-ingress]
- [cert-manager][cert-manager]

さらに、以下を実行できます。

- [外部のネットワーク接続を使用して基本的なイングレス コントローラーを作成する][aks-ingress-basic]
- [HTTP アプリケーションのルーティング アドオンを有効にする][aks-http-app-routing]
- [内部のプライベート ネットワークと IP アドレスを使用するイングレス コントローラーを作成する][aks-ingress-internal]
- [ご自身の TLS 証明書を使用するイングレス コントローラーを作成する][aks-ingress-own-tls]
- [動的パブリック IP アドレスを使用してイングレス コントローラーを作成し、Let's Encrypt を構成して TLS 証明書を自動的に生成する][aks-ingress-tls]

<!-- LINKS - external -->
[helm-cli]: https://docs.microsoft.com/azure/aks/kubernetes-helm#install-helm-cli
[cert-manager]: https://github.com/jetstack/cert-manager
[cert-manager-certificates]: https://cert-manager.readthedocs.io/en/latest/reference/certificates.html
[cert-manager-cluster-issuer]: https://cert-manager.readthedocs.io/en/latest/reference/clusterissuers.html
[cert-manager-issuer]: https://cert-manager.readthedocs.io/en/latest/reference/issuers.html
[lets-encrypt]: https://letsencrypt.org/
[nginx-ingress]: https://github.com/kubernetes/ingress-nginx
[helm-install]: https://docs.helm.sh/using_helm/#installing-helm

<!-- LINKS - internal -->
[use-helm]: kubernetes-helm.md
[azure-cli-install]: /cli/azure/install-azure-cli
[az-aks-show]: /cli/azure/aks#az-aks-show
[az-network-public-ip-create]: /cli/azure/network/public-ip#az-network-public-ip-create
[aks-ingress-internal]: ingress-internal-ip.md
[aks-ingress-basic]: ingress-basic.md
[aks-ingress-tls]: ingress-tls.md
[aks-http-app-routing]: http-application-routing.md
[aks-ingress-own-tls]: ingress-own-tls.md

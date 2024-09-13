# 認証用Kubernetes設定ファイルの生成

本実習では、[Kubernetes設定ファイル](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/)(kubeconfigsとしても知られています)を生成します。このファイルは、KubernetesクライアントがKubernetes APIサーバーの場所を特定し、認証することを可能にします。

## クライアント認証設定

このセクションでは、`kubelet` と `admin` ユーザー用の kubeconfig ファイルを生成します。

### kubeletのKubernetes設定ファイル

Kubelet用のkubeconfigファイルを生成する際には、Kubeletのノード名に一致するクライアント証明書を使用する必要があります。これにより、KubeletがKubernetes [Node Authorizer](https://kubernetes.io/docs/admin/authorization/node/)によって適切に認証されます。

> 以下のコマンドは、[TLS証明書の生成](04-certificate-authority.md)実習でSSL証明書を生成するのに使用したのと同じディレクトリで実行する必要があります。

node-0ワーカーノードのkubeconfigファイルを生成：

```bash
for host in node-0 node-1; do
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://server.kubernetes.local:6443 \
    --kubeconfig=${host}.kubeconfig

  kubectl config set-credentials system:node:${host} \
    --client-certificate=${host}.crt \
    --client-key=${host}.key \
    --embed-certs=true \
    --kubeconfig=${host}.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:node:${host} \
    --kubeconfig=${host}.kubeconfig

  kubectl config use-context default \
    --kubeconfig=${host}.kubeconfig
done
```

結果:

```text
node-0.kubeconfig
node-1.kubeconfig
```

### kube-proxy Kubernetes設定ファイル

`kube-proxy`サービスのkubeconfigファイルを生成：

```bash
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://server.kubernetes.local:6443 \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-credentials system:kube-proxy \
    --client-certificate=kube-proxy.crt \
    --client-key=kube-proxy.key \
    --embed-certs=true \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-proxy \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config use-context default \
    --kubeconfig=kube-proxy.kubeconfig
}
```

結果:

```text
kube-proxy.kubeconfig
```

### kube-controller-manager Kubernetes設定ファイル

`kube-controller-manager`サービス用のkubeconfigファイルを生成：

```bash
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://server.kubernetes.local:6443 \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-credentials system:kube-controller-manager \
    --client-certificate=kube-controller-manager.crt \
    --client-key=kube-controller-manager.key \
    --embed-certs=true \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-controller-manager \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config use-context default \
    --kubeconfig=kube-controller-manager.kubeconfig
}
```

結果:

```text
kube-controller-manager.kubeconfig
```


### kube-scheduler Kubernetes設定ファイル

`kube-scheduler`サービス用のkubeconfigファイルを生成：

```bash
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://server.kubernetes.local:6443 \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-credentials system:kube-scheduler \
    --client-certificate=kube-scheduler.crt \
    --client-key=kube-scheduler.key \
    --embed-certs=true \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-scheduler \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config use-context default \
    --kubeconfig=kube-scheduler.kubeconfig
}
```

結果:

```text
kube-scheduler.kubeconfig
```

### admin Kubernetes設定ファイル

`admin`ユーザー用のkubeconfigファイルを生成：

```bash
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=admin.kubeconfig

  kubectl config set-credentials admin \
    --client-certificate=admin.crt \
    --client-key=admin.key \
    --embed-certs=true \
    --kubeconfig=admin.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=admin \
    --kubeconfig=admin.kubeconfig

  kubectl config use-context default \
    --kubeconfig=admin.kubeconfig
}
```

結果:

```text
admin.kubeconfig
```

## Kubernetes設定ファイルの配布

`kubelet` と `kube-proxy` の kubeconfig ファイルを node-0 インスタンスにコピーします：

```bash
for host in node-0 node-1; do
  ssh root@$host "mkdir /var/lib/{kube-proxy,kubelet}"
  
  scp kube-proxy.kubeconfig \
    root@$host:/var/lib/kube-proxy/kubeconfig \
  
  scp ${host}.kubeconfig \
    root@$host:/var/lib/kubelet/kubeconfig
done
```

`kube-controller-manager` と `kube-scheduler` の kubeconfig ファイルをコントローラインスタンスにコピー：

```bash
scp admin.kubeconfig \
  kube-controller-manager.kubeconfig \
  kube-scheduler.kubeconfig \
  root@server:~/
```

Next: [Generating the Data Encryption Config and Key](06-data-encryption-keys.md)

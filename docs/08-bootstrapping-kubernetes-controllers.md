# Kubernetesコントロールプレーンのブートストラップ

本実習では、Kubernetesコントロールプレーンをブートストラップします。これらのコンポーネントをコントローラーマシンにインストールします： Kubernetes API Server、Scheduler、Controller Manager

## 前提条件

Kubernetes のバイナリと systemd のユニットファイルを `server` インスタンスにコピー：

```bash
scp \
  downloads/kube-apiserver \
  downloads/kube-controller-manager \
  downloads/kube-scheduler \
  downloads/kubectl \
  units/kube-apiserver.service \
  units/kube-controller-manager.service \
  units/kube-scheduler.service \
  configs/kube-scheduler.yaml \
  configs/kube-apiserver-to-kubelet.yaml \
  root@server:~/
```

本実習のコマンドは、コントローラインスタンス `server` で実行する必要があります。`ssh` コマンドを使用して、コントローラインスタンスにログイン：

```bash
ssh root@server
```

## Kubernetesコントロールプレーンの準備

Kubernetes設定ディレクトリを作成：

```bash
mkdir -p /etc/kubernetes/config
```

### Kubernetesコントローラバイナリをインストール

Kubernetesのバイナリをインストール：

```bash
{
  chmod +x kube-apiserver \
    kube-controller-manager \
    kube-scheduler kubectl
    
  mv kube-apiserver \
    kube-controller-manager \
    kube-scheduler kubectl \
    /usr/local/bin/
}
```

### Kubernetes APIサーバーの設定

```bash
{
  mkdir -p /var/lib/kubernetes/

  mv ca.crt ca.key \
    kube-api-server.key kube-api-server.crt \
    service-accounts.key service-accounts.crt \
    encryption-config.yaml \
    /var/lib/kubernetes/
}
```

`kube-apiserver.service`のsystemdユニットファイルを作成：

```bash
mv kube-apiserver.service \
  /etc/systemd/system/kube-apiserver.service
```

### Kubernetes Controller Managerの設定

`kube-controller-manager` kubeconfigを所定の位置に移動：

```bash
mv kube-controller-manager.kubeconfig /var/lib/kubernetes/
```

`kube-controller-manager.service`のsystemdユニットファイルを作成：

```bash
mv kube-controller-manager.service /etc/systemd/system/
```

### Kubernetesスケジューラーの設定

`kube-scheduler`のkubeconfigを所定の位置に移動：

```bash
mv kube-scheduler.kubeconfig /var/lib/kubernetes/
```

`kube-scheduler.yaml`設定ファイルを作成：

```bash
mv kube-scheduler.yaml /etc/kubernetes/config/
```

`kube-scheduler.service`のsystemdユニットファイルを作成：

```bash
mv kube-scheduler.service /etc/systemd/system/
```

### コントローラーサービスの開始

```bash
{
  systemctl daemon-reload
  
  systemctl enable kube-apiserver \
    kube-controller-manager kube-scheduler
    
  systemctl start kube-apiserver \
    kube-controller-manager kube-scheduler
}
```

> Kubernetes API Serverが完全に初期化されるまで、最大10秒待ちます。


### 確認

```bash
kubectl cluster-info \
  --kubeconfig admin.kubeconfig
```

```text
Kubernetes control plane is running at https://127.0.0.1:6443
```

## Kubelet認証のためのRBAC

このセクションでは、Kubernetes API Serverが各ワーカーノードのKubelet APIにアクセスできるようにRBAC権限を設定します。Kubelet APIへのアクセスは、メトリクスやログの取得、Podでのコマンド実行に必要です。

> このチュートリアルでは、Kubelet の `--authorization-mode` フラグを `Webhook` に設定します。Webhook モードでは、[SubjectAccessReview](https://kubernetes.io/docs/admin/authorization/#checking-api-access) API を使用して認可を決定します。

このセクションのコマンドはクラスタ全体に影響し、コントローラノードで実行する必要があります。

```bash
ssh root@server
```

Kubelet APIにアクセスし、Podの管理に関連するほとんどの一般的なタスクを実行する権限を持つ`system:kube-apiserver-to-kubelet` [ClusterRole](https://kubernetes.io/docs/admin/authorization/rbac/#role-and-clusterrole)を作成：

```bash
kubectl apply -f kube-apiserver-to-kubelet.yaml \
  --kubeconfig admin.kubeconfig
```

### 確認

この時点でKubernetesのコントロールプレーンは稼働している。`jumpbox`マシンから以下のコマンドを実行して、動作していることを確認できます：

Kubernetesのバージョン情報をHTTPリクエスト：

```bash
curl -k --cacert ca.crt https://server.kubernetes.local:6443/version
```

```text
{
  "major": "1",
  "minor": "28",
  "gitVersion": "v1.28.3",
  "gitCommit": "a8a1abc25cad87333840cd7d54be2efaf31a3177",
  "gitTreeState": "clean",
  "buildDate": "2023-10-18T11:33:18Z",
  "goVersion": "go1.20.10",
  "compiler": "gc",
  "platform": "linux/arm64"
}
```

Next: [Kubernetesワーカーノードのブートストラップ](09-bootstrapping-kubernetes-workers.md)

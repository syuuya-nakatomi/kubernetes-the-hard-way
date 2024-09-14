# Kubernetesワーカーノードのブートストラップ

本実習では、2つのKubernetesワーカーノードをブートストラップします。これらのコンポーネントをインストールします： [runc](https://github.com/opencontainers/runc)、[container networking plugins](https://github.com/containernetworking/cni)、[containerd](https://github.com/containerd/containerd)、[kubelet](https://kubernetes.io/docs/admin/kubelet)、[kube-proxy](https://kubernetes.io/docs/concepts/cluster-administration/proxies)。

## 前提条件

Kubernetesバイナリとsystemdユニットファイルを各ワーカーインスタンスにコピー：

```bash
for host in node-0 node-1; do
  SUBNET=$(grep $host machines.txt | cut -d " " -f 4)
  sed "s|SUBNET|$SUBNET|g" \
    configs/10-bridge.conf > 10-bridge.conf 
    
  sed "s|SUBNET|$SUBNET|g" \
    configs/kubelet-config.yaml > kubelet-config.yaml
    
  scp 10-bridge.conf kubelet-config.yaml \
  root@$host:~/
done
```

```bash
for host in node-0 node-1; do
  scp \
    downloads/runc.arm64 \
    downloads/crictl-v1.28.0-linux-arm.tar.gz \
    downloads/cni-plugins-linux-arm64-v1.3.0.tgz \
    downloads/containerd-1.7.8-linux-arm64.tar.gz \
    downloads/kubectl \
    downloads/kubelet \
    downloads/kube-proxy \
    configs/99-loopback.conf \
    configs/containerd-config.toml \
    configs/kube-proxy-config.yaml \
    units/containerd.service \
    units/kubelet.service \
    units/kube-proxy.service \
    root@$host:~/
done
```

### 補足：

---

上記のコマンドですが、元の方では

```bash
for host in node-0 node-1; do
  scp \
    downloads/runc.arm64 \
    downloads/crictl-v1.28.0-linux-arm.tar.gz \
    downloads/cni-plugins-linux-arm64-v1.3.0.tgz \
    downloads/containerd-1.7.8-linux-arm64.tar.gz \
    downloads/kubectl \
    downloads/kubelet \
    downloads/kube-proxy \
    configs/99-loopback.conf \
    configs/containerd-config.toml \
    configs/kubelet-config.yaml \ <<<なぜか2回送っているところ
    configs/kube-proxy-config.yaml \
    units/containerd.service \
    units/kubelet.service \
    units/kube-proxy.service \
    root@$host:~/
done
```

となっており、前項での`SUBNET`を書き込んだファイルを送っているのに対し、もう一度素のファイルを送っているために、ファイルが初期化してしまう問題が起きるため、このページでは事前に行を消去させていただきました。

こちらについてはIssueを建てさせて頂いたため、どこかで対応されるかもしれません。

[kubelet-config.yaml is initialized and kubelet cannot be started #808](https://github.com/kelseyhightower/kubernetes-the-hard-way/issues/808)

---

本実習の以下のコマンドは、それぞれのワーカーインスタンスで実行する必要があります: `node-0`, `node-1`. ssh` コマンドを使用してワーカーインスタンスにログイン：

```bash
ssh root@node-0
```

## Kubernetesワーカーノードの準備

OSの依存関係をインストール：

```bash
{
  apt-get update
  apt-get -y install socat conntrack ipset
}
```

> socatバイナリは`kubectl port-forward`コマンドのサポートを有効にします。

### スワップを無効にする

デフォルトでは、[swap](https://help.ubuntu.com/community/SwapFaq)が有効になっていると、kubeletは起動に失敗します。Kubernetesが適切なリソース割り当てとサービス品質を提供できるように、スワップを無効にすることが[推奨](https://github.com/kubernetes/kubernetes/issues/7294)されています。

スワップが有効になっているか確認：

```bash
swapon --show
```

出力が空の場合、スワップは有効になっていません。スワップが有効になっている場合は、以下のコマンドを実行してスワップを直ちに無効にしてください：

```bash
swapoff -a
```

> 再起動後もスワップがオフのままであることを確認するには、Linuxディストロのドキュメントを参照してください。

インストール用ディレクトリを作成：

```bash
mkdir -p \
  /etc/cni/net.d \
  /opt/cni/bin \
  /var/lib/kubelet \
  /var/lib/kube-proxy \
  /var/lib/kubernetes \
  /var/run/kubernetes
```

ワーカーバイナリをインストール：

```bash
{
  mkdir -p containerd
  tar -xvf crictl-v1.28.0-linux-arm.tar.gz
  tar -xvf containerd-1.7.8-linux-arm64.tar.gz -C containerd
  tar -xvf cni-plugins-linux-arm64-v1.3.0.tgz -C /opt/cni/bin/
  mv runc.arm64 runc
  chmod +x crictl kubectl kube-proxy kubelet runc 
  mv crictl kubectl kube-proxy kubelet runc /usr/local/bin/
  mv containerd/bin/* /bin/
}
```

### CNIネットワーキングの設定

`bridge` ネットワーク設定ファイルを作成：

```bash
mv 10-bridge.conf 99-loopback.conf /etc/cni/net.d/
```
### 補足：

---

原文でもこの文周辺に作成とあるが、実行しているコマンドはファイルの移動です。

---

### containerdの設定

`containerd`設定ファイルをインストール：

```bash
{
  mkdir -p /etc/containerd/
  mv containerd-config.toml /etc/containerd/config.toml
  mv containerd.service /etc/systemd/system/
}
```

### Kubeletの設定

`kubelet-config.yaml`設定ファイルを作成：

```bash
{
  mv kubelet-config.yaml /var/lib/kubelet/
  mv kubelet.service /etc/systemd/system/
}
```

### Kubernetesプロキシの設定

```bash
{
  mv kube-proxy-config.yaml /var/lib/kube-proxy/
  mv kube-proxy.service /etc/systemd/system/
}
```

### ワーカー・サービスの開始

```bash
{
  systemctl daemon-reload
  systemctl enable containerd kubelet kube-proxy
  systemctl start containerd kubelet kube-proxy
}
```

## 確認

このチュートリアルで作成したコンピュートインスタンスには、このセクションを完了する権限がありません。`jumpbox`マシンから以下のコマンドを実行してください。

登録されているKubernetesノードをリストアップ：

```bash
ssh root@server \
  "kubectl get nodes \
  --kubeconfig admin.kubeconfig"
```

```
NAME     STATUS   ROLES    AGE    VERSION
node-0   Ready    <none>   1m     v1.28.3
node-1   Ready    <none>   10s    v1.28.3
```

Next: [Configuring kubectl for Remote Access](10-configuring-kubectl.md)

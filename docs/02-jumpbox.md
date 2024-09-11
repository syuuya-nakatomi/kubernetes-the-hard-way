# Jumpboxのセットアップ

この実習では4台のマシンのうち1台を `jumpbox` としてセットアップします。このマシンはこのチュートリアルのコマンドを実行するのに使われます。一貫性を確保するために専用のマシンを使用していますが、これらのコマンドはmacOSやLinuxを実行している個人のワークステーションを含む、ほぼすべてのマシンから実行することもできます。

`jumpbox`は、Kubernetesクラスタを一からセットアップする際にホームベースとして使用する管理マシンだと考えてください。始める前にやっておかなければならないことの1つは、いくつかのコマンドラインユーティリティをインストールし、Kubernetes The Hard Wayのgitリポジトリをクローンすることです。

`jumpbox`へログイン:

```bash
ssh root@jumpbox
```

すべてのコマンドは `root` ユーザーとして実行します。これは利便性のためであり、すべてのセットアップに必要なコマンドの数を減らすためです。

### コマンドラインユーティリティのインストール

これで `root` ユーザーとして `jumpbox` マシンにログインしたので、チュートリアルを通して様々なタスクを実行するために使用するコマンドラインユーティリティをインストールします。

```bash
apt-get -y install wget curl vim openssl git
```

### GitHubリポジトリを同期する

このチュートリアルには、Kubernetesクラスタを一から構築するための設定ファイルとテンプレートが含まれています。 `git` コマンドを使って Kubernetes The Hard Way の git リポジトリをクローンします：
```bash
git clone --depth 1 \
  https://github.com/kelseyhightower/kubernetes-the-hard-way.git
```

`kubernetes-the-hard-way`ディレクトリに移動：

```bash
cd kubernetes-the-hard-way
```

チュートリアルの残りの部分はこのディレクトリで行います。もし道に迷ったら、`pwd`コマンドを実行して、`jumpbox`上でコマンドを実行するときには正しいディレクトリにいることを確認してください：

```bash
pwd
```

```text
/root/kubernetes-the-hard-way
```

### バイナリのダウンロード

このセクションでは、様々なKubernetesコンポーネントのバイナリをダウンロードします。バイナリは `jumpbox` 上の `downloads` ディレクトリに保存され、Kubernetes クラスタ内の各マシンにバイナリを複数回ダウンロードする必要がなくなるため、このチュートリアルを完了するのに必要なインターネット帯域幅を削減できます。

`kubernetes-the-hard-way` ディレクトリから `mkdir` コマンドを使用して `downloads` ディレクトリを作成する：

```bash
mkdir downloads
```

ダウンロードされるバイナリは`downloads.txt`ファイルにリストアップされ、`cat`コマンドを使って確認することができます：

```bash
cat downloads.txt
```

`wget`コマンドを使用して、`downloads.txt`ファイルに記載されているバイナリをダウンロード：

```bash
wget -q --show-progress \
  --https-only \
  --timestamping \
  -P downloads \
  -i downloads.txt
```

インターネットの接続速度によっては、584メガバイトのバイナリをダウンロードするのに時間がかかる場合があります。ダウンロードが完了したら、`ls`コマンドを使用してバイナリを一覧表示できます：

```bash
ls -loh downloads
```

```text
total 584M
-rw-r--r-- 1 root  41M May  9 13:35 cni-plugins-linux-arm64-v1.3.0.tgz
-rw-r--r-- 1 root  34M Oct 26 15:21 containerd-1.7.8-linux-arm64.tar.gz
-rw-r--r-- 1 root  22M Aug 14 00:19 crictl-v1.28.0-linux-arm.tar.gz
-rw-r--r-- 1 root  15M Jul 11 02:30 etcd-v3.4.27-linux-arm64.tar.gz
-rw-r--r-- 1 root 111M Oct 18 07:34 kube-apiserver
-rw-r--r-- 1 root 107M Oct 18 07:34 kube-controller-manager
-rw-r--r-- 1 root  51M Oct 18 07:34 kube-proxy
-rw-r--r-- 1 root  52M Oct 18 07:34 kube-scheduler
-rw-r--r-- 1 root  46M Oct 18 07:34 kubectl
-rw-r--r-- 1 root 101M Oct 18 07:34 kubelet
-rw-r--r-- 1 root 9.6M Aug 10 18:57 runc.arm64
```

### kubectlのインストール

このセクションでは、公式のKubernetesクライアントコマンドラインツールである `kubectl` を `jumpbox` マシンにインストールします。kubectlは、このチュートリアルの後半でクラスタを準備した後に、Kubernetesコントロールとの対話に使用されます。

`chmod` コマンドを使って `kubectl` バイナリを実行可能にして、`/usr/local/bin/` ディレクトリにコピー：

```bash
{
  chmod +x downloads/kubectl
  cp downloads/kubectl /usr/local/bin/
}
```

この時点で `kubectl` がインストールされ、`kubectl` コマンドを実行して確認できます：

```bash
kubectl version --client
```

```text
Client Version: v1.28.3
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
```

これにより、`jumpbox`にはこのチュートリアルのラボを完了するのに必要なすべてのコマンドラインツールとユーティリティがセットアップされました。

Next: [Provisioning Compute Resources](03-compute-resources.md)

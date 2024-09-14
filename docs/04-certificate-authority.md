# CAの準備とTLS証明書の生成

本実習では、opensslを使用して[PKIインフラストラクチャ](https://en.wikipedia.org/wiki/Public_key_infrastructure)を準備し、認証局をブートストラップして、次のコンポーネントのTLS証明書を生成します: kube-apiserver、kube-controller-manager、kube-scheduler、kubelet、kube-proxy。

このセクションのコマンドは `jumpbox` から実行してください。

## 認証局

このセクションでは、他のKubernetesコンポーネント用に追加のTLS証明書を生成するために使用できる認証局をプロビジョニングします。`openssl`を使用して認証局をセットアップして証明書を生成するのは、特に初めて行う場合は時間がかかります。本実習を効率化するために、各Kubernetesコンポーネントの証明書を生成するために必要なすべての詳細を定義したopenssl設定ファイル`ca.conf`が含まれています。

`ca.conf`設定ファイルを確認：

```bash
cat ca.conf
```

このチュートリアルを完了するために、`ca.conf`ファイルのすべてを理解する必要はありませんが、`openssl`と証明書を高いレベルで管理するための設定を学ぶための出発点だと考えてください。

すべての認証局は秘密鍵とルート証明書から始まります。このセクションでは、自己署名認証局を作成します。このチュートリアルで必要なのはこれだけですが、これは実際の本番レベルの環境で行うものではないと考えるべきです。

CA設定ファイル、証明書、秘密鍵を生成：

```bash
{
  openssl genrsa -out ca.key 4096
  openssl req -x509 -new -sha512 -noenc \
    -key ca.key -days 3653 \
    -config ca.conf \
    -out ca.crt
}
```

結果:

```txt
ca.crt ca.key
```

## クライアント証明書とサーバ証明書の作成

このセクションでは、各Kubernetesコンポーネントのクライアント証明書とサーバー証明書、およびKubernetes `admin`ユーザーのクライアント証明書を生成します。

証明書と秘密鍵を生成：

```bash
certs=(
  "admin" "node-0" "node-1"
  "kube-proxy" "kube-scheduler"
  "kube-controller-manager"
  "kube-api-server"
  "service-accounts"
)
```

```bash
for i in ${certs[*]}; do
  openssl genrsa -out "${i}.key" 4096

  openssl req -new -key "${i}.key" -sha256 \
    -config "ca.conf" -section ${i} \
    -out "${i}.csr"
  
  openssl x509 -req -days 3653 -in "${i}.csr" \
    -copy_extensions copyall \
    -sha256 -CA "ca.crt" \
    -CAkey "ca.key" \
    -CAcreateserial \
    -out "${i}.crt"
done
```

上記のコマンドを実行した結果、Kubernetesの各コンポーネントの秘密鍵、証明書要求、署名付きSSL証明書が生成されます。生成されたファイルは以下のコマンドで一覧表示できます：

```bash
ls -1 *.crt *.key *.csr
```

## クライアント証明書とサーバ証明書の配布

このセクションでは、各Kubernetesコンポーネントが証明書ペアを検索するディレクトリの下に、様々な証明書を各マシンにコピーします。実際の環境では、これらの証明書はKubernetesコンポーネントが互いに認証するための認証情報として使用されることが多いため、機密性の高い秘密のセットのように扱う必要があります。

適切な証明書と秘密鍵を `node-0` と `node-1` のマシンにコピー：

```bash
for host in node-0 node-1; do
  ssh root@$host mkdir /var/lib/kubelet/
  
  scp ca.crt root@$host:/var/lib/kubelet/
    
  scp $host.crt \
    root@$host:/var/lib/kubelet/kubelet.crt
    
  scp $host.key \
    root@$host:/var/lib/kubelet/kubelet.key
done
```

適切な証明書と秘密鍵を `server` マシンにコピー：

```bash
scp \
  ca.key ca.crt \
  kube-api-server.key kube-api-server.crt \
  service-accounts.key service-accounts.crt \
  root@server:~/
```

> `kube-proxy`、`kube-controller-manager`、`kube-scheduler`、`kubelet` クライアント証明書は、次のラボでクライアント認証設定ファイルの生成に使用します。

Next: [認証用Kubernetes設定ファイルの生成](05-kubernetes-configuration-files.md)

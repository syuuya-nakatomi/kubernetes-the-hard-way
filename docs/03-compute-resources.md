# コンピューターリソースの準備

Kubernetesには、Kubernetesのコントロールプレーンと、コンテナが最終的に実行されるワーカーノードをホストするマシンのセットが必要です。このラボでは、Kubernetesクラスターのセットアップに必要なマシンを準備します。

## マシンデータベース

本チュートリアルでは、Kubernetesのコントロールプレーンとワーカーノードをセットアップする際に使用する様々なマシン属性を保存するために、マシンデータベースとなるテキストファイルを活用します。以下のスキーマは、マシンデータベースのエントリを表しており、1行に1エントリです：

```text
IPV4_ADDRESS FQDN HOSTNAME POD_SUBNET
```

各カラムは、マシンの IP アドレス `IPV4_ADDRESS`、完全修飾ドメイン名 `FQDN`、ホスト名 `HOSTNAME`、IP サブネット `POD_SUBNET` に対応している。Kubernetes は `pod` ごとに 1 つの IP アドレスを割り当て、`POD_SUBNET` はそのためにクラスタ内の各マシンに割り当てられた一意の IP アドレス範囲を表します。 

本チュートリアルの作成時に使用したものと同様のマシンデータベースの例を示します。IPアドレスがマスクされていることに注意してください。各マシンがお互いに、そして `jumpbox` から到達可能である限り、マシンにどのような IP アドレスを割り当てることもできます。

```bash
cat machines.txt
```

```text
XXX.XXX.XXX.XXX server.kubernetes.local server  
XXX.XXX.XXX.XXX node-0.kubernetes.local node-0 10.200.0.0/24
XXX.XXX.XXX.XXX node-1.kubernetes.local node-1 10.200.1.0/24
```

### 補足：

---

`IPV4_ADDRESS`の部分は各ホストの`Private IPv4 addresses`を書き込んでください。

FQDNやホスト名はここに書かれているもの以外のものを使うと途中で進行不能になります。

理由としてはpodとの通信にFQDNに制約があるためです。

---

次は、Kubernetesクラスタを作成するために使用する3台のマシンの詳細を記載した`machines.txt`ファイルを作成します。先ほどのマシンデータベースの例を参考に、マシンの詳細を追加してください。

## SSHアクセスの設定

クラスタ内のマシンの設定にはSSHを使用します。マシンデータベースにリストされている各マシンにroot SSHアクセスがあることを確認します。sshd_configファイルを更新してSSHサーバを再起動することで、各ノードでroot SSHアクセスを有効にする必要がある場合があります。

### root SSH アクセスの有効化

各マシンで root SSH アクセスが有効になっている場合は、このセクションは省略できます。

デフォルトでは、新しい debian インストールは root ユーザの SSH アクセスを無効にします。これはセキュリティ上の理由からで、root ユーザーは Linux システムでよく知られたユーザーだからです。インターネットに接続されたマシンで弱いパスワードが使われたら、マシンが他人のものになるのは時間の問題です。先に述べたように、このチュートリアルのステップを効率化するために、SSHでrootアクセスを有効にします。セキュリティはトレードオフであり、この場合は利便性を最適化する。各マシンでユーザーアカウントを使ってSSHでログインし、suコマンドを使ってrootユーザーに切り替えます：

```bash
su - root
```
### 補足：

---

`sudu su -`で問題ありません。

---

`etc/ssh/sshd_config` SSHデーモン設定ファイルを編集し、`PermitRootLogin`オプションを`yes`にしてください：

```bash
sed -i \
  's/^#PermitRootLogin.*/PermitRootLogin yes/' \
  /etc/ssh/sshd_config
```

### 補足：

---

この`sed`コマンドがうまく動作せず、ファイル内容が変更されないことがあるので、Vimを使って`etc/ssh/sshd_config`を直接書き換えてください。

ここで、変更する内容としては、上記の`sed`コマンドで行われている`PermitRootLogin`の項目を`PermitRootLogin yes`にするのと、`PasswordAuthentication yes`を追記してください。

そして、`etc/ssh/sshd_config`の変更を保存したら、次は`passwd`コマンドでrootアカウントのパスワードを付与します。

パスワードはこの後で入力を求められるため、覚えられるもので好きなものを設定してください。

---


`sshd`SSHサーバーを再起動して、更新されたコンフィギュレーションファイルを取り込む：

```bash
systemctl restart sshd
```


### SSH キーの生成と配布

このセクションでは、SSH キーペアを生成して `server`、`node-0`、`node-1` の各マシンに配布します。`jumpbox` マシンから以下のコマンドを実行してください。

新しい SSH 鍵を生成：

```bash
ssh-keygen
```

```text
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa
Your public key has been saved in /root/.ssh/id_rsa.pub
```

SSH 公開鍵を各マシンにコピー：

```bash
while read IP FQDN HOST SUBNET; do 
  ssh-copy-id root@${IP}
done < machines.txt
```

### 補足：

---

この周辺の操作で各ホストで設定したrootパスワードを求められるため、入力をお願いします。

---

それぞれの鍵が追加されたら、SSH公開鍵アクセスが機能していることを確認：

```bash
while read IP FQDN HOST SUBNET; do 
  ssh -n root@${IP} uname -o -m
done < machines.txt
```

```text
aarch64 GNU/Linux
aarch64 GNU/Linux
aarch64 GNU/Linux
```

## ホスト名

このセクションでは `server`、`node-0`、`node-1` の各マシンにホスト名を割り当てます。ホスト名は `jumpbox` から各マシンにコマンドを実行する際に使用されます。ホスト名はクラスタ内でも大きな役割を果たします。KubernetesクライアントがIPアドレスを使用してKubernetes APIサーバーにコマンドを発行する代わりに、これらのクライアントは `server` ホスト名を使用します。ホスト名は、Kubernetesクラスタに登録する際に各ワーカーマシン、`node-0`と`node-1`でも使用されます。

各マシンのホスト名を設定するには、`jumpbox` 上で以下のコマンドを実行してください。

`machines.txt` ファイルに記載されている各マシンのホスト名を設定：

```bash
while read IP FQDN HOST SUBNET; do 
    CMD="sed -i 's/^127.0.1.1.*/127.0.1.1\t${FQDN} ${HOST}/' /etc/hosts"
    ssh -n root@${IP} "$CMD"
    ssh -n root@${IP} hostnamectl hostname ${HOST}
done < machines.txt
```

各マシンにホスト名が設定されていることを確認：

```bash
while read IP FQDN HOST SUBNET; do
  ssh -n root@${IP} hostname --fqdn
done < machines.txt
```

```text
server.kubernetes.local
node-0.kubernetes.local
node-1.kubernetes.local
```

## DNS

このセクションでは、DNS の `hosts` ファイルを生成し、それを `jumpbox` のローカルの `/etc/hosts` ファイルと、このチュートリアルで使用する3台のマシンの `/etc/hosts` ファイルに追加します。これにより、各マシンは `server`、`node-0`、`node-1` のようなホスト名でアクセスできるようになります。

新しい `hosts` ファイルを作成し、追加するマシンを識別するためのヘッダーを追加：

```bash
echo "" > hosts
echo "# Kubernetes The Hard Way" >> hosts
```

各マシンの DNS エントリを `machines.txt` ファイルに作成し、それを `hosts` ファイルに追加：

```bash
while read IP FQDN HOST SUBNET; do 
    ENTRY="${IP} ${FQDN} ${HOST}"
    echo $ENTRY >> hosts
done < machines.txt
```

`hosts`ファイルのDNSエントリーを確認する：

```bash
cat hosts
```

```text

# Kubernetes The Hard Way
XXX.XXX.XXX.XXX server.kubernetes.local server
XXX.XXX.XXX.XXX node-0.kubernetes.local node-0
XXX.XXX.XXX.XXX node-1.kubernetes.local node-1
```

## ローカルマシンへのDNSエントリーの追加

このセクションでは `hosts` ファイルの DNS エントリを `jumpbox` マシンのローカルの `/etc/hosts` ファイルに追加します。

`hosts`のDNSエントリを `/etc/hosts` に追加：

```bash
cat hosts >> /etc/hosts
```

etc/hosts`ファイルが更新されていることを確認：

```bash
cat /etc/hosts
```

```text
127.0.0.1       localhost
127.0.1.1       jumpbox

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters



# Kubernetes The Hard Way
XXX.XXX.XXX.XXX server.kubernetes.local server
XXX.XXX.XXX.XXX node-0.kubernetes.local node-0
XXX.XXX.XXX.XXX node-1.kubernetes.local node-1
```

この時点で、`machines.txt`ファイルにリストされている各マシンに、ホスト名を使ってSSH接続できるはずです。

```bash
for host in server node-0 node-1
   do ssh root@${host} uname -o -m -n
done
```

```text
server aarch64 GNU/Linux
node-0 aarch64 GNU/Linux
node-1 aarch64 GNU/Linux
```

## リモートマシンへのDNSエントリーの追加

このセクションでは、`machines.txt` テキストファイルに記載されている各マシンの `hosts` から DNS エントリを `/etc/hosts` に追加します。

`hosts`ファイルを各マシンにコピーし、その内容を `/etc/hosts`に追加：

```bash
while read IP FQDN HOST SUBNET; do
  scp hosts root@${HOST}:~/
  ssh -n \
    root@${HOST} "cat hosts >> /etc/hosts"
done < machines.txt
```

この時点で、`jumpbox` マシンや Kubernetes クラスタ内の 3 台のマシンからマシンに接続する際にホスト名を使用できるようになりました。IP アドレスを使う代わりに、`server`、`node-0`、`node-1` などのホスト名を使用してマシンに接続できるようになりました。

Next: [Provisioning a CA and Generating TLS Certificates](04-certificate-authority.md)

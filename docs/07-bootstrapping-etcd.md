# etcdクラスタのブートストラップ

Kubernetesコンポーネントはステートレスで、クラスタの状態を[etcd](https://github.com/etcd-io/etcd)に保存します。本実習では、3ノードのetcdクラスターをブートストラップし、高可用性とセキュアなリモートアクセス用に設定します。

## 前提条件

`etcd` のバイナリと systemd のユニットファイルを `server` インスタンスにコピーする：

```bash
scp \
  downloads/etcd-v3.4.27-linux-arm64.tar.gz \
  units/etcd.service \
  root@server:~/
```

本実習のコマンドは `server` マシンで実行してください。`ssh` コマンドを使って `server` マシンにログイン：

```bash
ssh root@server
```

## etcdクラスタのブートストラップ

### etcdバイナリをインストールする

`etcd`サーバーと `etcdctl` コマンドラインユーティリティを解凍してインストールする：

```bash
{
  tar -xvf etcd-v3.4.27-linux-arm64.tar.gz
  mv etcd-v3.4.27-linux-arm64/etcd* /usr/local/bin/
}
```

### etcdサーバーの設定

```bash
{
  mkdir -p /etc/etcd /var/lib/etcd
  chmod 700 /var/lib/etcd
  cp ca.crt kube-api-server.key kube-api-server.crt \
    /etc/etcd/
}
```

etcdの各メンバーは、etcdクラスタ内で一意な名前を持つ必要がある。etcd名を現在のコンピュートインスタンスのホスト名と一致するように設定します：

`etcd.service`のsystemdユニットファイルを作成：

```bash
mv etcd.service /etc/systemd/system/
```

### etcdサーバーの起動

```bash
{
  systemctl daemon-reload
  systemctl enable etcd
  systemctl start etcd
}
```

## 確認

etcdクラスターメンバーをリストする：

```bash
etcdctl member list
```

```text
6702b0a34e2cfd39, started, controller, http://127.0.0.1:2380, http://127.0.0.1:2379, false
```

Next: [Kubernetesコントロールプレーンのブートストラップ](08-bootstrapping-kubernetes-controllers.md)

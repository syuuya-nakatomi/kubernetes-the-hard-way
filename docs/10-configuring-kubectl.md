# リモートアクセス用kubectlの設定

本実習では、`admin` ユーザー資格情報に基づいて `kubectl` コマンドラインユーティリティ用の kubeconfig ファイルを生成します。

> 本実習のコマンドは`jumpbox`マシンから実行してください。

## Admin Kubernetes設定ファイル

各kubeconfigは、接続するKubernetes API Serverを必要とします。

前項の`/etc/hosts`DNSエントリに基づいて`server.kubernetes.local`にpingできるはずです。

```bash
curl -k --cacert ca.crt \
  https://server.kubernetes.local:6443/version
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

`admin`ユーザーとして認証するのに適したkubeconfigファイルを生成：

```bash
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://server.kubernetes.local:6443

  kubectl config set-credentials admin \
    --client-certificate=admin.crt \
    --client-key=admin.key

  kubectl config set-context kubernetes-the-hard-way \
    --cluster=kubernetes-the-hard-way \
    --user=admin

  kubectl config use-context kubernetes-the-hard-way
}
```
上記のコマンドを実行した結果、`kubectl` コマンドラインツールが使用するデフォルトの場所 `~/.kube/config` に kubeconfig ファイルが作成されるはずです。これは、config を指定せずに `kubectl` コマンドを実行できることも意味します。

## 確認

リモートKubernetesクラスタのバージョンを確認：

```bash
kubectl version
```

```text
Client Version: v1.28.3
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
Server Version: v1.28.3
```

リモートKubernetesクラスタのノードをリストアップ：

```bash
kubectl get nodes
```

```
NAME     STATUS   ROLES    AGE   VERSION
node-0   Ready    <none>   30m   v1.28.3
node-1   Ready    <none>   35m   v1.28.3
```

Next: [Provisioning Pod Network Routes](11-pod-network-routes.md)

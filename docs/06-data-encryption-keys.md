# データ暗号化コンフィグとキーの生成

Kubernetesは、クラスタの状態、アプリケーションの設定、シークレットを含むさまざまなデータを保存します。Kubernetesは、クラスタデータを静止状態で[暗号化](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data)する機能をサポートしています。

本実習では、Kubernetesシークレットの暗号化に適した暗号化キーと[encryption config](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#understanding-the-encryption-at-rest-configuration)を生成します。

## 暗号化キー

暗号鍵を生成：

```bash
export ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)
```

## 暗号化設定ファイル

暗号化設定ファイル `encryption-config.yaml` を作成：

```bash
envsubst < configs/encryption-config.yaml \
  > encryption-config.yaml
```

### 補足

---

ここで`configs/encryption-config.yaml`が出てくるが、過去のコミットでこのファイルは削除されているため、Vimを使って次の内容のconfigs/encryption-config.yamlを作成します。

```
kind: EncryptionConfig
apiVersion: v1
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: ${ENCRYPTION_KEY}
      - identity: {}
```

その後に下記のコマンドを実行すると`encryption-config.yaml`の`secret`の部分に暗号鍵が内包される。

```bash
envsubst < configs/encryption-config.yaml \
  > encryption-config.yaml
```

なお、Issueも建てられています。

[configs/encryption-config.yaml is missing #768](https://github.com/kelseyhightower/kubernetes-the-hard-way/issues/768)

---

暗号化設定ファイル `encryption-config.yaml` を各コントローラインスタンスにコピー：

```bash
scp encryption-config.yaml root@server:~/
```

Next: [etcdクラスタのブートストラップ](07-bootstrapping-etcd.md)

# ポッドネットワークルートの準備

ノードにスケジュールされたポッドは、ノードのポッドCIDR範囲からIPアドレスを受け取ります。この時点では、ネットワーク[ルート](https://cloud.google.com/compute/docs/vpc/routes)が見つからないため、ポッドは異なるノードで動作している他のポッドと通信できません。

本実習では、ノードのPod CIDR範囲をノードの内部IPアドレスにマップするルートを各ワーカーノードに作成します。

> Kubernetesのネットワーキング・モデルを実装する[他の方法](https://kubernetes.io/docs/concepts/cluster-administration/networking/#how-to-achieve-this)もあります。

## ルーティングテーブル

このセクションでは、`kubernetes-the-hard-way` VPCネットワークでルートを作成するために必要な情報を収集します。

各ワーカーインスタンスの内部IPアドレスとPod CIDR範囲を表示：

```bash
{
  SERVER_IP=$(grep server machines.txt | cut -d " " -f 1)
  NODE_0_IP=$(grep node-0 machines.txt | cut -d " " -f 1)
  NODE_0_SUBNET=$(grep node-0 machines.txt | cut -d " " -f 4)
  NODE_1_IP=$(grep node-1 machines.txt | cut -d " " -f 1)
  NODE_1_SUBNET=$(grep node-1 machines.txt | cut -d " " -f 4)
}
```

```bash
ssh root@server <<EOF
  ip route add ${NODE_0_SUBNET} via ${NODE_0_IP}
  ip route add ${NODE_1_SUBNET} via ${NODE_1_IP}
EOF
```

```bash
ssh root@node-0 <<EOF
  ip route add ${NODE_1_SUBNET} via ${NODE_1_IP}
EOF
```

```bash
ssh root@node-1 <<EOF
  ip route add ${NODE_0_SUBNET} via ${NODE_0_IP}
EOF
```

## 確認

```bash
ssh root@server ip route
```

```text
default via XXX.XXX.XXX.XXX dev ens160 
10.200.0.0/24 via XXX.XXX.XXX.XXX dev ens160 
10.200.1.0/24 via XXX.XXX.XXX.XXX dev ens160 
XXX.XXX.XXX.0/24 dev ens160 proto kernel scope link src XXX.XXX.XXX.XXX 
```

```bash
ssh root@node-0 ip route
```

```text
default via XXX.XXX.XXX.XXX dev ens160 
10.200.1.0/24 via XXX.XXX.XXX.XXX dev ens160 
XXX.XXX.XXX.0/24 dev ens160 proto kernel scope link src XXX.XXX.XXX.XXX 
```

```bash
ssh root@node-1 ip route
```

```text
default via XXX.XXX.XXX.XXX dev ens160 
10.200.0.0/24 via XXX.XXX.XXX.XXX dev ens160 
XXX.XXX.XXX.0/24 dev ens160 proto kernel scope link src XXX.XXX.XXX.XXX 
```


Next: [Smoke Test](12-smoke-test.md)

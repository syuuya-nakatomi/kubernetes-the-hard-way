# Kubernetes The Hard Way(ARM64移行版 日本語訳)

### 【注意】この翻訳はKubernetes The Hard WayがARM64での動作環境版へ移行したことにより、仕様が大きく変更されたため、日本語訳と実行する際の解説、補足を追加したものです。以前のIntel版での動作についての日本語訳は[Inducter](https://github.com/inductor)さんの[Kubernetes The Hard Way(日本語版)](https://github.com/inductor/kubernetes-the-hard-way?tab=readme-ov-file)をご参照ください。Typoや誤訳などを見つけた場合はPR、Issue、または[X(Twitter)](https://x.com/eatshuumai)にてお知らせください。

このチュートリアルでは、Kubernetesのセットアップを地道な方法で行なっていきます。このガイドは、Kubernetesクラスタを立ち上げるための完全自動化ツールを探している人向けではありません。Kubernetes The Hard Wayは学習用に最適化されており、Kubernetesクラスタのブートストラップに必要な各タスクを確実に理解するために長い道のりを選ぶことを意味します。

> 本チュートリアルの結果は、本番環境での利用を想定したものではなく、コミュニティからのサポートも限定的なものになるかもしれませんが、それでも学ぶことを止めないでください！

## Copyright

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License</a>.


## 対象者

本チュートリアルの対象者は、Kubernetesの基礎とコアコンポーネントがどのように組み合わされているかを理解したい人です。

## クラスターの詳細

Kubernetes The Hard Wayでは、基本的なKubernetesクラスタをブートストラップする上で、すべてのコントロールプレーンコンポーネント動作させるノードを1つと、2つのワーカーノードを作成します。

コンポーネントのバージョン:

* [kubernetes](https://github.com/kubernetes/kubernetes) v1.28.x
* [containerd](https://github.com/containerd/containerd) v1.7.x
* [cni](https://github.com/containernetworking/cni) v1.3.x
* [etcd](https://github.com/etcd-io/etcd) v3.4.x

## 実習内容

本チュートリアルでは、同じネットワークに接続された4台のARM64ベースの仮想マシンまたは物理マシンが必要です。本チュートリアルではARM64ベースのマシンを使用しますが、学んだことは他のプラットフォームにも適用できます。

* [前提条件](docs/01-prerequisites.md)
* [Jumpboxのセットアップ](docs/02-jumpbox.md)
* [コンピューターリソースの準備](docs/03-compute-resources.md)
* [CAの準備とTLS証明書の生成](docs/04-certificate-authority.md)
* [認証用Kubernetes設定ファイルの生成](docs/05-kubernetes-configuration-files.md)
* [データ暗号化コンフィグとキーの生成](docs/06-data-encryption-keys.md)
* [etcdクラスタのブートストラップ](docs/07-bootstrapping-etcd.md)
* [Kubernetesコントロールプレーンのブートストラップ](docs/08-bootstrapping-kubernetes-controllers.md)
* [Kubernetesワーカーノードのブートストラップ](docs/09-bootstrapping-kubernetes-workers.md)
* [リモートアクセス用kubectlの設定](docs/10-configuring-kubectl.md)
* [ポッドネットワークルートの準備](docs/11-pod-network-routes.md)
* [スモークテスト](docs/12-smoke-test.md)
* [お掃除](docs/13-cleanup.md)

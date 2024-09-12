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

次は、Kubernetesクラスタを作成するために使用する3台のマシンの詳細を記載した`machines.txt`ファイルを作成します。先ほどのマシンデータベースの例を参考に、マシンの詳細を追加してください。

## Configuring SSH Access

SSH will be used to configure the machines in the cluster. Verify that you have `root` SSH access to each machine listed in your machine database. You may need to enable root SSH access on each node by updating the sshd_config file and restarting the SSH server.

### Enable root SSH Access

If `root` SSH access is enabled for each of your machines you can skip this section.

By default, a new `debian` install disables SSH access for the `root` user. This is done for security reasons as the `root` user is a well known user on Linux systems, and if a weak password is used on a machine connected to the internet, well, let's just say it's only a matter of time before your machine belongs to someone else. As mention earlier, we are going to enable `root` access over SSH in order to streamline the steps in this tutorial. Security is a tradeoff, and in this case, we are optimizing for convenience. On each machine login via SSH using your user account, then switch to the `root` user using the `su` command:

```bash
su - root
```

Edit the `/etc/ssh/sshd_config` SSH daemon configuration file and the `PermitRootLogin` option to `yes`:

```bash
sed -i \
  's/^#PermitRootLogin.*/PermitRootLogin yes/' \
  /etc/ssh/sshd_config
```

Restart the `sshd` SSH server to pick up the updated configuration file:

```bash
systemctl restart sshd
```

### Generate and Distribute SSH Keys

In this section you will generate and distribute an SSH keypair to the `server`, `node-0`, and `node-1`, machines, which will be used to run commands on those machines throughout this tutorial. Run the following commands from the `jumpbox` machine.

Generate a new SSH key:

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

Copy the SSH public key to each machine:

```bash
while read IP FQDN HOST SUBNET; do 
  ssh-copy-id root@${IP}
done < machines.txt
```

Once each key is added, verify SSH public key access is working:

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

## Hostnames

In this section you will assign hostnames to the `server`, `node-0`, and `node-1` machines. The hostname will be used when executing commands from the `jumpbox` to each machine. The hostname also play a major role within the cluster. Instead of Kubernetes clients using an IP address to issue commands to the Kubernetes API server, those client will use the `server` hostname instead. Hostnames are also used by each worker machine, `node-0` and `node-1` when registering with a given Kubernetes cluster.

To configure the hostname for each machine, run the following commands on the `jumpbox`.

Set the hostname on each machine listed in the `machines.txt` file:

```bash
while read IP FQDN HOST SUBNET; do 
    CMD="sed -i 's/^127.0.1.1.*/127.0.1.1\t${FQDN} ${HOST}/' /etc/hosts"
    ssh -n root@${IP} "$CMD"
    ssh -n root@${IP} hostnamectl hostname ${HOST}
done < machines.txt
```

Verify the hostname is set on each machine:

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

In this section you will generate a DNS `hosts` file which will be appended to `jumpbox` local `/etc/hosts` file and to the `/etc/hosts` file of all three machines used for this tutorial. This will allow each machine to be reachable using a hostname such as `server`, `node-0`, or `node-1`.

Create a new `hosts` file and add a header to identify the machines being added:

```bash
echo "" > hosts
echo "# Kubernetes The Hard Way" >> hosts
```

Generate a DNS entry for each machine in the `machines.txt` file and append it to the `hosts` file:

```bash
while read IP FQDN HOST SUBNET; do 
    ENTRY="${IP} ${FQDN} ${HOST}"
    echo $ENTRY >> hosts
done < machines.txt
```

Review the DNS entries in the `hosts` file:

```bash
cat hosts
```

```text

# Kubernetes The Hard Way
XXX.XXX.XXX.XXX server.kubernetes.local server
XXX.XXX.XXX.XXX node-0.kubernetes.local node-0
XXX.XXX.XXX.XXX node-1.kubernetes.local node-1
```

## Adding DNS Entries To A Local Machine

In this section you will append the DNS entries from the `hosts` file to the local `/etc/hosts` file on your `jumpbox` machine.

Append the DNS entries from `hosts` to `/etc/hosts`:

```bash
cat hosts >> /etc/hosts
```

Verify that the `/etc/hosts` file has been updated:

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

At this point you should be able to SSH to each machine listed in the `machines.txt` file using a hostname.

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

## Adding DNS Entries To The Remote Machines

In this section you will append the DNS entries from `hosts` to `/etc/hosts` on each machine listed in the `machines.txt` text file.

Copy the `hosts` file to each machine and append the contents to `/etc/hosts`:

```bash
while read IP FQDN HOST SUBNET; do
  scp hosts root@${HOST}:~/
  ssh -n \
    root@${HOST} "cat hosts >> /etc/hosts"
done < machines.txt
```

At this point hostnames can be used when connecting to machines from your `jumpbox` machine, or any of the three machines in the Kubernetes cluster. Instead of using IP addresess you can now connect to machines using a hostname such as `server`, `node-0`, or `node-1`.

Next: [Provisioning a CA and Generating TLS Certificates](04-certificate-authority.md)

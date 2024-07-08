---
title: TorqueをCentOS 6にインストールするメモ
tags:
  - CentOS
  - openmpi
  - torque
private: false
updated_at: '2020-08-10T15:48:16+09:00'
id: c3b78f7fabe1a1253466
organization_url_name: null
slide: false
ignorePublish: false
---
# 検証環境 & 想定環境
Host Computer
* OS: Win 10
* CPU: i5 4550
* VirtualHost: Oracle VM VirtualBox
Remote
* OS: CentOS 6.10 (Final)

想定環境（検証環境はあくまで練習）
最終的にはXeonを使ったシングルノードのマルチプロセス環境の構築を想定しています。
Torqueを使う以上、マルチクラスタをしたいものですが、LDAPを保守できる環境にないので、必然的にノードは一つが前提になります。
XeonなのでIntel Studioを使った環境も想定されますが、これはOpenMPIのコンパイル時にoptionをしっかり指定すればいいだけなので、詳細は省きます。

# Torque について
Torqueはjob managerと呼ばれるソフトウェアの一つで、マルチクラスタ計算を行うときにハードウェアリソースを管理し適切にプロセスを割り当てる管理ソフトです。
ほかにもいくつか種類がありますが、OSSで簡単に導入できるので人気なjob managerです。これ以上の詳細は省きます。

# 依存ライブラリのインストール
大学内などで（筆者環境）proxy下にある環境の場合、proxyの設定をyum,wget,curlなどに忘れず行ってください。また、CentOS6.4などの古いOSでは、そもそもyumのリポジトリが切れている可能性があるのでyumのリポジトリの再設定などをしてから行ってください。

```
＃yum install epel-release
# yum update
# yum upgrade
# yum install openssl-devel libtool libxml2-devel boost-devel gcc gcc-c++
# yum install git
```

ここで、参考サイト[1]（公式docs）[2]では、libtool-develと書かれていますが、centos6.10のyumリポジトリにはないです。libtoolでいいみたいなので変更。

## NUMActlとhwlocのインストール
Hardware Locality(hwloc)は複雑怪奇なCPUの環境（とくにXeonのような環境）をMPIなどのマルチプロセスアプリケーションに統一された表現で情報を与えるためのツール[3]。Torqueもhwlocからハードウェアリソースの情報を得るので先にインストールする必要があります。
hwlocのバージョンはこの後インストールするTorque5系に合わせてhwloc1系をインストールします。
筆者環境は、マルチクラスタを想定していないけれども、単体のマシンはつよつよなので、NUMAを使用してもう少し効率向上を図ります[4]参照。

```
# yum install numactl-devel
# wget https://download.open-mpi.org/release/hwloc/v1.11/hwloc-1.11.13.tar.gz
# tar xzvf hwloc-1.11.13.tar.gz
# cd hwloc-1.11.13
# mkdir build
# cd build
# ../configure --prefix=/usr/local
# make
# make install
```

# Torqueのインストール（サーバのみ）
というわけで、本題のTorqueのインストール。tar.gzファイルが見つからなかったため、gitよりdownload。

```
# git clone https://github.com/adaptivecomputing/torque.git -b 5.1.3 torque-5.1.3
# cd torque-5.1.3
# ./autogen.sh
# ./configure
# mkdir ./build
# cd build
```

ここで、このあと./configureを実行するわけですが、一度```./configure --help```を実行してoptionを確認しましょう。環境によっておそらくoptionは大きく変わります。
例えば、マルチクラスタを行う場合では、マスターは```--disable-clients```を付けたり逆にクライアントは```--diable-server```を付ける必要があるでしょう（クライアント（＝ノード）は後述）。
いまの想定環境は大体defaultでいいですが、参考にする人は、注意してください。
また、出力を確認して指定したものが入っているかも確認しましょう。例えば下記の```./configure <optioin> > log```のように保存し、```log | grep NUMA```や```log | grep hwloc```のように確認できます。

```
# ../configure --enable-numa-support --with-hwloc-path=/usr/local/
# make
# make install
```
## trqauthdデーモンのセットアップ
ここまででインストールは終了しました。セットアップを行います。

まずは、システム(OS)に認識してもらいます。

```
# cp contrib/init.d/trqauthd /etc/init.d/
# chkconfig --add trqauthd
# echo /usr/local/lib > /etc/ld.so.conf.d/torque.conf
# ldconfig
# service trqauthd start
```

## 名前解決とポートの解放
ここからはTorqueの設定になります。
設定の方法はあまたあると思うので、一例と思って自分でしっかり調べてください。
まずは、各ノードにipアドレスではなく名前を振る（名前を解決する）必要があります。今回は、```/etc/hosts```ファイルを使います。が、localhostはすでにlocalhost.localdomainで名前解決しているのでスルーします。マルチクラスタでは、hostsはDNSといった手法で名前解決の手法を用意してください。

```
# cat /etc/hosts
127.0.0.1    localhost.localdomain localhost
::1          localhost6.localdomain localhost6
```

続いてポートも開けます。（多分共通）

```
# iptables-save > /tmp/iptables.mod
# vi /tmp/iptables.mod

[追記：変更] 
# "-A INPUT -j REJECT --reject-with icmp-host-prohibited"
 
-A INPUT -p tcp --dport 15001 -j ACCEPT
 
-A INPUT -p tcp --dport 15002 -j ACCEPT
-A INPUT -p tcp --dport 15003 -j ACCEPT
 
# iptables-restore < /tmp/iptables.mod
# service iptables save
```

# torqueのサーバの設定
torqueにserverを指定します。（サーバのみ）

```
# echo localhost.localdomain > /var/spool/torque/server_name
```

serverを立ち上げます。初回一回、サーバのみ。qtermしてからやり直すことができますが、やり直すとあとの設定ファイルが全部飛びます。

```
# ../torque.setup root
```

## nodesファイルの設定（サーバのみ）
計算に用いるノードの設定を行います。今回はnodeが一つなので非常に簡単です。詳しくは[5]を見ましょう。
ただし、前述の通りNUMAを使用してセットアップするので、[4,7]を参照してセットアップします。

```
# vi /var/spool/torque/server_priv/nodes
locahost.localdomain np=1 num_node_boards=1
```

## pbs_serverの設定
つぎに、pbs_serverというサーバデーモンを起動します（trqauthdとの違いがわからん）。

```
# cp contrib / init.d / pbs_server /etc/init.d 
# chkconfig --add pbs_server 
# service pbs_server restart
```

# Torqueのノードの設定
Torqueで管理・利用されるノードへのインストールは、ここまでbuildした環境と同一であれば個別buildや```./configure```を行う必要はなく、build内に、client用のインストローラーを生成できます[6]。

```
# make packages
```

今回の想定環境はシングルノードでserverとclientが同一なので、これ以上話しませんが、マルチクラスタの場合は、LDAPとか使ってuidを統一化したり、nfsを使って共通ディレクトリを提供したり、sshのキーを配ったりしたうえで、[6]を参考にnode用のソフトをインストールしてください。



## mom.layout
NUMAを使う場合、ノードのCPU構成をmom.layouに設定する必要があります。[7]を参考にしてください。

```
# vi /var/spool/torque/mom_prov/mom.layout
nodes=0
```

## configファイルの設定
ノード側の設定でserverを指定します

```
# vi /var/spool/torque/mom_priv/config
$pbsserver localhost.localdomain
$logevent 225 
```

## torque-momのセットアップ

```
# cp contrib/init.d/pbs_mom /etc/init.d
# chkconfig --add pbs_mom
# service pbs_mom start
```

# OpenMPIのインストール
Torqueを使う主な目的はマルチクラスタ計算であり、これを行うためにはMPIを使うことになります。OpenMPIではTorqueと連携することができます。OpenMPIはyumでインストールすることができますが、これではTorqueと連携できません（と思ってます）。なので、ソースコードを落としてコンパイルする必要があります[8]。
OpenMPIのソースコードをダウンロードします。バージョンはyumでインストールできる最新バージョンと同じOpenMPI-1.10を使います。

```
# cd
# wget https://download.open-mpi.org/release/open-mpi/v1.10/openmpi-1.10.7.tar.gz
# tar xzvf openmpi-1.10.7.tar.gz
# cd openmpi-1.10.7
# mkdir build
# cd build
# ../configure --with-hwloc=/usr/local --with-tm
# make 
# make install
```

インストールが終わったらパスを通します。この時、既存のopenmpiを使っている場合、/usr/localにインストール(default)すると競合して問題を起こす場合があります。この例では、そんなことは気にせずパスを通しますが、必要に応じてインストールディレクトリを変えて、Torqueを使うユーザだけ、.bashrcなどにPATHとLD_LIBRARY_PATHを設定するようにしましょう

```
# echo /usr/local/lib/openmpi > etc/ld.so.conf/openmpi.conf
# ldconfig
```

# 参考サイト
1. http://docs.adaptivecomputing.com/torque/5-0-3/Content/topics/hpcSuiteInstall/manual/1-installing/installingTorque.htm
2. https://qiita.com/Yasushi-Shinohara/items/bd1218c79724f042d878
3. https://www.open-mpi.org/projects/hwloc/doc/v2.0.2/
4. http://docs.adaptivecomputing.com/torque/4-1-4/Content/topics/1-installConfig/buildingWithNUMA.htm
5. http://docs.adaptivecomputing.com/8-1-3/basic/installGuide/Content/topics/torque/1-installConfig/specifyComputeNodes.htm
6. http://docs.adaptivecomputing.com/8-1-3/basic/installGuide/Content/topics/torque/1-installConfig/computeNodes.htm
7. https://qiita.com/Baruim/items/a3a839fd71545d4cd109
8. https://www.jstage.jst.go.jp/article/mssj/17/4/17_209/_pdf

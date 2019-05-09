### apach-MySQL-PHP7.2

VirtualBox に CentOS を入れて仮想環境を構築

### VirtualBox インストール

https://www.virtualbox.org/wiki/Downloads

- OS X hosts をクリック
- dmg ファイルをダウンロード(VirtualBox-6.0.6-130049-OSX.dmg)
- dmg ファイルを開いて、VirtualBox.pkg をダブルクリック
- インストール

参考 URL：https://pc-karuma.net/mac-virtualbox-install/

### ホストオンリーネットワークの作成

ホストマシンと仮想マシン、仮想マシンと仮想マシン同士の通信をするために必要な設定

- VirtualBox の「ファイル」メニューから「Host Network Manager」を選択
- 「作成」をクリックすると、ホストオンリーネットワーク「vboxnet0」が作成される。そのまま「閉じる」をクリック。

ホストオンリーネットワークでは「192.168.56.1」がホストマシンの macOS に割り当てられるので、仮想マシンには 192.168.56.2 から 192.168.56.254 までの IP アドレスを割り当てることができる。

参考 URL：https://blog.apar.jp/linux/9309/

### 仮想マシン作成

- 「新規」をクリック
- 「エキスパートモード」をクリック
- 仮想マシンの名前を入力し、今回は CentOS7.6 をインストールするため、タイプは「Linux」バージョンは「Red Hat（64-bit）」を選択。メモリーサイズは 1024MB 以上を指定。（CentOS7 は少なくとも 1024MB のメモリが必要）もろもろの入力/選択が終わったら「作成」をクリック。

### ホストオンリーアダプターの追加

作成した仮想マシンのネットワークアダプターは「NAT」（仮想マシンがインターネットと通信するためのアダプター）のみ有効になっている。このままでは、ホストマシンと仮想マシンの通信ができないため「ホストオンリーアダプター」を追加する。

- 作成した仮想マシンを選択して「設定」をクリック
- 「ネットワーク」→「アダプター２」を選択
- 「ネットワークアダプターを有効化」にチェックを入れ、割り当て「ホストオンリーアダプター」を選択すると、名前に先ほど作成したホストオンリーネットワーク「vboxnet0」が表示され流ので「OK」をクリック

### CentOS ダウンロード

http://isoredirect.centos.org/centos/7/isos/x86_64/CentOS-7-x86_64-Minimal-1810.iso
から適当なものをダウンロード

### VB に CentOS をインストール

- 仮想マシンを選択して「設定」をクリック
- 「ストレージ」を選択し、コントローラー IDE の CD/DVD デバイス追加ボタン「＋」をクリック
- 「ディスクを選択」をクリック
- ダウンロードした「CentOS-7-x86_64-Minimal-1810.iso」を選択し「開く」をクリック
- 確認ができたら「OK」をクリック
- 仮想マシンを起動

### 仮想マシン起動

- Install CentOS7 を選択
- 日本語を選択して続行
- 「インストール先」をクリックし、ローカルの標準デバイスを選択して「完了」
- 「インストールの開始」をクリック
- 「root パスワード」をクリックし、パスワードを設定したら「完了」
- 再起動

### ネットワーク設定

参考 URL：https://blog.apar.jp/linux/8257/

- `cd /etc/sysconfig/network-scripts
- `vi ifcfg-enp0s3`
  `ONBOOT=no` を `ONBOOT=yes` に変更
- `vi ifcfg-enp0s8` としてファイルを開き，以下の変更を加える
  - `BOOTPROTO=dhcp` を `BOOTPROTO=none` に変更
  - `IPV6INIT=yes` を `IPV6INIT=no` に変更
  - `ONBOOT=no` を `ONBOOT=yes` に変更
  - 末尾に以下を追加
    ```
    IPADDR=192.168.56.110
    NETMASK=255.255.255.0
    ```
- ターミナルから ssh 接続

```
$ ssh root@192.168.56.110
The authenticity of host '192.168.56.110 (192.168.56.110)' can't be established.
ECDSA key fingerprint is SHA256:pKRtcddOwSGhJ+zJXsKufILow2pQ//lcUy5P8ABlH3I.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.56.110' (ECDSA) to the list of known hosts.
root@192.168.56.110's password:
[root@localhost ~]#
```

### 共有フォルダ設定

- 共有フォルダ指定

  - 仮想マシンの設定から「共有フォルダ」
  - 右端の「追加」をクリック
  - フォルダーのパスに共有したいフォルダを設定
  - 自動マウント、永続化にチェック

- 前提パッケージがインストールされているか確認

```
rpm -qa | grep kernel-deaders
rpm -qa | grep kernel-devel
rpm -qa | grep make
rpm -qa | grep gcc
```

- インストール

```
yum install kernel-headers
yum install kernel-devel
yum install make
yum install gcc
```

- Linux に Guest Additions をインストールする

  - ゲストマシン上のメニュー「デバイス」→「Guest Additions の CD イメージを挿入」をクリック
  - 再起動

- マウントする

```
[root@localhost ~]# mkdir /mnt/cdrom
[root@localhost ~]# mount -r /dev/cdrom /mnt/cdrom
[root@localhost ~]# cd /mnt/cdrom
[root@localhost cdrom]# ls -l
合計 71751
-r--r--r--. 1 root root      763  1月 22 02:01 AUTORUN.INF
dr-xr-xr-x. 2 root root     1824  4月 16 19:03 NT3x
dr-xr-xr-x. 2 root root     2652  4月 16 19:03 OS2
-r--r--r--. 1 root root      547  4月 16 19:03 TRANS.TBL
-r--r--r--. 1 root root  3710817  4月 16 18:56 VBoxDarwinAdditions.pkg
-r--r--r--. 1 root root     3949  4月 16 18:56 VBoxDarwinAdditionsUninstall.tool
-r-xr-xr-x. 1 root root  9665969  4月 16 18:57 VBoxLinuxAdditions.run
-r--r--r--. 1 root root 20608000  4月 16 18:58 VBoxSolarisAdditions.pkg
-r-xr-xr-x. 1 root root 26216000  4月 16 19:02 VBoxWindowsAdditions-amd64.exe
-r-xr-xr-x. 1 root root 12977032  4月 16 19:00 VBoxWindowsAdditions-x86.exe
-r-xr-xr-x. 1 root root   270104  4月 16 18:57 VBoxWindowsAdditions.exe
-r-xr-xr-x. 1 root root     6384  4月 16 18:56 autorun.sh
dr-xr-xr-x. 2 root root      792  4月 16 19:03 cert
-r-xr-xr-x. 1 root root     4821  4月 16 18:56 runasroot.sh
```

```
[root@localhost cdrom]# cd /mnt/cdrom/
[root@localhost cdrom]# ./VBoxLinuxAdditions.run
Verifying archive integrity... All good.
Uncompressing VirtualBox 6.0.6 Guest Additions for Linux........
VirtualBox Guest Additions installer
Copying additional installer modules ...
Installing additional modules ...
VirtualBox Guest Additions: Starting.
VirtualBox Guest Additions: Building the VirtualBox Guest Additions kernel
modules.  This may take a while.
VirtualBox Guest Additions: To build modules for other installed kernels, run
VirtualBox Guest Additions:   /sbin/rcvboxadd quicksetup <version>
VirtualBox Guest Additions: or
VirtualBox Guest Additions:   /sbin/rcvboxadd quicksetup all
VirtualBox Guest Additions: Building the modules for kernel
3.10.0-957.12.1.el7.x86_64.
```

- 確認
  デフォルトでは/media 以下に sf\_[指定フォルダ名]が作成される．

```
[root@localhost media]# ls
sf_share
```

ローカルで txt ファイルを追加すると、ゲスト OS 側でも確認できた。

参考 URL：
https://vboxmania.net/guest-additions%E3%81%AE%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB/
https://mpweb.mobi/windows/guestadditions-centos.php
https://vboxmania.net/guest-additions%E3%81%AE%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB/

### apache

- インストール

```
$ yum install httpd
読み込んだプラグイン:fastestmirror
Loading mirror speeds from cached hostfile
 * base: ftp-srv2.kddilabs.jp
 * epel: ftp.iij.ad.jp
 * extras: ftp-srv2.kddilabs.jp
 * updates: ftp-srv2.kddilabs.jp
依存性の解決をしています
--> トランザクションの確認を実行しています。
---> パッケージ httpd.x86_64 0:2.4.6-89.el7.centos を インストール
--> 依存性の処理をしています: httpd-tools = 2.4.6-89.el7.centos のパッケージ: httpd-2.4.6-89.el7.centos.x86_64
--> 依存性の処理をしています: /etc/mime.types のパッケージ: httpd-2.4.6-89.el7.centos.x86_64
--> トランザクションの確認を実行しています。
---> パッケージ httpd-tools.x86_64 0:2.4.6-89.el7.centos を インストール
---> パッケージ mailcap.noarch 0:2.1.41-2.el7 を インストール
--> 依存性解決を終了しました。

依存性を解決しました

==========================================================================================
 Package              アーキテクチャー
                                      バージョン                   リポジトリー      容量
==========================================================================================
インストール中:
 httpd                x86_64          2.4.6-89.el7.centos          updates          2.7 M
依存性関連でのインストールをします:
 httpd-tools          x86_64          2.4.6-89.el7.centos          updates           90 k
 mailcap              noarch          2.1.41-2.el7                 base              31 k

トランザクションの要約
==========================================================================================
インストール  1 パッケージ (+2 個の依存関係のパッケージ)

総ダウンロード容量: 2.8 M
インストール容量: 9.6 M
Is this ok [y/d/N]: y
Downloading packages:
(1/3): httpd-tools-2.4.6-89.el7.centos.x86_64.rpm                  |  90 kB  00:00:00
(2/3): mailcap-2.1.41-2.el7.noarch.rpm                             |  31 kB  00:00:00
(3/3): httpd-2.4.6-89.el7.centos.x86_64.rpm                        | 2.7 MB  00:00:02
------------------------------------------------------------------------------------------
合計                                                      1.3 MB/s | 2.8 MB  00:00:02
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  インストール中          : mailcap-2.1.41-2.el7.noarch                               1/3
  インストール中          : httpd-tools-2.4.6-89.el7.centos.x86_64                    2/3
  インストール中          : httpd-2.4.6-89.el7.centos.x86_64                          3/3
  検証中                  : httpd-tools-2.4.6-89.el7.centos.x86_64                    1/3
  検証中                  : mailcap-2.1.41-2.el7.noarch                               2/3
  検証中                  : httpd-2.4.6-89.el7.centos.x86_64                          3/3

インストール:
  httpd.x86_64 0:2.4.6-89.el7.centos

依存性関連をインストールしました:
  httpd-tools.x86_64 0:2.4.6-89.el7.centos          mailcap.noarch 0:2.1.41-2.el7

完了しました!
```

- 起動（自動起動の設定）

```
[root@localhost sf_share]# systemctl start httpd.service
[root@localhost sf_share]# systemctl enable httpd.service
Created symlink from /etc/systemd/system/multi-user.target.wants/httpd.service to /usr/lib/systemd/system/httpd.service.
```

### MySQL(5.7)

CentOS7 では、mariaDB(MySQL 互換の DB)がデフォルトでインストールされている場合があるため、
MySQL と競合を起こさないように削除する。

- mariaDB、既存 MySQL 削除

```
$ yum remove mariadb-libs
読み込んだプラグイン:fastestmirror
依存性の解決をしています
--> トランザクションの確認を実行しています。
---> パッケージ mariadb-libs.x86_64 1:5.5.60-1.el7_5 を 削除
--> 依存性の処理をしています: libmysqlclient.so.18()(64bit) のパッケージ: 2:postfix-2.10.1-7.el7.x86_64
--> 依存性の処理をしています: libmysqlclient.so.18(libmysqlclient_18)(64bit) のパッケージ: 2:postfix-2.10.1-7.el7.x86_64
--> トランザクションの確認を実行しています。
---> パッケージ postfix.x86_64 2:2.10.1-7.el7 を 削除
--> 依存性解決を終了しました。

依存性を解決しました

=================================================================================================================================================
 Package                            アーキテクチャー             バージョン                                リポジトリー                     容量
=================================================================================================================================================
削除中:
 mariadb-libs                       x86_64                       1:5.5.60-1.el7_5                          @anaconda                       4.4 M
依存性関連での削除をします:
 postfix                            x86_64                       2:2.10.1-7.el7                            @anaconda                        12 M

トランザクションの要約
=================================================================================================================================================
削除  1 パッケージ (+1 個の依存関係のパッケージ)

インストール容量: 17 M
上記の処理を行います。よろしいでしょうか？ [y/N]y
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  削除中                  : 2:postfix-2.10.1-7.el7.x86_64                                                                                    1/2
  削除中                  : 1:mariadb-libs-5.5.60-1.el7_5.x86_64                                                                             2/2
  検証中                  : 2:postfix-2.10.1-7.el7.x86_64                                                                                    1/2
  検証中                  : 1:mariadb-libs-5.5.60-1.el7_5.x86_64                                                                             2/2

削除しました:
  mariadb-libs.x86_64 1:5.5.60-1.el7_5

依存性の削除をしました:
  postfix.x86_64 2:2.10.1-7.el7

完了しました!
```

```
$ yum remove mysql*
読み込んだプラグイン:fastestmirror
引数に一致しません: mysql*
削除対象とマークされたパッケージはありません。
```

- インストール

```
$ yum install http://dev.mysql.com/get/mysql57-community-release-el7-9.noarch.rpm
読み込んだプラグイン:fastestmirror
mysql57-community-release-el7-9.noarch.rpm                                                                                | 9.0 kB  00:00:00
/var/tmp/yum-root-ku0nhC/mysql57-community-release-el7-9.noarch.rpm を調べています: mysql57-community-release-el7-9.noarch
/var/tmp/yum-root-ku0nhC/mysql57-community-release-el7-9.noarch.rpm をインストール済みとして設定しています
依存性の解決をしています
--> トランザクションの確認を実行しています。
---> パッケージ mysql57-community-release.noarch 0:el7-9 を インストール
--> 依存性解決を終了しました。

依存性を解決しました

=================================================================================================================================================
 Package                                 アーキテクチャー     バージョン             リポジトリー                                           容量
=================================================================================================================================================
インストール中:
 mysql57-community-release               noarch               el7-9                  /mysql57-community-release-el7-9.noarch               8.6 k

トランザクションの要約
=================================================================================================================================================
インストール  1 パッケージ

合計容量: 8.6 k
インストール容量: 8.6 k
Is this ok [y/d/N]: y
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  インストール中          : mysql57-community-release-el7-9.noarch                                                                           1/1
  検証中                  : mysql57-community-release-el7-9.noarch                                                                           1/1

インストール:
  mysql57-community-release.noarch 0:el7-9

完了しました!
```

```
$ yum -y install mysql-community-server
読み込んだプラグイン:fastestmirror
Loading mirror speeds from cached hostfile
 * base: ftp-srv2.kddilabs.jp
 * epel: ftp.iij.ad.jp
 * extras: ftp-srv2.kddilabs.jp
 * updates: ftp-srv2.kddilabs.jp
mysql-connectors-community                                                                                                | 2.5 kB  00:00:00
mysql-tools-community                                                                                                     | 2.5 kB  00:00:00
mysql57-community                                                                                                         | 2.5 kB  00:00:00
(1/3): mysql-connectors-community/x86_64/primary_db                                                                       |  41 kB  00:00:00
(2/3): mysql-tools-community/x86_64/primary_db                                                                            |  58 kB  00:00:00
(3/3): mysql57-community/x86_64/primary_db                                                                                | 177 kB  00:00:00
依存性の解決をしています
--> トランザクションの確認を実行しています。
---> パッケージ mysql-community-server.x86_64 0:5.7.26-1.el7 を インストール
--> 依存性の処理をしています: mysql-community-common(x86-64) = 5.7.26-1.el7 のパッケージ: mysql-community-server-5.7.26-1.el7.x86_64
--> 依存性の処理をしています: mysql-community-client(x86-64) >= 5.7.9 のパッケージ: mysql-community-server-5.7.26-1.el7.x86_64
--> 依存性の処理をしています: net-tools のパッケージ: mysql-community-server-5.7.26-1.el7.x86_64
--> トランザクションの確認を実行しています。
---> パッケージ mysql-community-client.x86_64 0:5.7.26-1.el7 を インストール
--> 依存性の処理をしています: mysql-community-libs(x86-64) >= 5.7.9 のパッケージ: mysql-community-client-5.7.26-1.el7.x86_64
---> パッケージ mysql-community-common.x86_64 0:5.7.26-1.el7 を インストール
---> パッケージ net-tools.x86_64 0:2.0-0.24.20131004git.el7 を インストール
--> トランザクションの確認を実行しています。
---> パッケージ mysql-community-libs.x86_64 0:5.7.26-1.el7 を インストール
--> 依存性解決を終了しました。

依存性を解決しました

=================================================================================================================================================
 Package                                アーキテクチャー       バージョン                                リポジトリー                       容量
=================================================================================================================================================
インストール中:
 mysql-community-server                 x86_64                 5.7.26-1.el7                              mysql57-community                 166 M
依存性関連でのインストールをします:
 mysql-community-client                 x86_64                 5.7.26-1.el7                              mysql57-community                  24 M
 mysql-community-common                 x86_64                 5.7.26-1.el7                              mysql57-community                 274 k
 mysql-community-libs                   x86_64                 5.7.26-1.el7                              mysql57-community                 2.2 M
 net-tools                              x86_64                 2.0-0.24.20131004git.el7                  base                              306 k

トランザクションの要約
=================================================================================================================================================
インストール  1 パッケージ (+4 個の依存関係のパッケージ)

総ダウンロード容量: 192 M
インストール容量: 866 M
Downloading packages:
警告: /var/cache/yum/x86_64/7/mysql57-community/packages/mysql-community-common-5.7.26-1.el7.x86_64.rpm: ヘッダー V3 DSA/SHA1 Signature、鍵 ID 5072e1f5: NOKEY
mysql-community-common-5.7.26-1.el7.x86_64.rpm の公開鍵がインストールされていません
(1/5): mysql-community-common-5.7.26-1.el7.x86_64.rpm                                                                     | 274 kB  00:00:00
(2/5): mysql-community-libs-5.7.26-1.el7.x86_64.rpm                                                                       | 2.2 MB  00:00:03
(3/5): net-tools-2.0-0.24.20131004git.el7.x86_64.rpm                                                                      | 306 kB  00:00:01
(4/5): mysql-community-client-5.7.26-1.el7.x86_64.rpm                                                                     |  24 MB  00:00:05
(5/5): mysql-community-server-5.7.26-1.el7.x86_64.rpm                                                                     | 166 MB  00:00:24
-------------------------------------------------------------------------------------------------------------------------------------------------
合計                                                                                                             6.8 MB/s | 192 MB  00:00:28
file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql から鍵を取得中です。
Importing GPG key 0x5072E1F5:
 Userid     : "MySQL Release Engineering <mysql-build@oss.oracle.com>"
 Fingerprint: a4a9 4068 76fc bd3c 4567 70c8 8c71 8d3b 5072 e1f5
 Package    : mysql57-community-release-el7-9.noarch (installed)
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  インストール中          : mysql-community-common-5.7.26-1.el7.x86_64                                                                       1/5
  インストール中          : mysql-community-libs-5.7.26-1.el7.x86_64                                                                         2/5
  インストール中          : mysql-community-client-5.7.26-1.el7.x86_64                                                                       3/5
  インストール中          : net-tools-2.0-0.24.20131004git.el7.x86_64                                                                        4/5
  インストール中          : mysql-community-server-5.7.26-1.el7.x86_64                                                                       5/5
  検証中                  : mysql-community-server-5.7.26-1.el7.x86_64                                                                       1/5
  検証中                  : net-tools-2.0-0.24.20131004git.el7.x86_64                                                                        2/5
  検証中                  : mysql-community-client-5.7.26-1.el7.x86_64                                                                       3/5
  検証中                  : mysql-community-libs-5.7.26-1.el7.x86_64                                                                         4/5
  検証中                  : mysql-community-common-5.7.26-1.el7.x86_64                                                                       5/5

インストール:
  mysql-community-server.x86_64 0:5.7.26-1.el7

依存性関連をインストールしました:
  mysql-community-client.x86_64 0:5.7.26-1.el7    mysql-community-common.x86_64 0:5.7.26-1.el7    mysql-community-libs.x86_64 0:5.7.26-1.el7
  net-tools.x86_64 0:2.0-0.24.20131004git.el7

完了しました!
```

- バージョン確認

```
$ mysqld --version
mysqld  Ver 5.7.26 for Linux on x86_64 (MySQL Community Server (GPL))
```

- 起動（自動起動の設定）

```
$ systemctl start mysqld.service
$ systemctl enable mysqld.service
```

- root ユーザーの初期パスワード

```
$ cat /var/log/mysqld.log | grep 'password is generated'
2019-05-06T07:48:36.480588Z 1 [Note] A temporary password is generated for root@localhost: DZD1m37rEx-d
```

- 初期設定

```
$ mysql_secure_installation
Enter password for user root: //ログに出力されていた初期パスワードを入力
Set root password?　//ルートのパスワードを変更しますか？ Yでパスワード設定
Remove anonymous users?　//誰でもログイン可能になっているが消しますか？ Yで消しとく
Disallow root login remotely?　//リモートからrootログインを許可しませんか？ Yでしない
Remove test database and access to it?　//テストデータベース消していいですか？ Yで消す
Reload privilege tables now?　//設定をすぐに反映しますか？ Yで反映
All done!
```

- MySQL 接続確認

```
mysql -u root -p
```

- my.conf 設定

```
mysql> SHOW VARIABLES LIKE 'char%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | latin1                     |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | latin1                     |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.00 sec)
```

- ストレージエンジンを、InnoDB、データファイル・ログファイルをテーブルごとに設定
- 文字コードを utf8 に

```
$ vi /etc/my.cnf

[mysqld]
default-storage-engine=InnoDB
innodb_file_per_table

character-set-server = utf8
collation-server = utf8_general_ci

[mysql]
default-character-set = utf8

[client]
default-character-set = utf8
```

- 最終的な my.cnf

```
# For advice on how to change settings please see
# http://dev.mysql.com/doc/refman/5.7/en/server-configuration-defaults.html

[mysqld]
default-storage-engine=InnoDB
innodb_file_per_table
character-set-server = utf8
collation-server = utf8_general_ci
#
# Remove leading # and set to the amount of RAM for the most important data
# cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
# innodb_buffer_pool_size = 128M
#
# Remove leading # to turn on a very important data integrity option: logging
# changes to the binary log between backups.
# log_bin
#
# Remove leading # to set options mainly useful for reporting servers.
# The server defaults are faster for transactions and fast SELECTs.
# Adjust sizes as needed, experiment to find the optimal values.
# join_buffer_size = 128M
# sort_buffer_size = 2M
# read_rnd_buffer_size = 2M
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

[mysql]
default-character-set = utf8
[client]
default-character-set = utf8
```

- my.conf が更新されない
  再起動する

```
[root@localhost~]# systemctl stop mysqld.service
[root@localhost~]# systemctl start mysqld.service
```

参考 URL：
https://qiita.com/kyophp/items/0110d4307eff747c7092#apache24
my.conf が更新されない
https://qiita.com/yutaro1985/items/7ee4251531e2f1ededca

### PHP7.2

- libmcrypt
  mcrypt のインストールに必要なので先にインストール

```
# yum --enablerepo=epel install libmcrypt
読み込んだプラグイン:fastestmirror
Loading mirror speeds from cached hostfile
 * base: ftp-srv2.kddilabs.jp
 * epel: ftp.iij.ad.jp
 * extras: ftp-srv2.kddilabs.jp
 * updates: ftp-srv2.kddilabs.jp
依存性の解決をしています
--> トランザクションの確認を実行しています。
---> パッケージ libmcrypt.x86_64 0:2.5.8-13.el7 を インストール
--> 依存性解決を終了しました。

依存性を解決しました

=================================================================================================================================================
 Package                            アーキテクチャー                バージョン                               リポジトリー                   容量
=================================================================================================================================================
インストール中:
 libmcrypt                          x86_64                          2.5.8-13.el7                             epel                           99 k

トランザクションの要約
=================================================================================================================================================
インストール  1 パッケージ

総ダウンロード容量: 99 k
インストール容量: 283 k
Is this ok [y/d/N]: y
Downloading packages:
libmcrypt-2.5.8-13.el7.x86_64.rpm                                                                                         |  99 kB  00:00:00
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  インストール中          : libmcrypt-2.5.8-13.el7.x86_64                                                                                    1/1
  検証中                  : libmcrypt-2.5.8-13.el7.x86_64                                                                                    1/1

インストール:
  libmcrypt.x86_64 0:2.5.8-13.el7

完了しました!
```

- インストール
  FastCGI で動かせる PHP-FPM、キャッシュ周りの OPcache、APCu はインストールしておく

```
# yum -y install http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
読み込んだプラグイン:fastestmirror
remi-release-7.rpm                                                                                                        |  16 kB  00:00:00
/var/tmp/yum-root-ku0nhC/remi-release-7.rpm を調べています: remi-release-7.6-2.el7.remi.noarch
/var/tmp/yum-root-ku0nhC/remi-release-7.rpm をインストール済みとして設定しています
依存性の解決をしています
--> トランザクションの確認を実行しています。
---> パッケージ remi-release.noarch 0:7.6-2.el7.remi を インストール
--> 依存性解決を終了しました。

依存性を解決しました

=================================================================================================================================================
 Package                           アーキテクチャー            バージョン                             リポジトリー                          容量
=================================================================================================================================================
インストール中:
 remi-release                      noarch                      7.6-2.el7.remi                         /remi-release-7                       19 k

トランザクションの要約
=================================================================================================================================================
インストール  1 パッケージ

合計容量: 19 k
インストール容量: 19 k
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  インストール中          : remi-release-7.6-2.el7.remi.noarch                                                                               1/1
  検証中                  : remi-release-7.6-2.el7.remi.noarch                                                                               1/1

インストール:
  remi-release.noarch 0:7.6-2.el7.remi

完了しました!
```

```
# yum -y install --enablerepo=remi,remi-php72 php php-mbstring php-xml php-xmlrpc php-gd php-pdo php-pecl-mcrypt php-mysqlnd php-pecl-mysql
読み込んだプラグイン:fastestmirror
Loading mirror speeds from cached hostfile
 * base: ftp-srv2.kddilabs.jp
 * epel: ftp.iij.ad.jp
 * extras: ftp-srv2.kddilabs.jp
 * remi: ftp.riken.jp
 * remi-php72: ftp.riken.jp
 * remi-safe: ftp.riken.jp
 * updates: ftp-srv2.kddilabs.jp
remi                                                                                                                      | 3.0 kB  00:00:00
remi-php72                                                                                                                | 3.0 kB  00:00:00
remi-safe                                                                                                                 | 3.0 kB  00:00:00
(1/3): remi-php72/primary_db                                                                                              | 222 kB  00:00:00
(2/3): remi-safe/primary_db                                                                                               | 1.4 MB  00:00:00
(3/3): remi/primary_db                                                                                                    | 2.3 MB  00:00:00
依存性の解決をしています
--> トランザクションの確認を実行しています。
---> パッケージ php.x86_64 0:7.2.18-1.el7.remi を インストール
--> 依存性の処理をしています: php-cli(x86-64) = 7.2.18-1.el7.remi のパッケージ: php-7.2.18-1.el7.remi.x86_64
--> 依存性の処理をしています: php-common(x86-64) = 7.2.18-1.el7.remi のパッケージ: php-7.2.18-1.el7.remi.x86_64
--> 依存性の処理をしています: libargon2.so.0()(64bit) のパッケージ: php-7.2.18-1.el7.remi.x86_64
---> パッケージ php-gd.x86_64 0:7.2.18-1.el7.remi を インストール
--> 依存性の処理をしています: gd-last(x86-64) >= 2.1.1 のパッケージ: php-gd-7.2.18-1.el7.remi.x86_64
--> 依存性の処理をしています: libX11.so.6()(64bit) のパッケージ: php-gd-7.2.18-1.el7.remi.x86_64
--> 依存性の処理をしています: libXpm.so.4()(64bit) のパッケージ: php-gd-7.2.18-1.el7.remi.x86_64
--> 依存性の処理をしています: libgd.so.3()(64bit) のパッケージ: php-gd-7.2.18-1.el7.remi.x86_64
--> 依存性の処理をしています: libjpeg.so.62()(64bit) のパッケージ: php-gd-7.2.18-1.el7.remi.x86_64
---> パッケージ php-mbstring.x86_64 0:7.2.18-1.el7.remi を インストール
--> 依存性の処理をしています: libonig.so.5()(64bit) のパッケージ: php-mbstring-7.2.18-1.el7.remi.x86_64
---> パッケージ php-mysqlnd.x86_64 0:7.2.18-1.el7.remi を インストール
---> パッケージ php-pdo.x86_64 0:7.2.18-1.el7.remi を インストール
---> パッケージ php-pecl-mcrypt.x86_64 0:1.0.2-2.el7.remi.7.2 を インストール
---> パッケージ php-pecl-mysql.x86_64 0:1.0.0-0.17.20160812git230a828.el7.remi.7.2 を インストール
---> パッケージ php-xml.x86_64 0:7.2.18-1.el7.remi を インストール
--> 依存性の処理をしています: libxslt.so.1(LIBXML2_1.0.11)(64bit) のパッケージ: php-xml-7.2.18-1.el7.remi.x86_64
--> 依存性の処理をしています: libxslt.so.1(LIBXML2_1.0.13)(64bit) のパッケージ: php-xml-7.2.18-1.el7.remi.x86_64
--> 依存性の処理をしています: libxslt.so.1(LIBXML2_1.0.18)(64bit) のパッケージ: php-xml-7.2.18-1.el7.remi.x86_64
--> 依存性の処理をしています: libxslt.so.1(LIBXML2_1.0.22)(64bit) のパッケージ: php-xml-7.2.18-1.el7.remi.x86_64
--> 依存性の処理をしています: libxslt.so.1(LIBXML2_1.0.24)(64bit) のパッケージ: php-xml-7.2.18-1.el7.remi.x86_64
--> 依存性の処理をしています: libexslt.so.0()(64bit) のパッケージ: php-xml-7.2.18-1.el7.remi.x86_64
--> 依存性の処理をしています: libxslt.so.1()(64bit) のパッケージ: php-xml-7.2.18-1.el7.remi.x86_64
---> パッケージ php-xmlrpc.x86_64 0:7.2.18-1.el7.remi を インストール
--> トランザクションの確認を実行しています。
---> パッケージ gd-last.x86_64 0:2.2.5-8.el7.remi を インストール
--> 依存性の処理をしています: libtiff.so.5(LIBTIFF_4.0)(64bit) のパッケージ: gd-last-2.2.5-8.el7.remi.x86_64
--> 依存性の処理をしています: libfontconfig.so.1()(64bit) のパッケージ: gd-last-2.2.5-8.el7.remi.x86_64
--> 依存性の処理をしています: libtiff.so.5()(64bit) のパッケージ: gd-last-2.2.5-8.el7.remi.x86_64
--> 依存性の処理をしています: libwebp.so.7()(64bit) のパッケージ: gd-last-2.2.5-8.el7.remi.x86_64
---> パッケージ libX11.x86_64 0:1.6.5-2.el7 を インストール
--> 依存性の処理をしています: libX11-common >= 1.6.5-2.el7 のパッケージ: libX11-1.6.5-2.el7.x86_64
--> 依存性の処理をしています: libxcb.so.1()(64bit) のパッケージ: libX11-1.6.5-2.el7.x86_64
---> パッケージ libXpm.x86_64 0:3.5.12-1.el7 を インストール
---> パッケージ libargon2.x86_64 0:20161029-3.el7 を インストール
---> パッケージ libjpeg-turbo.x86_64 0:1.2.90-6.el7 を インストール
---> パッケージ libxslt.x86_64 0:1.1.28-5.el7 を インストール
---> パッケージ oniguruma5.x86_64 0:6.9.1-1.el7.remi を インストール
---> パッケージ php-cli.x86_64 0:7.2.18-1.el7.remi を インストール
---> パッケージ php-common.x86_64 0:7.2.18-1.el7.remi を インストール
--> 依存性の処理をしています: php-json(x86-64) = 7.2.18-1.el7.remi のパッケージ: php-common-7.2.18-1.el7.remi.x86_64
--> トランザクションの確認を実行しています。
---> パッケージ fontconfig.x86_64 0:2.13.0-4.3.el7 を インストール
--> 依存性の処理をしています: fontpackages-filesystem のパッケージ: fontconfig-2.13.0-4.3.el7.x86_64
--> 依存性の処理をしています: dejavu-sans-fonts のパッケージ: fontconfig-2.13.0-4.3.el7.x86_64
---> パッケージ libX11-common.noarch 0:1.6.5-2.el7 を インストール
---> パッケージ libtiff.x86_64 0:4.0.3-27.el7_3 を インストール
--> 依存性の処理をしています: libjbig.so.2.0()(64bit) のパッケージ: libtiff-4.0.3-27.el7_3.x86_64
---> パッケージ libwebp7.x86_64 0:1.0.2-1.el7.remi を インストール
---> パッケージ libxcb.x86_64 0:1.13-1.el7 を インストール
--> 依存性の処理をしています: libXau.so.6()(64bit) のパッケージ: libxcb-1.13-1.el7.x86_64
---> パッケージ php-json.x86_64 0:7.2.18-1.el7.remi を インストール
--> トランザクションの確認を実行しています。
---> パッケージ dejavu-sans-fonts.noarch 0:2.33-6.el7 を インストール
--> 依存性の処理をしています: dejavu-fonts-common = 2.33-6.el7 のパッケージ: dejavu-sans-fonts-2.33-6.el7.noarch
---> パッケージ fontpackages-filesystem.noarch 0:1.44-8.el7 を インストール
---> パッケージ jbigkit-libs.x86_64 0:2.0-11.el7 を インストール
---> パッケージ libXau.x86_64 0:1.0.8-2.1.el7 を インストール
--> トランザクションの確認を実行しています。
---> パッケージ dejavu-fonts-common.noarch 0:2.33-6.el7 を インストール
--> 依存性解決を終了しました。

依存性を解決しました

=================================================================================================================================================
 Package                              アーキテクチャー    バージョン                                               リポジトリー             容量
=================================================================================================================================================
インストール中:
 php                                  x86_64              7.2.18-1.el7.remi                                        remi-php72              3.2 M
 php-gd                               x86_64              7.2.18-1.el7.remi                                        remi-php72               79 k
 php-mbstring                         x86_64              7.2.18-1.el7.remi                                        remi-php72              494 k
 php-mysqlnd                          x86_64              7.2.18-1.el7.remi                                        remi-php72              234 k
 php-pdo                              x86_64              7.2.18-1.el7.remi                                        remi-php72              126 k
 php-pecl-mcrypt                      x86_64              1.0.2-2.el7.remi.7.2                                     remi-php72               29 k
 php-pecl-mysql                       x86_64              1.0.0-0.17.20160812git230a828.el7.remi.7.2               remi-php72               38 k
 php-xml                              x86_64              7.2.18-1.el7.remi                                        remi-php72              207 k
 php-xmlrpc                           x86_64              7.2.18-1.el7.remi                                        remi-php72               81 k
依存性関連でのインストールをします:
 dejavu-fonts-common                  noarch              2.33-6.el7                                               base                     64 k
 dejavu-sans-fonts                    noarch              2.33-6.el7                                               base                    1.4 M
 fontconfig                           x86_64              2.13.0-4.3.el7                                           base                    254 k
 fontpackages-filesystem              noarch              1.44-8.el7                                               base                    9.9 k
 gd-last                              x86_64              2.2.5-8.el7.remi                                         remi                    134 k
 jbigkit-libs                         x86_64              2.0-11.el7                                               base                     46 k
 libX11                               x86_64              1.6.5-2.el7                                              base                    606 k
 libX11-common                        noarch              1.6.5-2.el7                                              base                    164 k
 libXau                               x86_64              1.0.8-2.1.el7                                            base                     29 k
 libXpm                               x86_64              3.5.12-1.el7                                             base                     55 k
 libargon2                            x86_64              20161029-3.el7                                           epel                     23 k
 libjpeg-turbo                        x86_64              1.2.90-6.el7                                             base                    134 k
 libtiff                              x86_64              4.0.3-27.el7_3                                           base                    170 k
 libwebp7                             x86_64              1.0.2-1.el7.remi                                         remi                    265 k
 libxcb                               x86_64              1.13-1.el7                                               base                    214 k
 libxslt                              x86_64              1.1.28-5.el7                                             base                    242 k
 oniguruma5                           x86_64              6.9.1-1.el7.remi                                         remi                    188 k
 php-cli                              x86_64              7.2.18-1.el7.remi                                        remi-php72              4.8 M
 php-common                           x86_64              7.2.18-1.el7.remi                                        remi-php72              1.1 M
 php-json                             x86_64              7.2.18-1.el7.remi                                        remi-php72               64 k

トランザクションの要約
=================================================================================================================================================
インストール  9 パッケージ (+20 個の依存関係のパッケージ)

総ダウンロード容量: 14 M
インストール容量: 55 M
Downloading packages:
(1/29): dejavu-fonts-common-2.33-6.el7.noarch.rpm                                                                         |  64 kB  00:00:00
(2/29): jbigkit-libs-2.0-11.el7.x86_64.rpm                                                                                |  46 kB  00:00:00
(3/29): libX11-1.6.5-2.el7.x86_64.rpm                                                                                     | 606 kB  00:00:00
(4/29): libX11-common-1.6.5-2.el7.noarch.rpm                                                                              | 164 kB  00:00:00
(5/29): libXau-1.0.8-2.1.el7.x86_64.rpm                                                                                   |  29 kB  00:00:00
(6/29): libXpm-3.5.12-1.el7.x86_64.rpm                                                                                    |  55 kB  00:00:00
(7/29): fontpackages-filesystem-1.44-8.el7.noarch.rpm                                                                     | 9.9 kB  00:00:00
warning: /var/cache/yum/x86_64/7/remi/packages/gd-last-2.2.5-8.el7.remi.x86_64.rpm: Header V4 DSA/SHA1 Signature, key ID 00f97f56: NOKEY
gd-last-2.2.5-8.el7.remi.x86_64.rpm の公開鍵がインストールされていません
(8/29): gd-last-2.2.5-8.el7.remi.x86_64.rpm                                                                               | 134 kB  00:00:00
(9/29): libjpeg-turbo-1.2.90-6.el7.x86_64.rpm                                                                             | 134 kB  00:00:00
(10/29): fontconfig-2.13.0-4.3.el7.x86_64.rpm                                                                             | 254 kB  00:00:00
(11/29): libtiff-4.0.3-27.el7_3.x86_64.rpm                                                                                | 170 kB  00:00:00
(12/29): libwebp7-1.0.2-1.el7.remi.x86_64.rpm                                                                             | 265 kB  00:00:00
(13/29): libxcb-1.13-1.el7.x86_64.rpm                                                                                     | 214 kB  00:00:00
(14/29): oniguruma5-6.9.1-1.el7.remi.x86_64.rpm                                                                           | 188 kB  00:00:00
(15/29): dejavu-sans-fonts-2.33-6.el7.noarch.rpm                                                                          | 1.4 MB  00:00:01
(16/29): libxslt-1.1.28-5.el7.x86_64.rpm                                                                                  | 242 kB  00:00:00
(17/29): libargon2-20161029-3.el7.x86_64.rpm                                                                              |  23 kB  00:00:00
php-cli-7.2.18-1.el7.remi.x86_64.rpm の公開鍵がインストールされていません=====================                 ] 2.2 MB/s | 9.3 MB  00:00:02 ETA
(18/29): php-cli-7.2.18-1.el7.remi.x86_64.rpm                                                                             | 4.8 MB  00:00:01
(19/29): php-mbstring-7.2.18-1.el7.remi.x86_64.rpm                                                                        | 494 kB  00:00:00
(20/29): php-json-7.2.18-1.el7.remi.x86_64.rpm                                                                            |  64 kB  00:00:00
(21/29): php-mysqlnd-7.2.18-1.el7.remi.x86_64.rpm                                                                         | 234 kB  00:00:00
(22/29): php-pecl-mcrypt-1.0.2-2.el7.remi.7.2.x86_64.rpm                                                                  |  29 kB  00:00:00
(23/29): php-pecl-mysql-1.0.0-0.17.20160812git230a828.el7.remi.7.2.x86_64.rpm                                             |  38 kB  00:00:00
(24/29): php-xml-7.2.18-1.el7.remi.x86_64.rpm                                                                             | 207 kB  00:00:00
(25/29): php-7.2.18-1.el7.remi.x86_64.rpm                                                                                 | 3.2 MB  00:00:01
(26/29): php-xmlrpc-7.2.18-1.el7.remi.x86_64.rpm                                                                          |  81 kB  00:00:00
(27/29): php-gd-7.2.18-1.el7.remi.x86_64.rpm                                                                              |  79 kB  00:00:01
(28/29): php-pdo-7.2.18-1.el7.remi.x86_64.rpm                                                                             | 126 kB  00:00:00
(29/29): php-common-7.2.18-1.el7.remi.x86_64.rpm                                                                          | 1.1 MB  00:00:02
-------------------------------------------------------------------------------------------------------------------------------------------------
合計                                                                                                             4.1 MB/s |  14 MB  00:00:03
file:///etc/pki/rpm-gpg/RPM-GPG-KEY-remi から鍵を取得中です。
Importing GPG key 0x00F97F56:
 Userid     : "Remi Collet <RPMS@FamilleCollet.com>"
 Fingerprint: 1ee0 4cce 88a4 ae4a a29a 5df5 004e 6f47 00f9 7f56
 Package    : remi-release-7.6-2.el7.remi.noarch (installed)
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-remi
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  インストール中          : libjpeg-turbo-1.2.90-6.el7.x86_64                                                                               1/29
  インストール中          : fontpackages-filesystem-1.44-8.el7.noarch                                                                       2/29
  インストール中          : libargon2-20161029-3.el7.x86_64                                                                                 3/29
  インストール中          : dejavu-fonts-common-2.33-6.el7.noarch                                                                           4/29
  インストール中          : dejavu-sans-fonts-2.33-6.el7.noarch                                                                             5/29
  インストール中          : fontconfig-2.13.0-4.3.el7.x86_64                                                                                6/29
  インストール中          : php-common-7.2.18-1.el7.remi.x86_64                                                                             7/29
  インストール中          : php-json-7.2.18-1.el7.remi.x86_64                                                                               8/29
  インストール中          : php-cli-7.2.18-1.el7.remi.x86_64                                                                                9/29
  インストール中          : php-pdo-7.2.18-1.el7.remi.x86_64                                                                               10/29
  インストール中          : php-mysqlnd-7.2.18-1.el7.remi.x86_64                                                                           11/29
  インストール中          : libX11-common-1.6.5-2.el7.noarch                                                                               12/29
  インストール中          : jbigkit-libs-2.0-11.el7.x86_64                                                                                 13/29
  インストール中          : libtiff-4.0.3-27.el7_3.x86_64                                                                                  14/29
  インストール中          : libXau-1.0.8-2.1.el7.x86_64                                                                                    15/29
  インストール中          : libxcb-1.13-1.el7.x86_64                                                                                       16/29
  インストール中          : libX11-1.6.5-2.el7.x86_64                                                                                      17/29
  インストール中          : libXpm-3.5.12-1.el7.x86_64                                                                                     18/29
  インストール中          : libwebp7-1.0.2-1.el7.remi.x86_64                                                                               19/29
  インストール中          : gd-last-2.2.5-8.el7.remi.x86_64                                                                                20/29
  インストール中          : libxslt-1.1.28-5.el7.x86_64                                                                                    21/29
  インストール中          : php-xml-7.2.18-1.el7.remi.x86_64                                                                               22/29
  インストール中          : oniguruma5-6.9.1-1.el7.remi.x86_64                                                                             23/29
  インストール中          : php-mbstring-7.2.18-1.el7.remi.x86_64                                                                          24/29
  インストール中          : php-xmlrpc-7.2.18-1.el7.remi.x86_64                                                                            25/29
  インストール中          : php-gd-7.2.18-1.el7.remi.x86_64                                                                                26/29
  インストール中          : php-pecl-mysql-1.0.0-0.17.20160812git230a828.el7.remi.7.2.x86_64                                               27/29
  インストール中          : php-7.2.18-1.el7.remi.x86_64                                                                                   28/29
  インストール中          : php-pecl-mcrypt-1.0.2-2.el7.remi.7.2.x86_64                                                                    29/29
  検証中                  : libtiff-4.0.3-27.el7_3.x86_64                                                                                   1/29
  検証中                  : fontconfig-2.13.0-4.3.el7.x86_64                                                                                2/29
  検証中                  : php-json-7.2.18-1.el7.remi.x86_64                                                                               3/29
  検証中                  : php-mbstring-7.2.18-1.el7.remi.x86_64                                                                           4/29
  検証中                  : libargon2-20161029-3.el7.x86_64                                                                                 5/29
  検証中                  : oniguruma5-6.9.1-1.el7.remi.x86_64                                                                              6/29
  検証中                  : fontpackages-filesystem-1.44-8.el7.noarch                                                                       7/29
  検証中                  : libjpeg-turbo-1.2.90-6.el7.x86_64                                                                               8/29
  検証中                  : php-cli-7.2.18-1.el7.remi.x86_64                                                                                9/29
  検証中                  : php-7.2.18-1.el7.remi.x86_64                                                                                   10/29
  検証中                  : php-common-7.2.18-1.el7.remi.x86_64                                                                            11/29
  検証中                  : php-pecl-mcrypt-1.0.2-2.el7.remi.7.2.x86_64                                                                    12/29
  検証中                  : php-pdo-7.2.18-1.el7.remi.x86_64                                                                               13/29
  検証中                  : dejavu-fonts-common-2.33-6.el7.noarch                                                                          14/29
  検証中                  : libxcb-1.13-1.el7.x86_64                                                                                       15/29
  検証中                  : libXpm-3.5.12-1.el7.x86_64                                                                                     16/29
  検証中                  : libxslt-1.1.28-5.el7.x86_64                                                                                    17/29
  検証中                  : libX11-1.6.5-2.el7.x86_64                                                                                      18/29
  検証中                  : dejavu-sans-fonts-2.33-6.el7.noarch                                                                            19/29
  検証中                  : gd-last-2.2.5-8.el7.remi.x86_64                                                                                20/29
  検証中                  : libwebp7-1.0.2-1.el7.remi.x86_64                                                                               21/29
  検証中                  : php-mysqlnd-7.2.18-1.el7.remi.x86_64                                                                           22/29
  検証中                  : php-gd-7.2.18-1.el7.remi.x86_64                                                                                23/29
  検証中                  : libXau-1.0.8-2.1.el7.x86_64                                                                                    24/29
  検証中                  : php-pecl-mysql-1.0.0-0.17.20160812git230a828.el7.remi.7.2.x86_64                                               25/29
  検証中                  : jbigkit-libs-2.0-11.el7.x86_64                                                                                 26/29
  検証中                  : libX11-common-1.6.5-2.el7.noarch                                                                               27/29
  検証中                  : php-xmlrpc-7.2.18-1.el7.remi.x86_64                                                                            28/29
  検証中                  : php-xml-7.2.18-1.el7.remi.x86_64                                                                               29/29

インストール:
  php.x86_64 0:7.2.18-1.el7.remi                                                    php-gd.x86_64 0:7.2.18-1.el7.remi
  php-mbstring.x86_64 0:7.2.18-1.el7.remi                                           php-mysqlnd.x86_64 0:7.2.18-1.el7.remi
  php-pdo.x86_64 0:7.2.18-1.el7.remi                                                php-pecl-mcrypt.x86_64 0:1.0.2-2.el7.remi.7.2
  php-pecl-mysql.x86_64 0:1.0.0-0.17.20160812git230a828.el7.remi.7.2                php-xml.x86_64 0:7.2.18-1.el7.remi
  php-xmlrpc.x86_64 0:7.2.18-1.el7.remi

依存性関連をインストールしました:
  dejavu-fonts-common.noarch 0:2.33-6.el7             dejavu-sans-fonts.noarch 0:2.33-6.el7         fontconfig.x86_64 0:2.13.0-4.3.el7
  fontpackages-filesystem.noarch 0:1.44-8.el7         gd-last.x86_64 0:2.2.5-8.el7.remi             jbigkit-libs.x86_64 0:2.0-11.el7
  libX11.x86_64 0:1.6.5-2.el7                         libX11-common.noarch 0:1.6.5-2.el7            libXau.x86_64 0:1.0.8-2.1.el7
  libXpm.x86_64 0:3.5.12-1.el7                        libargon2.x86_64 0:20161029-3.el7             libjpeg-turbo.x86_64 0:1.2.90-6.el7
  libtiff.x86_64 0:4.0.3-27.el7_3                     libwebp7.x86_64 0:1.0.2-1.el7.remi            libxcb.x86_64 0:1.13-1.el7
  libxslt.x86_64 0:1.1.28-5.el7                       oniguruma5.x86_64 0:6.9.1-1.el7.remi          php-cli.x86_64 0:7.2.18-1.el7.remi
  php-common.x86_64 0:7.2.18-1.el7.remi               php-json.x86_64 0:7.2.18-1.el7.remi

完了しました!
```

- バージョン確認

```
[root@localhost ~]# php -v
PHP 7.2.18 (cli) (built: Apr 30 2019 15:26:52) ( NTS )
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.2.0, Copyright (c) 1998-2018 Zend Technologies
```

- php.ini 設定
  初期ファイルのバックアップ

```
$ cp /etc/php.ini /etc/php.ini.bk
```

設定

```
$ vi /etc/php.ini

[PHP]
# レスポンスヘッダにPHPのバージョンを表示しない
expose_php = Off
# 全てのログを出力
error_reporting = E_ALL
# ブラウザでエラーを表示しない
display_errors = Off
# エラーをログに残す
log_errors = On
# エラーログの長さを設定
log_errors_max_len = 4096
# エラーログ出力先
error_log = "/var/log/php_errors.log"
# 文字エンコーディング
default_charset = "UTF-8"

[Date]
# タイムゾーン
date.timezone = "Asia/Tokyo"

[mbstring]
# デフォルト言語
mbstring.language = Japanese
# 内部文字エンコーディング
mbstring.internal_encoding = UTF-8
# HTTP入力文字エンコーディングのデフォルト
mbstring.http_input = auto
# 文字エンコーディング検出順序のデフォルト
mbstring.detect_order = auto
```

- 動作確認
  ドキュメントルートに phpinfo を出力する php ファイルを作成
  ※/var/www/html/ …ドキュメントルート

```
$ vi /var/www/html/info.php

<?php
    // すべての情報を表示します。デフォルトは INFO_ALL です。
    phpinfo();
```

ブラウザで`http://192.168.56.110/info.php`にアクセス

- 接続できなかったのでポートの設定を追加

```
[root@localhost ~]# systemctl start httpd.service
// http用に80番ポートを開放
[root@localhost ~]# firewall-cmd --add-service=http --permanent
success
[root@localhost ~]# firewall-cmd --reload
success
```

php.info が確認できた

### CodeIgniter

- ダウンロード
  https://codeigniter.com/download
  Download CodeIgniter 3 をクリックして zip をダウンロード
- zip ファイルを解凍して、ホスト側の共有ファイルに置く
- CodeIgniter の中身をすべてドキュメントルート下に配置

```
# mv CodeIgniter-3.1.10/* /var/www/html/
```

`http://192.168.56.110/index.php`で接続確認

### 共有フォルダをホームディレクトリ以下にする

- 仮想マシンの設定をクリック
- 共有フォルダをクリック
- マウントポイントに「/var/www/html/hoge」

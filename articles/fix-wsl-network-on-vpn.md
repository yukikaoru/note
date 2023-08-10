---
title: "VPN利用時にWSLがネットワークに接続できなくなる問題を直す"
emoji: ""
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["wsl"]
published: true
---

# VPN利用時にWSLがネットワークに接続できなくなる問題を直す

WSLにはVPNを利用するとネットワーク接続ができなくなる問題があります。
これを解決するために[wsl-vpnkit](https://github.com/sakai135/wsl-vpnkit/)というものがあるのですが、セットアップ手順が複雑でしたのでここにまとめようかと思います。

## ゲストOSのDNS設定

DNSサーバーの設定をするのでWSLの自動設定を切る必要があります。

```conf
[network]
generateResolvConf=false
```

設定を有効にするためにWSLを再起動します。

```powershell
wsl --shutdown
wsl
```

次にresolv.confに必要な設定を入れます。
VPNは切断した状態で行います。

① ネームサーバーIP
```powershell
Get-DnsClientServerAddress -AddressFamily IPv4 | Select-Object -ExpandProperty ServerAddresses
```
② 検索ドメイン
```powershell
Get-DnsClientGlobalSetting | Select-Object -ExpandProperty SuffixSearchList
```

①と②の値をresolv.confに記述します。
```
nameserver ①
search ②
```

## wsl-vpnkitのセットアップ

[Setup as a standalone script](https://github.com/sakai135/wsl-vpnkit/#setup-as-a-standalone-script)にある通りに進めます。

### ゲストOSへのwsl-vpnkitのインストール

```bash
sudo apt-get install iproute2 iptables iputils-ping dnsutils wget
mkdir wsl-vpnkit
cd wsl-vpnkit
wget https://github.com/sakai135/wsl-vpnkit/releases/download/v0.4.1/wsl-vpnkit.tar.gz
tar --strip-components=1 -xf wsl-vpnkit.tar.gz \
    app/wsl-vpnkit \
    app/wsl-gvproxy.exe \
    app/wsl-vm \
    app/wsl-vpnkit.service
rm wsl-vpnkit.tar.gz
cd ..
sudo mv wsl-vpnkit /opt/wsl-vpnkit
sudo chown -R root:root /opt/wsl-vpnkit
```

### Systemdの設定

wsl-vpnkitをいちいち手動実行するのはめんどうなので自動起動するようにします。
もし自動起動が嫌な場合は次のコマンドで手動実行してください。
```bash
sudo VMEXEC_PATH=/opt/wsl-vpnkit/wsl-vm GVPROXY_PATH=/opt/wsl-vpnkit/wsl-gvproxy.exe /wsl-vpnkit
```

まず、WSLではデフォルトではsystemdが有効になっていないのでゲストOSの`/etc/wsl.conf`で有効にします。
```conf
[boot]
systemd=true
```

serviceファイル書き換えて設置します。
元になるファイルは先程インストールした中にある`wsl-vpnkit.service`となります。

書き換える箇所は`[Service]`の部分の`setup as a distro`のブロックをコマントアウトし、`setup as a standalone script`のブロックをコメントインします。
そしてパスを適当なものに変更します。

変更後の全体像は次のようになります。
```conf
[Unit]
Description=wsl-vpnkit
After=network.target

[Service]
# for wsl-vpnkit setup as a distro
#ExecStart=/mnt/c/Windows/system32/wsl.exe -d wsl-vpnkit --cd /app wsl-vpnkit

# for wsl-vpnkit setup as a standalone script
ExecStart=/opt/wsl-vpnkit/wsl-vpnkit
Environment=VMEXEC_PATH=/opt/wsl-vpnkit/wsl-vm GVPROXY_PATH=/opt/wsl-vpnkit/wsl-gvproxy.exe

Restart=always
KillMode=mixed

[Install]
WantedBy=multi-user.target
```

ファイルを書き換えたらsystemdのディレクトリに配置します。
```bash
sudo mkdir -p /etc/systemd/system
sudo mv /opt/wsl-vpnkit/wsl-vpnkit.service /etc/systemd/system/
```

サービスを有効化します。
```bash
sudo systemctl enable wsl-vpnkit
```

WSLを再起動して設定を有効にします。
```powershell
wsl --shutdown
wsl
```

statusにてエラーが出ていないかをチェックしてください。
```bash
systemctl status wsl-vpnkit
```

## 接続の確認

VPNを接続状態にしdigコマンドでgoogle.comや御自身のVPNで利用するシステムのドメインのIPが引けるか確かめてください。

wsl-vpnkitはVPNの接続切り替えでDNSの設定をWSLで通常動作するものか、カスタムで設定したホストと同様の設定を利用するかを切り替えてくれる(実際にはプロキシー)ものなので、VPNに接続したままWSLを起動した場合はVPNを再接続してみてください。
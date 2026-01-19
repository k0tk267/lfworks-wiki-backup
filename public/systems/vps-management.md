---
title: VPSの管理
description: 
published: true
date: 2026-01-19T05:27:30.598Z
tags: 
editor: markdown
dateCreated: 2026-01-19T05:27:30.598Z
---

# VPSの管理
AWSは高いので、ドイツ製のContaboVPSを借りて諸々のツールを運用している。
(Memoryが12GBで、SSDが500GBが、12ユーロぐらい)
サーバーはUbuntu24.04LTSを使用

## セットアップメモ
基本的に以下のページを参考に、VPSの設定を行った。
https://zenn.dev/moribook/articles/c2e9cdee9cf5cc

Ubuntu24.04を使う場合
- `PasswordAuthentication`は/etc/ssh/sshd_confiｇ.d配下にも設定が存在するので、その中を書きかる必要がある
- `systemctl restart sshd`ではなく、`systemctl restart ssh`で再起動する
- ファイアーウォールの有効化は`sudo ufw enable`だけではなく、`systemctl daemon-reload`と`systemctl restart ssh.socket`も行う

点が異なるため注意が必要。


## 以下ページの魚拓

### はじめに

VPS（Virtual Private Server）は、仮想化技術を用いてインターネット上に仮想的なサーバーを提供するサービスです。VPSを借りることで、物理的なサーバーの購入や管理の手間を省きながら、自身のウェブサイトやアプリケーションをホスティングすることが可能となります。  
私は、コスパの良い [Contabo VPS](https://contabo.com/en/vps/) でubuntuサーバを契約しました！

この記事では、VPSを借りた際にすぐに行うべき重要な手順について解説します。これらの手順を実施することで、セキュリティの強化やシステムの安定性を確保することができます。

### 大まかな内容

1. root以外のユーザーを作成して、ssh接続できるようにする
2. rootユーザーでのssh接続を禁止する
3. パスワード認証でのssh接続を禁止する
4. セキュリティリスク軽減のため、よく使われる22番ポートではないポート番号に、SSHの使用ポートを変更する
5. ファイアウォールを設定する

### 手順

ターミナルの複数のタブを利用するので、実行するタブを記載しています。

1. root以外のユーザーを作成して、ssh接続できるようにする

```
# 【タブ1】 rootでssh接続
ssh root@<hostname or IP>

# 【タブ1】 ユーザーを作成
adduser <user_name>

# 【タブ1】 sudoを使えるようにする
gpasswd -a <user_name> sudo

# 【タブ2】 作成したユーザーでssh接続
ssh <user_name>@<hostname or IP>

# 【タブ1】 rootのssh接続を切断
exit
```

1. rootユーザーでのssh接続を禁止する
1. パスワード認証でのssh接続を禁止する  
	まず、以下記事を参考にローカルPC（Mac）でSSHキーを作成する。  
	[https://qiita.com/wakahara3/items/52094d476774f3a2f619](https://qiita.com/wakahara3/items/52094d476774f3a2f619)

```
# 【タブ2】 作成した公開鍵をサーバ側に登録
vi ~/.ssh/authorized_keys

# 【タブ2】 公開鍵の中身をペーストする

# 【タブ3】 公開鍵認証でssh接続
ssh -i <秘密鍵のパス> <user_name>@<hostname or IP>

# 【タブ3】 一旦抜ける
exit

# 【タブ2】 sshd_configをviで開く
sudo vi /etc/ssh/sshd_confiｇ

# 【タブ2】 デフォルトで(a)になっている部分を(b)に変更
(a) #PasswordAuthentication yes
(b) PasswordAuthentication no

# 【タブ2】 sshdを再起動して変更を反映させる
sudo systemctl restart sshd

# 【タブ3】 パスワード認証でssh接続できないことを確認
ssh <user_name>@<hostname or IP>

Permission denied (publickey)となればOK。
```

1. セキュリティリスク軽減のため、よく使われる22番ポートではないポート番号に、SSHの使用ポートを変更する

```
# 【タブ2】 sshd_configをviで開く
sudo vi /etc/ssh/sshd_confiｇ
# 【タブ2】 デフォルトで(a)になっている部分を(b)に変更
(a) #Port 22
(b) Port 22222
# 【タブ2】 sshdを再起動して変更を反映させる
sudo systemctl restart sshd
```

1. ファイアウォールを設定する

```
# 【タブ2】 22222番ポートを開く
sudo ufw allow 22222  #← 上記で設定したポート

# 【タブ2】 ufwを有効化
sudo ufw enable

# 【タブ2】 デフォルトを遮断にする
sudo ufw default DENY

# 【タブ2】 ステータスを確認
sudo ufw status

以下のようになっていればOK。
＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝
Status: active

To                         Action      From
--                         ------      ----
22222                      ALLOW       Anywhere
＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝
```
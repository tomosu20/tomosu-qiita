---
title: AWSを使って1からインフラ構築をしてみる
tags:
  - WordPress
  - AWS
private: false
updated_at: '2024-05-28T15:30:17+09:00'
id: 17a22b7086503c9bbca3
organization_url_name: null
slide: false
ignorePublish: false
---


# はじめに

本記事は、AWS、インフラ構築の初学者向けの記事です。
私自身も、エンジニア歴 5 年目にして、1 からインフラ構築をした経験がなく、Udemy のハンズオン講座で学習しながら得た知見を、本記事にまとめようと思います。

## そもそもインフラ構築とは？

アプリケーションが稼働できる状態を作ること。

アプリケーションが稼働するには？
アプリケーションが動く(環境)サーバーを構築して、そのアプリケーションに接続できるように、ネットワークを構成する必要がある。

## インフラ構築ができると何が嬉しい？

### 自分でサービスを作れるようになる

プログラムを書いて、良いアプリケーションを開発し、それを誰かに提供したい！となったとき、自分でできるようになる。

### 自分で開発時のテスト環境を作れるようになる

何かしら検証作業が必要な場合、本番環境で実施するわけにもいかないので、テスト環境が必要な状況ある。
そんなときに、必要な環境を自分で用意することができるようになる。

### 障害が起きても、どこに問題があるのか原因の切り分けができるようになる

「サービスにアクセスしても繋がらない」「メールが送られない」といった障害が発生した際に、アプリケーション側の問題なのか、インフラ側の問題なのか、自分で原因を切り分けて調査することができる。

## インフラを構築する基本的な流れ

### サーバーを構築(用意)する

1. どのようなサーバーが必要なのかを検討
2. 必要なサーバーを設置
3. サーバーの OS をインストールし、各種設定
4. 必要なソフトウェアをインストールし、各種設定

### 構築したサーバーをネットワークに接続する

1. ネットワークで使用する IP アドレスの範囲を決める
2. サーバーに IP アドレスを割り当てる
3. ドメイン名と IP アドレスの対応を割り当てる

# AWS（Amazon Web Service）

インフラ構築における各サービスを提供しているクラウドサービス。

https://aws.amazon.com/jp/

サービスが豊富であり、高負荷に耐えられる、信頼性の高いシステムを手間なく運用できる。
リソースを必要なときに必要な分だけ調達でき、従量課金制である。

## アカウントを作成して初期セットアップを行う

AWS アカウントの登録には、メールアドレス、クレジットカードが必要。
※クレジットカードを登録するが、特定のサービスを無料枠で利用することが可能。

### AWSアカウントを作成

上記サイトからアカウントをサインアップ。

### 請求アラートの作成

請求が発生する場合に、登録したメールアドレスへ通知する設定。

https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/monitoring/monitor_estimated_charges_with_cloudwatch.html

### 作業ユーザーの作成

AWS アカウント作成時は、「ルートユーザー」にて操作を行っている状態である。
ルートユーザーは、基本的にアカウントの変更・解約・サポートプランの変更などをする際に使用するが、普段は使用しない。

通常は、作業用のユーザー(IAM ユーザー)を利用する。
IAM Identity Center にて、アカウントを管理することも可能。

https://docs.aws.amazon.com/ja_jp/IAM/latest/UserGuide/introduction_identity-management.html#intro-identity-users

上記のドキュメントによると、
IAM は、AWS リソースに長期的な認証情報が付与され、
IAM Identity Center には、AWS リソースに短期間の認証が付与される。

一元的にアクセス管理するには、IAM Identity Center を利用することを勧めている。

## VPCでネットワークを構築する

### 用語

VPC：Virtual Private Cloud. ユーザー専用のプライベートなクラウド環境。

- サブネット：VPC 内を細かく区切ったネットワーク
  - パブリックサブネット：インターネットに公開されたネットワーク
  - プライベートサブネット：インターネットには非公開、特定のサーバーからの通信のみを許可したネットワーク

```text
サブネット例）
Web サーバーは誰でもアクセスできるように公開したい → パブリックサブネット
DB サーバーは公開せずに、Webサーバーからアクセスできる状態にしたい → プライベートサブネット
```

ルートテーブル：VPC 内（サブネット内）の通信を振り分けるルール一覧

### VPC構築の流れ

1. VPC を作成（IP アドレス例：10.0.0.0/16）
2. サブネットを作成
   1. パブリックサブネットを作成（IP アドレス例：10.0.10.0/24）
   2. プライベートサブネットを作成（IP アドレス例：10.0.20.0/24）
3. インターネットゲートウェイを作成
   1. 作成した VPC にインターネットゲートウェイをアタッチ
4. ルートテーブルを作成
   1. サブネットの関連付けにて、パブリックサブネットを関連付ける
   2. ルートの編集から、送信先（0.0.0.0/0）をターゲット：インターネットゲートウェイに指定する

## EC2(Webサーバー)を構築する

### 用語

EC2：Elastic Compute Cloud. AWS クラウド上の仮想サーバー
インスタンス：EC2 から立てられたサーバーのこと

- EBS：Elastic Block Store
  - OS や DB などの永続性と耐久性が必要なデータを置く
  - Snapshot を取得し S3 に保存できる
- インスタンスストア
  - インスタンス専用の一時的なストレージ

基本的には EBS を利用することが多い。

### EC2構築の流れ

1. AMI：Amazon Machine Image の選択
2. インスタンスタンプ（例：m5.xlarge）の選択
3. SSH キーペアを設定
4. セキュリティグループを設定
5. ストレージを設定

### EC2へsshログイン後の作業

yum で管理されているパッケージを update

```shell
[ec2-user@ip-xx-x-xx-xx ~]$ sudo yum update -y
```

apache を install

```shell
[ec2-user@ip-xx-x-xx-xx ~]$ sudo yum -y install httpd
```

apache を起動する

```shell
[ec2-user@ip-xx-x-xx-xx ~]$ sudo systemctl start httpd.service
```

apache の起動確認

```shell
[ec2-user@ip-xx-x-xx-xx ~]$ sudo systemctl status httpd.service
```

apache の自動起動設定

```shell
[ec2-user@ip-xx-x-xx-xx ~]$ sudo systemctl enable httpd.service
```

自動起動設定確認

```shell
[ec2-user@ip-xx-x-xx-xx ~]$ sudo systemctl is-enabled httpd.service
enabled
```

### ファイアウォールを設定

1. EC2 インスタンスのセキュリティタブからセキュリティグループを表示
2. インバウンドのルールを編集し、タイプ：HTTP を追加

### パブリックIPアドレスを固定する

Elastic IP アドレス：インターネット経由でアクセス可能な固定グローバル IP アドレスを取得でき、インスタンスに付与できるサービス。

**注意！！**
Elastic IP アドレスは、「EC2 インスタンスに関連付けられている」かつ「インスタンスが起動中」であれば無料なので、EC2 インスタンスを停止したり、紐づけをしていない場合は、IP アドレスを解放したほうが良い。

## Route53にドメインを登録する

### 用語

DNS：Domain Name System. ドメインとサーバー（IP アドレス）を DNS で紐づけるもの。
Route53 は、AWS の DNS システムのこと。

### ドメインを登録する手順

1. お名前ドットコムでドメインを購入する
2. Route53 でホストゾーンを作成
   1. お名前ドットコムで購入したドメイン名を入力
3. ドメインのネームサーバーを Route53 に変更する
   1. お名前ドットコムページから対象のドメインのネームサーバーを変更する
   2. Route53 にある NS レコードのネームサーバーをすべて登録する
4. ドメインに紐づく IP アドレスを登録
   1. Route53 のホストゾーンから A レコードを追加する

下記のコマンドで、ドメインに紐づくネームサーバーの変更が適用されたかを確認できる。

```shell
dig sample.com NS +short
```

## RDSでDBサーバーを構築する

### 構築手順

1. プライベートサブネットの追加（IP アドレス例：10.0.20.0/24）
2. セキュリティグループの作成
   1. web サーバーからの mysql へのアクセスのみを許可
3. サブネットグループの作成
   1. 作成した 2 つのプライベートサブネットを追加
4. パラメータグループ (RDS の設定項目) の作成
5. オプショングループの作成
6. RDS のデータベースから作成

### WebサーバーからMysqlにアクセスを確認

作成した EC2 インスタンスにアクセスし、下記のコマンドで作成した DB に接続ができるか確認する。

```shell
mysql -h DBへのエンドポイント -u 設定したUser名 -p
```

# WordPressを構築してみる

## DBを準備

EC2 インスタンスへアクセスし、DB へ接続。
作業用の DB（以下では wordpress_db とする）を作成する。

```shell
[ec2-user@ip-xx-x-xx-xx ~]$ mysql -h DBへのエンドポイント -u 設定したUser名 -p

mysql> CREATE DATABASE wordpress_db DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
Query OK, 1 row affected, 2 warnings (0.04 sec)

mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| wordpress_db       |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.01 sec)

mysql>
```

ユーザー（以下では、wordpress_user とする）を作成し、権限の付与する。

```shell
mysql> CREATE USER 'wordpress_user'@'%' IDENTIFIED BY 'password';
Query OK, 0 rows affected (0.02 sec)

mysql> GRANT ALL ON wordpress_db.* TO 'wordpress_user'@'%';
Query OK, 0 rows affected (0.01 sec)

mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.01 sec)

mysql> SELECT user, host FROM mysql.user;
+--------------------+-----------+
| user               | host      |
+--------------------+-----------+
| wordpress_user     | %         |
| rds_superuser_role | %         |
| root               | %         |
| mysql.infoschema   | localhost |
| mysql.session      | localhost |
| mysql.sys          | localhost |
| rdsadmin           | localhost |
+--------------------+-----------+
7 rows in set (0.00 sec)
```

## WordPressを導入

### PHPをインストール

今回は、php 8.1 系で構築してみた。

```shell
[ec2-user@ip-xx-x-xx-xx ~]$ sudo yum install -y php
```

```shell
[ec2-user@ip-xx-x-xx-xx ~]$ sudo yum -y install php-mysqli
```

### WordPressをインストールして設定

WordPress をインストール

```shell
[ec2-user@ip-xx-x-xx-xx ~]$ wget https://ja.wordpress.org/latest-ja.tar.gz
[ec2-user@ip-xx-x-xx-xx ~]$ tar xzvf latest-ja.tar.gz
```

apache が参照しているディレクトリに移動して、権限を変更。
apache を再起動する。

```shell
[ec2-user@ip-xx-x-xx-xx ~]$ cd wordpress/
[ec2-user@ip-xx-x-xx-xx ~]$ sudo cp -r * /var/www/html/

[ec2-user@ip-xx-x-xx-xx ~]$ sudo chown apache:apache /var/www/html/ -R

[ec2-user@ip-xx-x-xx-xx ~]$ sudo systemctl restart httpd.service
```

その後は、お名前ドットコムで購入したドメインに、ブラウザでアクセスし、初期設定を実施。

## S3バケットに画像を保存する

1. 新しく WordPress 用の S3 バケットを作成
2. IAM を作成し、S3 の Full Access 権限をアタッチ
3. IAM のアクセスキーを作成

WordPress でプラグイン「WP Offload Media Lite」をインストールし有効化する。

（上記プラグインに説明があるが、）下記のファイルに IAM で作成したアクセスキーを設定する。

```php:/var/www/html/wp-config.php
define( 'AS3CF_SETTINGS', serialize( array(
 'provider' => 'aws',
 'access-key-id' => '********************',
 'secret-access-key' => '**************************************',
) ) );
```

プラグイン設定から、S3 バケットを選択し、server 側には保存されないように設定変更。

## CloudFront経由でコンテンツを配信する

CloudFront は高速にコンテンツを配信する CDN：Content Delivery Network のサービス。

1. ディストリビューションを作成
2. SSL 証明書を作成
3. CloudFront の独自ドメインを設定
4. Route53 に CloudFront のドメイン名と独自ドメインの紐づけを追加する
5. WP Offload Media Lite の Delivery Settings を CloudFront に変更し、Custom Domain Name を設定する

# 安定した運用に向けて

可用性：システムが継続して稼働できる能力。可用性を上げるには稼働率を上げる必要がある。

稼働率を上げるには？

- 障害発生間隔を長くする
- 平均復旧時間を短くする

上記を達成するために、「冗長化」を行う必要がある。

具体的には、、

- 要素単体の稼働率を高くする
- 要素を組み合わせて、全体の稼働率を高くする
  - 冗長化構成
    - Active-Active：冗長化した両方が利用可能
    - Active-Standby：冗長化した片方は利用不可
      - Hot Standby：スタンバイ側は普段起動しすぐに利用可能
      - Warm Standby：スタンバイ側は普段起動しているが、利用するのに準備が必要
      - Cold Standby：スタンバイ側は普段停止している
- 負荷を適切なプロビジョニングで回避する
  - スケールアップ：サーバーの CPU やメモリなどの増設によって処理能力を高める
  - スケールアウト：サーバーのぢ亜数を増やして分散処理する

## ELBでWebレイヤを冗長化する

### 用語

ロードバランサー：各サーバーにアクセスを振り分けて、負荷を分散させるもの
ELB：Elastic Load Bracer. AWS クラウド上のロードバランサー

### 構築手順

1. AMI から EC2 を起動
    1. public サブネットを追加
    2. 既存の EC2 インスタンスから AMI を作成
    3. 新しい public サブネットに、作成した AMI を使って EC2 インスタンスを追加
2. ELB を追加
    1. セキュリティグループを追加
    2. ターゲットグループを作成
3. DNS を変更
    1. A レコードをエイリアスにして、作成した ALB を選択する

## DBレイヤの冗長化

マスタースレーブ構成にする

### 構築手順

データベースの設定からマルチ AZ を「あり」に変更する

## CloudWatchでシステム監視をする

- 死活監視
  - 正常にシステムが動作しているかを確認
- メトリクス監視
  - パフォーマンスを定量的に確認
  - 指標を決めて、指標が閾値以上／以下を確認

Amazon SNS を使えば、メールなどで通知することができる。

### 構築手順

CloudWatch で、アラームを作成する

1. EC2 を選択
2. 指定のインスタンスの監視したいメトリクスを選択
3. 閾値を設定
4. SNS トピックを作成
5. アラーム名を設定

# AWS無料枠への対応

作成、登録した AWS のサービスをそのまま放置しておくと、無料枠を超えて課金対象になってしまうので、私のほうで取った対応を書き留めておきます。

- 冗長化した EC2 インスタンスは削除しておく（無料枠の使用時間が、稼働台数分消費されちゃうので）
- 特に構築した WordPress を使わないのであれば、EC2 インスタンスも停止する
  - EC2 インスタンスを停止したら、紐づけている Elastic IP を解放する
- 冗長化した DB は、マスタースレーブ構成をやめて、元に戻しておく
- CloudTrail でログを S3 に保存している場合は、止めておく

# 学習を通しての感想

今まで、AWS の各サービスの一部を、アプリケーション側の SDK を使って連携したり、保守運用目的でコンソール画面を眺めることはありましたが、実際に構築したこともなく、どのサービスがどのような役割でどうやって繋がっているのかがわからない状態でしたが、ところどころの知識はあった分、「実際に手を動かして作ってみる」ことで、各サービスのイメージがつかみやすくて、点と点が線となって繋がったように理解ができたかなと思います。
AWS の無料枠の範囲内で、実際に構築するだけで、おおよその仕組みは理解できるとともに、様々なサービスのインフラ構成はどのような要件をどのような工夫で構築されているのかが、気になるようにはなりました。

これからも実際の開発経験や AWS の資格の勉強などを通して、インフラに関する知見を溜めたいなと思います。

# 参考教材

https://www.udemy.com/course/aws-and-infra

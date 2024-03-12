## 概要
　本ドキュメントは、AWSとGCP間のVPN Connectionを構築するためのマニュアルです。<br/>
以下の通り、VPN ConnectionでAWS VGW～GCP VGW間を接続します。<br/>

![vpn-connection-between-aws-and-gcp](https://github.com/yamamototis1105/vpn-connection-between-aws-and-gcp/assets/114621183/7b65c8f8-d777-4987-8615-2ea936a0b127)

## 利用方法
### AWSインスタンス作成
1. VPC作成
   * 名前「sample-vpc」、IPv4 CIDRブロックは「10.0.0.0/24」で作成。
1. サブネット作成 
   * 名前「sample-subnet」、アベイラビリティ―ゾーンは「ap-northeast-1a」、IPv4 CIDRブロックは「10.0.0.0/24」で作成。 
1. インターネットゲートウェイ作成 
   * 名前「sample-igw」で作成。 
   * VPCへアタッチ。 
1. ルートテーブル更新 
   * 宛先「0.0.0.0/0」、ネクストホップ「(インターネットゲートウェイのID)」を追加。 
1. 仮想プライベートゲートウェイ作成 
   * 名前「sample-vgw」で作成。 
   * VPCへアタッチ。 
1. カスタマゲートウェイ×2作成 
   * 名前「sample-cgw」、ルーティング「動的」、BGP ASN「65001」、IPアドレス「(GCP側のアドレス)」で作成。 
1. サイト間VPN接続×2作成 
   * 名前「sanmple-vpn」、ターゲットゲートウェイタイプ「仮想プライベートゲートウェイ」で作成。 <br/>
※事前共有鍵、ピアアドレスは予め設計しとくべき。自動の場合、設定をダウンロードのうえ確認する必要あり。 
   * Tunnel1、Tunnel2のピアアドレスを確認。 
1. ルートテーブル更新 
   * 宛先「10.1.0.0/24」、ネクストホップ「(仮想プライベートゲートウェイのID)」を追加。 
1. セキュリティグループ作成 
   * セキュリティグループ名「sample-sg」、説明「sample-sg」で作成。 
   * プロトコル「すべてのICMP」、送信元「0.0.0.0/0」を追加。 
1. キーペア作成 
   * 名前「sample-kp」で作成。 
1. EC2インスタンス作成 
   * AMI「Amazon Linux」、インスタンスタイプ「t2 medium」、 ネットワーク「sample-vpc」、サブネット「sample-subnet」、自動割り当てパブリックIP「有効」、タグ「Name: sample-ec2」、セキュリティグループ「(セキュリティグループのID)」、キーペア「(キーペアのID)」で作成。 
1. SSH用の秘密鍵・共通鍵を作成 
   * ssh-keygen -t rsa 
   * sudo chmod 600 .ssh/id_rsa 
1. ping実行 
   * nohup ping 10.0.0.4 -c 86400 > result & 。
<br/>

### GCPインスタンス作成
1. VPCネットワーク作成 
   * 名前「sample-vpc」で作成。 
1. サブネット作成 
   * 名前「sample-subnet」、リージョン「asia-northeast1」、IPアドレス範囲「10.0.1.0/24」で作成。 
1. ファイアウォール作成 
   * 名前「sample-fw」、タグ「sample」で作成。 
   * プロトコル/ポート「tcp/22」、「icmp/-」を追加。 
1. CloudRouter作成 
   * 名前「sample-router」、ネットワーク「sample-vpc」、リージョン「asia-northeast1」、Google ASN「65001」で作成。 
1. VPN作成 
   * VPNオプション「HA VPN」で作成。 
   * Cloud HA VPNゲートウェイ作成 
   * 名前「sample-vgw」、ネットワーク「sample-vpc」、リージョン「asia-northeast1」、 
   * インターフェース: 0、インターフェース: 1のアドレスを確認。 
   * ピアVPNゲートウェイ作成 
   * 名前「sample-pvgw」、インタフェース「4つ」、インタフェース0～3「(AWS側のアドレス)」で作成。 
   * VPNトンネル作成 
   * 高可用性「4つのVPNトンネルを作成」、名前「sample-tunnel」で作成。 
   * BGPセッション作成 
   * 名前「sample-bgp」、ピアASN「64512」、Cloud RouterのBGP IP「(GCP側のアドレス)」 
1. ルートの更新 
   * (不要) <br/>※BGPによって自動学習 
1. VMインスタンス作成 
   * 名前「sample-vm」、リージョン「asia-northeast1」、ネットワークタグ「sample」、SSH認証鍵「(id_rsa.pub貼り付け)」で作成。 

## 関連技術
<img src="https://img.shields.io/badge/AWS-Site_to_Site_VPN-orange"></img> <img src="https://img.shields.io/badge/GCP-HA_VPN-blue"></img>

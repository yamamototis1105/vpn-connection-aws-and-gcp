## 概要
　本ドキュメントは、AWSとGCP間のVPN Connectionを構築するためのマニュアルです。<br/>
以下の通り、VPN ConnectionでAWS VGW～GCP VGW間を接続します。<br/>

![vpn-connection-between-aws-and-gcp](https://github.com/yamamototis1105/vpn-connection-between-aws-and-gcp/assets/114621183/0644cc78-4921-4884-aa80-45283d340480)

## 利用方法
### AWSインスタンス作成
1. VPC作成
   * 名前「sample-vpc」、IPv4 CIDRブロック「10.0.0.0/24」で作成。
1. サブネット作成 
   * VPC「sample-vpc」、名前「sample-subnet」、アベイラビリティ―ゾーン「ap-northeast-1a」、IPv4 CIDRブロックは「10.0.0.0/24」で作成。 
1. 仮想プライベートゲートウェイ作成 
   * 名前「sample-vgw」、ASN「AmazonデフォルトASN」で作成。 
   * VPC「sample-vpc」へアタッチ。 ※GCPの1へ
1. カスタマゲートウェイ×2作成 
   * 名前「sample-cgw」、ルーティング「動的」、BGP ASN「65001」、IPアドレス「(GCP側のアドレス)」で作成。 
1. サイト間VPN接続×2作成 
   * 名前「sample-vpn」、ターゲットゲートウェイタイプ「仮想プライベートゲートウェイ」、トンネル1の内部IPv4 CIDR「169.254.6.0/30」、トンネル1の事前共有キー「samplekey」、トンネル2の内部IPv4 CIDR「169.254.6.4/30」、トンネル1の事前共有キー「samplekey」、で作成。
   * Tunnel1、Tunnel2のピアアドレスを確認。 ※GCPの5へ
<br/>

### GCPインスタンス作成
1. VPCネットワーク作成 
   * 名前「sample-vpc」で作成。 
1. サブネット作成 
   * 名前「sample-subnet」、リージョン「asia-northeast1」、IPアドレス範囲「10.0.1.0/24」、動的ルーティング モード「グローバル」で作成。 
1. ファイアウォール作成 
   * 名前「sample-fw」、ネットワーク「sample-vpc」、ターゲットタグ「sample」、送信元IPv4範囲「0.0.0.0/0」、プロトコルとポート「その他/icmp」で作成。
1. CloudRouter作成 
   * 名前「sample-router」、ネットワーク「sample-vpc」、リージョン「asia-northeast1」、Google ASN「65000」で作成。 
1. VPN作成 
   * VPNオプション「HA VPN」で作成。 
   * Cloud HA VPNゲートウェイ作成 
     * 名前「sample-vgw」、ネットワーク「sample-vpc」、リージョン「asia-northeast1」で作成。
     * インターフェース: 0、インターフェース: 1のアドレスを確認。 ※AWSの4へ
   * ピアVPNゲートウェイ作成 
     * 名前「sample-pvgw」、インタフェース「4つ」、インタフェース0～3「(AWS側のアドレス)」で作成。 
   * VPNトンネル作成 
     * 高可用性「4つのVPNトンネルを作成」、名前「sample-tunnel」、IKE事前共有キー「samplekey」で作成。 
   * BGPセッション作成 
     * 名前「sample-bgp」、ピアASN「64512」、Cloud RouterのBGP IP「169.254.6.2」 、BGPピアIPv4アドレス「169.254.6.1」

## 関連技術
<img src="https://img.shields.io/badge/AWS-Site_to_Site_VPN-orange"></img> <img src="https://img.shields.io/badge/GCP-HA_VPN-blue"></img>

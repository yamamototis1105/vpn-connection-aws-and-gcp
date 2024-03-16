## 概要
　本ドキュメントは、AWSとGCP間のVPN Connectionを構築するためのマニュアルです。<br/>
以下の通り、VPN ConnectionでAWS VGW～GCP VGW間を接続します。<br/>

![](/images/vpn-connection-between-aws-and-gcp.png)

## 利用方法
### AWS作成
1. VPC作成
   * 名前「sample-vpc」、IPv4 CIDRブロック「10.0.0.0/24」で作成。
   * 名前「sample-subnet」、アベイラビリティ―ゾーン「ap-northeast-1a」、IPv4 CIDRブロックは「10.0.0.0/24」で作成。 
1. 仮想プライベートゲートウェイ作成 
   * 名前「sample-vgw」、ASN「AmazonデフォルトASN(64512)」で作成。 
   * VPC「sample-vpc」へアタッチ。 ※「GCP作成」の「1.VPCネットワーク作成」へ
1. カスタマゲートウェイ作成 
   * 名前「sample-cgw」、ルーティング「動的」、BGP ASN「65000」、IPアドレス「(GCP側のアドレス1)」で作成。 
   * 名前「sample-cgw」、ルーティング「動的」、BGP ASN「65000」、IPアドレス「(GCP側のアドレス2)」で作成。 
1. サイト間VPN接続作成 
   * 名前「sample-vpn1」、ターゲットゲートウェイタイプ「仮想プライベートゲートウェイ」、トンネル1の内部IPv4 CIDR「169.254.6.0/30」、トンネル1の事前共有キー「samplekey」、トンネル2の内部IPv4 CIDR「169.254.6.4/30」、トンネル1の事前共有キー「samplekey」、で作成。
   * 名前「sample-vpn2」、ターゲットゲートウェイタイプ「仮想プライベートゲートウェイ」、トンネル1の内部IPv4 CIDR「169.254.7.0/30」、トンネル1の事前共有キー「samplekey」、トンネル2の内部IPv4 CIDR「169.254.7.4/30」、トンネル1の事前共有キー「samplekey」、で作成。
   * Tunnel1、Tunnel2のピアアドレスを確認。 ※「GCP作成」の「4.VPN作成 > ピアVPNゲートウェイ作成」へ
<br/>

### GCP作成
1. VPCネットワーク作成 
   * 名前「sample-vpc」で作成。 
   * 名前「sample-subnet」、リージョン「asia-northeast1」、IPアドレス範囲「10.0.1.0/24」、動的ルーティング モード「グローバル」で作成。 
1. ファイアウォール作成 
   * 名前「sample-fw」、ネットワーク「sample-vpc」、ターゲットタグ「sample」、送信元IPv4範囲「0.0.0.0/0」、プロトコルとポート「その他/icmp」で作成。
1. CloudRouter作成 
   * 名前「sample-router」、ネットワーク「sample-vpc」、リージョン「asia-northeast1」、Google ASN「65000」で作成。 
1. VPN作成 
   * VPNオプション「HA VPN」で作成。 
   * Cloud VPNゲートウェイ作成 
     * 名前「sample-vgw」、ネットワーク「sample-vpc」、リージョン「asia-northeast1」で作成。
     * インターフェース: 0、インターフェース: 1のアドレスを確認。 ※「AWS作成」の「3.カスタマゲートウェイ」へ
   * ピアVPNゲートウェイ作成 
     * 名前「sample-pvgw」、インタフェース「4つ」、インタフェース0「(AWS側のアドレス)」、インタフェース1「(AWS側のアドレス)」、インタフェース2「(AWS側のアドレス)」、インタフェース3「(AWS側のアドレス)」で作成。 
   * VPNトンネル作成 
     * 高可用性「4つのVPNトンネルを作成」
     * 名前「sample-tunnel1」、IKE事前共有キー「samplekey」で作成。 
     * 名前「sample-tunnel2」、IKE事前共有キー「samplekey」で作成。 
     * 名前「sample-tunnel3」、IKE事前共有キー「samplekey」で作成。 
     * 名前「sample-tunnel4」、IKE事前共有キー「samplekey」で作成。 
   * BGPセッション作成 
     * 名前「sample-bgp1」、ピアASN「64512」、Cloud RouterのBGP IP「169.254.6.2」 、BGPピアIPv4アドレス「169.254.6.1」で作成。
     * 名前「sample-bgp2」、ピアASN「64512」、Cloud RouterのBGP IP「169.254.6.6」 、BGPピアIPv4アドレス「169.254.6.5」で作成。
     * 名前「sample-bgp3」、ピアASN「64512」、Cloud RouterのBGP IP「169.254.7.2」 、BGPピアIPv4アドレス「169.254.7.1」で作成。
     * 名前「sample-bgp4」、ピアASN「64512」、Cloud RouterのBGP IP「169.254.7.6」 、BGPピアIPv4アドレス「169.254.7.5」で作成。
<br/>

### AWS削除
1. サイト間VPN接続削除
   * 名前「sample-vpn1」を削除。
   * 名前「sample-vpn2」を削除。
1. カスタマゲートウェイ削除
   * 名前「sample-cgw1」を削除。
   * 名前「sample-cgw2」を削除。
1. 仮想プライベートゲートウェイ削除 
   * VPC「sample-vpc」からデタッチ。
   * 名前「sample-vgw」を削除。 
1. VPC削除
   * VPC「sample-vpc」を削除。
<br/>

### GCP削除
1. VPN削除
   * VPNトンネル削除 
     * 名前「sample-tunnel1」を削除。
     * 名前「sample-tunnel2」を削除。
     * 名前「sample-tunnel3」を削除。
     * 名前「sample-tunnel4」を削除。
   * ピアVPNゲートウェイ削除
     * 名前「sample-pvgw」を削除。
   * Cloud VPNゲートウェイ削除
     * 名前「sample-vgw」を削除
1. CloudRouter削除
   * 名前「sample-router」を削除。
1. ファイアウォール削除
   * 名前「sample-fw」を削除。 
1. VPC削除
   * VPC「sample-vpc」を削除。

## 関連技術
<img src="https://img.shields.io/badge/AWS-Site_to_Site_VPN-orange"></img> <img src="https://img.shields.io/badge/GCP-VPN-blue"></img>

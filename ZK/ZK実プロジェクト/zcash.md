<center>
  **Zcash: Bitcoin送金にプライバシーを**
</center>

Zcashは、Bitcoin Protocolのプライバシー問題を解決するためのプロトコルであり、Electric Coin Co.によって2016年の10月にローンチされました。
Bitcoin送金におけるプライバシーを実現するためにBitcoin Protocolの様々な仕様を追加しています。
では具体的に、どこを / どのように / なぜそのように 追加したのでしょうか？
この資料は、Zcashを理解する上で誰しもが抱くであろう上記の疑問を解消することを目指して書かれています。
尚、取り扱っているバージョンは[Sprout](https://github.com/zcash/zips/blob/main/protocol/sprout.pdf)になります。

`Writer: @ashWhiteHat`

`Donated by: `

- [Zcashとは何か？](#zcashとは何か)
    - [プライバシー送金が可能なBitcoin Protocolである](#プライバシー送金が可能なBitcoin Protocolである)
- [プライバシーについて](#プライバシーについて)
- [アーキテクチャー](#アーキテクチャー)
- [要素技術](#要素技術)
- [まとめ](#まとめ)

# Zcashとは何か？

## 匿名送金が可能なBitcoin Protocolである

匿名送金: Bitcoin Protocolの課題

## 2種類のプライバシー

- ブロックチェーン業界におけるプライバシーには、一般的に`Confidential`と`Anonoymous`の2つの種類が存在する

**Confidential**
- 送金額が秘匿化されている

**Anonymous**
- 送金額が秘匿化されている
- 送金に関連するアカウントが秘匿化されている

# Zcashの革新性

## 匿名送金の課題

- Bitcoin Protocolの匿名性に取り組んでいるプロジェクトはこれまでにも存在していた

**ミキサー**
- 集権サービスを利用することで送金の追跡を不可能にする
- この方法には以下のような欠点が存在する
1. ミキシングに必要な量のコインを集めるのに時間がかかる
2. ミキシングの提供者は送金を追跡できる
3. ミキシングの提供者がコインを盗む可能性がある

**Zerocoin**
- ゼロ知識証明により集権サービスを必要としない匿名送金を実現した
- この方法には以下のような欠点が存在する
1. 証明サイズが45kBで検証に450msの時間がかかる
2. 固定額の送金しかできない
3. 直接送金する方法がない
4. 送金額や取引のメタデータは秘匿していない

## Zcashの貢献

- 従来の匿名送金では
1. 集権サービスを用いた匿名送金
2. 分散で少ない機能性で低いスケーラビリティの匿名送金
- の2つの方法しか存在しなかった

- Zcashでは、[匿名送金の課題](#匿名送金の課題)を分散性を保ちながら以下のように改善した
1. トランザクションサイズを1kB以下にする(97.7%の改善)
2. 検証時間を6ms以下にする(98.6%の改善)
3. あらゆる送金量での匿名送金を可能にする
4. 送金額とユーザーの保有するコインの価値を秘匿する
5. ユーザーの固定アドレスに直接送金される

# プロトコル

## 概要

ZcashはBitcoin ProtocolのUTXOの管理方法を変更し、匿名送金を実現している。
Zcashの特性を一言で表すと送金者が使用したUTXOが識別できないことである。

変更されている箇所は以下の通り
- UTXOを生データで管理するのではなくそのコミットメントを管理する
- UTXOの送金の際のトランザクションの情報をゼロ知識証明で隠蔽
- UTXOの二重支払いの防止にNullifierという文字列を用いている

## 用語集

**鍵管理**
- Public Key: Paying KeyとTransmission Keyで構成される
- Paying Key: Bitcoinのアドレスにあたるもの(の一部)
- Spending Key: Bitcoinの秘密鍵にあたるもの

**トランザクション**
- Note: UTXOの残高情報にあたるもの
- Note Commitment: UTXOの残高情報にあたるものの暗号学的コミットメント
- JoinSplit: 匿名送金の命題に対するゼロ知識証明の証明
- Nullifier: 二重支払い防止に用いられている文字列

# 要素技術

- アーキテクチャーの中で用いられている要素技術の概要と役割を解説する

**Note Commitment**

**Nullifier**
- 二重支払い防止のために用いられる文字列
- 送金時に入力として用いられたNoteのNullifierが公開される

# まとめ

# 参照

- [Learn Zcash](https://z.cash/learn/who-created-zcash/)
- [A Privacy-Protecting Digital Currency Built On Strong Science.](https://www.binance.com/ja/research/projects/zcash)
- [匿名暗号資産（Monero/Zcash/Grin）ブロックチェーンの匿名性に関する考察](https://cir.nii.ac.jp/crid/1050292572094389888)
- [Zcash Protocol Specification Version 2021.1.19 [Sprout]](https://github.com/zcash/zips/blob/main/protocol/sprout.pdf)

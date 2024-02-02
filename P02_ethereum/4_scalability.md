- [ガバナンスとEIP(Ethereum Improvement Proporsal)](#ガバナンスとeipethereum-improvement-proporsal)
  - [どのように仕様変更するのか？](#どのように仕様変更するのか)
  - [いくつかのEIP紹介](#いくつかのeip紹介)
- [スケーラビリティ問題](#スケーラビリティ問題)
  - [The Merge](#the-merge)
  - [Layer2](#layer2)
  - [ブロックチェーンのトリレンマ](#ブロックチェーンのトリレンマ)
- [再びEthereumとは何か？](#再びethereumとは何か)


# ガバナンスとEIP(Ethereum Improvement Proporsal)
## どのように仕様変更するのか？
- Ethereumの仕様変更は、Bitcoinと同様、EIP (Ethereum Improvement Proposal) に基づく
- オープンソースソフトウェア開発と同様、Githubや掲示板、メーリングリストなどで議論される
- とはいえ新たな仕様は各ノードがそれぞれの意志で反映させることになる
- コンセンサスが取れない場合、最終的にブロックチェーンは分岐し市場の判断に委ねられる

e.g., 
EthereumにおけるThe DAOハッキング事件 (2016)
- Ethereum
- Ethereum Classic

BitcoinにおけるSegwit (Segregated Witness) 機能の実装 (2017)
- Bitcoin
- Bitcoin Cash
  - Bitcoin Cash ABC
- Bitcoin Gold
- Bitcoin Diamond
- …

EthereumにおけるThe Merge (2022)
- Ethereum
- Ethereum PoW


## いくつかのEIP紹介
EIPの一覧は https://eips.ethereum.org/all で確認可能
- 仕様変更の提案 (Core) のみならず、プログラマへの呼びかけ (ERC; Ethereum Request for Comments) も存在する

ERCに分類されるEIPには、例えば以下が存在する

EIP20, EIP721, EIP1155: 異なるEOAが作成したスマートコントラクトを跨げるように、トークンの規格を設定した

- EIP20
  - 1つ1つがユニークではないトークン (Fungible Token) の規格を設定した
  - e.g.,投票用に総発行量1,000枚の伊東トークンを発行する
- EIP721 
  - 1つ1つがユニークなトークン (NFT; Non-Fungible Token) の規格を設定した
  - e.g., 1000種類の異なる画像と紐付けられた伊東トークンを発行する
- EIP1155
  - 条件を満たすとユニークになるトークン (Semi-Fungible Token) の規格を設定した
  - e.g., 投票後にNFTへと変わる1000枚の伊東トークンを発行する

これらはEthereumの仕様変更ではなく、利便性のために書式を統一しよう、という呼びかけ

Coreに分類されるEIPには、例えば以下が存在する

EIP86, EIP2938 (未実装, 現在検討中)
- Account Abstractionの提案
- EOAとCAの区別を無くして、全てのアカウントをCAに統一する (！)
- 署名は、トランザクションのデータ領域であるinputにコントラクトとして記載する
- これにより、楕円曲線暗号以外の署名も使用出来る
- これにより、署名無しのトランザクションも必要に応じて作成出来る

このように、Bitcoinと比較して根本的な仕様変更が積極的に議論・実装されている印象がある


# スケーラビリティ問題

ブロックチェーンの文脈におけるスケーラビリティ問題とは、分散的な合意形成の処理がトランザクションの増加に追いつかなくなる問題を意味する。これはBitcoinと共通する課題である

TPS (transaction per second)
- Bitcoin: 5
- Ethereum (PoW): 10-20
- VISAカード: 1500-2000

しかし「スマートコントラクト」「ワールドコンピューター」を謳う以上、Ethereumにとってスケーラビリティはより重要な問題である

## The Merge
The Mergeは、まさにこのスケーラビリティ問題への対処が主目的 (EIP2982, EIP3675などで議論された)
- PoWからPoS (Proof-of-Stake) への移行
  - マイニングを廃止。代わりにブロックを作成するノードを確率的に選択。その確率はノードが保持するEtherの量で重み付けされる (平均TPSが10%向上)
- 将来的なShardingの導入
  - トランザクションの検証やブロックの作成などの作業をノードが分担して行う (i.e., 並列処理) 

しかし10%程度にとどまらず、Ethreumはここからさらなる大幅アップデートを繰り返すことで 100,000 TPS (！) を目指している

## Layer2
加えて、スマートコントラクト部分をEthereumのブロックチェーンから一旦切り離して別の仕組みで実行、その結果だけを戻すという “Layer2” の提案も行われている
- Zero-knowledge rollups
- Optimistic rollups
- ...

Layer2は、専ら集権的な存在を置くがそいつが内容を改ざん出来ないような構造になっており、スケーラビリティ問題の解消とEOAが負担する手数料の引き下げを目指している


一方で 「スマートコントラクト部分を切り離すことは本末転倒であり、それならBitcoinを使えば良いのでは？」 という批判もある

また、分散的な合意形成自体を諦めてしまうことでスケーラビリティ問題と高い手数料に対処するブロックチェーンも登場している
- Binance Smart Chain (Ethereumのソースコードを転用している)
- Solana
- ...

## ブロックチェーンのトリレンマ
以下の要素のうち、2つまでしか同時に実現出来ない
- Scalability
- Security
- Decentralization


参考: Vitalik Buterinの発言

> “If Eth fails to scale, then Eth definitely failed. If Eth succeeds at scaling, but it turns into something that’s centralized, then I think it also failed. If Eth succeeds at scaling and decentralization, but nothing interesting gets built on top of it then it also fails.”

> “もしスケーリングに失敗したら、 Ethereumは失敗する。もしスケーリングに成功しても、そこに集権的な要素があればEthereumは失敗する。もしスケーリングと分散化に成功しても、そこで面白いものが作られなければEthereumは失敗する。” (拙訳)


# 再びEthereumとは何か？

端的に言えば、**Bitcoinの一般化**であった
- 処理の一般化: 送金 から プログラム へ
- 管理対象の一般化: Bitcoinの移転記録 から アカウントが持つ状態 (state) の遷移記録 へ

この一般化を計算機科学の文脈で捉えると...
- 電卓と (プログラム内蔵方式の) コンピューター
- Ethereumは「ワールドコンピューター」である

これを実現するために、Bitcoinの様々な (本当に様々な！) 仕様を変更していた
- アカウント
- トランザクション
- ブロック
- ブロックチェーン
- コンセンサス
- etc…


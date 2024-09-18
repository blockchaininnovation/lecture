- [ガバナンスとEIP(Ethereum Improvement Proporsal)](#ガバナンスとeipethereum-improvement-proporsal)
- [スケーラビリティ問題](#スケーラビリティ問題)
- [再びEthereumとは何か？](#再びethereumとは何か)
  - [端的に言えば、**Bitcoin Protocolの一般化**であった](#端的に言えばbitcoin-protocolの一般化であった)
  - [この一般化を計算機科学の文脈で捉えると...](#この一般化を計算機科学の文脈で捉えると)
  - [これを実現するために、Bitcoin Protocolの様々な (本当に様々な！) 仕様を変更していた](#これを実現するためにbitcoin-protocolの様々な-本当に様々な-仕様を変更していた)
  - [スケーラビリティをどこまで改善させられるか](#スケーラビリティをどこまで改善させられるか)


# [ガバナンスとEIP(Ethereum Improvement Proporsal)](../P02_ethereum/4_scalability.md#ガバナンスとeipethereum-improvement-proporsal)
以前のEthereumと同様である。
補足として、スケーラビリティに関するEIPを紹介する

シャーディングに関するもの

# [スケーラビリティ問題](../P02_ethereum/4_scalability.md#スケーラビリティ問題)
以前のEthereumと同様である。
補足として、今後のアップデート予定を紹介する
補足として、data availabilityについても説明する

# 再びEthereumとは何か？

## 端的に言えば、**Bitcoin Protocolの一般化**であった
- 処理の一般化: 送金 から プログラム へ
- 管理対象の一般化: Bitcoin Protocolの移転記録 から アカウントが持つ状態 (state) の遷移記録 へ

## この一般化を計算機科学の文脈で捉えると...
- 電卓から (プログラム内蔵方式の) コンピューターへ
- Ethereumは「ワールドコンピューター」である

## これを実現するために、Bitcoin Protocolの様々な (本当に様々な！) 仕様を変更していた
- アカウント
- トランザクション
- ブロック
- ブロックチェーン
- コンセンサス
- etc…

## スケーラビリティをどこまで改善させられるか
- トランザクションが捌ききれなければ、自律分散的な「ワールドコンピューター」の夢は実現したとはいえない
- スケーラビリティ向上のために、プロトコルのアップデート(The Merge)やLayer2アプローチを試みている



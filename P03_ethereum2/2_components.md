- [各構成要素が保持するデータ](#各構成要素が保持するデータ)
  - [バリデータが保持するデータ](#バリデータが保持するデータ)
- [まとめ](#まとめ)


yellowmagic33
Pass: x3C594HBVoPG

# 各構成要素が保持するデータ

各構成要素が保持するデータは、以前のEthereumと同様である

- [EOA (Externally Owned Account)](../P02_ethereum/2_components.md#eoa-externally-owned-account)
- [CA (Contract Account)](../P02_ethereum/2_components.md#ca-contract-account)
- [Message Call トランザクションのデータ構造](../P02_ethereum/2_components.md#message-call-トランザクションのデータ構造)
- [Ethereumの手数料 (gas)](../P02_ethereum/2_components.md#ethereumの手数料-gas)
- [Message Call トランザクションのレシート](../P02_ethereum/2_components.md#message-call-トランザクションのレシート)
- [Contract Creation トランザクションのデータ構造](../P02_ethereum/2_components.md#contract-creation-トランザクションのデータ構造)
- [Contract Creation トランザクションのレシート](../P02_ethereum/2_components.md#contract-creation-トランザクションのレシート)
- [データ構造を実際に確認してみましょう](../P02_ethereum/2_components.md#データ構造を実際に確認してみましょう)


## バリデータが保持するデータ

- 各バリデータは、アカウント用の鍵とは別の**Validator Signing Key**と**Withdrawal Key**の2種類 (それぞれ秘密・公開鍵から成る) で構成されている
  - `なぜ？: セキュリティ上、アカウント用の鍵とバリデータ用の鍵は分けておきたいから`
  - `なぜ？: 楕円曲線暗号よりも署名の集約に適した暗号形式 (BLS署名) を用いたいから`
- Validator Signing Key: ブロックの提案・投票における署名
  - 公開鍵 バリデータのID的なものとして使われる
  - 秘密鍵 これを持つ者だけがブロックの提案・投票を行える
- Withdrawal Key: deposit contractからのether引き出し
  - 公開鍵 バリデータのウォレットアドレス的なものとして使われる
  - 秘密鍵 これを持つ者だけがstakeしたetherを引き出せる

<center>
<img src="./img/validator.png" width="70%">

Source: https://ethereum.org/en/developers/docs/consensus-mechanisms/pos/keys/
</center>

> 2024年4月において、バリデータの数は約1,000,000個

- 具体的に、バリデータは以下のプロセスで作成される
  - EOAの所有者がWithdrawal Keyのペアを作成する
  - EOAがdeposit contractに32etherを送るトランザクションのinput部分にWithdrawal Keyの公開鍵情報を埋め込む (秘密鍵の情報は手元に残す)
  - deposit contractが32etherにバリデータインデックスを割り当てる
- 2種類の鍵はConsesus Clientが管理、Conseus Layerで管理、手元で管理
- バリデータ用の鍵を別途作ることにより、次のような運用が可能になる
  - Validator Signing Keyをノードに渡すことで、Stakingを委任できる。この場合、自身でConsensus Clientやフルノードを稼働させずともStakingが行える (Staking as a Service; SaaS)
【キャスレー：素人目線ですが、Stakingを委任した場合では、Consensus Clientやフルノードを稼働させずにStakingを行うことができ、バリデータを自分で管理しなくてよくなるので、Solo StakingよりもSaaSで運用する方が良いのではないかと感じてしまいました。Solo Stakingで運用したほうが良い場合はどんな時でしょうか？】
  - また、そもそもバリデータ自体を自分で用意せずにStakingを行うことも出来る。つまり、Bitcoin Protocolにおけるマイニングプールのように少額のetherを集約して共用のバリデータ (Staking Pool) を運用することが可能

Solo Staking, SaaS, Staking Poolの3つの運用方法について、それぞれの特徴は以下のとおり
  
|  | 自身で用意するもの | 自身で作成するもの | 自身で管理するもの | トラストへの依存 |
| ---- | ---- | ---- | ---- | ---- |
| Solo Staking | full (or archive) node, EOA, 32ether以上のether | バリデータ | バリデータ | 低
| Staking as a Service (SaaS) | EOA, 32ether以上のether | バリデータ |  | 中
| Staking Pool | EOA, 任意量のether | |  | 高


# まとめ
- 各構成要素が保持するデータは、以前のEthereumと同様である
- バリデータが保持するデータにはValidator Signing KeyとWithdrawal Keyがある
- しかしこれらの構成要素がどう連動するかについて説明する次の資料からは、内容が大きく異なりはじめる

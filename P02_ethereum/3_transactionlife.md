- [トランザクションのライフサイクル](#トランザクションのライフサイクル)
  - [トランザクションの伝搬](#トランザクションの伝搬)
    - [Ethereumネットワークのノードについて](#ethereumネットワークのノードについて)
    - [Ethereumネットワークへの参加について](#ethereumネットワークへの参加について)
  - [独立したトランザクションの検証](#独立したトランザクションの検証)
    - [Intrinsic gasについて](#intrinsic-gasについて)
  - [EthereumのMempool](#ethereumのmempool)
  - [ブロックの作成](#ブロックの作成)
    - [Etherreumブロックのデータ構造](#etherreumブロックのデータ構造)
    - [ブロックヘッダの新要素](#ブロックヘッダの新要素)
    - [Uncle Blockについて](#uncle-blockについて)
    - [Block Gas Limitについて](#block-gas-limitについて)
    - [baseFeePerGasの決定ルールについて](#basefeepergasの決定ルールについて)
    - [State Root, Receipts Rootについて](#state-root-receipts-rootについて)
  - [トランザクションの実行](#トランザクションの実行)
  - [ブロックチェーンの構造](#ブロックチェーンの構造)
  - [EthereumのPoW (Proof-of-Work)](#ethereumのpow-proof-of-work)
    - [block intervalと難易度調整について](#block-intervalと難易度調整について)
  - [独立したブロックの検証](#独立したブロックの検証)
  - [コンセンサスとマイニング報酬](#コンセンサスとマイニング報酬)
    - [GHOSTプロトコルについて](#ghostプロトコルについて)
    - [EthererumのGHOSTプロトコルについて](#ethererumのghostプロトコルについて)
    - [Ethererumのマイニング報酬について](#ethererumのマイニング報酬について)
- [まとめ](#まとめ)


# トランザクションのライフサイクル
 
Ethereumの構成要素がどのように動くのか、トランザクションのライフサイクルを軸に詳細をみてみよう。

```
EOAは、トランザクション(Message Call or Contract Creation)を各ノードに伝搬する
各ノードは、受け取ったトランザクションを独立に検証する
各ノードは、問題が無いトランザクションのみを溜め、かつ他のノードに伝搬する
マイナーノードは、溜まりから任意のトランザクションをブロックに格納する
マイナーノードは、ブロック内のトランザクションを実行する
マイナーノードは、ブロックを既存のチェーンに含まれるいずれかのブロックに繋ぐ
マイナーノードは、PoW (Proof-of-Work) を経てブロックを完成させる
マイナーノードは、完成したブロックを各ノードに伝搬する
各ノードは、受け取ったブロックに問題が無いかを独立に検証する
各ノードは、問題が無いブロックのみを自身のチェーンに反映する
Ethereumは、最も重いチェーンを「正しい」状態遷移の記録とする
```

## トランザクションの伝搬

### Ethereumネットワークのノードについて
Ethereumネットワークのノードは、full node, light node, archive nodeの3種類に大別される
- Bitcoinは、full node, SPV (軽量) nodeの2種類に大別される
- ただしarchive nodeをfull nodeの派生版とみなす分類もある
- Ethereumにはブロックデータ以外に状態 (state) データというものが存在する (詳細は後述)

|  | 保持データ | 検証の対象 | マイニングおよびトランザクション実行 |
| ---- | ---- | ---- | ---- |
| full node | 全てのブロックデータ, 直近128ブロックの状態 (state) データ | トランザクション, ブロック | 可 |
| light node | ブロックヘッダ | ブロック (ブロックヘッダを使った検証のみ) | 不可 |
| archive node | 全てのブロックデータ, 全ての状態 (state) データ | トランザクション, ブロック | 可 (だがほぼ全てのマイナーはfull node) |


`なぜ？：マイニングに参加する障壁を下げることで、マイナーノードの分散化を図りたいから`
- full node: 550GB-1.1TB, archive node: 9.5TB (2022年1月時点)
- Bitcoinのfull node: 375GB

### Ethereumネットワークへの参加について
- トランザクションはfull nodeとarchive nodeへ伝搬される
- Bitcoinと同様に隣接するノードへの伝搬が繰り返される (gossip-based flooding protocol)

ただし手前の、最初にノードを建ててネットワークに参加する際の手続きがBitcoinとは異なる

- Ethereumでは、最初に接続するノードは所与の8個のノードの中から選択される
  - 自身のノードとIDが近いノードが複数個(デフォルトでは3つ)選択される
  - ネットワークのトポロジーに制約がある → Structured P2P network
- Bitcoinでは、最初に接続するノードを自分で設定することが出来る
  - 基本的に所与のノードに接続する (DNSシード) 形だが、オプションでこういうことも可能
  - ネットワークのトポロジーに制約がない → Unstructured P2P network

`なぜ？：ネットワークのトポロジーを、トランザクションやブロックの伝搬をより速く効率的に行える形にしておきたいから`


## 独立したトランザクションの検証

- 各ノードは、受け取ったトランザクションについて以下の内容を検証する
- 検証内容は基本的にビットコインと同様 (だがUTXOとアカウントベースという設計の違いが存在)

  - トランザクションがEthereumで採用されているRLPフォーマットという形式に沿っているか
    - RLPフォーマットの詳細な説明については時間の都合上割愛
    - 興味がある方はこちらの動画をご参照ください
  - トランザクションのgasLimitがintrinsic gas以上に設定されているか
    - 次で説明
  - 送信者(EOA) の残高が必要な額 gasLimit × feePerGas + value(送金するetherの量) 以上か
  - デジタル署名が有効か
  - トランザクションのnonceが、送信者(EOA)のnonceよりも大きいか？
    - 復習: トランザクションのnonceは、送信者(EOA)によるトランザクションがこれまでいくつ実行されてきたかを表していました

### Intrinsic gasについて

- トランザクションに必要な計算量 (gas) は、正確には実行してみなければわからない
- しかし検証の段階でも「少なくとも絶対にこれだけのgasは必要だよね」という量はわかる
- この部分をintrinsic gasと呼ぶ

具体的には、intrinsic gasは以下の仕様に沿って計算される
- トランザクションの実行には21,000gasが必要 (predefined gas fee)
- トランザクションのInput部分に対して最低68gasが必要で、そこから送るデータが1byte増えるごとに4gas増える (storage fee)
- Contract creationトランザクションの場合は32,000gasが必要

これにも満たないgasLimitが設定されていたら、トランザクションは絶対に実行不可なので弾く


## EthereumのMempool
各ノードが検証の済んだトランザクションを溜めておく領域をMempoolと呼ぶ
- Bitcoinと同じ

EthereumのMempoolは、以下の2階層に分かれている

- Pending pool
- Queued pool
  - Pending pool内のトランザクションの順序を揃えるために、噛ませておくpool
  - 伝搬されたトランザクションのnonceが、送信者(EOA)のnonce+1にならない場合がある
    - あるEOAはこれまで4つトランザクションを送信している
    - 1番目と2番目は既に実行された
    - あるノードへ、3番目より先に4番目のトランザクションが伝搬された
  - この場合、4番目のトランザクションをQueued poolに一旦保持し、3番目が伝搬された後に3番目、4番目の両方をPending poolへと移動する
  

BitcoinのMempoolもTransaction poolとOrphan transaction poolがあった
- 名前は違うがやっていることは同じ
- ただしUTXO型のBitcoinは親を見ていたが、アカウントベース型のEthereumはnonceを見る


## ブロックの作成
### Etherreumブロックのデータ構造
- マイナーノードはMempool (のPending pool) に溜まったトランザクションから、任意のトラン
ザクションを選択してブロックに格納する
  - このとき、gasLimitとgasPriceがトランザクションを選ぶ際の参考になる
  - 普通は計算量に対して得られる手数料が高そうなトランザクションを選択する
- トランザクションは、マークル・パトリシアツリー構造でブロックに格納される
- ここまでは基本的にBitcoinと同じ
  - しかし、Ethereumにおけるブロックの作成作業はこれだけに留まらない!
  - Ethereumのブロックは、Bitcoinのそれ以上に様々な情報を格納している
  
### ブロックヘッダの新要素

| 名称 | 役割 |
| ---- | ---- |
| shaUncles | Uncleブロックのブロックヘッダリストをハッシュ化したもの
| extraData | 任意のデータ (32byteまで)
| gasLimit | ブロック全体のgasLimit
| gasUsed | ブロックに格納された全トランザクションのgasUsedの合計
| logsBloom | ブロック内の全トランザクションの実行ログ (Bloom Filter形式)
| miner | ブロックを作成したノードが持つ、報酬受取用のアドレス
| mixHash | nonce と合わさることでPoWのための十分な計算がされたことの証明になる256bitのハッシュ
| number | ブロックの番号 (genesis blockを0として累積)
| totalDifficulty | このブロック以前のブロックのdifficultyの総和
| State Root | このブロックの全トランザクションが実行された後の、全てのアカウント状態をマークルパトリシアツリーで要約したRoot値のハッシュ値(KECCAK-256ハッシュ形式)
| Receipts Root | このブロックの全レシートをマークルパトリシアツリーで要約したRoot値のハッシュ値(KECCAK-256ハッシュ形式)

### Uncle Blockについて
- マイナーノードは、トランザクションに加えてUncle blockの情報もブロックに格納する
  - Uncle block: メインチェーンから分岐したブロックのこと
  - 具体的には、任意のUncle blockのブロックヘッダを2つまで格納することが出来る
- `なぜ?:Ethereumのブロックチェーンは、Bitcoinと比べてより頻繁に分岐するから`
  - (後述するが) Ethereumのblock intervalは15秒に設定されている。
  - Block intervalが短くなると、同時にマイニングに成功する可能性が高まるため、ブロックチェーンはより頻繁に分岐してしまう
  - (これも後述するが) その対策として、Nakamoto Consensusの代わりにUncle blockを考慮した合意形成 (GHOST protocol) を採用し、かつUncle blockにも報酬を与えている

### Block Gas Limitについて
- Ethereumのブロックには、ブロックサイズの上限が設定されていない
  - Bitcoinは現状4MB
- その代わりに、block用のgasLimitが設定されており、ブロックに格納された全トランザクションのgasLimitの合計がこのblock gasLimitを超えてはならないことになっている
  - Block gasLimitとTransaction gasLimitはしばしば混同されるので注意！
  - 前者はマイナーノードが決める変数で、後者はEOAが決める変数である
- なぜ？：Ethereumにおいてマイナーノードはトランザクションの実行も担うため、彼らの負担は容量よりも計算量で把握した方が適切だから
  - このような「スマートコントラクト用のプラットフォームとして計算量(gas)を軸に物事を考えよう」という思想はEthereumのブロックチェーンにおいて一貫しています
- マイナーノードは、Block gasLimitを30,000,000Gweiまで増やすことが出来る
  -  1Gwei = 10^8 wei
  - ただし次の仕組みが示すとおり、ターゲット値は15,000,000Gweiである

### baseFeePerGasの決定ルールについて
- baseFeePerGasはブロック毎に固定で、以下のルールに沿って内生的に決まる
  - 1つ前のブロックのBlock gasLimitが…
    - 最大値 (30,000,000Gwei) の0%である → baseFeePerGasは-12.5%
    - 最大値 (30,000,000Gwei) の0〜50%である → baseFeePerGasは-12.5〜±0%
    - 最大値 (30,000,000Gwei) の50%である → baseFeePerGasは±0%
    - 最大値 (30,000,000Gwei) の50〜100%である → baseFeePerGasは±0〜 +12.5 %
    - 最大値 (30,000,000Gwei) の100%である → baseFeePerGasは+12.5% 
- つまり... Txが増える → マイナーノードがBlock gasLimitを増やす → 次のbaseFeePerGasが増加する → EOAが高い手数料を忌避してTxを作らなくなる → 混雑の緩和に繋がる、という流れが想定されている (and vice versa)
- 直感的に言えば、手数料に難易度調整のような仕組みを導入している
- `なぜ？: (Bitcoinのように) EOAにfeePerGasを決めさせるよりも、支払う手数料を最適化出来そうだから`

### State Root, Receipts Rootについて
- Ethereumでは、トランザクションを実行した結果として、状態とレシートをマークルパトリシアツリー状にしたRootもブロックに格納する 
- ただしトランザクションとは違い、ブロックに格納されるのはRoot部分のみ!
- 復習: EOAとCAが保持する状態データ
- 復習: Message CallトランザクションとContract Creationのレシート
- 状態ツリーとレシートツリーの全歴史は、archive nodeが保持している

<!-- 芝野さん授業スライドの図を挿入 -->

改めて整理すると、状態(state)に関して各ノードが保持するデータは以下の通り:

- Light node
  - Transaction root, state root, receipt root (block header)
- Full node
  - Transaction root, state root, receipt root (block header)
  - Transaction data (block)
  - State data (latest 120 blocks)
  - Receipt data (latest 120 blocks)
- Archive node
  - Transaction root, state root, receipt root (block header)
  - Transaction data (block)
  - State data
  - Receipt data

つまりスマートコントラクト用に書かれたプログラムの実行結果は、archive nodeも持っていない!
- ハッシュ化したものだけをstorage rootとして保持している。
- (プログラムのコード自体はエンコード化されたものがcontract creation トランザクションのinputに入っている)
- 「あるEOAの過去の残高」などの情報は、ブロックチェーン内のトランザクションデータを計算することでfull
nodeでも把握することは出来る。ただしarchive nodeが持つ状態データを参照した方が早い。

## トランザクションの実行
マイナーノードは、各トランザションに対して以下の処理を行う:
1. 状態ツリーにあるトランザクション作成者(EOA)のnonceを1増やす
2. 実行に必要であろうether (gasLimit * feePerGas + value) を作成者 (EOA) の残高から徴収
3. inputにある処理を (EVMを用いて) 実行
   1. もし結局Gasが足りなければ、out of gas exceptionエラーとして処理を中断
4. 実行結果に関するレシートを発行
5. 処理にかかったGas feeのうち、priority fee部分 (priorityFeePerGas * gasUsed) を自身のアドレスへ送金
   1. もしGasが余ったならば、その分は作成者(EOA)に返金


マイナーノードは、ブロックに対して以下の処理を行う:
1. 状態ツリーとレシートツリーを作成する
   1. Full nodeの場合、120ブロック経過したらlocal storageから削除
2. ブロックヘッダーにState rootとReceipt rootを書き込む
3. ブロックヘッダーにgasUsedを書き込む

## ブロックチェーンの構造
ここはBitcoinと同じ。親のブロックヘッダのハッシュを自身のブロックヘッダに格納することでチェーンを形成する

## EthereumのPoW (Proof-of-Work)
- 現在のEthereumはBitcoinと同様にPoWを採用している
  - 現在はProof-of-Stake型へと以降している
  - `なぜ?: ブロック作成にかかる時間を短縮し、処理速度の向上を図りたいから (後述)`
- ただしEtherumのPoWは、オリジナルのEthashというアルゴリズムを採用している
  - `なぜ?: マイナーノードの寡占化を可能な限り防ぎたいから`
  - 当時Bitcoinのマイニングは、それ専用の演算マシンであるASICが開発されたためにマイナーの寡占化が進んでいた
  - この現状を鑑みて、Ethereumは演算を繰り返す形ではなく、メモリからデータを繰り返し呼び出して比較する形でnonceを探すPoWを実装した (ASIC-resistantなどと呼ばれる)
- Ethashの詳細については時間の都合上説明を割愛するが、以下の資料が非常にわかりや
すい
  - 【第5回】Ethereumの全体像を理解する - MiningとConsensus
  - https://www.etarou.work/posts/4983481

### block intervalと難易度調整について
- Ethereumのblock intervalは、Bitcoinの10分に対して、15秒に設定されている
  - `なぜ?:スマートコントラクト用プラットフォームとして、処理速度の向上を図りたいから`
  - 送金ならば10分待てるかも知れないが、プログラムの実行はそんなに待てない
- ただし先述のとおり、block intervalが短いとその分チェーンが分岐しやすくなる
- Ethereumの難易度調整は、 Bitcoinは2016ブロック毎だったが、1ブロック毎に行われる
  - `なぜ?:より安定的に規定のblock intervalを維持したいため`
  - 難易度調整の仕様はメジャーアップデートの度にコロコロ変更されているが、現在は
    - Parent blockのdifficultyとtimestamp
    - Uncle blockのdifficultyとtimestampを元に計算されている。
  - 昔はuncle blockは用いられていなかった
- 計算方法の詳細については時間の都合上説明を割愛するが、以下の資料が非常にわかり
やすい
  - Ethereumのディフィカルティに関するまとめ
    - https://blockchain.gunosy.io/entry/ethereum-difficulty-summary - Ethereumのディ
フィカルティ算出方法

## 独立したブロックの検証
- 各ノードは、受け取ったブロックについて以下の内容を検証する
- 検証内容は基本的にビットコインと同様 (だがUTXOとアカウントベースという設計の違いが存在)
- トランザクションと同様、検証に通ったブロックのみを隣接ノードへと転送する
  - ブロックのデータ構造が正しいか
  - ブロックヘッダに含まれるtimestampが、ノードが持つ時間より+2時間以内に収まっているか
  - ブロックヘッダに含まれているdifficultyが正しいか
  - ブロックヘッダに含まれるnonceがdifficultyの条件を満たしているか (PoWの検証)
  - Transaction gasLimitの合計 ≦ Block gasLimitになっているか
  - 適切なUncle blockを格納しているか
  - ブロックに含まれるすべてのトランザクションが 「独立したトランザクション検証」のチェックリストをすべて満たすか
  - ブロックに含まれるすべてのトランザクションを実行した結果である、State ｒoot, Receipts root, gasUsed, logsBloomが正しいか

謝罪: ブロックの検証はBitcoinと同様に全てのfull (or archive) nodeが行うのか？それともマイナーノードだけが行うのか？ギリギリまで調べていたのですが、結局わかりませんでした。申し訳ありません。
*トランザクションの検証はlight node含む全てのノードが行っています。


## コンセンサスとマイニング報酬
### GHOSTプロトコルについて
- Ethereumでは、Bitcoinとは異なり「最も長いチェーン」を正統な記録とする (Nakamoto
consensus) わけではない!
- 代わりに「最も重いチェーン」を正統とする
- これはGHOST (Greedy Heaviest Observed Subtree) プロトコルと呼ばれる
  - *元々はBitcoinに対する改善提案でした
- チェーン選択のアルゴリズムは比較的単純で、小ブロックのサブツリー数を比較し続けるだけ
  - 特に頻繁に分岐するブロックチェーンでは、このように「長さより重さ」で選択すべきではないか?
- 頻繁に分岐するブロックチェーンにおいては、攻撃に必要なコストもより高くなる
  - 以下の例では、攻撃者がメインチェーンとなるためには、さらに6ブロックがAチェーンに必要
  - *言い換えれば、合意形成においてUncle (or Orphan) blocksにも意味を持たせることが出来る

### EthererumのGHOSTプロトコルについて
Ethereumでは、このようなGHOSTプロトコルに以下の変更を加えている
- サブツリー数の比較は、現在のメインチェーンの先端から7つ前にあるブロックから始める
  - `なぜ?: Genesisブロックから始めると計算が大変だから`
- メインチェーンの先端のブロックが2つ以上になった場合、ブロックが含むUncle block (のブロックヘッダ) に記載されたdifficultyの合計が大きい方を先端のブロックとする

### Ethererumのマイニング報酬について
Ethereumでは、ブロックの作成に成功したマイナーノードへ2ETHが新規発行される
- Bitcoinと異なり半減期はない
- Bitcoinと異なりblock maturityの設定 (100 confirmations) もない

さらに、もしブロックがuncle blockのブロックヘッダを含んでいる場合...
- 2ETHの報酬が3.125%増える (Uncle inclusion rewards)
  - `なぜ?:マイナーノードに、uncle blockの格納を促したいから`
- Uncle blockを作成した各ノードに対して、2ETHの87.5%分の報酬が新規発行される
- Uncle blockの子ブロック (nephew block) を作成した各ノードに対して、 2ETHの3.125%分の報酬が新規発行される
  - `なぜ?: 頻繁にブロックチェーンが分岐する環境でもマイニングへの参入を促したいから`
  - ただし対象となるUncle blockは (EthereumのGHOSTプロトコルが扱う範囲である) 現在のメインチェーンの先端から7つ前以内のブロックから分岐している必要がある

*この仕様が本当に上手く機能するものなのか?特に、block maturityの設定が存在せずとも良い
のか?については、個人的に疑問を持っています。

# まとめ
- トランザクションのライフサイクルを詳細に確認した　

詳細は構造はここまで。最後にEthereumのガバナンスと課題をみてみよう。
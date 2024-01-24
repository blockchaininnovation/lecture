# 詳細な仕様について

Ethereumの構成要素について、もう少し詳細をみてみよう。

## EOA (Externally Owned Account)
- EOAはBitcoinにおけるアドレスのようなもの
- 作成の基本的なプロセスはBitcoinと同じ

ただし...
- Bitcoinとは異なり、SHA-256ではない
    - `なぜ?: 新しいハッシュ関数でより安全と言われているから`
- Bitcoin Addressとは異なり、後ろにチェックサムが付いていない
    - `なぜ?: 将来的にアドレスに文字を対応させて使う想定だから (e.g., knskito.eth)`
    - *アドレス自体にチェックサムを付けかつ誤読を防ぐ文字列にエンコーディングする仕組みもあるにはある

**EOAが保持するデータ ( = 状態 = state )**
| 名称 | 役割 |
| ---- | ---- |
| Nonce | EOAのトランザクションがこれまでにいくつ実行されたか |
| Balance | EOAが所持している残高 (wei) |

  - Bitcoin (のUTXO) とは異なり、アカウントが残高を保持している (この設計をアカウントベースと呼ぶ)
  - 取引 (0.3BTC from Alice to Carol) ではなく残高 (Alice 0.7→0.4ETH; Carol 0→0.3ETH)
  - `なぜ?: (スマートコントラクトの結果を含む)状態遷移を管理するには、トランザクションにアドレスを紐付けるよりもアドレスにトランザクションを紐付けたほうが設計しやすいからなのだと思う...おそらく`

### 補足: Ethereumの通貨単位
- Ethereum通貨の1単位は「イーサ」「ether」と呼ばれ「ETH」と表記する.「Ξ」「♦」とも書く
  - 1 ether = 1 ETH = Ξ1 = ♦1
  - 通貨はあくまでetherであって、Ethereumはシステム名

- weiという単位もあり、etherの最小単位
  - 1 ether = 1 * 10^18 or 1,000,000,000,000,000,000 wei
  - 1 eth = 100京 wei

- 参考:Bitcoinにおけるsatoshiは・・・
  - 1 BTC = 1 * 10^8 or 100,000,000 satoshi
  - 1 BTC = 1億 satoshi

## CA (Contract Account)
- CAもアドレスを持つが、コントラクトの入れ物であるため作り方は大きく異なる
  - そもそもCAは秘密鍵を持たないことを思い出しましょう
- 具体的には、EOAのアドレスとそのEOAが保持するnonceをハッシュ化して作成する

**CAが保持するデータ ( = 状態 = state )**
| 名称 | 役割 |
| ---- | ---- |
| Nonce | CAのトランザクションがこれまでにいくつ実行されたか |
| Balance | CAが所持している残高 (wei) |
| Code Hash | コントラクトの中身 (プログラムコード) をハッシュ化 |
| Storage Root | コントラクトの結果をハッシュ化 |

- コントラクトの結果は、複数のアウトプット (e.g., コントラクト5回目, ‘送金が完了しました’) をマークル・パトリシアツリーに格納する形式で保存される。
- ただし、CAが保持するのは、根(root)の部分のみ。
- `なぜ?: コントラクトの全てのアウトプットを結果としてブロックチェーンに記録すると、容量を圧迫してしまうから`

### 補足: マークル・パトリシアツリー
- マークルツリー + パトリシアツリー
- データ探索の効率性をさらに高めるために採用されている
- Ethereumにおける木構造でのデータ保存は、全てマークル・パトリシアツリーとなっている
- 時間の都合上詳細な説明は割愛、以下の資料を参照のこと
  - https://qiita.com/yanagisawa-kentaro/items/bfdbb5564d1751c3d2ea (JP)
  - https://wiki.nebulas.io/en/latest/go-nebulas/design-overview/merkle_trie.html (EN)

## Message Call トランザクションのデータ構造

Bitcoinの送金に相当する、いわゆるトランザクション

| 名称 | 役割 |
| ---- | ---- |
|blockhash| このtxを含むブロックのブロックヘッダーハッシュ
|blocknumber| このtxを含むブロックが何番目か
|transaction| index このtxがブロックの中で何番目のトランザクションか
|from| このtxを作成したEOAのアドレス
|to| このtxの宛先 (EOA or CA) のアドレス
|nonce| 作成者であるEOAにとってこのtxが何番目か
|hash| このtx自体のハッシュ
|value| 送金するetherの量 (wei)
|data| データ領域 (16進数の数列にエンコード化されている)
|gasLimit| EOAがtx実行に対して支払えるGasの最大量
|maxPriorityFeePerGas| EOAがバリデータノードに支払えるGasの最大金額
|maxFeePerGas| EOAがtx実行のために支払えるGasの最大金額


## Ethereumの手数料 (gas)
## Contract Creation トランザクション

| TH | TH |
| ---- | ---- |
| TD | TD |
| TD | TD |

次回はこれらがどのようなプロセスで動いていくのか？というブロックチェーン運用の議論を、トランザクションのライフサイクルを元に示す。
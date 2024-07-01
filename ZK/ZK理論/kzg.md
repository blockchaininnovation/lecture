<center>

  # KZG Commitments: 係数を秘匿した多項式の評価結果の証明

</center>

## はじめに
KZG commitmentsは、polynomial commitments (PC)と呼ばれる暗号スキームの一つです [1]。PCを使うことで、証明者は自身の知るある多項式$f(x)$が$f(\alpha)=\beta$を満たすことを、$f(x)$に明らかにせずに、検証者に示すことができます。この性質により、PCはゼロ知識証明 (ZKP) [2]や、検証可能な秘密分散 [3]、データ可用性サンプリング (data availability samping; DAS) [4-5]など、より実用的な暗号スキームを構成するために応用されています。特に、PCのうちKZG commitmentsは、検証のための計算量が非常に小さい、具体的には多項式の次数に関わらず一定であるため、計算リソースが限られているEthereumなどのブロックチェーン上でも用いることが可能です [6]。この資料では、PCの一般的な性質と安全性を定義し、その後KZG commitmentsの構成を紹介します。

`Writer: @SoraSuegami`


## 定義
PCは、その名前が示す通り、多項式 (polynomial)に対するcommitmentsスキームになります。そのため、最初にcommitmentsスキームの定義を述べ、その後にPCについて説明します。

### Commitments
Commitmentsスキームは、封筒と同じような性質を持つデジタルデータを実現します [7]。具体的には、最初にメッセージを封筒に入れて後でその封を開けて中身を確認できるように、commitmentsスキームではメッセージからcommitmentというデータを生成し (コミットフェーズ)、その後にそのメッセージがcommitmentに入っていることを確認できるようにするための、openingというデータを生成します (オープンフェーズ)。ここで、封筒の封を開けるまで中のメッセージがわからないのと同様に、commitmentはopeningが開示されるまでメッセージを秘匿します (Hiding)。また、封を開けずに封筒の中のメッセージを入れ替えることができないのと同様に、commitmentの生成時に使われたものとは異なるメッセージに対してopeningを生成することは、commitmentの生成者であってもできません (Binding)。

Commitmentsスキームを応用することで、例えばオンラインじゃんけんを実現することができます。じゃんけんでは、次の2つの要件が求められます。
1. 各プレイヤーは、自分の手を出すまで他のプレイヤーの手を知ることはできない。
2. 各プレイヤーは、一度自分の手を出した後、その手を入れ替えることはできない。

単純に自分の手を表すデータを他のプレイヤーに送信すると、要件1が達成されないことがわかるでしょう。そこで、最初に各プレイヤーは自分の手に対するcommitmentを他のプレイヤーに送信し、全員がcommitmentを送信した後に、それぞれの手のopeningを送信します。この方式は、Hiding、Bindingの性質により、それぞれ要件1、2を達成することができます。以上のように、commitmentsスキームを使うことで、メッセーずをある時点まで隠しながら、そのメッセージに紐づくデータを送信することが可能になります。

厳密には、commitmentsスキームは次のアルゴリズムによって定義されます [8]。
- $\textsf{Setup}(1^{\lambda}) \rightarrow \textsf{pp}$: 入力としてセキュリティパラメータ$1^{\lambda}$を受け取り、公開パラメータ$\textsf{pp}$を生成する。
- $\textsf{Commit}(\textsf{pp}, m, r) \rightarrow c$: 入力として公開パラメータ$\textsf{pp}$とメッセージ$m$、乱数$r$を受け取り、 commitment $c$を出力する。
- $\textsf{Open}(\textsf{pp}, m, r, c) \rightarrow d$: 入力として公開パラメータ$\textsf{pp}$とメッセージ$m$、乱数$r$、commitment $c$を受け取り、 opening $d$を出力する。
- $\textsf{Verify}(\textsf{pp}, m, c, d) \rightarrow 1/0$: 入力として公開パラメータ$\textsf{pp}$とメッセージ$m$、commitment $c$、opening $d$を受け取り、1 (受理)または0 (拒否)を出力する。

注意点として、封筒の封は誰でも勝手に開けることができるのに対して、commitmentsスキームは$\textsf{Open}$アルゴリズムが乱数$r$を必要としているため、**commitment $c$を生成した人だけがその$c$に対するopeningを生成できる**ようになっています。

これらのアルゴリズムが以下の関係を満たすとき、commitmentsスキームは正当性 (correctness)があると言われます。
$$
Pr[\textsf{Verify}(\textsf{pp}, m, c, \textsf{Open}(\textsf{pp}, m, r, c))=1] \geq 1 - \textsf{negl}(\lambda)
$$
ただし、$\textsf{negl}(\lambda)$は無視できるほど小さい確率を表します。

HidingとBindingは、直感的には次のように定義されます。
- Hiding: ランダムなメッセージと乱数$m, r$に対するcommitment $c \leftarrow \textsf{Commit}(\textsf{pp}, m, r)$を見て、（多項式時間の計算ができる）攻撃者が$m$を正しく推定できる確率は$\textsf{negl}(\lambda)$未満である。
- Binding: 2つの異なるメッセージと乱数のペア$(m_0, r_0), (m_1, r_1)$に対して、$c_0 \leftarrow \textsf{Commit}(\textsf{pp}, m_0, r_0)$と$c_1 \leftarrow \textsf{Commit}(\textsf{pp}, m_1, r_1)$が$c_0 = c_1$を満たす確率は$\textsf{negl}(\lambda)$未満である。


### Polynomial Commitments
PCスキームは、多項式$f(x)=\Sigma_{i=0}^n a_i x^i$の係数ベクトル$(a_0, \dots, a_n)$をメッセージとするcommitmentsスキームです [3]。つまり、$(a_0, \dots, a_n)$からcommitment $c$を生成し、それを開示するopeningを生成します。加えて、PCでは$c$の中の多項式$f(x)$をある評価点$\alpha$で評価した結果が$\beta$になる、つまり$f(\alpha)=\beta$であることだけを、$(a_0, \dots, a_n)$を明かさずに開示することも可能です。この性質により、**多項式そのものを隠したまま、その多項式が特定の条件を満たしていることを他者に証明できます**。

厳密には。PCスキームは次のアルゴリズムによって定義されます [3]。
- $\textsf{Setup}(1^{\lambda}, n) \rightarrow \textsf{pp}$: 入力としてセキュリティパラメータ$1^{\lambda}$と最大の次数$n$を受け取り、公開パラメータ$\textsf{pp}$を生成する。
- $\textsf{Commit}(\textsf{pp}, f(x), r) \rightarrow c$: 入力として公開パラメータ$\textsf{pp}$と多項式$f(x)$、乱数$r$を受け取り、 commitment $c$を出力する。ただし、$f(x)$の実際のデータは、$f(x)$の係数ベクトル$(a_0, \dots, a_n)$になる。また、$f(x)$の次数はアルゴリズム$\textsf{Setup}$で指定された整数$n$以下でなければならない。
- $\textsf{Open}(\textsf{pp}, f(x), r, c) \rightarrow d$: 入力として公開パラメータ$\textsf{pp}$と多項式$f(x)$、乱数$r$、commitment $c$を受け取り、 opening $d$を出力する。
- $\textsf{VerifyPoly}(\textsf{pp}, f(x), c, d) \rightarrow 1/0$: 入力として公開パラメータ$\textsf{pp}$と多項式$f(x)$、commitment $c$、opening $d$を受け取り、1 (受理)または0 (拒否)を出力する。
- $\textsf{CreateWitness}(\textsf{pp}, f(x), \alpha, r) \rightarrow (\beta = f(\alpha), \omega_{\alpha})$: 力として公開パラメータ$\textsf{pp}$と多項式$f(x)$、評価点$\alpha$、乱数$r$を受け取り、評価結果$\beta = f(\alpha)$とwitness $\omega_{\alpha}$を出力します。
- $\textsf{VerifyEval}(\textsf{pp}, f(x), c, \alpha, \beta, \omega_{\alpha}) \rightarrow 1/0$: 入力として公開パラメータ$\textsf{pp}$と多項式$f(x)$、commitment $c$、評価点$\alpha$、評価結果$\beta$、witness $\omega_{\alpha}$を入力として受け取り、1 (受理)または0 (拒否)を出力する。

これらのアルゴリズムが以下の2つの関係を満たすとき、commitmentsスキームは正当性 (correctness)があると言われます。
$$
    Pr[\textsf{VerifyPoly}(\textsf{pp}, f(x), c, \textsf{Open}(\textsf{pp}, f(x), r, c))=1] \geq 1 - \textsf{negl}(\lambda)
$$
$$
    Pr[\textsf{VerifyEval}((\textsf{pp}, f(x), c, \alpha, \textsf{CreateWitness}(\textsf{pp}, f(x), \alpha, r)))] \geq 1 - \textsf{negl}(\lambda)
$$
特に後者の条件は、$f(\alpha)=\beta$を満たす正当な$f(x)$、$\alpha$、$\beta$に対して生成されたwitnessは、十分に高い確率で$\textsf{VerifyEval}$によって受理されることを要請しています。



## KZG Commitmentの構成
### 基本的な構成
### 複数の評価点・単一の評価結果の証明のバッチ化
### 複数の評価点・複数の評価結果の証明のバッチ化
## References
1. Kate, A., Zaverucha, G. M., & Goldberg, I. (2010). Constant-size commitments to polynomials and their applications. In Advances in Cryptology-ASIACRYPT 2010: 16th International Conference on the Theory and Application of Cryptology and Information Security, Singapore, December 5-9, 2010. Proceedings 16 (pp. 177-194). Springer Berlin Heidelberg.
2. Gabizon, A., Williamson, Z. J., & Ciobotaru, O. (2019). Plonk: Permutations over lagrange-bases for oecumenical noninteractive arguments of knowledge. Cryptology ePrint Archive.
3. Kate, A., Zaverucha, G. M., & Goldberg, I. (2010). Polynomial commitments. Tech. Rep.
4. Hall-Andersen, M., Simkin, M., & Wagner, B. (2023). Foundations of data availability sampling. Cryptology ePrint Archive.
5. Nikolaenko, V., & Boneh, D. (2023). Data availability sampling and danksharding: An overview and a proposal for improvements. Available at https://a16zcrypto.com/posts/article/an-overview-of-danksharding-and-a-proposal-for-improvement-of-das/
6. Arditi, A. (n.d.). KZG in Practice: Polynomial Commitment Schemes and Their Usage in Scaling Ethereum. Available at https://scroll.io/blog/kzg
7. 岡本, 龍明. (2019). 現代暗号の誕生と発展: ポスト量子暗号・仮想通貨・新しい暗号. 近代科学社.
8. Micciancio, D. (2019). Lattice Algorithms and Applications. Commitment Schemes. Available at https://cseweb.ucsd.edu/classes/fa19/cse206A-a/LecCommit.pdf
<center>

  # KZG Commitments: 係数を秘匿した多項式の評価結果の証明

</center>

## はじめに
KZG commitmentsは、polynomial commitments (PC)と呼ばれる暗号スキームの一つです [1]。PCを使うことで、証明者は自身の知るある多項式$f(x)$が$f(\alpha)=\beta$を満たすことを、$f(x)$に明らかにせずに、検証者に示すことができます。この性質により、PCはゼロ知識証明 (ZKP) [2]や、検証可能な秘密分散 [1]、データ可用性サンプリング (data availability samping; DAS) [3-4]など、より実用的な暗号スキームを構成するために応用されています。特に、PCのうちKZG commitmentsは、検証のための計算量が非常に小さい、具体的には多項式の次数に関わらず一定であるため、計算リソースが限られているEthereumなどのブロックチェーン上でも用いることが可能です [5]。この資料では、PCの一般的な性質と安全性を定義し、その後KZG commitmentsの構成を紹介します。

`Writer: @SoraSuegami`


## 定義
PCは、その名前が示す通り、多項式 (polynomial)に対するcommitmentsスキームになります。そのため、最初にcommitmentsスキームの定義を述べ、その後にPCについて説明します。

### Commitments
Commitmentsスキームは、封筒と同じような性質を持つデジタルデータを実現します [6]。具体的には、最初にメッセージを封筒に入れて後でその封を開けて中身を確認できるように、commitmentsスキームではメッセージからcommitmentというデータを生成し (コミットフェーズ)、その後にそのメッセージがcommitmentに入っていることを確認できるようにするための、openingというデータを生成します (オープンフェーズ)。ここで、封筒の封を開けるまで中のメッセージがわからないのと同様に、commitmentはopeningが開示されるまでメッセージを秘匿します (Hiding)。また、封を開けずに封筒の中のメッセージを入れ替えることができないのと同様に、commitmentの生成時に使われたものとは異なるメッセージに対してopeningを生成することは、commitmentの生成者であってもできません (Binding)。

Commitmentsスキームを応用することで、例えばオンラインじゃんけんを実現することができます。じゃんけんでは、次の2つの要件が求められます。
1. 各プレイヤーは、自分の手を出すまで他のプレイヤーの手を知ることはできない。
2. 各プレイヤーは、一度自分の手を出した後、その手を入れ替えることはできない。

単純に自分の手を表すデータを他のプレイヤーに送信すると、要件1が達成されないことがわかるでしょう。そこで、最初に各プレイヤーは自分の手に対するcommitmentを他のプレイヤーに送信し、全員がcommitmentを送信した後に、それぞれの手のopeningを送信します。この方式は、Hiding、Bindingの性質により、それぞれ要件1、2を達成することができます。以上のように、commitmentsスキームを使うことで、メッセーずをある時点まで隠しながら、そのメッセージに紐づくデータを送信することが可能になります。

厳密には、commitmentsスキームは次のアルゴリズムによって定義されます [7]。
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
PCスキームは、多項式$f(x)=\Sigma_{i=0}^n a_i x^i$の係数ベクトル$(a_0, \dots, a_n)$をメッセージとするcommitmentsスキームです [1]。つまり、$(a_0, \dots, a_n)$からcommitment $c$を生成し、それを開示するopeningを生成します。加えて、PCでは$c$の中の多項式$f(x)$をある評価点$\alpha$で評価した結果が$\beta$になる、つまり$f(\alpha)=\beta$であることだけを、$(a_0, \dots, a_n)$を明かさずに開示することも可能です。この性質により、**多項式そのものを隠したまま、その多項式が特定の条件を満たしていることを他者に証明できます**。

厳密には。PCスキームは次のアルゴリズムによって定義されます [1]。
- $\textsf{Setup}(1^{\lambda}, n) \rightarrow \textsf{pp}$: 入力としてセキュリティパラメータ$1^{\lambda}$と最大の次数$n$を受け取り、公開パラメータ$\textsf{pp}$を生成する。
- $\textsf{Commit}(\textsf{pp}, f(x), r) \rightarrow c$: 入力として公開パラメータ$\textsf{pp}$と多項式$f(x)$、乱数$r$を受け取り、 commitment $c$を出力する。ただし、$f(x)$の実際のデータは、$f(x)$の係数ベクトル$(a_0, \dots, a_n)$になる。また、$f(x)$の次数はアルゴリズム$\textsf{Setup}$で指定された整数$n$以下でなければならない。
- $\textsf{Open}(\textsf{pp}, f(x), r, c) \rightarrow d$: 入力として公開パラメータ$\textsf{pp}$と多項式$f(x)$、乱数$r$、commitment $c$を受け取り、 opening $d$を出力する。
- $\textsf{VerifyPoly}(\textsf{pp}, f(x), c, d) \rightarrow 1/0$: 入力として公開パラメータ$\textsf{pp}$と多項式$f(x)$、commitment $c$、opening $d$を受け取り、1 (受理)または0 (拒否)を出力する。
- $\textsf{CreateWitness}(\textsf{pp}, f(x), \alpha, r) \rightarrow (\beta = f(\alpha), \omega_{\alpha})$: 力として公開パラメータ$\textsf{pp}$と多項式$f(x)$、評価点$\alpha$、乱数$r$を受け取り、評価結果$\beta = f(\alpha)$とwitness $\omega_{\alpha}$を出力します。
- $\textsf{VerifyEval}(\textsf{pp}, c, \alpha, \beta, \omega_{\alpha}) \rightarrow 1/0$: 入力として公開パラメータ$\textsf{pp}$とcommitment $c$、評価点$\alpha$、評価結果$\beta$、witness $\omega_{\alpha}$を入力として受け取り、1 (受理)または0 (拒否)を出力する。

これらのアルゴリズムが以下の2つの関係を満たすとき、commitmentsスキームは正当性 (correctness)があると言われます [1]。
$$
    Pr[\textsf{VerifyPoly}(\textsf{pp}, f(x), c, \textsf{Open}(\textsf{pp}, f(x), r, c))=1] \geq 1 - \textsf{negl}(\lambda)
$$
$$
    Pr[\textsf{VerifyEval}((\textsf{pp}, c, \alpha, \textsf{CreateWitness}(\textsf{pp}, f(x), \alpha, r)))] \geq 1 - \textsf{negl}(\lambda)
$$
特に後者の条件は、$f(\alpha)=\beta$を満たす正当な$f(x)$、$\alpha$、$\beta$に対して生成されたwitnessは、十分に高い確率で$\textsf{VerifyEval}$によって受理されることを要請しています。

PCはcommitmentsスキームのBindingに加えて、多項式の評価結果を拘束する、Evaluation Bindingという安全性を満たします [1]。具体的には、攻撃者が1つのcommitment $c$と評価点$\alpha$、2つの評価結果とwitnessのペア$(\beta, \omega_{\alpha}), (\beta^{'}, \omega_{\alpha}^{'})$を提出した時、$\textsf{VerifyEval}(\textsf{pp}, f(x), c, \alpha, \beta, \omega_{\alpha})=1$かつ$\textsf{VerifyEval}(\textsf{pp}, f(x), c, \alpha, \beta^{'}, \omega_{\alpha}^{'})=1$になる確率、つまり同じcommitmentで2つの異なる評価結果に対して証明が成功する確率は、$\textsf{negl}(\lambda)$未満です。また、Hidingは、攻撃者が$c$と$\textsf{VerifyEval}$をパスするような複数の$(\alpha, \beta, \omega_{\alpha})$のタプルを受け取った時、まだ受け取っていない評価点$\alpha^{'}$については、その評価結果$f(\alpha^{'})$を無視できる確率$\textsf{negl}(\lambda)$を除いて推定できないことを保証します。

## KZG Commitmentの構成
### 準備
KZG commitmentの構成を説明するために必要な記号を準備します。自然数$n \in \mathbb{N}$に対して、$[n]$は集合$\{1,2,\dots,n\}$を表します。ただし、$[0]$は空集合$\empty$とします。

位数$q$の有限体を$F_q$、$F_q$上で定義される楕円曲線を$E(F_q)$でそれぞれ表記します。ECDSAの説明で述べられたように、楕円曲線上の有理点には加算$P + Q \in E(F_q)$と2倍演算$2P \in E(F_q)$、これらを組み合わせたスカラー倍$nP \in E(F_q)$が定義されます。

さらに、BN curveやBLS curveなどいくつかの楕円曲線の種類では、楕円曲線の点同士の乗算に該当する、pairingと呼ばれる演算が可能です [8]。具体的には、2つの有理点の集合$\mathbb{G}_1, \mathbb{G}_2$とある有限体の集合$\mathbb{G}_T$に対して、次の関数$e$のように定義されます。
$$
    e: \mathbb{G}_1 \times \mathbb{G}_2 \rightarrow \mathbb{G}_T
$$
ただし、$\mathbb{G}_1, \mathbb{G}_2$はそれぞれある点$P \in \mathbb{G}_1, Q \in \mathbb{G}_2$の加算を繰り返して得られる集合として定義され、それぞれ$r$倍すると0に該当する点$\mathcal{O}$になります。すなわち、
$$
    rP = \mathcal{O}, \\
    rQ = \mathcal{O}, \\
    \mathbb{G}_1 = \{\mathcal{O}, P, 2P, \dots, (r-1)P\}, \\
    \mathbb{G}_2 = \{\mathcal{O}, Q, 2Q, \dots, (r-1)Q\}
$$
同様に、$\mathbb{G}_T$はある有限体上の整数$g$の乗算を繰り返して得られる集合として定義され、$r$乗すると1になります。つまり、
$$
    g^r = 1 \\
    \mathbb{G}_T = \{1, g, g^2, \dots, g^{r-1}\}
$$
pairing $e$は次の2つの性質を満たします [9]。
1. 非縮退性
任意の$P \in \mathbb{G}_1$（$Q \in \mathbb{G}_2$）に対して、$e(P, Q) = 1$ならば$Q = \mathbb{O}$ ($P = \mathbb{O}$)である。
2. 双線型性
$$
    e(P_1 + P_2, Q) = e(P_1, Q) e(P_2, Q) \\
    e(P, Q_1 + Q_2) = e(P, Q_1) e(P, Q_2)
$$

これらの性質より、pairingは$aP$、$bQ$のスカラーの乗算に対応していることがわかります。
$$
    e(aP, bQ) = e(P, bQ)^a = (e(P, Q)^b)^a = e(P, Q)^{ab}
$$

現在実用されているpairingの構成では、$\mathbb{G}_{T}$は$\mathbb{G}_1, \mathbb{G}_2$と異なる集合であり、効率的に（多項式時間以内に）$g^{x} \in \mathbb{G}_{T}$の要素を$xP \in \mathbb{G}_{1}$や$xQ \in \mathbb{G}_{2}$に変換することはできません。したがって、pairing $e$の乗算結果を$e$の入力として使うことはできないので、**pairingでは1度しか点同士の乗算を行えません**。以上のようなpairingの性質は、bilinear mapとして抽象化されています [10]。

### 基本的な構成
KZG commitmentsは、端的には剰余の定理の関係が成り立つことをpairingを用いて検証することで、$f(\alpha)=\beta$を確かめています [1]。剰余の定理は、多項式$f(x) - f(\alpha)$が$(x-\alpha)$で割り切れることを保証します。つまり、以下の関係を満たす$n-1$次以下の多項式$q(x)$が一意に存在します。
$$
    f(x) - f(\alpha) = q(x)(x - \alpha)
$$
直感的には、証明者が検証者に$f(\alpha)=\beta$になることを示すためには、
証明者がこのような$q(x)$を直接検証者に送信し、検証者には$f(x) - \beta$が$q(x)(x - \alpha)$と多項式として等しいことを確かめてもらえば良いと言えます。しかし、この方法では検証者の計算量・通信量が多項式の次数$n$に対して線形以上のオーダーで増加するため、効率的ではありません。

そこで、多項式として等式を検証する代わりに、あるランダムな一点$s$で多項式を評価した結果が等しいこと、すなわち$f(s) - \beta = q(s)(s - \alpha)$を検証することを考えます。証明者は$f(s), q(s)$を検証者に送信すると、検証者はこの等式が整数として成立することだけを確かめられれば良いため、検証者の計算量・通信量は$n$に関わらず一定になることがわかります。では、この方式はBindingの性質を実現している、つまり悪意のある証明者が$f(\alpha) \neq \beta$ではない$f(x), \alpha, \beta$に対して有効な$q(s)$を提出することを防いでいるでしょうか？仮に$s$がランダムかつ、証明者が$s$を知ることができなければ、Schwartz–Zippelの補題によりBindingが無視できるほど小さい確率を除いて成り立つことを証明できます [11]。具体的には、非零の次数$n$の多項式で$g(x)=0$となる点$x$は高々$n$個しかないため、サイズ$|S|$の集合から評価点$s \in S$を一様にランダムに取った時に、$g(s)=0$となる確率、すなわち$s$がその$n+1$個の点のいずれかに一致する確率は高々$\frac{n}{|S|}$になります。今回の例では、$g(x) = f(x) - \beta - q(x)(x - \alpha)$、$|S| \approx 2^{254}$であるため、次数$n$が100万程度でも$g(s)=0$になる確率は$\frac{1000000}{2^{254}} \approx 3.45447×10^{-71}$で、無視できる程小さい値になります。

では、いかにして$s$を証明者に対して隠しながら、同時に証明者に$f(s), q(s)$を計算させることができるでしょうか？これを実現するために、KZG commitmentsは楕円曲線とpairingを利用します。以下がその基本的な構成になります [1, 12]。
- $\textsf{Setup}(1^{\lambda}, n) \rightarrow \textsf{pp}$: 
    1. ランダムな$s$を$\mathbb{F}_r$から取る。
    2. $(P, sP, s^2P, \dots, s^nP) \in \mathbb{G}_1^{n+1}$を計算する。
    3. $(Q, sQ) \in \mathbb{G}_2^{2}$を計算する。
    2. $\textsf{pp}=((P, sP, s^2P, \dots, s^nP)、(Q, sQ))$を出力する。
- $\textsf{Commit}(\textsf{pp}, f(x), r) \rightarrow c$:
    1. $f(x)$を係数ベクトル$(a_0, a_1, \dots, a_n) \in \mathbb{F}_r$としてパースする。
    2. $c \leftarrow \Sigma_{i = 0}^{n} a_{i}(s^{i}P) \in \mathbb{G}_1$を計算する。
    3. $c$を出力する。
- $\textsf{CreateWitness}(\textsf{pp}, f(x), \alpha, r) \rightarrow (\beta = f(\alpha), \omega_{\alpha})$:
    1. $q(x) = \frac{f(x) - f(\alpha)}{x - \alpha}$を計算する。
    2. $q(x)$を係数ベクトル$(q_0, q_1, \dots, q_n) \in \mathbb{F}_r$としてパースする。
    3. $\omega_{\alpha} \leftarrow \Sigma_{i = 0}^{n} q_{i}(s^{i}P) \in \mathbb{G}_1$を計算する。
    4. $\omega_{\alpha}$を出力する。
- $\textsf{VerifyEval}(\textsf{pp}, c, \alpha, \beta, \omega_{\alpha}) \rightarrow 1/0$:
    1. $e(c - \beta P, Q) = e(\omega_{\alpha}, (sQ) - \alpha Q)$が成り立てば1を、そうでなければ0を出力します。

上記の構成は次のように正当です。
$$
    e(c - \beta P, Q) = e(\Sigma_{i = 0}^{n} f_{i}(s^{i}P) - \beta P, Q) \\
    = e((f(s) - \beta)P, Q) \\
    = e(P,Q)^{f(s)-\beta}\\
    = e(P,Q)^{q(s)(s-\alpha)}\\
    = e(q(s)P, ((sQ) - \alpha)Q)\\
    = e(\omega_{\alpha}, (sQ) - \alpha Q)
$$

また、証明者は$f(s)P, q(s)P$を計算できるにも関わらず、離散対数問題の困難性より$\textsf{pp}$から$s$を知ることはできません。したがって、これは$s$を証明者に対して隠しながら、証明者に$f(s), q(s)$を計算させるという機能を実現しています。よって、先述のSchwartz–Zippelの補題を用いた議論と合わせると、これはBindingを実現していると言えます。ただし、原論文 [1]の定理3.2で述べられているBindingの証明は、Schwartz–Zippelの補題ではなく、t-SDH仮定という計算量困難性の仮定に基づいていていることに注意してください。

上記の構成は、部分的にHidingを満たしています。具体的には、$f(x)$の係数のうち少なくとも一つが十分にランダムである、例えば$a_i$が254 bit程度の位数の有限体$F_r$からランダムに取られる場合であれば、攻撃者が$c$と次数$n$以下の評価点・評価結果から$f(x)$の係数や他の評価結果を推定することは計算量的に困難だと言えます。しかし、例えば次数が$n=31$で各係数が0もしくは1の場合、係数ベクトルの候補は$2^{32}$個しかないため、攻撃者は$\textsf{pp}$から各候補に対応する$2^{32}$個のcommitmentをそれぞれ生成できます。そのため、どの候補のものが渡されたcommitmentと一致するか確かめることによって、攻撃者はそのcommitmentに対応する$f(x)$の係数および任意の評価結果$f(\alpha)$を知ることができます。この問題を修正するために、原論文 [1]は3.3章でランダムな多項式で元の多項式を秘匿した構成を提案しています。ただし、その構成は検証者がペアリング$e$を3回計算する必要があるため、検証者の計算量を増やしています。それに対して、Plonk [2]などZKPの構成では、評価したい点$\alpha_1,\dots,\alpha_n$で評価結果が0になるような多項式をランダムに生成し、それを$f(x)$に足した多項式に本章で述べたKZG commitmentsの構成を適用することで、$f(x)$の係数を秘匿しています。


### 複数の評価点・単一の評価結果の証明のバッチ化


### 複数の評価点・複数の評価結果の証明のバッチ化
## References
1. Kate, A., Zaverucha, G. M., & Goldberg, I. (2010). Polynomial commitments. Tech. Rep.
2. Gabizon, A., Williamson, Z. J., & Ciobotaru, O. (2019). Plonk: Permutations over lagrange-bases for oecumenical noninteractive arguments of knowledge. Cryptology ePrint Archive.
3. Hall-Andersen, M., Simkin, M., & Wagner, B. (2023). Foundations of data availability sampling. Cryptology ePrint Archive.
4. Nikolaenko, V., & Boneh, D. (2023). Data availability sampling and danksharding: An overview and a proposal for improvements. Available at https://a16zcrypto.com/posts/article/an-overview-of-danksharding-and-a-proposal-for-improvement-of-das/
5. Arditi, A. (n.d.). KZG in Practice: Polynomial Commitment Schemes and Their Usage in Scaling Ethereum. Available at https://scroll.io/blog/kzg
6. 岡本龍明. (2019). 現代暗号の誕生と発展: ポスト量子暗号・仮想通貨・新しい暗号. 近代科学社.
7. Micciancio, D. (2019). Lattice Algorithms and Applications. Commitment Schemes. Available at https://cseweb.ucsd.edu/classes/fa19/cse206A-a/LecCommit.pdf
8. Sakami, Y., Kobayashi, T., Saito, T., & Wahby, R. (2020). Pairing-Friendly Curves, draft-irtf-cfrg-pairing-friendly-curves-08. Available at https://www.ietf.org/archive/id/draft-irtf-cfrg-pairing-friendly-curves-08.
9. 岡本栄司, 岡本健, & 金山直樹. (2007). ペアリングに関する最近の研究動向. 電子情報通信学会 基礎・境界ソサイエティ Fundamentals Review, 1(1), 1_51-1_60.
10. Bethencourt, J. (n.d.). Intro to Bilinear Maps. Available at https://people.csail.mit.edu/alinush/6.857-spring-2015/papers/bilinear-maps.pdf
11. Oveis Gharan, S. (2017). CSE 521: Design and Analysis of Algorithms I. Lecture 7: Schwartz-Zippel Lemma, Perfect Matching. Available at https://courses.cs.washington.edu/courses/cse521/17wi/521-lecture-7.pdf
12. Fleischhacker, N., Hall-Andersen, M., & Simkin, M. (2024). Extractable Witness Encryption for KZG Commitments and Efficient Laconic OT. Cryptology ePrint Archive.
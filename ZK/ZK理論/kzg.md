<center>

  # KZG Commitments: 係数を秘匿した多項式の評価結果の証明

</center>

## はじめに
KZG commitmentsは、polynomial commitments (PC)と呼ばれる暗号スキームの一つです [1]。PCを使うことで、証明者は自身の知るある多項式 $f(x)$が $f(\alpha)=\beta$を満たすことを、 $f(x)$を明らかにせずに検証者に示すことができます。この性質により、PCはゼロ知識証明 (ZKP) [2]や、検証可能な秘密分散 [1]、データ可用性サンプリング (data availability samping; DAS) [3-4]など、より実用的な暗号スキームを構成するために応用されています。特に、PCのうちKZG commitmentsは、検証のための計算量が非常に小さい、具体的には多項式の次数に関わらず一定であるため、計算リソースが限られているEthereumなどのブロックチェーン上でも用いることが可能です [5]。この資料では、PCの一般的な性質と安全性を定義し、その後KZG commitmentsの構成を紹介します。

`Writer: @SoraSuegami`


## 定義
PCは、その名前が示す通り、多項式 (polynomial)に対するcommitmentsスキームになります。そのため、最初にcommitmentsスキームの定義を述べ、その後にPCについて説明します。

### Commitments
Commitmentsスキームは、封筒と同じような性質を持つデジタルデータを実現します [6]。具体的には、最初にメッセージを封筒に入れて後でその封を開けて中身を確認できるように、commitmentsスキームではメッセージからcommitmentというデータを生成し (コミットフェーズ)、その後にそのメッセージがcommitmentに入っていることを確認できるようにするための、openingというデータを生成します (オープンフェーズ)。ここで、封筒の封を開けるまで中のメッセージがわからないのと同様に、commitmentはopeningが開示されるまでメッセージを秘匿します (Hiding)。また、封を開けずに封筒の中のメッセージを入れ替えることができないのと同様に、commitmentの生成時に使われたものとは異なるメッセージに対してopeningを生成することは、commitmentの生成者であってもできません (Binding)。

Commitmentsスキームを応用することで、例えばオンラインじゃんけんを実現することができます。じゃんけんでは、次の2つの要件が求められます。
1. 各プレイヤーは、自分の手を出すまで他のプレイヤーの手を知ることはできない。
2. 各プレイヤーは、一度自分の手を出した後、その手を入れ替えることはできない。

単純に自分の手を表すデータを他のプレイヤーに送信すると、要件1が達成されないことがわかるでしょう。そこで、最初に各プレイヤーは自分の手に対するcommitmentを他のプレイヤーに送信し、全員がcommitmentを送信した後に、それぞれの手のopeningを送信します。この方式は、Hiding、Bindingの性質により、それぞれ要件1、2を達成することができます。以上のように、commitmentsスキームを使うことで、メッセージをある時点まで隠しながら、そのメッセージに紐づくデータを送信することが可能になります。

厳密には、commitmentsスキームは次のアルゴリズムによって定義されます [7]。
- $\textsf{Setup}(1^{\lambda}) \rightarrow \textsf{pp}$: 入力としてセキュリティパラメータ $1^{\lambda}$を受け取り、公開パラメータ $\textsf{pp}$を生成する。
<!-- 1^{\lambda}とはどういうノーテーション？
ビット長がラムダの特定の値をセキュリティパラメータとして使用する，ということ？ 
-->
- $\textsf{Commit}(\textsf{pp}, m, r) \rightarrow c$: 入力として公開パラメータ $\textsf{pp}$とメッセージ $m$、乱数 $r$を受け取り、 commitment $c$を出力する。
- $\textsf{Open}(\textsf{pp}, m, r, c) \rightarrow d$: 入力として公開パラメータ $\textsf{pp}$とメッセージ $m$、乱数 $r$、commitment $c$を受け取り、 opening $d$を出力する。
- $\textsf{Verify}(\textsf{pp}, m, c, d) \rightarrow 1/0$: 入力として公開パラメータ $\textsf{pp}$とメッセージ $m$、commitment $c$、opening $d$を受け取り、1 (受理)または0 (拒否)を出力する。

注意点として、封筒の封は誰でも勝手に開けることができるのに対して、commitmentsスキームは $\textsf{Open}$アルゴリズムが乱数 $r$を必要としているため、**commitment $c$を生成した人だけがその $c$に対するopeningを生成できる**ようになっています。

これらのアルゴリズムが以下の関係を満たすとき、commitmentsスキームは正当性 (correctness)があると言われます。
```math
\begin{align*}
Pr[\textsf{Verify}(\textsf{pp}, m, c, \textsf{Open}(\textsf{pp}, m, r, c))=1] \geq 1 - \textsf{negl}(\lambda)
\end{align*}
```
ただし、 $\textsf{negl}(\lambda)$は無視できるほど小さい確率を表します。

HidingとBindingは、直感的には次のように定義されます。
- Hiding: ランダムなメッセージと乱数 $m, r$に対するcommitment $c \leftarrow \textsf{Commit}(\textsf{pp}, m, r)$を見て、（多項式時間の計算ができる）攻撃者が $m$を正しく推定できる確率は $\textsf{negl}(\lambda)$未満である。
- Binding: 2つの異なるメッセージと乱数のペア $(m_0, r_0), (m_1, r_1)$に対して、 $c_0 \leftarrow \textsf{Commit}(\textsf{pp}, m_0, r_0)$と $c_1 \leftarrow \textsf{Commit}(\textsf{pp}, m_1, r_1)$が $c_0 = c_1$を満たす確率は $\textsf{negl}(\lambda)$未満である。


### Polynomial Commitments
PCスキームは、多項式 $f(x)=\Sigma_{i=0}^n f_i x^i$の係数ベクトル $(f_0, \dots, f_n)$をメッセージとするcommitmentsスキームです [1]。つまり、 $(f_0, \dots, f_n)$からcommitment $c$を生成し、それを開示するopeningを生成します。加えて、PCでは $c$の中の多項式 $f(x)$をある評価点 $\alpha$で評価した結果が $\beta$になる、つまり $f(\alpha)=\beta$であることだけを、 $(f_0, \dots, f_n)$を明かさずに開示することも可能です。この性質により、**多項式そのものを隠したまま、その多項式が特定の条件を満たしていることを他者に証明できます**。

厳密には。PCスキームは次のアルゴリズムによって定義されます [1]。
- $\textsf{Setup}(1^{\lambda}, n) \rightarrow \textsf{pp}$: 入力としてセキュリティパラメータ $1^{\lambda}$と最大の次数 $n$を受け取り、公開パラメータ $\textsf{pp}$を生成する。
- $\textsf{Commit}(\textsf{pp}, f(x), r) \rightarrow c$: 入力として公開パラメータ $\textsf{pp}$と多項式 $f(x)$、乱数 $r$を受け取り、 commitment $c$を出力する。ただし、 $f(x)$の実際のデータは、 $f(x)$の係数ベクトル $(f_0, \dots, f_n)$になる。また、 $f(x)$の次数はアルゴリズム $\textsf{Setup}$で指定された整数 $n$以下でなければならない。
- $\textsf{Open}(\textsf{pp}, f(x), r, c) \rightarrow d$: 入力として公開パラメータ $\textsf{pp}$と多項式 $f(x)$、乱数 $r$、commitment $c$を受け取り、 opening $d$を出力する。
- $\textsf{VerifyPoly}(\textsf{pp}, f(x), c, d) \rightarrow 1/0$: 入力として公開パラメータ $\textsf{pp}$と多項式 $f(x)$、commitment $c$、opening $d$を受け取り、1 (受理)または0 (拒否)を出力する。
- $\textsf{CreateWitness}(\textsf{pp}, f(x), \alpha, r) \rightarrow (\beta = f(\alpha), \omega_{\alpha})$: 力として公開パラメータ $\textsf{pp}$と多項式 $f(x)$、評価点 $\alpha$、乱数 $r$を受け取り、評価結果 $\beta = f(\alpha)$とwitness  $\omega_{\alpha}$を出力します。
- $\textsf{VerifyEval}(\textsf{pp}, c, \alpha, \beta, \omega_{\alpha}) \rightarrow 1/0$: 入力として公開パラメータ $\textsf{pp}$とcommitment $c$、評価点 $\alpha$、評価結果 $\beta$、witness $\omega_{\alpha}$を入力として受け取り、1 (受理)または0 (拒否)を出力する。

これらのアルゴリズムが以下の2つの関係を満たすとき、commitmentsスキームは正当性 (correctness)があると言われます [1]。
```math
\begin{align*}
    &Pr[\textsf{VerifyPoly}(\textsf{pp}, f(x), c, \textsf{Open}(\textsf{pp}, f(x), r, c))=1] \geq 1 - \textsf{negl}(\lambda) \\
    &Pr[\textsf{VerifyEval}((\textsf{pp}, c, \alpha, \textsf{CreateWitness}(\textsf{pp}, f(x), \alpha, r)))] \geq 1 - \textsf{negl}(\lambda)
\end{align*}
```
特に後者の条件は、 $f(\alpha)=\beta$を満たす正当な $f(x)$、 $\alpha$、 $\beta$に対して生成されたwitnessは、十分に高い確率で $\textsf{VerifyEval}$によって受理されることを要請しています。

PCはcommitmentsスキームのBindingに加えて、多項式の評価結果を拘束する、Evaluation Bindingという安全性を満たします [1]。具体的には、攻撃者が1つのcommitment $c$と評価点 $\alpha$、2つの評価結果とwitnessのペア $(\beta, \omega_{\alpha}), (\beta^{'}, \omega_{\alpha}^{'})$を提出した時、 $\textsf{VerifyEval}(\textsf{pp}, f(x), c, \alpha, \beta, \omega_{\alpha})=1$かつ $\textsf{VerifyEval}(\textsf{pp}, f(x), c, \alpha, \beta^{'}, \omega_{\alpha}^{'})=1$になる確率、つまり同じcommitmentで2つの異なる評価結果に対して証明が成功する確率は、 $\textsf{negl}(\lambda)$未満です。また、Hidingは、攻撃者が $c$と $\textsf{VerifyEval}$をパスするような複数の $(\alpha, \beta, \omega_{\alpha})$のタプルを受け取った時、まだ受け取っていない評価点 $\alpha^{'}$については、その評価結果 $f(\alpha^{'})$を無視できる確率 $\textsf{negl}(\lambda)$を除いて推定できないことを保証します。

## KZG Commitmentの構成
### 準備
KZG commitmentの構成を説明するために必要な記号を準備します。自然数 $n \in \mathbb{N}$に対して、 $[n]$は集合 $\{1,2,\dots,n\}$を表します。ただし、 $[0]$は空集合 $\empty$とします。

位数 $q$の有限体を $F_q$、 $F_q$上で定義される楕円曲線を $E(F_q)$でそれぞれ表記します。ECDSAの説明で述べられたように、楕円曲線上の有理点には加算 $P + Q \in E(F_q)$と2倍演算 $2P \in E(F_q)$、これらを組み合わせたスカラー倍 $nP \in E(F_q)$が定義されます。

さらに、BN curveやBLS curveなどいくつかの楕円曲線の種類では、楕円曲線の点同士の乗算に該当する、pairingと呼ばれる演算が可能です [8]。具体的には、2つの有理点の集合 $\mathbb{G}_1, \mathbb{G}_2$とある有限体の集合 $\mathbb{G}_T$に対して、次の関数 $e$のように定義されます。
```math
\begin{align*}
    e: \mathbb{G}_1 \times \mathbb{G}_2 \rightarrow \mathbb{G}_T
\end{align*}
```
ただし、 $\mathbb{G}_1, \mathbb{G}_2$はそれぞれある点 $P \in \mathbb{G}_1, Q \in \mathbb{G}_2$の加算を繰り返して得られる集合として定義され、それぞれ $r$倍すると0に該当する点 $\mathcal{O}$になります。すなわち、
```math
\begin{align*}
    rP &= \mathcal{O}, \\
    rQ &= \mathcal{O}, \\
    \mathbb{G}_1 &= \{\mathcal{O}, P, 2P, \dots, (r-1)P\}, \\
    \mathbb{G}_2 &= \{\mathcal{O}, Q, 2Q, \dots, (r-1)Q\}
\end{align*}
```
同様に、 $\mathbb{G}_T$はある有限体上の整数 $g$の乗算を繰り返して得られる集合として定義され、 $r$乗すると1になります。つまり、
```math
\begin{align*}
    g^r = 1 \\
    \mathbb{G}_T = \{1, g, g^2, \dots, g^{r-1}\}
\end{align*}
```
pairing $e$は次の2つの性質を満たします [9]。
1. 非縮退性
任意の $P \in \mathbb{G}_1$（ $Q \in \mathbb{G}_2$）に対して、 $e(P, Q) = 1$ならば $Q = \mathbb{O}$ ( $P = \mathbb{O}$)である。
2. 双線型性
```math
\begin{align*}
    e(P_1 + P_2, Q) &= e(P_1, Q) e(P_2, Q) \\
    e(P, Q_1 + Q_2) &= e(P, Q_1) e(P, Q_2)
\end{align*}
```

これらの性質より、pairingは $aP$、 $bQ$のスカラーの乗算に対応していることがわかります。
```math
\begin{align*}
    e(aP, bQ) = e(P, bQ)^a = (e(P, Q)^b)^a = e(P, Q)^{ab}
\end{align*}
```

現在実用されているpairingの構成では、 $\mathbb{G}_{T}$は $\mathbb{G}_1, \mathbb{G}_2$と異なる集合であり、効率的に（多項式時間以内に） $g^{x} \in \mathbb{G}_{T}$の要素を $xP \in \mathbb{G}_{1}$や $xQ \in \mathbb{G}_{2}$に変換することはできません。したがって、pairing $e$の乗算結果を $e$の入力として使うことはできないので、**pairingでは1度しか点同士の乗算を行えません**。以上のようなpairingの性質は、bilinear mapとして抽象化されています [10]。

### 基本的な構成
KZG commitmentsは、端的には剰余の定理の関係が成り立つことをpairingを用いて検証することで、 $f(\alpha)=\beta$を確かめています [1]。剰余の定理は、多項式 $f(x) - f(\alpha)$が $(x-\alpha)$で割り切れることを保証します。つまり、以下の関係を満たす $n-1$次以下の多項式 $q(x)$が一意に存在します。
```math
\begin{align*}
    f(x) - f(\alpha) = q(x)(x - \alpha)
\end{align*}
```
直感的には、証明者が検証者に $f(\alpha)=\beta$になることを示すためには、
証明者がこのような $q(x)$を直接検証者に送信し、検証者には $f(x) - \beta$が $q(x)(x - \alpha)$と多項式として等しいことを確かめてもらえば良いと言えます。しかし、この方法では検証者の計算量・通信量が多項式の次数 $n$に対して線形以上のオーダーで増加するため、効率的ではありません。
<!-- というかその方法だと直接方程式を検証者に渡してしまっていることと同じになってしまい，ダメなのでは？
多項式そのものを明かさずにf(α)=βを示す方法のはず．
直接f(x)渡しても同じな気がする．

検証者が知っているのがq(x)だけだと，どうやってf(x)-βがq(x)(x-α)と多項式として等しいかチェックできるんだろう？
 -->

そこで、多項式として等式を検証する代わりに、あるランダムな一点 $s$で多項式を評価した結果が等しいこと、すなわち $f(s) - \beta = q(s)(s - \alpha)$を検証することを考えます。証明者は $f(s), q(s)$を検証者に送信すると、検証者はこの等式が整数として成立することだけを確かめられれば良いため、検証者の計算量・通信量は $n$に関わらず一定になることがわかります。
<!-- f(s)，q(s)は計算結果の単なるスカラー値であるため．多項式の形で送る必要はなくなる． -->
では、この方式はBindingの性質を実現している、つまり悪意のある証明者が $f(\alpha) \neq \beta$ではない $f(x), \alpha, \beta$に対して有効な $q(s)$を提出することを防いでいるでしょうか？
<!--$f(\alpha) \neq \beta$である $f(x)に対して検証者側がf(α)=βであると誤検出してしまうq(s)を提出することを妨げているでしょうか？
ということ？  -->
仮に $s$がランダムかつ、証明者が $s$を知ることができなければ、Schwartz–Zippelの補題によりBindingが無視できるほど小さい確率を除いて成り立つことを証明できます [11]。具体的には、非零の次数 $n$の多項式で $g(x)=0$となる点 $x$は高々 $n$個しかないため、サイズ $|S|$の集合から評価点 $s \in S$を一様にランダムに取った時に、 $g(s)=0$となる確率、すなわち $s$がその $n+1$個の点のいずれかに一致する確率は高々 $\frac{n}{|S|}$になります。今回の例では、 $g(x) = f(x) - \beta - q(x)(x - \alpha)$、 $|S| \approx 2^{254}$であるため、次数 $n$が100万程度でも $g(s)=0$になる確率は $\frac{1000000}{2^{254}} \approx 3.45447×10^{-71}$で、無視できる程小さい値になります。
<!-- $|S| \approx 2^{254}$ってどこから出てきた？
↓の記述でF_rから取るとなっている．
F_rなどKZGコミットメントを使用する上での仮定を最初に書くと良い．

特にVerifyEvalでペアリングを行っている関係で隠したい多項式f(x)の仮定（係数が有限体F_r上の元？）の記載がほしい．

というかここの記載から，KZGコミットメントを使っている事例で，BLS12-381などペアリングフレンドリーな曲線を使う理由と，元のデータ（複数の数値データ）を多項式の係数にマッピングさせる際の条件（多分F_r上の元じゃないとダメという理解）がわかるようにしたい．

F_r：有限体，r：十分大きな素数，G_1^i： ・・・-->

では、いかにして $s$を証明者に対して隠しながら、同時に証明者に $f(s), q(s)$を計算させることができるでしょうか？これを実現するために、KZG commitmentsは楕円曲線とpairingを利用します。以下がその基本的な構成になります [1, 12]。
- $\textsf{Setup}(1^{\lambda}, n) \rightarrow \textsf{pp}$: 
    1. ランダムな $s$を $\mathbb{F}_r$から取る。
    2. $(P, sP, s^2P, \dots, s^nP) \in \mathbb{G}_1^{n+1}$を計算する。
    3. $(Q, sQ) \in \mathbb{G}_2^{2}$を計算する。
    2. $\textsf{pp}=((P, sP, s^2P, \dots, s^nP)、(Q, sQ))$を出力する。
    <!-- ここで，PとQはG_1,G_2上の任意の点？ 
    ppはペアリングの値？じゃなくて異なる2つの有限体のそれぞれの点のペアか．
    最後の出力されるppはなんの元なのかははっきり記述したほうが理解しやすい．
    pp \in (G_1^n+1, G_2^2) など
    → ペアっていうか，iiとiiiを全部並べた点の集合．有限体が異なる点が混ざっているから()で区切って記載してあるという理解．-->
- $\textsf{Commit}(\textsf{pp}, f(x), r) \rightarrow c$:
<!-- ppじゃなくてP, sP, s^2P,...しか使ってないので上記iiだけあれば十分？ -->
    1. $f(x)$を係数ベクトル $(f_0, f_1, \dots, f_n) \in \mathbb{F}_r$としてパースする。
<!-- ここで隠したい多項式f(x)が登場．
係数ベクトルとは，多項式の各項の係数ってこと？
    f(x) = f_n*x^n + f_{n-1} *x^{n-1} + ... + f_1*x + f_0 -->
    2. $c \leftarrow \Sigma_{i = 0}^{n} f_{i}(s^{i}P) \in \mathbb{G}_1$を計算する。
    <!-- これはcはG_1の元ってことだと思う．←を使っていてさらに右辺にG_1の元という表記があると，何かしら違う集合の元にあらためて飛ばしているふうにも見えてしまうので，cを右辺に持ってきて c \in G_1 としたほうが読みやすい． -->
    3. $c$を出力する。
    <!-- これもなんの元なのかはっきり記述．
    c \in G_1 -->
- $\textsf{CreateWitness}(\textsf{pp}, f(x), \alpha, r) \rightarrow (\beta = f(\alpha), \omega_{\alpha})$:
<!-- これもppのiiiの部分は不要でiiだけで十分に見える． -->
    1. $q(x) = \frac{f(x) - f(\alpha)}{x - \alpha}$を計算する。
    2. $q(x)$を係数ベクトル $(q_0, q_1, \dots, q_n) \in \mathbb{F}_r$としてパースする。
    3. $\omega_{\alpha} \leftarrow \Sigma_{i = 0}^{n} q_{i}(s^{i}P) \in \mathbb{G}_1$を計算する。
    <!-- 上記と同じく右辺に持ってきた方が良い -->
    4. $\omega_{\alpha}$を出力する。
<!-- \omega_alpha \in G_1 
    ここまで証明者が行う作業．
-->
- $\textsf{VerifyEval}(\textsf{pp}, c, \alpha, \beta, \omega_{\alpha}) \rightarrow 1/0$:
<!-- ここの処理のみ検証者が行う検証作業．
ここはP, sP,...もQも使っているからppが必要なのでよい． -->
    1. $e(c - \beta P, Q) = e(\omega_{\alpha}, (sQ) - \alpha Q)$が成り立てば1を、そうでなければ0を出力します。
<!-- eはペアリング写像． -->
上記の構成は次のように正当です。
```math
\begin{align*}
    e(c - \beta P, Q) &= e(\Sigma_{i = 0}^{n} f_{i}(s^{i}P) - \beta P, Q) \\
    &= e((f(s) - \beta)P, Q) \\
    &= e(P,Q)^{f(s)-\beta}\\
    &= e(P,Q)^{q(s)(s-\alpha)}\\
    &= e(q(s)P, ((sQ) - \alpha)Q)\\
    &= e(\omega_{\alpha}, (sQ) - \alpha Q)
\end{align*}
```

また、証明者は $f(s)P, q(s)P$を計算できるにも関わらず、離散対数問題の困難性より $\textsf{pp}$から $s$を知ることはできません。なぜなら、 $\textsf{pp}$は証明者・検証者とは独立した信頼できる第三者によって生成され、その第三者は $\textsf{pp}$の生成後にスカラー $s$を破棄するからです。なお、実際にそのような第三者を用意することは難しいため、複数のユーザがそれぞれスカラー $s_i$を提供し、それらの総積 $s=\prod s_i$を $\textsf{pp}$のスカラーとして用いることが多いです [13]。このようなプロトコルはtrusted setup ceremonyと呼ばれ、少なくとも一人の参加者が誠実で自身のスカラーを破棄すれば、誰も $\textsf{pp}$のスカラーを知ることはできません。
<!-- ということは上でのppはやはり，P, sP, s^2P,...s^nP, Q, sQということか． -->

以上のように、KZG Commitmentsは $s$を証明者に対して隠しながら、証明者に $f(s), q(s)$を計算させるという機能を実現しています。よって、先述のSchwartz–Zippelの補題を用いた議論と合わせると、これはBindingを実現していると言えます。ただし、原論文 [1]の定理3.2で述べられているBindingの証明は、Schwartz–Zippelの補題ではなく、t-SDH仮定という計算量困難性の仮定に基づいていていることに注意してください。

上記の構成は、部分的にHidingを満たしています。具体的には、 $f(x)$の係数のうち少なくとも一つが十分にランダムである、例えばある $f_i$が254 bit程度の位数の有限体 $F_r$からランダムに取られる場合であれば、攻撃者が $c$と次数 $n$以下の評価点・評価結果から $f(x)$の係数や他の評価結果を推定することは困難だと言えます。しかし、例えば次数が $n=31$で各係数が0もしくは1の場合、係数ベクトルの候補は $2^{32}$個しかないため、攻撃者は $\textsf{pp}$から各候補に対応する $2^{32}$個のcommitmentをそれぞれ生成できます。そのため、どの候補のものが渡されたcommitmentと一致するか確かめることによって、攻撃者はそのcommitmentに対応する $f(x)$の係数および任意の評価結果 $f(\alpha)$を知ることができます。この問題を修正するために、原論文 [1]は3.3章でランダムな多項式で元の多項式を秘匿した構成を提案しています。ただし、その構成は検証者がペアリング $e$を3回計算する必要があるため、検証者の計算量を増やしています。それに対して、Plonk [2]などZKPの構成では、評価したい点 $\alpha_1,\dots,\alpha_n$で評価結果が0になるような多項式をランダムに生成し、それを $f(x)$に足した多項式に本章で述べたKZG commitmentsの構成を適用することで、 $f(x)$の係数を秘匿しています。


### 複数の評価点・複数の評価結果に対する証明のバッチ検証処理
上述の構成は効率的に $f(\alpha)=\beta$を検証できる一方、複数の多項式を複数の評価点で評価した結果を検証したい場合、単純な方法では各多項式・評価点の組ごとにpairingを実行する必要があります。pairingの計算は楕円曲線の加算やスカラー倍の計算よりも時間がかかるため、KZG commitmentsを多用するためにはpairingの計算回数を減らさなければなりません。そこで、pairingの計算回数を多項式や評価点の個数に関わらず一定(2回)にする圧縮方法を、[2]の3.1章に沿って説明します。

最初に、 $t$個の異なる多項式 $f^1(x), \dots, f^t(x)$を共通の評価点 $\alpha$で評価した結果を検証する場合を考えます。証明者と検証者は次のプロトコルを実行します。
1. 証明者が各多項式のcommitments $c_1 = \Sigma_{j=0}^{n} f^1_{j}(s^{j}P), \dots, c_t = \Sigma_{j=0}^{n} f^t_{j}(s^{j}P)$を生成し、それらを検証者に送信する。
2. 検証者は乱数 $\gamma$を $\mathbb{F}_r$から取り、それを証明者に送信する。
3. 証明者は多項式 $q(x) \leftarrow \Sigma_{i \in [t]} \gamma^{i-1} \cdot \frac{f^i(x)-f^i(\alpha)}{x-\alpha}$を計算し、 $\omega \leftarrow \Sigma_{j=0}^{n} q_i(s^jP) \in \mathbb{G}_1$を検証者に送信する。
4. 検証者は以下の点 $F, v \in \mathbb{G}_1$を計算する。
```math
\begin{align*}
    F \leftarrow \Sigma_{i \in [t]} \gamma^{i-1} c_i\\
    v \leftarrow \Sigma_{i \in [t]} \gamma^{i-1} \beta_i P
\end{align*}
```
5. 検証者は以下の等式が成り立つ場合に1を、そうでない場合に0を出力する。
```math
\begin{align*}
    e(F-v, Q) = e(\omega, sQ - \alpha Q)
\end{align*}
```

なお、証明者と検証者は何度か通信を繰り返してプロトコルを実行していますが、これはFiat-Shamir変換を用いて非対話的なプロトコル、つまり証明者が一度データを送信すれば通信を必要とせずにそれを検証できるものに変換できます。具体的には、検証者に乱数を選ばせる代わりに、証明者はその前ステップまでで本来検証者に送信するデータ（例: ステップ2では、ステップ1で送信されるcommitments）のハッシュ値を計算し（厳密には、ランダムオラクルと呼ばれる理想化された関数の機能）、その出力を検証者が選んだ乱数と見做します。検証者は、受け取ったデータから同様にハッシュ関数を計算することで、証明者が使用した乱数が正当なものであることを確かめられます。証明者はハッシュ値の原像の衝突を見つけない限り、その乱数を変えずにハッシュ関数へ入力されたデータを変えることが困難であるため、このように生成される乱数は、検証者が対話的に選択した乱数と同じように扱っても問題ないと見なされています。

上述の構成は明らかに正当性を満たします。つまり、すべての $i \in [t]$について $f^i(\alpha)=\beta_i$ならば、5の検証式は成立します。Bindingが成り立つことは、直感的には次のように理解できます。
1. 乱数 $s, \gamma$をそれぞれ変数 $x, y$に置き換えると、5の検証式は次の多項式の関係に対応する。
```math
\begin{align*}
    \Sigma_{i \in [t]}(\Sigma_{j=0}^{n}f_j^ix^j)y^{i-1} - \Sigma_{i \in [t]} \beta_i y^{i-1} = \Sigma_{i \in [t]} (\Sigma_{j=0}^{n}q_j^ix^j) y^{i-1}(x - \alpha)
\end{align*}
```
2. この多項式の等式が成り立つとき、 $y$の各項についてそれぞれ両辺が等しくなるため、すべての $i \in [t]$で $\Sigma_{j=0}^{n}f_j^ix^j - \beta_i = (\Sigma_{j=0}^{n}q_j^ix^j)(x - \alpha)$が同様に成り立つ。
3. いずれかの $i$で $f^i(\alpha) \neq \beta$である、すなわちすべての $q(x)$について $\Sigma_{j=0}^{n}f_j^ix^j - \beta_i \neq (\Sigma_{j=0}^{n}q_j^ix^j)(x - \alpha)$であるならば、あるランダムな点 $x=s, y=\gamma$で $\Sigma_{j=0}^{n}f_j^is^j - \beta_i = (\Sigma_{j=0}^{n}q_j^is^j)(s - \alpha)$が成り立つ確率は、Schwartz–Zippelの補題より無視できるほど小さいことがわかる。

次に、2つの異なる評価点 $\alpha_1, \alpha_2$で、それぞれ $t_1$個、 $t_2$個の多項式 $\{f^i(x)\}_{i \in [t_1]}$, $\{g^i(x)\}_{i \in [t_2]}$を評価した結果を検証することを考えます。上述のステップ1と同様に各多項式のcommitment $(c_1, \dots, c_{t_1})$, $(c^{'}_1, \dots, c^{'}_{t_2})$を検証者に送信した後、証明者と検証者は次のプロトコルを実行します。
1. 検証者は乱数 $\gamma, \gamma^{'}$をそれぞれ $\mathbb{F}_q$から取り、それらを証明者に送信する。
2. 証明者は以下の2つの多項式を計算し、それらのwitness $\omega, \omega^{'}$を検証者に送信する。
```math
\begin{align*}
    h(X) &\leftarrow \Sigma_{i \in [t_1]} \gamma^{i-1} \cdot \frac{f^i(x)-f^i(\alpha)}{x-\alpha} \\
    h^{'}(X) &\leftarrow \Sigma_{i \in [t_2]} {\gamma^{'}}^{i-1} \cdot \frac{g^i(x)-g^i(\alpha^{'})}{x-\alpha^{'}} \\
    \omega &\leftarrow \Sigma_{j=0}^{n} h_j(s^jP) \in \mathbb{G}_1 \\
    \omega^{'} &\leftarrow \Sigma_{j=0}^{n} h^{'}_j(s^jP) \in \mathbb{G}_1
\end{align*}
```
3. 検証者は乱数 $r^{'}$を $\mathbb{F}_q$から取り、次の値を計算する。
```math
\begin{align*}
    F \leftarrow (\Sigma_{i \in [t_1]} \gamma^{i-1}c_i - (\Sigma_{i \in [t_1]} \gamma^{i-1}\beta_i)P) + r^{'}(\Sigma_{i \in [t_2]}  {\gamma^{'}}^{i-1} c^{'}_i - (\Sigma_{i \in [t_2]}  {\gamma^{'}}^{i-1}\beta^{'}_i)P)
\end{align*}
```
4. 検証者は以下の等式が成り立つ場合に1を、そうでない場合に0を出力する。
```math
\begin{align*}
    e(F + \alpha \omega + r^{'} \alpha^{'} \omega^{'}, Q) = e(\omega + r^{'} \omega^{'}, sQ)
\end{align*}
```

同様にFiat-Shamir変換を適用することで、上記の構成を非対話的なプロトコルに変換することが可能です。正当性は次のように確かめられます。
```math
\begin{align*}
    &e(F + \alpha \omega + r^{'} \alpha^{'} \omega^{'}, Q)\\
    &= [(\Sigma_{i \in [t_1]} \gamma^{i-1} f(s) - (\Sigma_{i \in [t_1]} \gamma^{i-1}\beta_i)) + r^{'}(\Sigma_{i \in [t_2]}  {\gamma^{'}}^{i-1} g(s) - (\Sigma_{i \in [t_2]}  {\gamma^{'}}^{i-1}\beta^{'}_i)P) + \alpha h(s) + r^{'} \alpha^{'} h^{'}(s)]_T \\
    &= [(\Sigma_{i \in [t_1]} \gamma^{i-1} (f^i(s) - \beta_i) \cdot \alpha \cdot \frac{f^i(s)-f^i(\alpha)}{s-\alpha}) + r^{'}(\Sigma_{i \in [t_1]} {\gamma^{'}}^{i-1}(g^i(s) - \beta^{'}_i) \cdot \alpha^{'} \cdot \frac{g^i(s)-g^i(\alpha^{'})}{s-\alpha^{'}})]_T \\
    &= [(\Sigma_{i \in [t_1]} \gamma^{i-1} s \cdot \frac{f^i(s)-f^i(\alpha)}{s-\alpha}) + r^{'}(\Sigma_{i \in [t_1]} {\gamma^{'}}^{i-1}s \cdot \frac{g^i(s)-g^i(\alpha^{'})}{s-\alpha^{'}})]_T\\
    &= e(h(s) + r^{'}h^{'}(s), sQ)\\
    &= e(\omega + r^{'} \omega^{'}, sQ)
\end{align*}
```
Bindingも、前述の評価点が全て共通だった場合と同様に、2つの多項式が等しくない場合に乱数で評価した結果が一致する確率は無視できるほど小さいという性質から理解できます。このように、複数の等式を個別に検証する代わりに、左辺・右辺のランダムな線形和 (random linear combination)が一致するかを検証することで、pairingの計算回数を減らすことが可能です。このテクニックを応用することで、3個以上の評価点に関する検証を同様にバッチ化することができます。

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
13. Ethereum Community. (2022). KZG Ceremony. SUMMONING GUIDES. Available at https://ceremony.ethereum.org/
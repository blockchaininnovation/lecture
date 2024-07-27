<center>

  # ゼロ知識証明の理論 後編: Plonkの詳細

</center>

## はじめに
前編では、ゼロ知識証明 (ZKP)の構成の一つであるPlonkを例に、証明したNP問題がより証明するのが容易な形式、具体的には多項式の評価に対する証明に変換される過程を追いました [1]。後編では、その証明をKZG commitments [2]で行うために必要なテクニックを紹介します。

## KZG Commitmentsを適用するための課題
算術回路の充足可能性問題として記述されたNP問題は、ゲートごとの制約条件を表すgate constraintsとゲート間の制約条件を表すcopy constraintsに変換され、それらの証明は次の多項式の関係によってそれぞれ表されるのでした。
```math
\begin{align*}
&\forall x \in \{\textbf{g}^0, \dots, \textbf{g}^{n-1}\}, \textbf{q}_L(x)\textbf{a}(x) + \textbf{q}_R(x)\textbf{b}(x) + \textbf{q}_O(x)\textbf{c}(x) + \textbf{q}_M(x)\textbf{a}(x)\textbf{b}(x) + \textbf{q}_C(x) = 0\\
&\forall x \in \{\textbf{g}^0, \dots, \textbf{g}^{n-1}\}, z(x\textbf{g})(\textbf{a}(x) + \beta \sigma(i) + \gamma)(\textbf{b}(x) + \beta \sigma(n+i) + \gamma)(\textbf{c}(x) + \beta \sigma(2n+i) + \gamma) - z(x)(\textbf{a}(x) + \beta \textbf{g}^i + \gamma)(\textbf{b}(x) + \beta k_1 \textbf{g}^i + \gamma)(\textbf{c}(x) + \beta k_2 \textbf{g}^i + \gamma) = 0
\end{align*}
```

これらの多項式の関係にKZG commitmentsを適用しようとすると、次の3つの問題があります。
1. 複数の評価点 $\forall x \in \{\textbf{g}^0, \dots, \textbf{g}^{n-1}\}$で評価結果が0になることを証明しなければならない。しかし、証明のサイズは$n$によらず一定になる必要がある。
2. 検証者が指定する多項式 (例: $\textbf{q}_L(x), \sigma(i)$)と証明者が提供する多項式 $\textbf{a}(x), \textbf{b}(x), \textbf{c}(x)$の積を扱う必要がある。しかし、KZG commitmentsはどちらも$\mathbb{G}_1$上の要素になるため、2つのKZG commitmentsを直接かけることは不可能である。
3. 共通の多項式 $\textbf{a}(x), \textbf{b}(x), \textbf{c}(x)$がそれぞれの式で使われていることを証明する必要がある。しかし、これらの多項式は証明者の秘密情報 (例: 算術回路への入力)を含むため、直接検証者に公開することはできない。
<!-- 
上記の式は，多項式と評価点が混ざっているためKZGコミットメントに適用しようとしたときに使うイメージがわかない．
なので単純に多項式fは何なのか，ということと評価点αがどこでf(α)=βを示すべきは何なのかが不明瞭．

説明を読むとαがg_0, ... , g_{n-1}のn個の点で上の式と下の式がβ=0になる，という複数の評価点の話が出てくる．


これら多項式が与えられたときに示すことは以下：
・ $\forall x \in \{\textbf{g}^0, \dots, \textbf{g}^{n-1}\}$で $f(x)=0$ であること
・

 -->
##　複数の評価点でのKZG Commitments
一つ目の問題は、複数の評価点での評価結果に対して証明が行えるように、KZG Commitmentsを修正することで解決できます。KZG commitmentsの最も基本的な形を復習すると、ある多項式 $f(x)$のcommitment $c$について $f(\alpha)=\beta$であることを証明するためには、商の多項式 $q(x) = \frac{f(x) - f(\alpha)}{x - \alpha}$を計算し、それを $\mathbb{G}_1$上でランダムな点で評価した結果である $\pi \leftarrow \Sigma_{i = 0}^{n} q_{i}(s^{i}P)$を、証明として提出します。
<!-- ランダムな点というか，隠蔽化された乱数sで構築されている巡回群G_1中の点列． 
KZGコミットメントのときにはπじゃなくてω_α と表記されていた点．
-->
検証者は $e(c - \beta P, Q) = e(\pi, (sQ) - \alpha Q)$が成り立つ場合に、証明を受理します。

ここで、 $\forall \alpha^{'} \in \{\alpha_1, \dots, \alpha_n\}$で $f(\alpha^{'})=0$ならば、因数定理より次の式を満たす多項式 $q(x)$が存在します。
```math
  f(x) = q(x)(x - \alpha_1)(x - \alpha_2) \cdot (x - \alpha_n)
```
<!-- \cdotは\cdotsの誤記？ -->
特に $\{\alpha_1, \dots, \alpha_n\} = \{\textbf{g}^0, \dots, \textbf{g}^{n-1}\}$の場合、この集合の全ての元が1の累乗根である、つまり $x^n = 1$を満たすので、 $(x - \alpha_1)(x - \alpha_2) \cdot (x - \alpha_n) = x^n-1$と単純になります。
<!-- 
うーーーーん．
直感的にわからん．

例えばn=3のとき，
x^3 = 1
このときx=1もしくはx=g, g^2 （gは1の原始三乗根）

右辺をゼロにして，
x^3-1=0
(x-1)(x-g)(x-g^2) = 0
が成り立つ．
1=g^0なので
(x-g^0)(x-g^1)(x-g^2) = 0
とかける．
（任意のxで上記の式が成り立つのではなく，x^3=1という方程式から導かれる式なことに注意）

で，任意のxに対する式の因数分解を考えるとgを使って
x^3-1=(x-g^0)(x-g^1)(x-g^2)
が成り立つ．
これは右辺を愚直に計算すると示せる．

一般化して，gは1の原始n乗根で同じような性質が成り立つことを
円分多項式の性質として知られているらしい．
https://mathtano.com/enbun/

 -->


したがって、証明者は
```math
q(x) = \frac{f(x)}{x^n-1}
```
に対する証明 $\pi \leftarrow q(s)P$を提出し、検証者は
```math
e(c, Q) = e(\pi, s^nQ - Q)
```
が成り立つかを確かめることで、 $\forall x \in \{\textbf{g}^0, \dots, \textbf{g}^{n-1}\}$で $f(x)=0$ であることを証明・検証できます。
<!-- 
さっきは評価点αでの多項式qをfを変形して導き出して，f(α)=βを証明していたのに対して，今回は任意の
x \in \{\textbf{g}^0, \dots, \textbf{g}^{n-1}\}
かつgが原始ｎ乗根であるという性質 ・・・(*)

をつかってfを変形してqを導き出した．
なので，q導出のための仮定である(*)であることを導ける．


ここは，KZGコミットメントのページでのウィットネスの表記と合わせて，さらにおなじq()だと混乱するので適宜変数は置き換えたほうがわかりやすそう．

 -->

## ランダム評価点による多項式の定数化と線形化
2つ目と3つ目の問題は、多項式をランダムな点で評価した結果の定数に置き換え、多項式の積を含む等式を複数の多項式の線形和に変換することで解決できます。簡単な例として、証明者はある3つの多項式 $h_1(x), h_2(x), h_3(x)$について、それらのKZG commitments $c_1 = h_1(s)P, c_2 = h_2(s)P, c_3 = h_3(P)$を事前に検証者に渡し、これらが多項式として $h_1(x)h_2(x) - h_3(x) = 0$を満たすことを、検証者に証明したいとしましょう。単純な解決策は、検証者が選んだランダムな評価点 $\delta$でそれぞれの多項式を評価した結果 $t_1 = h_1(\delta), t_2 = h_2(\delta), t_3 = h_3(\delta)$を、証明者に計算・証明させ、検証者は$t_1 t_2 - t_3 = 0$というスカラーの等式が成り立つことを確かめるという方法です。
<!-- その評価点だけで成り立っていることだけ示せばよい場合はこれでよい．任意のxで成り立つことを示すには違う方法じゃないとダメ？ -->
しかし、これでは多項式の種類に比例した数だけ、証明者がそれぞれの多項式の評価結果を送信する必要があります。

そこで、証明者と検証者は、commitmentsが送信された後に次のようなプロトコルを行うことで、この関係を効率的に証明・検証できます。
1. 検証者はランダムな評価点 $\delta \in \mathbb{F}_r$を取り、これを証明者に送信する。
<!-- 
ランダムといいつつインタラクティブな証明・検証はないはずなので，適当な乱数で一致する数をお互いに取っているはず．
→ KZGコミットメントのページのフィアットシャミアのところか．

-->
2. 証明者は$h_1(\delta)=t_1$という評価結果について、証明$\pi_1$を検証者に送信する。
<!-- 
表記の問題だが，証明→ウィットネス，π→ωで統一． -->
3. さらに、証明者は $h^{'}(x) = t_1h_2(x) - h_3(x)$という多項式の、 $h^{'}(\delta) = 0$という評価結果について、証明$\pi_2$を検証者に送信する。
<!-- 
h_1(x)h_2(x)-h_3(x)=0を満たすことを示したいから
h'(x)=h_1(x)h_2(x)-h_3(x)
として
h'(δ) = h_1(δ)h_2(δ)-h_3(δ) = t_1 h_2(δ)-h_3(δ)
が0になるかどうかを検証者に対して証明しなきゃならないから，h'(δ)=0 ということが成り立つ証明を作る．

ということはKZGコミットメントについて，多項式はh_1()とh'()の2つを使ってコミットメントはc_1とc'を使う，という2回やるイメージ．
 -->
4. 検証者は$\pi_1$をcommitments $c_1$、$\pi_2$をcommitments $c^{'} = t_1c_2 + c_3$でそれぞれ検証する。

4の検証が成り立つのは、KZG commitmentsが加法準同型性、すなわち**多項式の和のcommitmentsは多項式のcommitmentsの和と等しい**という性質を満たすためです [1-2]。これにより、証明者は別の多項式との積になっている多項式についてのみその評価結果を送信すれば良く、線形関係の多項式については、それらの線形和の多項式の評価結果を送ることが可能です。

上記の例と同様に、Plonkの証明者は多項式 $\textbf{a}(x), \textbf{b}(x), \textbf{c}(x)$を、検証者が選んだランダムな評価点 $\delta$で評価し、その評価結果 $\bar{a} = \textbf{a}(\delta), \bar{b} = \textbf{b}(\delta), \bar{c} = \textbf{c}(\delta)$とそれらに対するKZG commitmentsの証明を検証者に送信します。さらに、商の多項式 
```math
q_1(x) = \frac{\textbf{q}_L(x)\textbf{a}(x) + \textbf{q}_R(x)\textbf{b}(x) + \textbf{q}_O(x)\textbf{c}(x) + \textbf{q}_M(x)\textbf{a}(x)\textbf{b}(x) + \textbf{q}_C(x)}{x^n - 1}
```
に対応する証明 $\pi_{q_1}$も送信します。検証者は、KZG commitmentsの線形和
```math
c^{'} = \bar{a}\textbf{q}_L(s)P + \bar{b}\textbf{q}_R(s)P + \bar{c}\textbf{q}_O(s)P +  \bar{a}\bar{b}\textbf{q}_M(s)P + \textbf{q}_C(s)P
```
に対して、
```math
e(c^{'}, Q) = e(\pi_{q_1}, s^nQ - Q)
```
が成り立つことを検証することで、gate contraintsに対応する多項式が正当であることを確かめられます。copy constraintsについても、同様の手法で証明・検証できます。なお、 $\bar{a}, \bar{b}, \bar{c}$は単なるスカラーであるため、多項式 $\textbf{a}(x), \textbf{b}(x), \textbf{c}(x)$の情報を漏らしません。よって、以上の方法は二つ目の問題を解決しています。また、証明者から渡された $\bar{a}, \bar{b}, \bar{c}$を適切に使い回す、例えば $\textbf{q}_M(s)P$の係数には積 $\bar{a}\bar{b}$を使う、gateとcopy constraintsの多項式で共有の$\bar{a}, \bar{b}, \bar{c}$を用いることで、検証者は同じ多項式 $\textbf{a}(x), \textbf{b}(x), \textbf{c}(x)$を使うように強制することができます。したがって、この手法は三つ目の問題も解決しています。

実際のPlonkのプロトコルでは、複数の評価点・複数の評価結果に対するKZG commitmentsのバッチ処理を行うことで、さらに証明のサイズと検証者の計算量を削減しています。厳密なプロトコルは[1]の第8章をご覧ください。


<!-- 
ここまで理解すると，
circomで使っているproving key, verification keyは

proving key：パブリックパラメータpp（トラステッドセットアップで作られた乱数sを含む．）
verification key: ppとコミットメント値c

という理解をすればよいか．


あとは，パブリックインプットとプライベートインプットをどう表現しているかがわからない．
a(), b()のコミットメントを作るときに一部を開示している？
→ https://research.metastate.dev/plonk-by-hand-part-1/
この辺を読んでいると，ゲート制約を作るときにパブリックインプットの値は制約を増やすらしいが・・・
> To bind a variable to a public value, we let qR, qO, qM equal to 0, qL equal to 1, and qC equal to the public value.
ただ，制約増やしても別にその制約を公開するわけではないので，謎．

というかOutputも回路そのものの共有と検証もよく考えるとどうやってやるんだろう？
上記のプロセスだと謎多項式が一定の制約を満たしているということだけしか検証できてなくて，多項式の情報が一切検証者側に渡ってない気がする．
-->


## References
1. Gabizon, A., Williamson, Z. J., & Ciobotaru, O. (2019). Plonk: Permutations over lagrange-bases for oecumenical noninteractive arguments of knowledge. Cryptology ePrint Archive.
2. Kate, A., Zaverucha, G. M., & Goldberg, I. (2010). Polynomial commitments. Tech. Rep.

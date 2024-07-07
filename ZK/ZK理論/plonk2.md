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

##　複数の評価点でのKZG Commitments
一つ目の問題は、複数の評価点での評価結果に対して証明が行えるように、KZG Commitmentsを修正することで解決できます。KZG commitmentsの最も基本的な形を復習すると、ある多項式 $f(x)$のcommitment $c$について $f(\alpha)=\beta$であることを証明するためには、商の多項式 $q(x) = \frac{f(x) - f(\alpha)}{x - \alpha}$を計算し、それを $\mathbb{G}_1$上でランダムな点で評価した結果である $\pi \leftarrow \Sigma_{i = 0}^{n} q_{i}(s^{i}P)$を、証明として提出します。検証者は $e(c - \beta P, Q) = e(\pi, (sQ) - \alpha Q)$が成り立つ場合に、証明を受理します。

ここで、 $\forall \alpha^{'} \in \{\alpha_1, \dots, \alpha_n\}$で $f(\alpha^{'})=0$ならば、因数定理より次の式を満たす多項式 $q(x)$が存在します。
```math
  f(x) = q(x)(x - \alpha_1)(x - \alpha_2) \cdot (x - \alpha_n)
```
特に $\{\alpha_1, \dots, \alpha_n\} = \{\textbf{g}^0, \dots, \textbf{g}^{n-1}\}$の場合、この集合の全ての元が1の累乗根である、つまり $x^n = 1$を満たすので、 $(x - \alpha_1)(x - \alpha_2) \cdot (x - \alpha_n) = x^n-1$と単純になります。したがって、証明者は
```math
q(x) = \frac{f(x)}{x^n-1}
```
に対する証明 $\pi \leftarrow q(s)P$を提出し、検証者は
```math
e(c, Q) = e(\pi, s^nQ - Q)
```
が成り立つかを確かめることで、 $\forall x \in \{\textbf{g}^0, \dots, \textbf{g}^{n-1}\}$で $f(x)=0$ であることを証明・検証できます。

## ランダム評価点による多項式の定数化と線形化
2つ目と3つ目の問題は、多項式をランダムな点で評価した結果の定数に置き換え、多項式の積を含む等式を複数の多項式の線形和に変換することで解決できます。簡単な例として、証明者はある3つの多項式 $h_1(x), h_2(x), h_3(x)$について、それらのKZG commitments $c_1 = h_1(s)P, c_2 = h_2(s)P, c_3 = h_3(P)$を事前に検証者に渡し、これらが多項式として $h_1(x)h_2(x) - h_3(x) = 0$を満たすことを、検証者に証明したいとしましょう。単純な解決策は、検証者が選んだランダムな評価点 $\delta$でそれぞれの多項式を評価した結果 $t_1 = h_1(\delta), t_2 = h_2(\delta), t_3 = h_3(\delta)$を、証明者に計算・証明させ、検証者は$t_1 t_2 - t_3 = 0$というスカラーの等式が成り立つことを確かめるという方法です。しかし、これでは多項式の種類に比例した数だけ、証明者がそれぞれの多項式の評価結果を送信する必要があります。

そこで、証明者と検証者は、commitmentsが送信された後に次のようなプロトコルを行うことで、この関係を効率的に証明・検証できます。
1. 検証者はランダムな評価点 $\delta \in \mathbb{F}_r$を取り、これを証明者に送信する。
2. 証明者は$h_1(\delta)=t_1$という評価結果について、証明$\pi_1$を検証者に送信する。
3. さらに、証明者は $h^{'}(x) = t_1h_2(x) - h_3(x)$という多項式の、 $h^{'}(\delta) = 0$という評価結果について、証明$\pi_2$を検証者に送信する。
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



## References
1. Gabizon, A., Williamson, Z. J., & Ciobotaru, O. (2019). Plonk: Permutations over lagrange-bases for oecumenical noninteractive arguments of knowledge. Cryptology ePrint Archive.
2. Kate, A., Zaverucha, G. M., & Goldberg, I. (2010). Polynomial commitments. Tech. Rep.

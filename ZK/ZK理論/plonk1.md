<center>

  # ゼロ知識証明の理論 前編: Plonkの概要

</center>

## はじめに
ゼロ知識証明 (ZKP)は、証明者と検証者という2者間で行われる暗号プロトコルで、証明者がある問題の答えを知っていることを、それ以外の情報を明らかにせずに検証者に証明することを可能にします。その特徴により、近年ZKPはプライバシーを保護した仮想通貨の送金 [1-3]やブロックチェーンのスケーリング [4-5]に応用されています。この資料では、ZKPスキームの一つであるPlonk [6]の構成を学ぶことで、ZKPが成立する理由を理解することを目指します。

## Plonkの概要
Plonkの証明者は、あるNP問題の答えを知っていることの証明を生成することができ、その証明のデータサイズや検証のための計算量は、**問題の複雑さによらず一定**になります（簡潔性）。また、証明者は検証者に証明データを一方的に送れば良いという特徴（非対話性）を持ちます。ZKPのうち、これらの特徴を満たすものはzero-knowledge succinct non-interactive argument of knowledge (zk-SNARKs)と呼ばれ [7]、Plonkはzk-SNARKsの構成の一つになります [6]。（厳密には、zk-SNARKsはさらに、知識の健全性という安全性の定義も満たす必要があります [7]。）

そのような証明を生成する過程で、NP問題は次のように効率的な証明が可能な形式へ変換されます。
1. 算術回路の充足可能性問題として定義されるNP問題
2. 制約条件 (constraints)
3. 多項式の評価

最後に問題を多項式の評価へ変換することで、KZG commitments [8]を利用して効率的に証明を行うことができます。具体的には、ある $\alpha$と $\beta$に対して $f(\alpha)=\beta$を満たす多項式 $f(x)$を知っていることを証明すれば良いならば、 $f(x)$のKZG commitmentに対する証明を使えばいいと言えます。その証明のデータサイズや検証のための計算量は一定になるため、Plonkの証明検証も簡潔になります。以降では、問題の変換の各過程をそれぞれ説明し、問題が（無視できる程小さい確率を除いて）等価になっていることを確かめます。
<!-- 
証明者が示したい問題（実行したい処理とその結果）を算術回路で表現する，というところから始まる．
算術回路とは2つの入力から1つの出力で，足し算もしくは掛け算で定義される「ゲート」を組み合わせてつくる一連の処理のこと．

まずは算術回路を作るところから始まる．
すべての起点はここから．
逆に言うと，PlonKでゼロ知識証明を行いたい人はこの算術回路を作るだけであとはオートマチックに処理がされる．
circom言語とかを使ってプログラミングを行っているのはこの算術回路を作っている作業．

ということは起点はcircom書くところだな．
circomで記述 → 算術回路へ変換 → 制約条件に変換 → 多項式に変換 → KZGコミットメント

最終目標はKZGコミットメント．
すなわち，算術回路をどうにかして多項式に変換する．
その過程で制約条件への変換を行う．

KZGコミットメントを使用するためにあれこれ変換可能なものが算術回路まで行き届いたという感じ．

 -->
## 算術回路の充足可能性問題
<!-- 
NP問題を算術回路に変換する問題のことを算術回路の充足可能性問題で，NP問題というのはいわゆる多くのプログラミング的な処理，すなわちプログラミング言語を使って作っている関数のインプット（引数）に対するアウトプット（返り値）を出力する，
-->
証明したいNP問題は、最初に有限体上の算術回路の充足可能性問題として定義されます。算術回路とは、論理回路と同じように、2入力1出力の加算・乗算を表すゲートを組み合わせることで、関数の計算を定義するモデルです。証明者は、自身が持つ値を回路に入力したとき、回路の出力が検証者によって指定された値になることを示します。

回路の充足可能性問題は任意のNP問題を表すことができ、回路や回路の出力がNP問題の命題 (instance)、回路への入力や途中のゲートの入出力がNP問題の解 (witness)に対応します [9]。ただし、回路の入力の一部が命題に含まれることもあります。算術回路が十分に幅広い種類の関数を表現できることを直感的に理解するために、有限体上の四則演算が加算・乗算の組み合わせによって表せることを確かめます。
- 加算: 加算ゲートを直接用いて計算できます。
- 減算: 位数 $r$の有限体上では、 $a + (r-b) = r = 0 \mod q$が成り立つため、 $-b$は  $r-b$で表すことができます。したがって、 $a - b$という減算は $a$と $r-b$の加算として計算できます。
- 乗算: 乗算ゲートを直接用いて計算できます。
- 除算: $c = a/b$を計算する代わりに、その出力 $c$を新たな入力として与え、 $bc - a$を計算します。検証者は $bc -a=0$であることを確かめることで、 $bc = a$、すなわち $c = a/b$であることを確かめられます。

実際に証明したい問題を算術回路で記述するためのツールとして、circom [10]などZKP専用のプログラミング言語や、arkworks [11]など既存のプログラミング言語で実装されたライブラリが挙げられます。これらのツールの利用者は、通常のプログラミングのように関数の出力を計算する過程を実装するだけではなく、各変数間が満たすべき制約条件を明確に定義する必要があります。例えば、 $c=a/b$を計算するコードを書いた後、 $bc = a$という制約条件を定義します。よって、制約条件を明確に定義していない既存のプログラミング言語での実装を、そのままZKPの回路として流用することは難しいです。近年、この問題を解決するために、ZKVMというシステムが提案・実装されています [12-16]。簡単には、ZKVMはCPUがプログラムを実行する過程を模倣して回路を実装することで、既存のプログラム言語の実装をそのまま実行することを可能にします。ただし、ZKVMはまだ発展途上の技術であり、直接回路を実装する場合と比べてパフォーマスが大きく悪化しうることに注意する必要があります。

また，プログラミング言語での処理を表現する例として仮に以下の関数myfuncという関数を算術回路で表現することを考えてみましょう．
要するに足し算「 $+$  」と掛け算「 $\times$ 」のみの数式で表現することを考えます．
```js
function myfunc(w, a, b) {
    if (w == true) {
        return a * (b + 3);
    } else {
        return a * b;
    }
}
```
これは， $v$ をアウトプットとして以下の算術式に変換すると同等の処理を表現できます．
```math
    v = \omega \times ( a \times (b + 3)) + (1- \omega)\times (a+b)
```

同様にもう少し複雑な以下の例を見てみましょう．
これは1から正の整数 $N$ までの数の和を求める関数です．
```js

N = 10
function sumUpTo() {
    let sum = 0;
    for (let i = 1; i <= N; i++) {
        sum += i;
    }
    return sum;
}
```
これを算術回路に表現し直すと以下のようになります．
最終的な出力値は同じく $v$ で記述しています．
```math
\begin{align*}
    s_0 &= 0 \\
    s_1 &= s_0 + 1 \\
    s_2 &= s_1 + 2 \\
    s_3 &= s_2 + 3 \\
    &... \\
    s_{N-1} &= s_{N-2} + N \\
    v &= s_{N-1} + N \\
\end{align*}
```
for文はNの数分の数式として表現されます．
ここで重要なのは，整数 $N$ は関数の引数として渡すことはできず，事前に固定な値として準備しておく必要があるということです．
以降の処理では，算術回路を固定し様々な変換処理を行います．
そのため，引数によって算術回路そのものが変わってしまう処理は実行することができません．

<!-- 
というか，ここでは上のように数式で表現するのではなく□と矢印からなる図で表現されることを示したほうが良いのか．
で，その□と矢印からなる図を下で，コンストレイントシステムとして数式に変換する，と．
 -->
## 算術回路の制約条件
Plonkの証明者は次に、算術回路の充足可能性問題を、ゲートの入出力間の制約を定義するgate constraintsと、一方の出力がもう一方の入力になっている2つのゲート、つまり同じワイヤを共有するゲートの間で同じ値がコピーされていることを要請するcopy constraintsに変換します。
<!-- この変換処理はKZGコミットメント利用のための多項式に変換するための途中のステップとして重要． -->

各ワイヤの値が両方の種類のconstraintsを満たすならば、（無視できる程小さい確率を除いて）それらは回路を正しく評価した時のワイヤの値と一致します。
<!-- ゲートやワイヤーが文章だけだとよくわからないので，よくある四角と矢印で表現される図が必要．
ここより上の章で書いたほうがいいかも？ 

ここでは算術回路をgate/copy constraintsに「変換」している処理．

gate/copy 制約への変換でできているすべての方程式を満たす
ということが
最初に作られていた算術回路全体の結果と一致（無視できる小さい確率を除いて．


「制約条件」と書くとイメージ湧きにくいが，連立方程式に変換している．
次数は最大2次．
2次なのは多分，ペアリングで一回までしか掛け算できないことによる？

-->

最初に、gate constraintsについて説明します。加算・乗算ゲートの左入力、右入力、出力の値をそれぞれ $x_a, x_b, x_c$とすると、各ゲートが正しく評価された場合にのみ、これらは次の制約条件を満たします。
- 加算: $x_a + x_b - x_c = 0$
- 乗算: $x_a x_b - x_c = 0$
<!-- 
x_a  x_b
   □
   x_c

ゲートが正しく評価された場合にのみこの制約が満たされる，ということはすなわち算術回路として表現されている一つのゲートはこの形式で書き換えが可能ということ．
同値変換（⇔）である．

 -->
また、あるワイヤの値 $x_c$をある定数 $q_c$に固定させたい場合、制約条件 $q_c - x_c = 0$を定義します。以降、定数になるワイヤは便宜的に定数ゲートの出力と見做します。

これらの制約条件を各ゲートに対して定めることで、回路が正しく評価された場合にのみ満たされるような制約条件のリストを作ることができます。ただし、このままだとゲートの種類ごとに制約条件が異なり扱いが面倒なので、ゲートごとに決まるパラメータ $(\mathbf{q}_L, \textbf{q}_R, \textbf{q}_O, \textbf{q}_M, \textbf{q}_C) \in (\mathbb{F}_r^n)^5$を導入して、式の形式を次のように統一します。
```math
    \textbf{q}_{L_i} x_{a_i} + \textbf{q}_{R_i} x_{b_i} + \textbf{q}_{O_i} x_{c_i} + \textbf{q}_{M_i} x_{a_i} x_{b_i} + \textbf{q}_{C_i} = 0
```
加算、乗算、定数ゲートの場合、パラメータの値をそれぞれ $(\textbf{q}_{L_i}, \textbf{q}_{R_i}, \textbf{q}_{O_i}, \textbf{q}_{M_i}, \textbf{q}_{C_i}) = (1, 1, -1, 0, 0), (0, 0, -1, 1, 0), (0, 0, -1, 0, q_C)$ と設定することで、前述の各制約条件と一致することがわかります。したがって、証明したい算術回路のゲート構成に応じてパラメータ $(\textbf{q}_L, \textbf{q}_R, \textbf{q}_O, \textbf{q}_M, \textbf{q}_C)$ を事前に定義し、証明者は各パラメータの値 $\textbf{q}_{L_i}, \textbf{q}_{R_i}, \textbf{q}_{O_i}, \textbf{q}_{M_i}, \textbf{q}_{C_i}$ を満たすようなワイヤの値を知っていることを示すことで、少なくとも各回路を正しく評価したことを証明できます。

次に、copy constraintsについて説明します。これは、あるゲート $i$の出力 $c_i$が別のゲート $j$の入力 $a_j$ (もしくは $b_j$)と等しい場合に、それらのワイヤの値が等しい、つまり $x_{c_i} = x_{a_j}$ (もしくは $x_{c_i} = x_{b_j}$ )であるという制約条件になります。もしワイヤの値がgate constraintsのみで制約される場合、悪意のある証明者は $x_{c_i}$とは異なる $x_{a_j}$を使うことで、回路を正しく評価していないにも関わらず全ての制約条件を満たすことができてしまうため、copy constraintsも追加で必要になります。

では、どのようにしてcopy constraintsを単純な等式で表すことができるでしょうか？各ワイヤごとにワイヤ間の等式 $x_{c_i} = x_{a_j}$ を定めると、等式の種類がワイヤ数に比例して増えるため望ましくありません。そこで、値のコピーが全てのワイヤで成り立つことだけを確認すればいいという点を利用して、ワイヤ間の等式をまとめて確認します。具体的には、次のような累積値 (アキュムレータ)を計算します。ただし、 $\beta, \gamma$は証明者が予測できないような乱数です。便宜的に、 $0 \leq i < n, n \leq i < 2n, 2n \leq i < 3n$ の場合、 $x_i$はそれぞれ $x_{a_i}, x_{b_i}, x_{c_i}$ を表すとし、 $\sigma(i)$は、 $i$番目のワイヤの値がコピーされるワイヤを表します。
```math
\begin{align*}
    F &= \prod_{i=0}^{3n-1} (x_{i} + \beta \cdot i + \gamma) \\
    G &= \prod_{i=0}^{3n-1} (x_{i} + \beta \cdot \sigma(i) + \gamma) 
\end{align*}
```
$x_i = x_{\sigma(i)}$ が全ての $i$で成り立つならば、累積値の値は $x_i$と $x_{\sigma(i)}$を入れ替えても同じように計算されるはずなので、 $F=G$ が成り立ちます。
逆に $F=G$ならば、高い確率で $x_i = x_{\sigma(i)}$ が全ての $i$ で成り立ちます。直感的には、 $j = \sigma(i)$ ならば $i = \sigma(j)$であるかつ、 $\beta, \gamma$のランダム性により $F, G$ の因数で $\beta$ の係数が同じもの同士がそれぞれ等しくなる必要があるので、  $x_i + \beta \cdot i + \gamma = x_{\sigma(i)} + \beta \cdot i + \gamma$ が高い確率で成り立つためです。このように、 $3n$個のワイヤのコピーの制約条件は、単縦な累積値の等式一つで表されます。


## 制約条件から多項式へ
最後に、KZG commitmentsを利用するために、制約条件を多項式へ変換します。その基礎となるテクニックが多項式補間です。ある2点を通る直線 (1次多項式)、3点を通る2次曲線 (2次多項式)を一意に決められるように、ある $n$点を通る $n-1$次多項式を求めることができます。よって、例えば左入力のワイヤの値 $(0, x_{a_0}), \dots, (n-1, x_{a_{n-1}})$ から $a(i) = x_{a_i}$ を満たす $n-1$次多項式 $a(x)$ を求められます。なお、実際のプロトコルでは、$\textbf{g}^n = 1$ を満たす1の累乗根 $\textbf{g}$を用いて、自然数 $i$ の代わりに $\textbf{g}^{i}$ を使用します。これにより、高速フーリエ変換 (fast Fourier transform; FFT)を利用して高速に多項式の評価と補間を行うことが可能になります。

gate constaintsは以下のように多項式に変換されます。
1. パラメータ $(\textbf{q}_L, \textbf{q}_R, \textbf{q}_O, \textbf{q}_M, \textbf{q}_C) \in (\mathbb{F}_r^n)^5$ を多項式補間して、 $n-1$次多項式 $\textbf{q}_L(x), \textbf{q}_R(x), \textbf{q}_O(x), \textbf{q}_M(x), \textbf{q}_C(x)$を求める。全ての $i\in \{0,\dots,n-1\}$ で、それぞれ $\textbf{q}_L(\textbf{g}^{i})=q_{L_i}, \textbf{q}_R(\textbf{g}^{i})=q_{R_i}, \textbf{q}_O(\textbf{g}^{i})=q_{O_i}, \textbf{q}_M(\textbf{g}^{i})=q_{M_i}, \textbf{q}_C(\textbf{g}^{i})=q_{C_i}$ が成立する。
2. ワイヤの値 $\textbf{x}_a=(x_{a_i})_{i \in \{0,\dots, n-1\}}, \textbf{x}_b=(x_{b_i})_{i \in \{0,\dots, n-1\}}, \textbf{x}_c=(x_{c_i})_{i \in \{0,\dots, n-1\}}$ を多項式補間して、 $n-1$次多項式 $\textbf{a}(x), \textbf{b}(x), \textbf{c}(x)$を求める。全ての $i\in \{0,\dots, n-1\}$ で、それぞれ $\textbf{a}(\textbf{g}^{i}) = x_{a_i}, \textbf{b}(\textbf{g}^{i}) = x_{b_i}, \textbf{c}(\textbf{g}^{i}) = x_{c_i}$ が成立する。
3. 多項式の等式 $\textbf{q}_L(x)\textbf{a}(x) + \textbf{q}_R(x)\textbf{b}(x) + \textbf{q}_O(x)\textbf{c}(x) + \textbf{q}_M(x)\textbf{a}(x)\textbf{b}(x) + \textbf{q}_C(x) = 0$ が成り立つことを、KZG commitmentsで証明する。

copy constraintsを多項式に変換する際は、 $i \geq n$についても 1の累乗根で表せるようにするために、定数 $k_1, k_2 \in \mathbb{F}_q$ を用いて、 $n \leq i < 2n$を $k_1\textbf{g}^i$、$2n \leq i < 3n$を $k_2\textbf{g}^i$ で表します。なお、集合 $(\textbf{g}^0, \dots, \textbf{g}^{n-1}), (k_1 \textbf{g}^{0}, \dots, k_1 \textbf{g}^{n-1}), (k_2 \textbf{g}^0, \dots, k_2 \textbf{g}^{n-1})$は別個なものになっているため、 $i \in \{0,\dots, 3n-1\}$ と1対1で対応しています。そして、累積値に対応する多項式を次のように定義します。ただし、 $L_i(x) = \prod_{j \in \{0,\dots, n-1\}, i\neq j} \frac{x-\textbf{g}^{j}}{\textbf{g}^{i}-\textbf{g}^{j}}$ であり、 $L_i(\textbf{g}^{i})=1$、 $j \neq i$ で $L_i(\textbf{g}^{j})=0$ であるという特徴を持ちます。
```math
z(x) = L_1(x) + \Sigma_{i \in [n-1]} (L_{i+1}(x) \prod_{j \in [i]} \frac{(x_{j} + \beta \textbf{g}^j + \gamma)(x_{n+j} + \beta k_1 \textbf{g}^j + \gamma)(x_{2n+j} + \beta k_2 \textbf{g}^j + \gamma)}{(x_{j} + \beta \sigma(j) + \gamma)(x_{n+j} + \beta \sigma(n+j) + \gamma)(x_{2n+j} + \beta \sigma(2n+j) + \gamma)})
```
$z(x)$は次の関係を満たす漸化式になっています。
```math
\begin{align*}
    &z(1)=1 \\
    &z(\textbf{g}^{i+1})=z(\textbf{g}^{i})\frac{(x_{i} + \beta \textbf{g}^i + \gamma)(x_{n+j} + \beta k_1 \textbf{g}^i + \gamma)(x_{2n+i} + \beta k_2 \textbf{g}^i + \gamma)}{(x_{i} + \beta \sigma(i) + \gamma)(x_{n+i} + \beta \sigma(n+i) + \gamma)(x_{2n+i} + \beta \sigma(2n+i) + \gamma)}
\end{align*}
```
特に2つ目の式を変形すると、
```math
z(\textbf{g}^{i+1})(a(\textbf{g}^i) + \beta \sigma(i) + \gamma)(b(\textbf{g}^i) + \beta \sigma(n+i) + \gamma)(c(\textbf{g}^i) + \beta \sigma(2n+i) + \gamma) = z(\textbf{g}^{i})(a(\textbf{g}^i) + \beta \textbf{g}^i + \gamma)(b(\textbf{g}^i) + \beta k_1 \textbf{g}^i + \gamma)(c(\textbf{g}^i) + \beta k_2 \textbf{g}^i + \gamma)
```
が成り立ちます。 $\textbf{g}^{n}=1$ であるため、その総積をとると、
```math
\begin{align*}
&\prod_{i=0}^{n-1}(a(\textbf{g}^i) + \beta \sigma(i) + \gamma)(b(\textbf{g}^i) + \beta \sigma(n+i) + \gamma)(c(\textbf{g}^i) + \beta \sigma(2n+i) + \gamma) \\
=&\prod_{i=0}^{n-1}(a(\textbf{g}^i) + \beta \textbf{g}^i + \gamma)(b(\textbf{g}^i) + \beta k_1 \textbf{g}^i + \gamma)(c(\textbf{g}^i) + \beta k_2 \textbf{g}^i + \gamma)
\end{align*}
```
が成り立ちますが、これはcopy constaintsにおける累積値の等式 $F=G$に対応していることがわかります。

## References
1. Sasson, E. B., Chiesa, A., Garman, C., Green, M., Miers, I., Tromer, E., & Virza, M. (2014, May). Zerocash: Decentralized anonymous payments from bitcoin. In 2014 IEEE symposium on security and privacy (pp. 459-474). IEEE.
2. Pertsev, A., Semenov, A., & Storm, R. (2019). Tornado cash privacy solution version 1.4. Tornado cash privacy solution version.
3. Bünz, B., Agrawal, S., Zamani, M., & Boneh, D. (2020, February). Zether: Towards privacy in a smart contract world. In International Conference on Financial Cryptography and Data Security (pp. 423-443). Cham: Springer International Publishing.
4. Buterin, V. (2018). On-chain scaling to potentially 500 tx/sec through mass tx validation, ethereum research. Available at https://ethresear.ch/t/on-chain-scaling-to-potentially-500-tx-sec-through-mass-tx-validation/3477
5. Thibault, L. T., Sarry, T., & Hafid, A. S. (2022). Blockchain scaling using rollups: A comprehensive survey. IEEE Access, 10, 93039-93054.
6. Gabizon, A., Williamson, Z. J., & Ciobotaru, O. (2019). Plonk: Permutations over lagrange-bases for oecumenical noninteractive arguments of knowledge. Cryptology ePrint Archive.
7. Nitulescu, A. (2020). zk-SNARKs: A gentle introduction. Ecole Normale Superieure.
8. Kate, A., Zaverucha, G. M., & Goldberg, I. (2010). Polynomial commitments. Tech. Rep.
9. LambdaClass. (2023). How to transform code into arithmetic circuits. Available at https://blog.lambdaclass.com/how-to-transform-code-into-arithmetic-circuits/ 
10. iden3. (n.d.). circom. zkSnark circuit compiler. Available at https://github.com/iden3/circom
11. arkworks. (n.d.). arkworks. Available at https://arkworks.rs/
12. Dokchitser, T., & Bulkin, A. (2023). Zero knowledge virtual machine step by step. Cryptology ePrint Archive.
13. Bruestle, J., Gafni, P., & the RISC Zero Team. (2023). RISC Zero zkVM: Scalable, Transparent Arguments of RISC-V Integrity. Available at https://dev.risczero.com/proof-system-in-detail.pdf
14. succinctlabs. (n.d.). SP1. Documentation for SP1 users and developers. Available at https://docs.succinct.xyz/#sp1
15. Zhu, M., & Ragsdale, S. (2024). Building Jolt: A fast, easy-to-use zkVM. Available at https://a16zcrypto.com/posts/article/building-jolt/
16. Marin, D., Abdalla, M., Govereau, P., Groth, J., Judson, S., Sosnin, K., Vamsi, Policharla, G., & Zhang, Y. (2024). Nexus 1.0: Enabling Verifiable Computation. Available at https://nexus-xyz.github.io/assets/nexus_whitepaper.pdf
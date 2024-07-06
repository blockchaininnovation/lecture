<center>

  # ゼロ知識証明の理論 後編: Plonkの詳細

</center>

## はじめに
前編では、ゼロ知識証明 (ZKP)の構成の一つであるPlonkを例に、証明したNP問題がより証明するのが容易な形式、具体的には多項式の評価に対する証明に変換される過程を追いました [1]。後編では、その証明をKZG commitments [2]で行うために必要なテクニックを紹介します。

## 証明したい多項式の復習
算術回路の充足可能性問題として記述されたNP問題は、ゲートごとの制約条件を表すgate constraintsとゲート間の制約条件を表すcopy constraintsに変換され、それらの証明は次の多項式の関係によってそれぞれ表されるのでした。
```math
\begin{align*}
&\forall x \in \{\textbf{g}^0, \dots, \textbf{g}^{n-1}\}, \textbf{q}_L(x)\textbf{a}(x) + \textbf{q}_R(x)\textbf{b}(x) + \textbf{q}_O(x)\textbf{c}(x) + \textbf{q}_M(x)\textbf{a}(x)\textbf{b}(x) + \textbf{q}_C(x) = 0\\
&\forall x \in \{\textbf{g}^0, \dots, \textbf{g}^{n-1}\}, z(x\textbf{g})(\textbf{a}(x) + \beta \sigma(i) + \gamma)(\textbf{b}(x) + \beta \sigma(n+i) + \gamma)(\textbf{c}(x) + \beta \sigma(2n+i) + \gamma) - z(x)(\textbf{a}(x) + \beta \textbf{g}^i + \gamma)(\textbf{b}(x) + \beta k_1 \textbf{g}^i + \gamma)(\textbf{c}(x) + \beta k_2 \textbf{g}^i + \gamma) = 0
\end{align*}
```


## References
1. Gabizon, A., Williamson, Z. J., & Ciobotaru, O. (2019). Plonk: Permutations over lagrange-bases for oecumenical noninteractive arguments of knowledge. Cryptology ePrint Archive.
2. Kate, A., Zaverucha, G. M., & Goldberg, I. (2010). Polynomial commitments. Tech. Rep.

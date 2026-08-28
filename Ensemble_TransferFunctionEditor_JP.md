# アンサンブル伝達関数エディタ                                                                                                                                                                                 
アンサンブル伝達関数エディタでは、伝達関数エディタと同様に物理値の統計量に割り当てられる色および不透明度を定義する伝達関数が作成できる。



通常のボリュームレンダリングでは伝達関数は1つの物理量のみによって定義されるがPBVRでは

1. 色と不透明度に独立な変量を割り当てる。
1. 各変量を座標X、Y、Z、変量q1、q2、q3...の任意の関数式で定義
1. 1次元伝達関数の色関数をC1、C2、...、不透明度関数をO1、O2、...で定義し、さらにそれら任意の関数式で合成して多次元伝達関数を定義

という、新たな伝達関数設計を可能にしたことで、極めて自由度の高い可視化処理を実現した。
<p align="center">
<img src="img/EnsembleTransferFunctionEditor/" alt="workload" width=100%>
</p>

- **Number of Transfer Functions**:作成可能な伝達関数の制限数を指定する。
- **Exportボタン**:作成した伝達関数をファイルに保存する
- **Importボタン**:作成した伝達関数ファイルを読み込む
- **Applyボタン**:作成した伝達関数ファイルを適用する

## Color Functionカテゴリ
- **Synthesizer**:色関数C1~C[N]による合成式を指定する※1
- **Function**:編集する色関数C1~C[N]を選択する
- **...ボタン**:Color Function Editorを表示して選択中の色関数C1~C[N]の引数となる合成変量を作成する

## Color Mapカテゴリ
- **C[N] Min:Max**: 合成変量に対して、色関数を割り当てる最小最大値を指定する
- **C[N] Server side Range Min:Max**: 合成変量の最小最大値を表示する
- **同期ボタン**:有効時、強制的にC[N]Server side Range Min:Maxの値が採用される。
- **Edit Color Map**:選択中の色関数に対するカラーマップを作成するためのColor Map Editorを開く
- **ヒストグラム**:C[N]Min:Maxで指定した最小最大値の範囲のヒストグラムが表示される

## Opacity Functionカテゴリ
- **Synthesizer**:不透明度関数O1~O[N]による合成式を指定する※1
- **Function**:編集する不透明度関数O1~O[N]を選択する
- **...ボタン**:Opacity Function Editorを表示して選択中の不透明度関数O1~O[N]の引数となる合成変量を作成する

## Opacity Mapカテゴリ
- **O[N] Min:Max**: 合成変量に対して、不透明度関数を割り当てる最小最大値を指定する
- **O[N] Server side Range Min:Max**: 合成変量の最小最大値を表示する
- **同期ボタン**:有効時、強制的にO[N]Server side Range Min:Maxの値が採用される。
- **Edit Opacity Map**:選択中の不透明度関数に対するオパシティマップを作成するためのOpacity Map Editorを開く
- **ヒストグラム**:O[N]Min:Maxで指定した最小最大値の範囲のヒストグラムが表示される

【代数式で使用可能な演算子】  
+, -, , /, ^, sqrt(), cbrt(), log(), log10(), exp(), abs(), floor(), ceil(), sin(), cos(), tan(), asin(), sinh(), cosh(), tanh(), asinh(), acosh(), atanh(), heavi(), rectfunc() Binary functions: sigmoid(,), gauss(,), min(,), max(,)

※1:[N]はNumber of Transfer Functionsで指定した伝達関数の制限数の値である。

## Color Function Editor
Color Function Editorではユーザが指定した代数式を使用して物理量を合成できる。
合成した物理量は色関数C1~C[N]の引数となる。
代数式で使用できる変量の名称は以下に示される。

- **物理量**:q1、q2、q3、...q[N]
- **座標値**:X、Y、Z

<p align="center">
<img src="img/TransferFunctionEditor/TransferFunctionEditor_2.svg" alt="workload" width=50%>
</p>

- **Color Function List**:作成されている伝達関数を表示する
- **f(algebraic formula)**:伝達関数C[N]に対応するf(変数)を入力する
- **Cancel**:設定を反映せず本パネルを閉じる
- **OK**:設定を反映して本パネルを閉じる

## Opacity Function Editor
Opacity Function Editorではユーザが指定した代数式を使用して物理量を合成できる。
合成した物理量は色関数O1~O[N]の引数となる。
代数式で使用できる変量の名称は以下に示される。

- **物理量**:q1、q2、q3、...q[N]
- **座標値**:X、Y、Z

<p align="center">
<img src="img/TransferFunctionEditor/TransferFunctionEditor_3.svg" alt="workload" width=50%>
</p>

- **Opacity Function List**:作成されている伝達関数を表示する
- **f(algebraic formula)**:伝達関数O[N]に対応するf(変数)を入力する
- **Cancel**:設定を反映せず本パネルを閉じる
- **OK**:設定を反映して本パネルを閉じる

## Color Map Editor
### Presets
PresetsではC1~C[N]に対応するカラーマップをあらかじめ用意されているカラーマップから選択できる
<p align="center">
<img src="img/TransferFunctionEditor/TransferFunctionEditor_4.svg" alt="workload" width=70%>
</p>

### Freeform Curve
Freeform CurveではC1~C[N]に対応するカラーマップをマウスによる自由曲線入力で作成できる。
<p align="center">
<img src="img/TransferFunctionEditor/TransferFunctionEditor_5.svg" alt="workload" width=70%>
</p>

- **Selected Drawing Color**:上塗りする色を選択できる
- **Resetボタン**:編集前のカラーマップバーに戻す
- **Undoボタン**:マウスの操作を一つ取り消す
- **Redoボタン**:取り消したマウス操作を再実行する

### Expression
ExpressionではC1~C[N]に対応するカラーマップを数式記述で作成できる
<p align="center">
<img src="img/TransferFunctionEditor/TransferFunctionEditor_6.svg" alt="workload" width=70%>
</p>

- **R**:色のR成分の色関数式を代数式で記述する
- **G**:色のG成分の色関数式を代数式で記述する
- **B**:色のB成分の色関数式を代数式で記述する


色関数の変数はxであり色関数の定義域は0から1の範囲である。

### Control Points
Control PointsではC1~C[N]に対応するカラーマップバーを制御点指定で作成できる
<p align="center">
<img src="img/TransferFunctionEditor/TransferFunctionEditor_7.svg" alt="workload" width=70%>
</p>

- **Number of Control Points**:制御点の個数を指定する
- **Point**:制御点の値を指定する
- **Red**:制御点の値に対応する色のR成分を指定する
- **Green**:制御点の値に対応する色のG成分を指定する
- **Blue**:制御点の値に対応する色のB成分を指定する

各制御点は区分線形関数で補間される、また定義域は0から1の範囲である。

## Opacity Map Editor
### Freeform Curve
Freeform CurveタブではO1~O[N]に対応するオパシティマップをマウスによる自由曲線入力で作成できる。
<p align="center">
<img src="img/TransferFunctionEditor/TransferFunctionEditor_8.svg" alt="workload" width=70%>
</p>

### Expression
ExpressionではO1~O[N]に対応するオパシティマップを数式記述で作成できる
<p align="center">
<img src="img/TransferFunctionEditor/TransferFunctionEditor_9.svg" alt="workload" width=70%>
</p>

### Control Points
Control PointsではO1~O[N]に対応するオパシティマップバーを制御点指定で作成できる
<p align="center">
<img src="img/TransferFunctionEditor/TransferFunctionEditor_10.svg" alt="workload" width=70%>
</p>

## 関数エディタ
伝達関数エディタにおける伝達関数合成、変量合成、カラーマップ曲線、不透明度曲線の入力に使用される関数エディタで使用できる組み込み関数は以下の通り。

|演算|書式|
|:--|:--|
|+|+|
|×|*|
|/|/|
|Sin|sin(x)|
|Cos|cos(x)|
|Tan|tan(x)|
|Log|log(x)|
|Exp|exp(x)|
|平方根|sqrt(x)|
|冪乗|x^y|                                                                                              

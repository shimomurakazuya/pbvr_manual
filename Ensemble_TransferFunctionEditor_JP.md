# アンサンブル伝達関数エディタ                                                                                                                                                                                 
アンサンブル伝達関数エディタでは、伝達関数エディタと同様に物理値の統計量に割り当てられる色および不透明度を定義する伝達関数が作成できる。

<p align="center">
<img src="img/EnsembleTransferFunctionEditor/ETFeditor.png" alt="workload" width=50%>
</p>

- **Variable**: 計算する統計量の関数式を指定する。
- **Statistic ボックス**: 可視化する統計量 または アンサンブルメンバ を指定する。
- **Log10 Scale (Mean / Variance) チェックボックス**: 統計量を常用対数表示するかしないかを切り替える。
- **Output Member Particles チェックボックス**: 各アンサンブルデータの出力するかしないかを切り替える。
- **Exportボタン**:作成した伝達関数をファイルに保存する
- **Importボタン**:作成した伝達関数ファイルを読み込む
- **Applyボタン**:エディタの現在の設定をサーバー側へ反映する

※ **Variable**で指定できる変数、演算子については[Opacity Function Editor](##Opacity_Function_Editor), [関数エディタ](##関数エディタ)参照

## 可視化表示する統計量の選択方法
 関数式にて計算される統計量は平均(average)、分散(varience)、変動係数(coefficient of variation)の3種類である。表示したい統計量について、Statisticボックスから選択し、applyボタンを押すことで設定が反映され、次に粒子が生成されたステップから表示される。

## 各アンサンブルメンバの指定方法
 各アンサンブルメンバを表示したい場合は Statisticボックスの"member"を選択する。すると、隣のスピンボックスが有効かされるので、idを入力する。画像における"2/3" について、"2"の部分が現在選択しているメンバidで、"3"が最大メンバidである。選択後、applyボタンを押すことで表示される。ただし、Output Member Particlesが**OFF**の場合またはOFFにしていたタイムステップをMain windowで指定した場合、出力データ自体がないため何も表示されないことに注意。出力の切り替えは後述。
 
## 各アンサンブルメンバの出力切り替え
 Output Member Particles チェックボックスをON にしてapplyボタンを押すと、各アンサンブルメンバが出力されるようになる。各メンバの出力処理が加わるため、反映までの全体の処理時間が長くなる。

## 常用対数表示の切り替え
 Variableでは、常用対数の数式に対応している。ただし、これを用いた場合常用対数を用いた値での統計量（アンサンブル間で幾何平均を対数化した値）を表示し、算術平均を対数化した値ではない。
 算術平均を対数化した値を表示したい場合、Log10 Scale (mean/varience) チェックボックスをONにし、applyボタンを押下する。

具体例：　f = log(q1) と f = q1　かつLog10 Scale　ボタンONの平均値について

f = log(q1)
<p align="center">
<img src="img/EnsembleTransferFunctionEditor/スクリーンショット 2026-11-18 16.31.03.png" alt="workload" width=30%>
</p>

f = q1　かつLog10 Scale　ボタンON
<p align="center">
<img src="img/EnsembleTransferFunctionEditor/スクリーンショット 2026-11-18 16.31.07.png" alt="workload" width=30%>
</p>


## Ensemble Transfer Functionカテゴリ
- **ColorMap**:variableにて指定された式の統計量に色付けするカラーマップを表示している。ダブルクリックでColor Map Editorを開く
- **OpacityMap**:variableにて指定された式に対するオパシティマップを表示している。ダブルクリックでOpacity Map Editorを開く
- **Histogram**: EnsembleMinMaxカテゴリにて指定した最小最大値の範囲のヒストグラムが表示される

※ Color Map Editor の 使い方は[Color Map Editor](##Color_Map_Editor)参照  

※ Opacity Map Editor の 使い方は[Opacity Map Editor](##Opcity_Map_Editor)参照



## Ensemble MinMax カテゴリ
- **User Defined MinMax**: Variableにて指定された式に対して、色関数/不透明度関数を割り当てる最小最大値を指定する 
- **Server Side MinMax**: Variableにて指定された式の統計量の最小最大値を表示する

                

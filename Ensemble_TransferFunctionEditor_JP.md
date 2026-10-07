# アンサンブル伝達関数エディタ                                                                                                                                                                                 
アンサンブル伝達関数エディタでは、伝達関数エディタと同様に物理値の統計量に割り当てられる色および不透明度を定義する伝達関数が作成できる。

<p align="center">
<img src="img/EnsembleTransferFunctionEditor/スクリーンショット 2026-08-28 16.53.47.png" alt="workload" width=50%>
</p>

- **Variable**: 計算する統計量の関数式を指定する。
- **Statistic**: 可視化する統計量を指定する。
- **Exportボタン**:作成した伝達関数をファイルに保存する
- **Importボタン**:作成した伝達関数ファイルを読み込む
- **Applyボタン**:作成した伝達関数ファイルを適用する

※ **Variable**で指定できる変数、演算子については[Opacity Function Editor](##Opacity_Function_Editor), [関数エディタ](##関数エディタ)参照

## Ensemble Transfer Functionカテゴリ
- **colorMap**:variableにて指定された式の統計量に色付けするカラーマップを表示している。ダブルクリックでColor Map Editorを開く
- **opacityMap**:variableにて指定された式に対するオパシティマップを表示している。ダブルクリックでColor Map Editorを開く
- **Histogram**: EnsembleMinMaxカテゴリにて指定した最小最大値の範囲のヒストグラムが表示される

※ Color Map Editor の 使い方は[Color Map Editor](##Color_Map_Editor)参照  

※ Opacity Map Editor の 使い方は[Opacity Map Editor](##Opcity_Map_Editor)参照



## Ensemble MinMax カテゴリ
- **User Defined MinMax**: Variableにて指定された式に対して、色関数/不透明度関数を割り当てる最小最大値を指定する 
- **Server Side MinMax**: Variableにて指定された式の統計量の最小最大値を表示する

                

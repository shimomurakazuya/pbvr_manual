
# アンサンブル In-Situ 可視化のセットアップ                                                                                                                                                                     
アンサンブルシミュレーションに対して、専用のサンプラを結合することで、代数式によって計算された物理値の統計量の対話的に可視化される。

環境設定に関しては[In-Situ 可視化のセットアップ](#In-Situ可視化のセットアップ)を参照

クライアント側の操作に関しては[EnsembleTransferFunctionEditor](#EnsembleTFE)を参照

## 組み込み方法

シミュレーション中において、以下のようにkvs_wrapper.h をインクルードし、アンサンブル可視化関数を呼び出す。

```
#include "kvs_wrapper.h"
#include "kvs_wrapper_common.h"

domain_parameters_unstruct dom ※ 可視化領域
    float**        values;//[nvariables][nnodes] ※変数配列、変数の数 * ノード数　の二次元行列 
    unsigned int*  connections;  ※ 接続情報　
    float*         coords;　※ 座標　
    int            ncells;　※ セル数
    int            nnodes;　※ ノード数
    int            nvariables;　※ 変数の数
    int            ens_num; ※ アンサンブル数
    int 　　　　　　time_step; ※ 　タイムステップ

...(計算処理)...

result = ensemble_generate_particles( time_step, ens_num, dom,
                       values, nvariables,
                       coords, nnodes,
                       connections, ncells, vismodule::VolumeObjectBase::CellType::(hoge) );
```


## サンプルコード
水素の電化密度分布をアンサンブルメンバ毎に変化させた分布を用いた時の統計量を可視化できるようになっている。
非構造格子データ、構造格子データのサンプルコードはそれぞれ以下のディレクトリに配置している。

| 対応タイプ       | ファイル名                                                             |
|----------------|------------------------------------------------------------------|
| 構造格子データ     |  ens_Hydrogen_struct   |
| 非構造格子データ   |  ens_Hydrogen_unstruct     |

### 計算条件

各アンサンブルのメンバ数が大きくなるほど値は増大し、分布は縦に圧縮されるような分布。
以下はアンサンブル数が0~4の画像でである。

0アンサンブル
<p align="center">
<img src="https://github.com/shimomurakazuya/pbvr_manual/blob/main/img/%E3%83%88%E3%83%AA%E3%83%9F%E3%83%B3%E3%82%AF%E3%82%99ens0_rl27%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202026-07-13%2015.21.28.png" alt="0アンサンブル" width=60%>
</p>


1アンサンブル
<p align="center">
<img src="https://github.com/shimomurakazuya/pbvr_manual/blob/main/img/%E3%83%88%E3%83%AA%E3%83%9F%E3%83%B3%E3%82%AF%E3%82%99ens1_rl27%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202026-07-13%2015.21.07.png" alt="1アンサンブル" width=60%>
</p>


2アンサンブル
<p align="center">
<img src="https://github.com/shimomurakazuya/pbvr_manual/blob/main/img/%E3%83%88%E3%83%AA%E3%83%9F%E3%83%B3%E3%82%AF%E3%82%99ens2_rl27%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202026-07-13%2015.20.48.png" alt="2アンサンブル" width=60%>
</p>


3アンサンブル
<p align="center">
<img src="https://github.com/shimomurakazuya/pbvr_manual/blob/main/img/%E3%83%88%E3%83%AA%E3%83%9F%E3%83%B3%E3%82%AF%E3%82%99ens3_rl27%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202026-07-13%2015.22.34.png" alt="3アンサンブル" width=60%>
</p>

### コンパイル方法

両者とも、Serverをビルドした時と同環境にてコンパイルできる。成功すると run　バイナリが生成される。

## サンプルコードの可視化結果

### 平均
<p align="center">
<img src="https://github.com/shimomurakazuya/pbvr_manual/blob/main/img/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202026-07-22%2014.47.40.png" alt="workload" width=60%>
</p>

### 分散
<p align="center">
<img src="https://github.com/shimomurakazuya/pbvr_manual/blob/main/img/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202026-07-22%2014.52.03.png" alt="workload" width=60%>
</p>

### 変動係数
<p align="center">
<img src="https://github.com/shimomurakazuya/pbvr_manual/blob/main/img/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202026-07-22%2014.53.51.png" alt="workload" width=60%>
</p>

                                                                                                            

# OpenFOAMへのpbvr組み込み

OpenFOAMにpbvr付属のFoamToPbvrアダプターを挿入することで、OpenFOAMソルバーが可視化用粒子を生成できるようになる、このアダプターはFilterディレクトリに格納されており、粒子サンプラ同様、粒子生成ライブラリを参照、リンクすることで使用可能となる。アダプターは計算された物理値の変数配列をvtkライブラリを用いてvtkUnstructuredGridクラスに変換し、粒子サンプラへと引き渡す。
以下ではPBVRに付属したOpenFOAMのサンプルコードの動作手順を示す。

## 環境設定

本事例ではJAEA所有のスパコン(SGI8600)でサンプルコードを動作させる。別サーバーで動作させる場合、モジュール等は別途読み替えるように。

【モジュール】　gnu/9.5.0　 openmpi/4.1.4 

【OpenFOAM】　10（財団版）(プリインストール済 ``/home/app/OpenFOAM/OpenFOAM-10``)

【サンプルコード名】 cavity flow 

【vtk】 v9.2　(プリインストール済 ``/home/center/viztools/VTK-9.2.2``)

【shell】 bash

なおモジュールのロードコマンドは以下の通り。

```
module purge; module load gnu/9.5.0　openmpi/4.1.4 
```

## ISPBVRのビルド&インストール

ここではインストールディレクトリのパスを($install_dir_pbvr)とする。

インストールディレクトリにて、git cloneを実施する。

```
git clone git@github.com:CCSEPBVR/CS-IS-PBVR.git -b release_v3.3.0
```


OpenFOAM用のライブラリ作成のために、以下のようにpbvrのコンパイル設定を図のように変更する。PBVR_MACHINEをgccに、VTK_PTAHをtrueに変更する。また、必要があればプリインストール済のvtkライブラリへパス設定を行う。（デフォルトで設定済）

 <p align="center">
 <img src="https://github.com/user-attachments/assets/b7dbf750-5179-400f-87f9-6ca20a1383a6" alt="workload" width=60%>
 </p>




設定が完了次第、コンパイルを実行する。

```
make -j 40
```


## OpenFOAM&ソルバーのコンパイル設定編集

OpenFOAMにてpbvrのライブラリを用いるために、コンパイル設定をpbvrと一致させる。 SGI8600ではプリインストールされたOpenFOAMをローカルにコピーし、設定を変更する。

### OpenFOAMコマンドの有効化

OpenFOAMのインストールディレクトリのパスを($install_dir_foam)とすると、コピーコマンドは以下の通り、

```
cp -r /home/app/OpenFOAM/OpenFOAM-10  ($install_dir_foam)
cp -r /home/app/OpenFOAM/ThirdParty-10  ($install_dir_foam)
```

コピーできたら、図の赤線部分のように($install_dir_foam)/OpenFOAM-10/etc/bashrcのFOAM_INST_DIRを($install_dir_foam)とし、環境変数WM_COMPIKERをGccに変更する。

 <p align="center">
 <img src="https://github.com/user-attachments/assets/ce5a7ff5-3d5e-4929-94f3-d60255144f41" alt="workload" width=40%>
 </p>


ここから、OpenFOAMコマンドを有効化する。まず、一度/etc/bashrcを実行する。
```
source ($install_dir_foam)/OpenFOAM-10/etc/bashrc
```

OpenFOAMディレクトリ内の./Allwmakeを実行する。

```
cd OpenFOAM-10
./Allwmake
```

/etc/bashrcを再び実行することで有効化準備は完了。以下のコマンドでヘルプログが出たら有効化されている。

```
source ($install_dir_foam)/OpenFOAM-10/etc/bashrc
icoFOAM –help 
```

### OpenFOAMのコンパイル設定変更

コンパイル設定ファイル(``$install_dir_foam)/OpenFOAM-10/wmake/rules/linux64Gcc/c++``)にてコンパイルコマンド部分( CC = … )を図のように変更する。

 <p align="center">
 <img src="https://github.com/user-attachments/assets/465930e1-b080-41e9-82e7-afddfe8604fb" alt="workload" width=60%>
 </p>

### OpenFOAMのコンパイルオプション変更

OpenFOAM用サンプルコードディレクトリ(ここではIS_DaemonAndSampler/Example/C/gcc_mpi_omp/cavity_flow_vtk)に移動し、pbvrのライブラリを用いるためにMake/optionsにてパスを以下のように追加する。
（サンプルコードではすでに追加済）

 <p align="center">
 <img src="https://github.com/user-attachments/assets/9060cb6b-e808-4670-b677-9e2574bccb95" alt="workload" width=150%>
 </p>

以上で、OpenFOAM側の設定は完了。

## サンプルコードのビルド

サンプルコードをビルドする。コンパイル完了後、$FOAM_APPBINにバイナリが生成される。

```
wmake -j
```

前処理として、メッシュ生成コマンドを実行する。

```
blockMesh
decomposePar
```
## コードの実行

In-Situセットアップの通りに環境変数を設定する。

```
export VIS_PARAM_DIR=($install_dir_pbvr)/CS-IS-PBVR/IS_DaemonAndSampler/Example/C/gcc_mpi_omp/cavity_flow_vtk
export  PARTICLE_DIR=($install_dir_pbvr)/CS-IS-PBVR/IS_DaemonAndSampler/Example/C/gcc_mpi_omp/cavity_flow_vtk/particle_out
```

粒子データ用ディレクトリとしてparticle_outを作成する。
```
mkdir particle_out
```

これでソルバー実行の準備が整ったので、OpenFOAMソルバーを実行する。サンプルコードとしてicoFoam.Cを使用しているので実行コマンドは以下の通り。

```
mpirun –n 2 icoFoam –parallel
```

## デーモンコードの実行

遠隔サーバーにてデーモンの実行手順をここでは示す。なお、ソルバーとデーモンは同時実行が想定されているので、新規タブでの作業を推奨する(モジュールロードを忘れずに)。

デーモンコードのビルド作業は、ソルバービルド時に同時に行っているため省略する。実行前に環境変数として、以下の２つを設定する。

```
export VIS_PARAM_DIR=($install_dir_pbvr)/CS-IS-PBVR/IS_DaemonAndSampler/Example/C/gcc_mpi_omp/cavity_flow_vtk
export  PARTICLE_DIR=($install_dir_pbvr)/CS-IS-PBVR/IS_DaemonAndSampler/Example/C/gcc_mpi_omp/cavity_flow_vtk/particle_out
```

Daemonディレクトリに移動し、pbvr_daemonを実行する。

```
cd ($install_dir_pbvr)/CS-IS-PBVR/IS_DaemonAndSampler/Daemon
./pbvr_daemon
```
“waiting connection… ”　というログが出れば成功。接続待ち状態となっているので、停止せずにクライアント側からの接続操作に移る。

遠隔サーバーへのポートフォワーディング、クライアントの実行、接続方法についてはチュートリアルを参照されたし。

## サンプルコードの可視化結果
サンプルコードの可視化結果は以下の図の通りである。この時の可視化パラメータを以下に記載する。

 <p align="center">
 <img src="https://github.com/user-attachments/assets/41439857-23fd-4a5c-a570-9ffe1294538c" alt="workload" width=80%>
 </p>

* 時間ステップ: 10 

Transfer function
* Color Function Synthesizer : C1
* Opacity Function Synthesizer : O1
* Color Map Function [C1] : f(q1)
* Opacity Map Function[O1] : f(q1)
* minmax mode: server size Min Max

グリフ
* scale factor : 0.1
* Distribution mode: AllPoints
* Size: constant
* Color : Variables Array
  * Number of variables: 3
    * q1
    * q2
    * q3

plot over line 
* Resolution:256
* Target: q1
* Start point: (0.5, 0, 0.05)
* End point: (0.5, 1, 0.05) 

## サンプルコードについて

### 計算内容

このサンプルコードの計算内容は2次元cavity flowである。z方向は周期境界条件を課して一様としている。

計算パラメータは以下の通りである。
* MPI並列数: 2MPI
* 上壁面速度(m/s): 1
* セル数(n_x, n_y, n_z): 400(20, 20, 1)
* セルタイプ:hexahedra
* 計算領域(L_x, L_y, L_z): (1, 1, 0.5)
* レイノルズ数(動粘性係数): 100


### インクルードファイルについて

 <p align="center">
 <img src="https://github.com/user-attachments/assets/75e10e61-750c-4304-b1db-850a092a0fd4" alt="workload" width=40%>
 </p>

上図はあくまで一例であり、必要なファイルはソルバーによって変化する。

### icoFoam.CでのPBVR関数組込＆出力間隔制御

ここではicoFoam.CへのPBVR関数組込と出力間隔制御の事例を説明する。

アダプターはOpenFOAMと同様、インクルードファイルを挿入する方式を採用している。図のように指定のフィルターファイル（赤線）を展開することで、PBVRの関数を呼び出している。（詳細は後述）

 <p align="center">
 <img src="https://github.com/user-attachments/assets/f7e5f588-3dcd-436d-88cf-1f6acd4c3c66" alt="workload" width=40%>
 </p>

アダプターには2種類あり、polyhedronタイプと混合要素タイプがある。混合要素タイプの方はOpenFOAM のcellModel Classに含まれるHEX, WEDGE、PRISM,　PYRタイプのセルのみ可視化可能だが処理速度が早い。一方で、polyhedronタイプはセルをそのまま扱えるので対応範囲が広い。

後述する使用方法について2つに違いはないが、混ぜ合わせて使用するとエラーの原因となるので、どちらか一方のみを使用するよう注意されたし。

| 対応タイプ       | ファイル名                                                             |
|-----------------|------------------------------------------------------------------|
| 混合要素タイプ    | Conversion_from_OpenFOAM_to_vtk_***.h    |
| 多面体タイプ　    | Conversion_from_OpenFOAM_to_vtkpolyhedron_***.h      |

アダプターは初回使用時には(**initial_step** )がついた.hファイルを、2回目以降は(**later_step**)が付いた.hファイルを使用する。

##  FoamToPbvrアダプターについて

ここではOpenFOAM形式からVTK形式への情報詰め替え処理を実施するFoamToPbvrアダプターについて説明する。
このアダプターはvtkライブラリ関数を用いて、OpenFOAM形式の座標、接続情報、変数データを粒子サンプラが対応しているvtk形式に変換を行うものである。
変数データについては毎出力ステップ毎に変換処理を行うが、座標、接続情報データに関しては初回出力ステップのみ変換処理を行い、処理コストを削減している。
また、変換処理の際にセルデータからポイントデータへの変換処理も行なっている。


### 可視化用変数を増やす手順

可視化したい計算パラメータを変更、追加したい場合はアダプターを編集する必要がある。

アダプターは``($install_dir_pbvr)/CS-IS-PBVR/IS_DaemonAndSampler /Insitulib/unstruct/Filter``ディレクトリに格納されている。

以下の図の黒枠部分について、コピー&ペーストし、可視化対象の変数配列をInsertNextValue()に挿入すると変数が増やせる。その際、変数名が重複しないように注意。

 <p align="center">
 <img src="https://github.com/user-attachments/assets/e9f6d60b-bf3b-4e53-a9e6-83f74d8cc815" alt="workload" width=40%>
 </p>




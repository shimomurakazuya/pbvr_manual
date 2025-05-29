# OpenFOAMの都市風況サンプルコードの可視化手順

OpenFOAMにpbvr付属のFoamToPbvrアダプターを挿入することで、OpenFOAMソルバーが可視化用粒子を生成できるようになる、このアダプターはFilterディレクトリに格納されており、粒子サンプラ同様、粒子生成ライブラリを参照、リンクすることで使用可能となる。アダプターは計算された物理値の変数配列をvtkライブラリを用いてvtkUnstructuredGridクラスに変換し、粒子サンプラへと引き渡す。
以下ではPBVRに付属したOpenFOAMの都市風況サンプルコードの動作手順を示す。

## 環境設定

本事例ではJAEA所有のスパコン(SGI8600)でサンプルコードを動作させる。別サーバーで動作させる場合、モジュール等は別途読み替えるように。

【モジュール】　gnu/9.5.0　 openmpi/4.1.4 

【OpenFOAM】　v2206（esi版）(プリインストール済 ``/home/app/OpenFOAM/OpenFOAM-v2206``)

【サンプルコード名】 wind around building 

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
git clone git@github.com:CCSEPBVR/CS-IS-PBVR.git 
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
rsync -r /home/app/OpenFOAM/OpenFOAM-v2206  ($install_dir_foam)
rsync -r /home/app/OpenFOAM/ThirdParty-v2206  ($install_dir_foam)
```

コピーできたら、図の赤線部分のように($install_dir_foam)/OpenFOAM-v2206/etc/bashrcのprojectDirを($install_dir_foam)とし、環境変数WM_COMPIKERをGccに変更する。

 <p align="center">
 <img src="https://github.com/user-attachments/assets/38d629a2-1ef1-42d3-b20b-b6870642adb9" alt="workload" width=40%>
 </p>




ここから、OpenFOAMコマンドを有効化する。まず、一度/etc/bashrcを実行する。
```
source ($install_dir_foam)/OpenFOAM-v2206/etc/bashrc
```

OpenFOAMディレクトリ内の./Allwmakeを実行する。

```
cd OpenFOAM-v2206
./Allwmake
```

/etc/bashrcを再び実行することで有効化準備は完了。以下のコマンドでヘルプログが出たら有効化されている。

```
source ($install_dir_foam)/OpenFOAM-v2206/etc/bashrc
icoFOAM –help 
```

### OpenFOAMのコンパイル設定変更

コンパイル設定ファイル(``($install_dir_foam)/wmake/rules/General/Gcc/c++``)にてコンパイルコマンド部分( CC = … )を図のように変更する。
```
CC          = mpic++$(COMPILER_VERSION) -std=c++14 -m64 -pthread -fopenmp
```

 <p align="center">
 <img src="https://github.com/user-attachments/assets/0ec04535-b3e0-4fc7-b727-0ad440d225af" alt="workload" width=60%>
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

前処理として、メッシュ生成スクリプト、コマンドを実行する。

```
./Allrun.pre 
decomposePar
```
## コードの実行

In-Situセットアップの通りに環境変数を設定する。

```
export VIS_PARAM_DIR=($install_dir_pbvr)/CS-IS-PBVR/IS_DaemonAndSampler/Example/C/gcc_mpi_omp/windAroundBuildings_v2206
export  PARTICLE_DIR=($install_dir_pbvr)/CS-IS-PBVR/IS_DaemonAndSampler/Example/C/gcc_mpi_omp/windAroundBuildings_v2206/particle_out
```

粒子データ用ディレクトリとしてparticle_outを作成する。
```
mkdir particle_out
```

これでソルバー実行の準備が整ったので、OpenFOAMソルバーを実行する。サンプルコードとしてicoFoam.Cを使用しているので実行コマンドは以下の通り。

```
mpirun -n 2 simpleFoam -parallel
```

## デーモンコードの実行

遠隔サーバーにてデーモンの実行手順をここでは示す。なお、ソルバーとデーモンは同時実行が想定されているので、新規タブでの作業を推奨する(モジュールロードを忘れずに)。

デーモンコードのビルド作業は、ソルバービルド時に同時に行っているため省略する。実行前に環境変数として、以下の２つを設定する。

```
export VIS_PARAM_DIR=($install_dir_pbvr)/CS-IS-PBVR/IS_DaemonAndSampler/Example/C/gcc_mpi_omp/windAroundBuildings_v2206
export  PARTICLE_DIR=($install_dir_pbvr)/CS-IS-PBVR/IS_DaemonAndSampler/Example/C/gcc_mpi_omp/windAroundBuildings_v2206/particle_out
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
 <img src="https://github.com/user-attachments/assets/7296e490-dcf8-40e8-b6b5-b525f7ac33fa" alt="workload" width=80%>
 </p>


* 時間ステップ: 400（定常状態）

Transfer function
* Color Function Synthesizer : C1
* Opacity Function Synthesizer : O1
* Color Map Function [C1] : $dq1x * dq2y + dq2y * dq3z + dq3z * dq1x - dq1y * dq2x - dq2z * dq3y - dq3x * dq1z$
* Opacity Map Function[O1] : $dq1x * dq2y + dq2y * dq3z + dq3z * dq1x - dq1y * dq2x - dq2z * dq3y - dq3x * dq1z$
* minmax mode: User defined Min Max

グリフ
* scale factor : 15
* Distribution mode: Uniform Distribution
* Size: constant
* Color : Variables Array
  * Number of variables: 3
    * q1
    * q2
    * q3

plot over line 
* Resolution:256
* Target: q1
* Start point: (-20, 160, 50)
* End point: (350, 160, 50) 

## サンプルコードについて

### 計算内容

このサンプルコードの計算内容は3次元空間における都市まわりの風況シミュレーション(WindAroundBuilding)である。x方向に（手前側の境界面から）風が流入している。

計算パラメータは以下の通りである。粒子はQ値、グリフは風速の大きさ、plot over line は風速のx成分分布を示す
* MPI並列数: 2MPI (並列数を変更したい場合は[後述](#MPI並列数の変更方法)を参照)
* 流入速度(m/s): 10
* セル数: 185,237
* セルタイプ:hexahedra, pyramid, polyhedron
* 計算領域(L_x, L_y, L_z): (350,280,140)
* レイノルズ数(動粘性係数): 2.33e8(0.00001)


### インクルードファイルについて

 <p align="center">
 <img src="https://github.com/user-attachments/assets/75e10e61-750c-4304-b1db-850a092a0fd4" alt="workload" width=40%>
 </p>

上図はあくまで一例であり、必要なファイルはソルバーによって変化する。

### simpleFoam.CでのPBVR関数組込＆出力間隔制御

ここではsimpleFoam.CへのPBVR関数組込と出力間隔制御の事例を説明する。

アダプターはOpenFOAMと同様、インクルードファイルを挿入する方式を採用している。図のように指定のフィルターファイル（赤線）を展開することで、PBVRの関数を呼び出している。（詳細は後述）

 <p align="center">
 <img src="https://github.com/user-attachments/assets/f7e5f588-3dcd-436d-88cf-1f6acd4c3c66" alt="workload" width=40%>
 </p>

アダプターには2種類あり、polyhedronタイプと混合要素タイプがある。ただし、このサンプルコードではpolyhedronタイプのみ使用可能。

| 対応タイプ       | ファイル名                                                             |
|-----------------|------------------------------------------------------------------|
| 多面体タイプ　    | Conversion_from_OpenFOAM_to_vtkpolyhedron_***.h      |

アダプターは初回使用時には(**initial_step** )がついた.hファイルを、2回目以降は(**later_step**)が付いた.hファイルを使用する。

##  FoamToPbvrアダプターについて

ここではOpenFOAM形式からVTK形式への情報詰め替え処理を実施するFoamToPbvrアダプターについて説明する。
このアダプターはvtkライブラリ関数を用いて、OpenFOAM形式の座標、接続情報、変数データを粒子サンプラが対応しているvtk形式に変換を行うものである。
変数データについては毎出力ステップ毎に変換処理を行うが、座標、接続情報データに関しては初回出力ステップのみ変換処理を行い、処理コストを削減している。
また、変換処理の際にセルデータからポイントデータへの変換処理も行なっている。

### MPI並列数の変更方法

ここではサンプルコードのMPI並列数を変更する手順について説明する。
MPI並列数をNに変更したい場合、前処理の段階からやり直す必要がある。``system/decomposeParDict`` の　``numberOfSubdomains``の数値をNに変更し、
下記のように、``N = x*y*z``が成り立つようにsimpleCoeffsの値を設定する。

変更後、Allrun.preを再実行する。
（この時、前処理のlogファイル(log....)を削除しておく必要がある）

```
  1 /*--------------------------------*- C++ -*----------------------------------*\
  2 | =========                 |                                                 |
  3 | \\      /  F ield         | OpenFOAM: The Open Source CFD Toolbox           |
  4 |  \\    /   O peration     | Version:  v2206                                 |
  5 |   \\  /    A nd           | Website:  www.openfoam.com                      |
  6 |    \\/     M anipulation  |                                                 |
  7 \*---------------------------------------------------------------------------*/
  8 FoamFile
  9 {
 10     version     2.0;
 11     format      ascii;
 12     class       dictionary;
 13     object      decomposeParDict;
 14 }
 15 // * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * //
 16 
 17 numberOfSubdomains N;   ←この数字
 18 
 19 method          simple;
 20 
 21 simpleCoeffs
 22 {
 23   n ( x y z );　← 三つの数字の積がNに等しくなるように設定
 24   delta 0.001;
 25 }
 26 
```

その後、コードの実行コマンドを並列数nで入力する。

```
mpirun -n N simpleFoam -parallel
```


### 可視化用変数を増やす手順

可視化したい計算パラメータを変更、追加したい場合はアダプターを編集する必要がある。

アダプターは``($install_dir_pbvr)/CS-IS-PBVR/IS_DaemonAndSampler /Insitulib/unstruct/Filter``ディレクトリに格納されている。

以下の図の黒枠部分について、コピー&ペーストし、可視化対象の変数配列をInsertNextValue()に挿入すると変数が増やせる。その際、変数名が重複しないように注意。

 <p align="center">
 <img src="https://github.com/user-attachments/assets/e9f6d60b-bf3b-4e53-a9e6-83f74d8cc815" alt="workload" width=40%>
 </p>



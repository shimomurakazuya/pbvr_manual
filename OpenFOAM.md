# OpenFOAMへのpbvr組み込み

OpenFOAMにpbvr付属のアダプターを挿入することで、OpenFOAMソルバーが可視化用粒子を生成できるようになる、このアダプターはFilterディレクトリに格納されており、粒子サンプラ同様、粒子生成ライブラリを参照、リンクすることで使用可能となる。アダプターは計算された物理値の変数配列をvtkライブラリを用いてvtkUnstructuredGridクラスに変換し、粒子サンプラへと引き渡す。
以下ではPBVRに付属したOpenFOAMのサンプルコードを動作させる手順を示す。

## 環境設定

本事例では機構所有のスパコン(SGI8600)でサンプルコードを動作させる。別サーバーで操作させる場合、別途読み替えるように。

【モジュール】　gnu/9.5.0　 openmpi/4.1.4 　vtk/7.1.1

【OpenFOAM】　10（財団版）(プリインストール済)

【サンプルコード名】 cavity flow 

【vtk】 v9.2　(プリインストール済)

【shell】 bash


## PBVRサーバーのビルド&インストール

ここではインストールディレクトリのパスを($install_dir_pbvr)とする。

インストールディレクトリにて、git cloneを実施する。

```
git clone git@github.com:CCSEPBVR/CS-IS-PBVR.git -b release_v3.3.0
```


OpenFOAM用のライブラリ作成のために、以下のようにpbvrのコンパイル設定を図のように変更する。PBVR_MACHINEをgccのものに変更し、プリインストール済のvtkライブラリへパス設定を行う。

 <p align="center">
 <img src="https://github.com/user-attachments/assets/d326e5f1-b216-4868-aebf-15cefc9e6a9e" alt="workload" width=60%>
 </p>

設定が完了次第、コンパイルを実行する。

```
make -j 40
```


## OpenFOAMのコンパイル設定編集

OpenFOAMにてpbvrのライブラリを用いるために、コンパイル設定をpbvrと一致させる。 SGI8600ではプリインストールされたOpenFOAMをローカルにコピーし、設定を変更する。

### OpenFOAMコマンドの有効化

OpenFOAMのインストールディレクトリのパスを($install_dir_foam)とすると、コピーコマンドは以下の通り、

```
cp -r /home/app/OpenFOAM/OpenFOAM-10  ($install_dir_foam)
cp -r /home/app/OpenFOAM/ThirdParty-10  ($install_dir_foam)
```

コピーできたら、図のように($install_dir_foam)/etc/bashrcの環境変数WM_COMPIKERをGccに変更し、FOAM_INST_DIRを($install_dir_foam)とする。

 <p align="center">
 <img src="https://github.com/user-attachments/assets/748e845f-92c5-4d36-b70d-7d3f5cf012a5" alt="workload" width=60%>
 </p>

```
source ($install_dir_foam) /etc/bashrc
```

を実行したのち、OpenFOAMディレクトリ内の./Allwmakeを実行する。

```
cd OpenFOAM-10
./Allwmake
```
/etc/bashrcを再び実行し、OpenFOAMコマンドを有効化する。以下のコマンドでヘルプログが出たら有効化されている。

```
icoFOAM –help 
```
### OpenFOAMのコンパイル設定変更

コンパイル設定ファイル($install_dir_foam)/wmake/rules/linux64Gcc/c++)を( CC = … )を図のように変更し、コンパイルコマンドを一致させる。

 <p align="center">
 <img src="https://github.com/user-attachments/assets/465930e1-b080-41e9-82e7-afddfe8604fb" alt="workload" width=60%>
 </p>

### OpenFOAMのコンパイルオプション変更

OpenFOAM用サンプルコードディレクトリ(ここではIS_DaemonAndSampler/Example/C/gcc_mpi_omp/cavity_flow_vtk)に移動し、pbvrのライブラリを用いるためにMake/optionにてパスを以下のように追加する。
（サンプルコードではすでに追加済）

 <p align="center">
 <img src="https://github.com/user-attachments/assets/9060cb6b-e808-4670-b677-9e2574bccb95" alt="workload" width=100%>
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

OpenFOAMソルバーの実行
% ./RESET.sh (粒子データがある場合、要実行)
%mpirun –n 2  icoFoam –parallel











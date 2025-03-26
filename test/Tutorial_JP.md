# チュートリアル
## ソケット通信によるクライアント・サーバの接続　
クライアントプログラムとサーバプログラムは、ポートフォワードで接続したポートを利用してソケット通信を行い、粒子データや可視化パラメータを送受信する。
以下ではsshを用いたポートフォワードの方法を記述する。

### ローカル接続
1台のマシン上でクライアントプログラムとサーバプログラムをスタンドアロンで起動する例を示す。
この例ではクライアントプログラムとサーバプログラムの両方がマシンAにおけるデフォルトのポート番号60000を用いて連携する。

1. **サーバ起動**
ターミナル上で以下のコマンドを実行する。
    - Linux
    ```
    machineA $ ./pbvr_server
    ```
    
    - Windows
    ```
    machineA >pbvr_server.exe
    ```
    
    - Mac
    ```
    machineA % ./pbvr_server
    ```

1. **クライアント起動**
クライアントプログラムの実行可能ファイルをダブルクリックする。
(ターミナルから実行する場合は以下を参考にすること。)
    - Linux
    ```
    machineA $ ./QTPBVR
    ```
    
    - Windows
    ```
    machineA >QTPBVR.exe
    ```
    
    - Mac
    ```
    machineA % open QTPBVR.app
    ```

### リモート接続
手元のマシン(machineA)と遠隔地のマシン(machineB)をsshポートフォワードで接続し、machineA上でクライアントプログラムを、machineB上でサーバプログラムを起動する例を示す。
基本的にsshポートフォワードが確立した後の起動方法はスタンドアロンと同じである。
<p align="center">
<img src="img/Tutorial/Tutorial_1.svg" alt="workload" style="width:50%;">
</p>

sshローカルフォワードのコマンド
```
クライアントPC> ssh -L 50000:ターゲットのアドレス:60000 遠隔サーバのアドレス
```

sshローカルフォワードのコマンドにおいて、遠隔サーバとターゲットが一致している場合、ターゲットのアドレスは"localhost"となる。
また、この例ではポート番号として50000番と60000番が利用されているが、TCPやUDPに割り当てられているポート番号をのぞいて任意のポート番号が利用可能である。
sshポートフォワードを用いてmachineAの50000番ポートをmachineBの60000番ポートに接続し、各マシンでクライアントプログラムとサーバプログラムを起動する例を示す。

1. sshポートフォワード
    machineAの50000番ポートをmachineBの60000番ポートに接続する。
    machineB自身がターゲットであるので、ターゲットのアドレスはlocalhostとなる。
    ```
    machineA $ ssh -L 50000:localhost:60000 username@machineB
    ```
    
1. サーバ起動
    ```
    machineB $ ./pbvr_server
    ```
    
1. クライアント起動
    ```
    machineA $ ./QTPBVR
    ```
    
machineBのログインノードから対話ノードinteractBに入り、sshポートフォワードを用いてmachineAの50000番をinteractBの60000に接続し、各マシンでクライアントプログラム、サーバプログラムを起動する例を示す。

1. sshポートフォワード
    ```
    machineA $ ssh -L 50000:interactB:60000 username@machineB
    (machineAの50000番ポートをmachineBを経由して、interactBの60000番ポートに接続)
    ```
    
1. サーバ起動
    ```
    machineB $ pbvr_server
    ```
    
1. クライアント起動
    ```
    machineA $ ./QTPBVR
    ```
    
## サンプルデータを用いた可視化事例
sample_data.tar.gzに格納されているfldデータを用いた可視化処理例を以下に示す。  

### フィルタ処理
sample_data/fldに格納されているparam.txtを以下のように編集する。
```
#
in_dir=/sample_data/fld
out_dir=/sample_data/fld/out #指定したディレクトリが存在していない場合は手動で作成すること。
out_prefix=gt5d
field_file=gt5d.fld
start_step=0
end_step=4
```


フィルタプログラムを実行する。
```
pbvr_filter param.txt
```

上記の例ではデフォルトのSPLITファイル形式、領域分割数1としている。
この処理によって指定ディレクトリに以下のファイルが生成される。

```
gt5d.pfi                                - pfiファイル
gt5d_YYYYYYY_ZZZZZZZ_connect.dat        - 要素構成ファイル
gt5d_YYYYYYY_ZZZZZZZ_coord.dat          - ノード座標ファイル
gt5d_XXXXX_YYYYYYY_ZZZZZZZ.kvsml        - kvsmlファイル
gt5d_XXXXX_YYYYYYY_ZZZZZZZ_value.dat    - 成分ファイル
```

### プログラム起動
sshポートフォワードを用いてmachineAの50000番ポートをmachineBの60000番ポートに接続し、各マシンでクライアントプログラムとサーバプログラムを起動する例を示す。

1. sshポートフォワード
    ```
    machineA % ssh -L 50000:localhost:60000 username@machineB
    ```

1. サーバ起動
    ```
    machineB % pbvr_server
    Server initialize done
    Server bind done
    Server listen done
    Waiting for connection ...
    ```
    
1. クライアント起動
    この例では、伝達関数ファイルdemo.tfを接続時に読み込み、粒子生成処理にメトロポリスサンプリングを使用している。
    
    <p align="center">
    <img src="img/Tutorial/Tutorial_2.svg" alt="workload" style="width:50%;">
    </p>
    
    1. Client Serverにチェックをつける
    1. Metropolisにチェックをつける
    1. ポート番号を指定する
    1. 遠隔サーバに存在するgt5d.pfiのフルパスをに指定する
    1. ローカルサーバに存在するdemo.tfeを指定する
    1. 接続を開始する
    
    <p align="center">
    <img src="img/Tutorial/Tutorial_3.svg" alt="workload" style="width:50%;">
    </p>
    赤枠のいずれかのボタンを押し粒子生成と表示が行われることを確認する。(図の例では右側のボタンを押している。)
    
### 伝達関数の設計
2変数の構造格子ボリュームデータであるgt5d.fldに対してPBVRの高度な伝達関数設計機能を適用した可視化事例を紹介する。

### 単変量のボリュームレンダリング
伝達関数t1を下図のように設定して変数q1の概要を確認する。
この例では、伝達関数パネル左側の色の設計と右側の不透明度の設計の両方にq1を使用して単変量のボリュームレンダリングをおこなっている。
* Color Function Synthesizer : C1
* Opacity Function Synthesizer : O1
* Color Map Function : f(q1)
* Opacity Map Function : f(q1)

<p align="center">
<img src="img/Tutorial/Tutorial_4.svg" alt="workload" style="width:50%;">
</p>

### 多変量のボリュームレンダリング
次に変量q1と変量q2を合成する多変量のボリュームレンダリングの事例を示す。
この例では、色の設計に変量q1を使用し、不透明度の設計に変量q2を使用している。
不透明度の伝達関数では、変量q2のトーラス状の等値面2枚を抽出し、そこでの変量q1の変化を色で示している。

<p align="center">
<img src="img/Tutorial/Tutorial_5.svg" alt="workload" style="width:50%;">
</p>

* Color Function Synthesizer : C1
* Opacity Function Synthesizer : O2
* Color Map Function : f(q1)
* Opacity Map Function : f(q2)

### 断面表示
多変量ボリュームレンダリングの応用例として断面表示例を示す。
この例では、不透明度の設計に座標を使用している。
PBVRでは伝達関数設計に任意の関数を利用可能であり、この例ではX^2+Z^2=constという円筒面を抽出して、そこでの変量q1の変化を示している。

<p align="center">
<img src="img/Tutorial/Tutorial_6.svg" alt="workload" style="width:50%;">
</p>

* Color Function Synthesizer : C1
* Opacity Function Synthesizer : O3
* Color Map Function : f(q1)
* Opacity Map Function : f(sqrt(X^2+Z^2))
* Opacity Range Min[O3] : 180
* Opacity Range Max[O3] : 380

### 伝達関数の合成
PBVRにおける伝達関数の合成機能を説明する。
以下の図では不透明度を座標Yで与え、Y>0の領域を透明にする伝達関数O4を定義し、伝達関数O4と先に定義した3つの伝達関数O1、O2、O3を(O1+O2)*O4+O3と合成することにより部分領域を抽出する可視化例を示している。
伝達関数の合成にあたり、C2とC3の色は(R、G、B)＝(0、0、0)と設定し、C4の色は(R、G、B)＝(1、1、1)と設定している。
これに上記合成式を適用すると、色の設計はC1で使用されているq1のrainbowによって定義される。
一方、不透明度の設計はO1とO2を合成したものにO4をかけてY<0の領域を抽出し、それとO3の円筒面を合成したものが定義される。
伝達関数による領域抽出機能は着目する等値面やボリュームレンダリングのみに適用できるので極めて自由度が高い。

<p align="center">
<img src="img/Tutorial/Tutorial_7.svg" alt="workload" style="width:50%;">
</p>


* Color Function Synthesizer : (C1+C2)*C4+C3
* Opacity Function Synthesizer : (O1+O2)*O4+O3

* Color Map Function  [C1] : f(q1)
* Color Range Min     [C1] : -0.05
* Color Range Max     [C1] : 0.05
* Opacity Map Function[O1] : f(q1)
* Opacity Range Min   [O1] : -0.05
* Opacity Range Max   [O1] : 0.05

* Color Map Function  [C2] : f(q1)
* Color Range Min     [C2] : -0.05
* Color Range Max     [C2] : 0.05
* Opacity Map Function[O2] : f(q2)
* Opacity Range Min   [O2] : 0.0
* Opacity Range Max   [O2] : 1.0

* Color Map Function  [C3] : f(q1)
* Color Range Min     [C3] : -0.05
* Color Range Max     [C3] : 0.05
* Opacity Map Function[O3] : f(sqrt(X^2+Z^2)
* Opacity Range Min   [O3] : 180.0
* Opacity Range Max   [O3] : 380.0

* Color Map Function  [C4] : f(q1)
* Color Range Min     [C4] : -0.05
* Color Range Max     [C4] : 0.05
* Opacity Map Function[O4] : f(Y)
* Opacity Range Min   [O4] : -109.0
* Opacity Range Max   [O4] : 109.0

<p align="center">
<img src="img/Tutorial/Tutorial_8.svg" alt="workload" style="width:50%;">
</p>

### 粒子データの結合
先で多次元伝達関数を用いたボリュームレンダリング表示、等値面表示、断面表示の統合例を示したが、マージパネルを用いても同様の複雑な可視化処理を実現できる。
以下に手順を示す。

#### 粒子データの保存
粒子データの保存はマージパネルを使用して行う、以下の図に粒子データファイル名の指定例を示す。
この場合にはプレフィックスがp1となり、以下のファイルが生成される。

```
./p1/p1_XXXXX.kvsml
./p1/p1_XXXXX.colors.dat
./p1/p1_XXXXX.coords.dat
./p1/p1_XXXXX.normals.dat
```

ここで、XXXXXは時刻を示し、colors、coords、normalsはそれぞれ色、座標、法線ベクトルのデータを示す。
粒子データファイル名を入力し、Exportボタンを押すと粒子保存が開始され、粒子データの保存処理中はExportボタンが非アクティブとなり、全時系列の粒子データを保存後に再度アクティブになる。
※粒子保存はサーバオブジェクトの時系列が全て画面上に表示された際に終了する。

<p align="center">
<img src="img/Tutorial/Tutorial_8.svg" alt="workload" width="500"">
</p>

<p align="center">
<img src="img/Tutorial/Tutorial_9.svg" alt="workload" width="500"">
</p>

#### 粒子データの読み込み
先ほど保存した粒子データを読み込み、合成表示する例を示す。
マージパネルから...ボタン押し保存したp1_XXXXX.kvsmlとp2_XXXXX.kvsmlファイルを選択する。
サーバ側のDisplayのチェックボックスを外しApplyボタンを押したのち時系列を更新するとp1とp2を合成したものが表示される。

<p align="center">
<img src="img/Tutorial/Tutorial_10.svg" alt="workload" width="500"">
</p>

<p align="center">
<img src="img/Tutorial/Tutorial_11.svg" alt="workload" width="500"">
</p>

<p align="center">
<img src="img/Tutorial/Tutorial_12.svg" alt="workload" width="500"">
</p>

### ボリュームとポリゴンの合成表示
サンプルデータに含まれる非構造格子データspx.impとその境界形状ポリゴンspx.stlを用いた合成表示の例を示す。
コネクトパネルでspx.pfiのパスを指定しボリュームレンダリングを実行する。
その後マージパネルからspx.stlの読み込みを行うことでボリュームオブジェクトと境界形状が合成される。

<p align="center">
<img src="img/Tutorial/Tutorial_13.svg" alt="workload" width="500"">
</p>

### 点群データの可視化
点群データとは、３次元空間の物体表面を、位置・色等の情報を持った点の集合で表現するデータである。点群データを取得する方法は、レーザースキャン、構造化光、フォトグラメトリが代表的で、用途に合わせて選択される。サンプルデータ公開ページからリンクしてある点群データの表示例を示す。
<p align="center">
<img src="img/Tutorial/Tutorial_14.svg" alt="workload" width="500"">
</p>

<p align="center">
<img src="img/Tutorial/Tutorial_15.svg" alt="workload" width="500"">
</p>

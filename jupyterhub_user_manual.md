# JupyterHubユーザマニュアル
このマニュアルはAIコンソーシアムのJupyterHubサービスの利用者のためのマニュアルです。

不明点・要望・トラブル等については `hai-help-group@keio.jp` までご連絡ください。

## スペック
JupyterHubはIntel XeonのCPUおよびGeForce RTX 2080 TiのGPUが搭載されたサーバ上で運用されています。

## 申請
JupyterHubへのアクセスはLDAP認証を用いているため、まずLDAPアカウントの作成が必要です。[ホームページ](https://aic.keio.ac.jp/events/jupyterhub)から申請を行ってください。申請が通り次第、ユーザ名・パスワードが送付されます。

## アクセス手順
認証の都合でJupyterHubのページに直接アクセスする前に一度だけSSH経由でのアクセスが必要になっています。一度アクセスを行うとユーザの設定が完了するので以降は直接URLにアクセスしログインすれば問題ありません。以下のコマンドを用いて、申請の際に伝えられたパスワードを入力しゲートウェイにログインを行ってください。

```
ssh -p 2221 [ユーザ名]@[サーバ].ai.hc.keio.ac.jp
```

サーバはtippy、chino、cocoa、rizeのうちから選んでください。どれにログインしても違いはありません。以下は例です。

```
ssh -p 2221 user@tippy.ai.hc.keio.ac.jp
```

ログインが完了するとパスワードを変更するよう促されるので、パスワードを変更します。

再び同じ場所にSSHアクセスを行い、これで正常にログインができれば設定は完了です。接続を切り、ブラウザで [https://cocoa.ai.hc.keio.ac.jp](https://cocoa.ai.hc.keio.ac.jp) にアクセスしてください。ここで、`https`ではなく`http`と打ってしまうと正しくアクセスできないので注意してください。出てきたログインフォームにてユーザ名と新しく設定したパスワードを入力するとログインができます。**初回のログイン時は30秒から数分程度の時間が掛かる場合があります。ご了承下さい。**

## カスタムパッケージ追加の方法
condaやpipのパッケージを入れようとしても、そのままではJupyterHubから利用できる状態にはなりません。そこで、次のようにするとcondaやpipで独自のパッケージを自由にインストールできるようになります。まずは、JupyterHubにログインし、「New」→「Terminal」をクリックしターミナルを起動します。

![JupyterHubのターミナルの起動](./images/jupyter-open-terminal.png)

![JupyterHubのターミナルの様子](./images/jupyter-terminal-image.png)

ターミナルで次のコマンドを実行し、自分用のcondaのenvironmentの作成し、Jupyter Notebookが認識するようにインストールします。今回は`custom-env`という名前で作成しますが、好きな名前で作ることもできます。

```sh
conda create -n custom-env --clone base
conda activate custom-env
conda install anaconda
ipython kernel install --user --name custom-env
```

新しく作ったenvironmentには`conda install [パッケージ名]`で自由にパッケージをインストールすることができ、Jupyter Notebookを起動するタイミングで以下のようにcustom-envを選択すればインストールしたパッケージを利用することができます。

![](./images/jupyter-select-env.png)

参考: <https://zonca.github.io/2017/02/customize-python-environment-jupyterhub.html>

## GPUを用いた機械学習の手順例
GPUドライバやCUDAは既にインストールされた状態になっているため、GPUを用いた機械学習を行うにはconda等によって必要なパッケージのインストールを行えばできるようになります。ここではTensorFlowという機械学習のフレームワークを用いたチュートリアルを実行する手順を例としてとりあげます。[カスタムパッケージ追加の方法](#カスタムパッケージ追加の方法)と同様にターミナルを開きましょう。

上で作成した`custom-env`を使いましょう。別な名前で作成した場合は、適宜読み替えてください。

```
conda activate custom-env
conda install tensorflow-gpu
wget 'https://raw.githubusercontent.com/tensorflow/docs/master/site/en/tutorials/keras/classification.ipynb' # Tensorflowのチュートリアル用ipynbファイル
```

JupyterHubのアイコンをクリックしファイル一覧に戻り、新しくダウンロードした`classification.ipynb`をクリックし開きます。「Kernel」→「Change kernel」をクリックし、`custom-env`を選択します。

![](./images/jupyter-change-kernel.png)

最後に、１つ１つのセルを実行すればチュートリアルに沿って実行を進めることができます。

## ポートフォワーディングの仕方
Jupyter Notebookの利用の上で、TensorBoardの利用やウェブアプリケーションの開発等によりどこかのポートにアクセスしたい場合は次のようにポートフォワーディングを行うことができます。例えば、8000番ポートにアクセスをしたい場合は、次のようなコマンドを実行します。

```
ssh -p 2221 -L 8000:jupyterhub-singleuser-instance-[ユーザ名]:8000 [ユーザ名]@[サーバ].ai.hc.keio.ac.jp
```

この時、ユーザ名とサーバは上と同様に設定します。すると、SSHが接続されている限り`locahost:8000`へのアクセスは透過的にJupyter Notebokインスタンスに転送されるようになります。

## CUDAバージョンの変更について
使用するライブラリやソフトウェアによっては、インストールされているCUDAバージョンに対応していない場合があります。それに伴い、別なCUDAバージョンを利用する場合は**Jupyter Notebookのシステムのバージョンによって手順が変わります**。

次のコマンドの出力によりCUDAバージョンの変更の手順がわかります:

```
apt install --installed | grep cuda-repo
```

もし、次のような出力が得られた場合はサーバ管理者による対応が必要となるため、お手数ですが `hai-help-group@keio.jp` までご連絡ください:

```
WARNING: apt does not have a stable CLI interface. Use with caution in scripts.

cuda-repo-ubuntu1804/unknown,now 10.2.89-1 amd64 [installed]
```

一方、次のような出力が得られた場合は次の手順に進んでください。

```
WARNING: apt does not have a stable CLI interface. Use with caution in scripts.
```

次のコマンドで現在のCUDAのバージョンを確認することができます。

```
apt list --installed | grep ^cuda-toolkit
```

例えば、CUDA 10.1（`cuda-toolkit-10-1`）がインストールされており、CUDA 11にアップグレードしたいとします。その場合、次のように一旦現在のCUDAツールキットを削除した後に新たなCUDAツールキットをインストールします。

```
sudo apt remove cuda-toolkit-10-1
sudo apt install cuda-toolkit-11-0
sudo apt autoremove
```

なお、インストールが可能なCUDAバージョンの一覧は次のコマンドで確認することができます。

```
sudo apt search ^cuda-toolkit
```

## 注意事項
### セキュリティについて
JupyterHubはパスワード認証となっており、インターネットのどこからでもアクセスできるようになっています。ログインにはパスワードマネージャを用いるなどして十分複雑かつ長いパスワードの利用を強く勧めます。

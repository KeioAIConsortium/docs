# JupyterHubユーザマニュアル
このマニュアルはAIコンソーシアムのJupyterHubサービスの利用者のためのマニュアルです。

不明点・要望・トラブル等については `hai-help-group@keio.jp` までご連絡ください。

## 申請
JupyterHubへのアクセスはLDAP認証を用いているため、まずLDAPアカウントの作成が必要です。[ホームページ](https://aic.keio.ac.jp/events/jupyterhub)から申請を行ってください。申請が通り次第、ユーザ名・パスワードが送付されます。

## アクセス手順
認証の都合でJupyterHubのページに直接アクセスする前に一度だけSSH経由でのアクセスが必要になっています。一度アクセスを行うとユーザの設定が完了するので以降は直接JupyterHubにアクセスしログインすれば問題ありません。

SSH接続では

- Windows 10の場合: コマンドプロンプトまたはPowerShellでOpenSSHが使用できます。
  - [こちら](https://docs.microsoft.com/ja-jp/windows-server/administration/openssh/openssh_install_firstuse)に記載されている手順に従ってOpenSSHクライアントをインストールする必要があります。
- Windows 10以前の場合: Tera TermやPuTTY等のSSHクライアントソフトウェアを使用してください。
- macOS (OS X) / Linuxの場合: ターミナル上でOpenSSHが使用できます。

OpenSSH以外の各種SSHクライアントソフトウェアを使用する場合は、それぞれのドキュメントを参考にしてください。個別のソフトウェアに対するサポートは致しかねます。ご了承ください。

以下のコマンドを用いて、申請の際に伝えられたユーザ名と初期パスワードを入力しゲートウェイにログインを行ってください。

```
ssh -p 2221 [ユーザ名]@[ゲートウェイサーバ].ai.hc.keio.ac.jp
```

ゲートウェイサーバはtippy、chino、cocoa、rizeのうちから選んでください。どれにログインしても違いはありません。また、どのゲートウェイサーバに接続したかを覚えておく必要はありません。以下は例です。

```
ssh -p 2221 user@tippy.ai.hc.keio.ac.jp
```

SSHクライアントソフトウェアを使用する場合は、接続先のポートを2221番に設定してください。
OpenSSH以外の各種SSHクライアントソフトウェアを使用する場合は、それぞれのドキュメントを参考にしてください。

ログインが完了するとパスワードを変更するよう促されるので、パスワードを変更します。

再び同じ場所にSSHアクセスを行い、これで正常にログインができれば設定は完了です。接続を切り、ブラウザで [https://cocoa.ai.hc.keio.ac.jp](https://cocoa.ai.hc.keio.ac.jp) にアクセスしてください。ここで、`https`ではなく`http`と打ってしまうと正しくアクセスできないので注意してください。出てきたログインフォームにてユーザ名と新しく設定したパスワードを入力するとログインができます。**初回のログイン時は30秒から数分程度の時間が掛かる場合があります。ご了承下さい。**エラーが出てしまった場合は一旦ページを再読込してください。

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

## JupyterHubインスタンスに直接アクセスしたい場合

ここでは、JupyterHubのWebインターフェイスを介さず、ご自身のインスタンスに直接アクセスする方法を説明します。

あらかじめインスタンスが起動している必要があります。インスタンスはユーザがJupyterHubにログインしたタイミングで立ち上がるようになっているため、接続できない場合はJupyterHubにブラウザからログインしてください。なおも立ち上がらない場合は`Start My Server`をクリックしてください。

### ポートフォワーディングの仕方
Jupyter Notebookインスタンスの利用の上で、TensorBoardの利用やウェブアプリケーションの開発等によりどこかのポートにアクセスしたい場合は次のようにポートフォワーディングを行うことができます。例えば、8000番ポートにアクセスをしたい場合は、次のようなコマンドを実行します。

```
ssh -p 2221 -L 8000:jupyterhub-singleuser-instance-[ユーザ名].lxd:8000 [ユーザ名]@[ゲートウェイサーバ].ai.hc.keio.ac.jp
```

この時、ユーザ名とゲートウェイサーバは[アクセス手順](#アクセス手順)と同様に設定します。すると、SSHが接続されている限り`locahost:8000`へのアクセスは透過的にJupyter Notebookインスタンスの8000番ポートに転送されるようになります。

### Jupyter NotebookインスタンスへのSSHアクセスの仕方

Jupyter Notebookインスタンスに直接SSH接続をしたい場合は、公開鍵認証を使用することでアクセスできるようになります。

事前の準備として

- SSHキーペアの生成
- 接続元の端末での`ssh-agent`の起動、SSHキーペアの登録
- Jupyter NotebookインスタンスへのSSH公開鍵の登録
  - `~/.ssh/autorized_keys`に書き込む、`ssh-import-id-gh [GitHubユーザ名]`でインポートする等
- (任意) ゲートウェイサーバへのSSH公開鍵の登録
  - これを行うことでゲートウェイコンテナでも公開鍵認証でSSH接続できるようになります
  - 各ゲートウェイサーバ間でホームディレクトリは同期しませんのでご注意ください
  
が必要になります。具体的な手順等についてはインターネット上の各種ドキュメントを参考にしてください。\
(参考: https://help.github.com/ja/github/authenticating-to-github/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent )

実際に接続する際には、まず

```
ssh -p 2221 -A [ユーザ名]@[ゲートウェイサーバ].ai.hc.keio.ac.jp
```

でゲートウェイサーバにSSH接続します。\
`-A`はagent forwardingを行うオプションです。OpenSSH以外の各種SSHクライアントソフトウェアを使用する場合は`Forward agent`オプションを有効にしてください。

次に

```
ssh ubuntu@jupyterhub-singleuser-instance-[ユーザ名].lxd
```

を実行することで、JupyterHub上のターミナルを使わなくてもご自身のPCのターミナルから直接コマンドが実行できるようになります。

## CUDAバージョンの変更について
使用するライブラリやソフトウェアによっては、インストールされているCUDAバージョンに対応していない場合があります。本サービスでは、ユーザ自身によるCUDAのバージョン変更に対応しています。

ただしCUDAのバージョン変更は**Jupyter Notebookインスタンスのバージョンによって手順が異なります**。

次のコマンドの出力によりCUDAバージョンの変更の手順がわかります:

```
apt list --installed | grep cuda-repo
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

例えば、CUDA 10.1（`cuda-toolkit-10-1`）がインストールされており、CUDA 11.0にアップグレードしたいとします。その場合、次のように一旦現在のCUDAツールキットを削除した後に新たなCUDAツールキットをインストールします。

```
sudo apt remove --purge --yes cuda-toolkit-10-1
sudo apt autoremove --purge --yes
sudo apt install --yes cuda-toolkit-11-0
```

なお、インストールが可能なCUDAバージョンの一覧は次のコマンドで確認することができます。

```
sudo apt search ^cuda-toolkit
```

## 注意事項
### セキュリティについて
JupyterHubはパスワード認証となっており、インターネットのどこからでもアクセスできるようになっています。ログインにはパスワードマネージャを用いるなどして十分複雑かつ長いパスワードの利用を強く勧めます。

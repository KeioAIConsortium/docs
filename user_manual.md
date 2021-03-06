# ユーザマニュアル
このマニュアルはAIコンソーシアムの計算資源利用者のためのマニュアルです。サーバにはLXDというコンテナマネージャが動いており、利用者の申請に応じて(rootを含む）LXCコンテナへのアクセスの提供を行っています。

不明点・要望・トラブル等については `aic-server-group@keio.jp` までご連絡ください。

## アクセス手順
特別な申請が無い限り、利用者に割り当てられるコンテナはNATされたネットワークに繋がっているため、直接SSHすることができません。したがって、一旦サーバ上で動いているゲートウェイコンテナにSSHでログインし、そこからもう一度SSHして自分のコンテナにアクセスする必要があります。ゲートウェイコンテナはサーバの2221番ポートからSSHできるようになっているため、コンテナの割当時に伝えられたサーバにまずSSHします。

```
ssh -p 2221 [ユーザ名]@(tippy|chino|cocoa|rize).ai.hc.keio.ac.jp
```

ここでゲートウェイコンテナのパスワードを入力します。次に割り当てられたコンテナにSSHします。

```
ssh ubuntu@[コンテナIP]
```

## 注意事項
### セキュリティについて
ゲートウェイおよびユーザコンテナへのSSHアクセスはパスワード認証となっており、前者はインターネットから広くアクセスできるようになっています。ログインには公開鍵認証の利用およびパスワードマネージャ等を用いた十分複雑かつ長いパスワードの利用を強く勧めます。
ユーザに付与されるコンテナは同一ネットワーク上に位置しており、お互いのコンテナにアクセスすることができるようになっています。提供されたコンテナのセキュリティの確保および管理はユーザご自身で行うことを想定しています。

### データのバックアップについて
現在コンテナのデータ等のバックアップ体制がないためインシデントの発生等によりユーザコンテナのデータが失われてしまう可能性があります。必須のデータは各自でバックアップをとるようお願いします。

### GPUの利用について
GPUの利用申請があったユーザにはCUDA 10.0・cuDNN 7・Anaconda3をコンテナにインストールした状態で提供しています。次のコードで簡単に動作確認をすることができます。まずは、通常のSSHと同時にポートフォワーディングを行うため、

```
ssh -p 2221 -L 8888:[コンテナIP]:8888 [ユーザ名]@(tippy|chino|cocoa|rize).ai.hc.keio.ac.jp
```

でゲートウェイコンテナにログインを行い、そこからは
```
ssh ubuntu@[コンテナIP]
```
でユーザコンテナにログインをします。
ユーザコンテナにログインできた後、次のコマンドの実行をします。

```sh
conda create -n test_env anaconda tensorflow-gpu # 時間が掛かるので注意
conda activate test_env
wget 'https://raw.githubusercontent.com/tensorflow/docs/master/site/en/tutorials/keras/classification.ipynb' # Tensorflowのチュートリアル用ipynbファイル
jupyter notebook --ip=0.0.0.0
```

最後に、ブラウザで `localhost:8888/?token=[トークン]` を開いた上で `basic_classification.ipynb` をクリックし、 `Cell -> Run All` で実行します。

### Anaconda削除の仕方
Anacondaを使わない場合、ホームディレクトリにある `~/anaconda3` を削除し、次の `~/.bashrc` の最後にある二行を消すことによりAnacondaをコンテナから消すことができます。

```sh
export PATH=$PATH:$HOME/anaconda3/bin
. /home/ubuntu/anaconda3/etc/profile.d/conda.sh
```

参考: <https://docs.anaconda.com/anaconda/install/uninstall/>

# AIC Slurmサーバ（試験運用版）ユーザマニュアル

このマニュアルは、AIC（AIコンソーシアム）が提供する **GPUサーバ（Slurm試験環境）** の利用者のためのガイドです。  
本サービスは試験運用中であり、利用条件・機能・構成は予告なく変更されることがあります。

不明点・要望・トラブル等については **aic-server-group@keio.jp** までご連絡ください。

---

## 利用の目的と範囲

本サーバは **教育・研究活動のためのGPU計算試験環境** として提供されます。  
AIC講習参加者・インターンシップ生らによる、学習・研究目的での利用に限ります。  
研究室・団体での共同アカウント使用は禁止です。

**禁止事項**  
- 仮想通貨マイニング、分散演算、ゲームプレイ等  
- 公開Webサービスの運用や外部からの常時アクセスを受けるサーバ構築  
- OSやネットワーク設定の変更・root権限操作  

上記の行為が発覚した場合は、警告なしにアカウントを停止することがあります。

---

## 利用申請・アカウント

- **JupyterHubアカウントの申請**  
  申請フォーム: https://forms.gle/in8MkCM5vLo7PwRL8

- **公開鍵の提出（必須）**  
  SSH接続に用いる公開鍵（例: `~/.ssh/id_ed25519.pub`）を提出してください。  
  公開鍵作成時は **必ずパスフレーズを設定** してください。

> 登録が完了すると、以下の接続設定でログインできるようになります。

---

## 接続情報（SSH / ProxyJump）

計算資源は **Slurmジョブスケジューラ** により管理されています。  
計算ノードに直接ログインしてGPUを占有することはできません。  
ジョブは `sbatch` または `srun` を用いて投入してください。

ローカルPCの `~/.ssh/config` に以下を追記します（**踏み台経由 / ProxyJump**）。

```sshconfig
### KEIO-AIC ###
Host casper
  HostName casper.ai.hc.keio.ac.jp

Host nadeko
  HostName 172.16.0.18

Host casper nadeko
  IdentityFile ~/.ssh/<private_key_file>   # 使用する秘密鍵
  User <username>                           # 付与されたユーザー名
  Port 2222

Host nadeko
  ProxyJump casper
```

接続:

```bash
ssh nadeko
```

---

## ファイル転送

作業ファイルのアップロードは `scp` を利用します（ホーム/共有領域は運用指針に従ってください）。

```bash
scp <local-file> <username>@nadeko:/share/<username>
```

---

## パーティション設計（試験運用）

用途に応じて以下のパーティションを指定してください。

| パーティション名 | 想定用途（例） |
|---|---|
| `gpu-short` | ハイパラ調整・コード動作確認など短時間ジョブ |
| `gpu-standard` | 一般的な学習・推論 |
| `gpu-strong` | 大規模学習（長時間・多GPU想定） |
| `gpu-interactive` | 対話的デバッグ・短時間実験（`srun --pty` など） |

> 各パーティションには **GPU数・メモリ・最大実行時間** の上限が設けられています。上限を超える指定はできません。

指定例：

```bash
# バッチ実行
sbatch --partition=gpu-standard job_script.sh

# 対話実行（シェルを割り当て）
srun --partition=gpu-interactive --pty bash
```

---

## Slurmの基本コマンド

- **ジョブ投入**  
  ```bash
  sbatch --partition=gpu-standard job_script.sh
  ```

- **ジョブ一覧（自分のジョブ）**  
  ```bash
  squeue -u <username>
  ```

- **ジョブ取消**  
  ```bash
  scancel <JOB_ID>
  ```

- **パーティションの確認**  
  ```bash
  scontrol show partition
  ```

参考:  
- Summary（コマンド早見）: https://slurm.schedmd.com/pdfs/summary.pdf  
- `sbatch` 詳細: https://slurm.schedmd.com/sbatch.html

---

## ジョブスクリプト例

```bash
#!/bin/bash
#SBATCH --job-name=memory_cpu_job
#SBATCH --partition=gpu-strong
#SBATCH --ntasks=1
#SBATCH --gres=gpu:4
#SBATCH --mem=64000MB
#SBATCH --cpus-per-task=16
#SBATCH --time=24:00:00
#SBATCH --output=output_mem_cpu_%j.txt

# 仮想環境の作成と有効化
python3 -m venv python_test
source python_test/bin/activate

# 依存パッケージの導入例
pip install torch torchvision torchaudio

# 実行コマンド
python3 large_model_training.py
```

**主要オプション**  
- `--gres=gpu:<N>`：利用するGPU枚数  
- `--mem=<MB>`：割当メモリ（MB）。不足時は OOM で失敗します  
- `--cpus-per-task=<N>`：タスクあたりのCPUコア数  
- `--time=HH:MM:SS`：最大実行時間（パーティション上限内で指定）  
- `--output=...%j...`：`%j` はジョブIDに置換

---

## データ管理

- ワーク領域（例：`/share/<username>`）を基本にご利用ください。  
- 不要データは適宜削除し、ストレージ圧迫を避けてください。  
- **試験環境のため、重要データは各自でバックアップ** をお願いします。

---

## セキュリティ上の注意

- **パスフレーズ付きのSSH鍵**を利用し、使い回しは避けてください。  
- 不審な挙動を検知した場合は速やかに **aic-server-group@keio.jp** へご連絡ください。  
- 試験環境のため、障害・データ損失に対する補償はありません。

---

## ログ・監視・運用ポリシー

- GPU利用状況、CPU・メモリ使用量、ジョブキュー状態は運営が定期監視します。  
- **不正利用・長時間アイドル**が続くジョブは、運営判断で強制終了される場合があります。  
- 利用ログは一定期間保管します。

---

## サポート・問い合わせ

技術的な質問・障害報告・要望などは下記またはDiscordまでお願いします。

- 障害・緊急停止に関する報告  
- ソフトウェア追加の要望  
- Slurmジョブの挙動・設定に関する質問  

**連絡先**：aic-server-group@keio.jp

---

## 免責事項

本サービスは試験運用中であり、**無保証** で提供されます。  
データ損失・障害発生・停電・メンテナンス等による影響について、運営チームは責任を負いません。  
サービスの構成・提供期間・リソースは予告なく変更されることがあります。

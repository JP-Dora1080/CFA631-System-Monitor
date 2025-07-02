# CFA631 System Monitor v1.0-beta  
for HomeDC Japan  

> Crystalfontz 20×2 LCD（CFA631-USB）に  
> ホスト情報／システム情報をリアルタイム表示するソフトウェア 

## ✨ 特長
* 1行目  
  * `HomeDC Japan - ホスト名` を自動スクロール表示  
* 2行目  
  * CPU使用率 / CPU温度 → メモリ使用率 → IPアドレスをサイクルで表示
* `config.yml` でスクロール速度・切替間隔・バックライトなどを簡単設定  
* 終了（Ctrl + C）時に 1行目を設定したテキスト、2行目を空白にする

## 🔧 ファイル構成

```
.
├─ cfa631_display.bin   # メインスクリプト
└─ config.yml          # 設定ファイル（無ければ自動でデフォルト値）
```

### config.yml 例

```yaml
# CFA631 Display Configuration
display:
  line1:
    text: "HomeDC Japan"  # 1行目の固定テキスト
    scroll_speed: 0.1     # スクロール間隔 [s]
  line2:
    cycle_interval: 3.0   # 2行目の情報切り替え周期 [s]

system:
  port: "/dev/ttyUSB0"    # CFA631 が刺さるデバイス
  baudrate: 115200        # デフォルト115200
  backlight: 100          # 0–100 %
```

## 🚀 使い方

1. スクリプトに実行権を付与  
   ```bash
   chmod +x cfa631_display.bin
   ```

2. デバイス名を確認  
   ```bash
   ls /dev/ttyUSB*
   # または
   dmesg | grep tty
   ```

3. 実行  
   ```bash
   ./cfa631_display.bin
   ```

   起動すると以下のようなバナーが表示されます。

```
╔══════════════════════════════════════════════════════════╗
║              CFA631 System Monitor v1.0-beta             ║
║                   for HomeDC Japan                       ║
╠══════════════════════════════════════════════════════════╣
║  作成者    : Veulx                                        ║
║  バージョン: 1.0-beta                                      ║
║  作成日    : 2025-07-01                                   ║
║  説明      : CFA631 LCD用システム監視ツール                  ║
╚══════════════════════════════════════════════════════════╝
⚙️  設定ファイルを読み込み中...
=== 設定情報 ===
📝 1行目テキスト: HomeDC Japan
⏱️  スクロール速度: 0.1秒
🔄 2行目切り替え間隔: 3.0秒
🔌 シリアルポート: /dev/ttyUSB0
📡 ボーレート: 115200
💡 バックライト: 100%

🔗 CFA631に接続中...
```

## 🛑 終了方法
`Ctrl + C` で停止。  
LCD には  
```
HomeDC Japan

```
のみが残るため、次回起動時に前回の文字列が残りません。

## OS起動時に自動で実行する方法（systemdサービス）

### 1. サービスファイルの作成

```bash
sudo nano /etc/systemd/system/cfa631-monitor.service
```

以下の内容を記述してください：

```ini
[Unit]
Description=CFA631 System Monitor for HomeDC Japan
After=network.target
Wants=network.target

[Service]
Type=simple
User=root
# 実際のファイルパスに変更してください
WorkingDirectory=/home/pi/cfa631-monitor
ExecStart=/home/pi/cfa631-monitor/cfa631_display.bin
Restart=always
RestartSec=5
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

### 2. サービスの有効化と開始

```bash
# サービスファイルを再読み込み
sudo systemctl daemon-reload

# サービスを有効化（起動時に自動実行）
sudo systemctl enable cfa631-monitor.service

# サービスを開始
sudo systemctl start cfa631-monitor.service

# 状態確認
sudo systemctl status cfa631-monitor.service
```

### 3. ログの確認

```bash
# リアルタイムログ表示
sudo journalctl -u cfa631-monitor.service -f

# 過去のログ表示
sudo journalctl -u cfa631-monitor.service
```

### 4. サービス管理コマンド

```bash
# サービス停止
sudo systemctl stop cfa631-monitor.service

# サービス再起動
sudo systemctl restart cfa631-monitor.service

# 自動起動を無効化
sudo systemctl disable cfa631-monitor.service
```

> **注意**: `WorkingDirectory` と `ExecStart` のパスは、実際にファイルを配置した場所に合わせて変更してください。

この設定により、OS起動時に自動的にCFA631 System Monitorが起動し、異常終了時には自動的に再起動されます。

## 💡 トラブルシューティング
| 症状 | 対処 |
|------|------|
| LCD が無反応 | `dmesg | grep tty` でデバイス名を確認し、`config.yml` の `port` を合わせる |
| Permission denied | `sudo usermod -aG dialout $USER` 後ログインし直し |
| 文字化け | LCD は ASCII 20 文字×2 行。全角や UTF-8 は「?」に置換されます |

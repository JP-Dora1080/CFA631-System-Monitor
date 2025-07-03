# CFA631 System Monitor v1.1
for HomeDC Japan

> Crystalfontz 20×2 LCD（CFA631-USB）に  
> ホスト情報／システム情報をリアルタイム表示するソフトウェア

## ✨ 特長
* **1行目**  
  * 設定可能なテキスト + ホスト名を自動スクロール表示  
* **2行目**  
  * CPU使用率/CPU温度 → メモリ使用率 → IPアドレスをサイクル表示
  * キーパッドで手動切り替え可能
  * 自動サイクルモードのON/OFF切り替え
* **キーパッド操作**
  * 左上: CPU情報表示
  * 右上: メモリ情報表示  
  * 左下: IP情報表示
  * 右下: 自動サイクル切り替え
* **設定機能**
  * `config.yml` でスクロール速度・切替間隔・バックライトなどを簡単設定
  * 設定ファイルがない場合は自動生成
* **EEPROM保存**
  * 終了時に現在の表示状態を電源投入時の状態として保存
  * 次回電源投入時に前回の表示が復元される

## 🔧 ファイル構成

```
.
├─ cfa631_display.bin   # メインスクリプト（バイナリ化済み）
└─ config.yml          # 設定ファイル（無ければ自動で生成）
```

### config.yml 例

```yaml
# CFA631 Display Configuration
display:
  line1:
    text: 'Crystalfontz'    # 1行目の固定テキスト
    scroll_speed: 0.1       # スクロール間隔 [s]
  line2:
    auto_cycle: false       # 自動サイクルの初期状態
    cycle_interval: 2.0     # 自動サイクル間隔 [s]

system:
  port: '/dev/ttyUSB0'      # CFA631 が刺さるデバイス
  baudrate: 115200          # デフォルト115200
  backlight: 100            # 0–100 %

keypad:
  debounce_time: 0.1        # キーパッドのデバウンス時間 [s]

eeprom:
  save_on_exit: true        # 終了時にEEPROMに保存するか
```

## 🚀 使い方

1. **スクリプトに実行権を付与**  
   ```bash
   chmod +x cfa631_display.bin
   ```

2. **デバイス名を確認**  
   ```bash
   ls /dev/ttyUSB*
   # または
   dmesg | grep tty
   ```

3. **実行**  
   ```bash
   ./cfa631_display.bin
   ```

   起動すると以下のようなバナーが表示されます：

```
╔══════════════════════════════════════════════════════════╗
║         CFA631 LCD Setting System v1.1                   ║
║                   for HomeDC Japan                       ║
╠══════════════════════════════════════════════════════════╣
║  作成者    : Veulx                                        ║
║  バージョン: v1.1                                          ║
║  作成日    : 2025-07-02                                   ║
║  説明      : CFA631 LCD用システム監視ツール                  ║
╚══════════════════════════════════════════════════════════╝
⚙️  設定ファイルを読み込み中...
=== 設定情報 ===
📝 1行目テキスト: Crystalfontz
⏱️ スクロール速度: 0.1秒
🔄 自動サイクル初期状態: False
⏰ 自動サイクル間隔: 2.0秒
⏰ デバウンス時間: 0.1秒
🔌 シリアルポート: /dev/ttyUSB0
📡 ボーレート: 115200
💡 バックライト: 100%
💾 終了時EEPROM保存: True

🔗 CFA631に接続中...
🖥️ システム監視を開始します
🎮 キーパッド操作:
   左上: CPU情報    | 右上: メモリ情報
   左下: IP情報     | 右下: 自動サイクル切り替え
```

## 🎮 キーパッド操作

| キー | 機能 | 説明 |
|------|------|------|
| 左上 | CPU情報 | CPU使用率/温度を表示（自動サイクル停止） |
| 右上 | メモリ情報 | メモリ使用率を表示（自動サイクル停止） |
| 左下 | IP情報 | IPアドレスを表示（自動サイクル停止） |
| 右下 | 自動サイクル切り替え | 自動モードのON/OFF切り替え |

* 手動でキーを押すと自動サイクルが停止します
* 右下キーで自動サイクルを再開できます
* モード切り替え時に一時メッセージが表示されます

## 🛑 終了方法
`Ctrl + C` で停止。  

**終了時の動作:**
* 設定ファイルで `eeprom.save_on_exit: true` の場合：
  * 1行目に設定テキスト、2行目を空白に設定
  * 現在の状態をEEPROMに保存（次回電源投入時に復元）
* `eeprom.save_on_exit: false` の場合：
  * 表示のみ設定（EEPROM保存はスキップ）

## 📊 表示される情報

### 1行目（スクロール表示）
* 設定テキスト + ホスト名
* 例: `Crystalfontz - hostname`

### 2行目（サイクル表示）
1. **CPU情報**: `CPU:45.2%/67.3C`
2. **メモリ情報**: `Memory:78.5%`
3. **IP情報**: `IP:192.168.1.100`

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

## 💡 トラブルシューティング

| 症状 | 対処 |
|------|------|
| LCD が無反応 | `dmesg \| grep tty` でデバイス名を確認し、`config.yml` の `port` を合わせる |
| Permission denied | `sudo usermod -aG dialout $USER` 後ログインし直し |
| 文字化け | LCD は ASCII 20 文字×2 行。全角や UTF-8 は「?」に置換されます |
| キーパッドが反応しない | デバウンス時間を調整（`config.yml` の `keypad.debounce_time`） |
| 自動サイクルが動作しない | 右下キーで自動モードを有効にしてください |
| EEPROM保存されない | `config.yml` の `eeprom.save_on_exit` が `true` になっているか確認 |

## 🔄 バージョン履歴

### v1.1 (2025-07-02)
- キーパッド操作機能を追加
- 自動サイクルモードの実装
- 一時メッセージ表示機能
- EEPROM保存機能
- 設定ファイルの自動生成
- CPU温度表示の改善

### v1.0-beta (2025-07-01)
- 初回リリース
- 基本的なシステム監視機能
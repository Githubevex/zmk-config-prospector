# Prospector Scanner - ZMK ステータス表示デバイス

> 🎉 **NEW: v2.0.0 "Touch & Precision" リリース！ (2025年11月20日)**
>
> **🎯 タッチパネル対応**: 任意で CST816S タッチを統合（スワイプジェスチャー対応）
>
> **⚡ USB 接続の修正**: キーボードが USB 接続のときに "> USB" を表示（Issue #5）
>
> **🔒 スレッドセーフ**: LVGL のミューテックス保護により完全にフリーズを排除
>
> **⏱️ タイムアウト減光**: キーボード操作がない場合の輝度低下を設定可能
>
> 👉 **[v2.0 のリリースノート全文 →](docs/RELEASES/v2.0.0/release_notes.md)**

<p align="center">
  <img src="https://img.shields.io/badge/version-v2.0.0-brightgreen" alt="Version 2.0.0 Touch & Precision">
  <img src="https://img.shields.io/badge/status-production%20ready-brightgreen" alt="Production Ready">
  <img src="https://img.shields.io/badge/ZMK-compatible-blue" alt="ZMK Compatible">
  <img src="https://img.shields.io/badge/license-MIT-green" alt="MIT License">
</p>

---

## 📋 目次

1. [Prospector Scanner とは？](#prospector-scanner-とは)
2. [主な機能](#-主な機能)
3. [タッチモード vs 非タッチモード](#タッチモード-vs-非タッチモード)
4. [クイックスタート](#クイックスタート)
5. [ハードウェア & 配線](#ハードウェア--配線)
6. [設定ガイド](#設定ガイド)
7. [アーキテクチャ概要](#️-アーキテクチャ概要)
8. [プロトコル仕様](#-プロトコル仕様)
9. [ディスプレイ機能](#-ディスプレイ機能)
10. [キーボード統合](#キーボード統合)
11. [ドキュメント](#ドキュメント)

---

## Prospector Scanner とは？

**Prospector Scanner** は、ZMK キーボード向けの独立した BLE ステータス表示デバイスです。キーボードの状態（バッテリー、レイヤー、修飾キー、WPM など）を、BLE 接続スロットを消費せずにリアルタイムで監視します。

### Prospector について

**Prospector** は、もともと [carrefinho](https://github.com/carrefinho/prospector) により、ZMK キーボード用の汎用ドングル（キーボード → ドングル → PC 接続）として作られたコミュニティ向けハードウェアプラットフォームです。本プロジェクトは同じハードウェアプラットフォームを使いながら、まったく別の用途で利用します。

**元の Prospector（ドングルモード）**:
- キーボードが Prospector に BLE 接続
- Prospector が PC に USB または BLE で接続
- ⚠️ **制限**: キーボードはドングルにしか接続できない（複数デバイス接続ができなくなる）
- ⚠️ **制限**: キーボード固有のドングル用シールド設定が必要
- ⚠️ **制限**: PC がキーボードへ直接接続できない

**本プロジェクト（スキャナーモード）**:
- キーボードが BLE Advertisement（ブロードキャスト）でステータスを送信（observer mode）
- Prospector は接続せず、**受信するだけ**
- ✅ **利点**: キーボードは最大 5 台までのデバイス接続を維持
- ✅ **利点**: どの ZMK キーボードでも動作（シールド不要）
- ✅ **利点**: キーボードを PC/タブレット/スマホに直接接続できる

### なぜスキャナーモード？

```
Dongle Mode (Original):              Scanner Mode (This Project):
┌─────────────┐                      ┌─────────────┐
│  Keyboard   │─BLE Connection──┐    │  Keyboard   │  BLE Adv (broadcast)
└─────────────┘                 │    └─────────────┘       ↓
                                ↓           │              ↓
                        ┌──────────────┐   ├── PC     ┌──────────────┐
                        │   Dongle     │   ├── Tablet │   Scanner    │
                        │ (Prospector) │   ├── Phone  │ (Prospector) │
                        └──────────────┘   └── ...    └──────────────┘
                                │                      (just monitors)
                                └─→ PC (only 1 device)
Keyboard loses multi-device!         Keyboard keeps 5 devices!
```

**最大のメリット**: Scanner を使っても、キーボードは複数デバイス接続を維持したまま、状態だけを表示できます。

### ハードウェアプラットフォーム

両モードとも同じハードウェアを使用します。
- **MCU**: Seeeduino XIAO BLE（nRF52840）
- **Display**: Waveshare 1.69" Round LCD（タッチパネル付き、ST7789V + CST816S）
- **3D ケース**: 元の Prospector のオープンソース設計

**重要**: 「非タッチモード」でもタッチ対応 LCD は同じで、タッチ用の 4 本のピンを配線しないだけです。元の Prospector も同じディスプレイを使いますが、タッチ機能は使用していません。

### 何が嬉しい？

- ✅ **全部見える**: バッテリー、レイヤー、修飾キー、WPM、信号強度
- ✅ **マルチキーボード**: 最大 5 台のキーボードを同時監視
- ✅ **接続コスト 0**: BLE Advertisement（observer mode）を利用
- ✅ **プロ品質 UI**: NerdFont アイコン付きの YADS 風ウィジェットレイアウト
- ✅ **分割キーボード対応**: 左右両側の情報を表示

## 🎯 主な機能

### 📱 **YADS 風のプロフェッショナル UI**
- **複数ウィジェット表示**: 接続状態、レイヤー表示、修飾キー、WPM、バッテリー、信号強度
- **分割キーボード対応**: 左右情報を 1 画面で統合表示（レイアウトは自動調整）
- **色分けステータス**: 5 段階バッテリー表示（緑/薄緑/黄/オレンジ/赤）、レイヤーはパステル色
- **リアルタイム更新**: サブ秒のレイテンシで即座に反映
- **WPM トラッキング**: アイドル時の減衰を含むリアルタイム WPM 計算
- **NerdFont アイコン**: 修飾キー表示（󰘴 Ctrl, 󰘵 Shift, 󰘶 Alt,  GUI）

### 🔋 **スマート電源管理**（v2.0 強化）
- **アクティビティ連動の送信間隔**: 入力中 10Hz（100ms）、アイドル時 0.03Hz（30 秒）で省電力
- **自動遷移**: アクティブ/アイドルの切り替えをシームレスに実行（タイムアウト設定可）
- **WPM 連動更新**: 入力セッション中の更新頻度を上げる（減衰アルゴリズム）
- **Scanner のバッテリー対応**: Scanner デバイス自身のバッテリーを監視（充電表示あり）
- **タイムアウト減光**: 一定時間受信がない場合に輝度を 5% へ自動低下（v2.0）
- **USB/バッテリープロファイル**: 電源種別ごとに輝度・電源設定を変更

### 🎮 **汎用互換性**
- **どの ZMK キーボードでも**: 分割/一体型を問わず、ZMK 対応デバイスで動作
- **非侵入**: キーボードは最大 5 台のデバイス接続を維持
- **マルチキーボード**: 最大 5 台を同時監視（チャンネル分離）
- **ペアリング不要**: BLE Advertisement（observer mode）で受信（接続スロット消費なし）

### 🎯 **タッチパネル対応**（v2.0 新機能）
- **任意の CST816S タッチ**: `CONFIG_PROSPECTOR_TOUCH_ENABLED=y` で有効化
- **4 方向スワイプ**: 設定画面を直感操作
- **ディスプレイ設定画面**: 輝度、オート輝度 ON/OFF をタッチスライダーで調整
- **システム設定画面**: 今後の拡張用
- **スレッドセーフな LVGL**: ミューテックスでフリーズを排除
- **動的メモリ**: ウィジェットはオンデマンド生成/破棄

### 🌞 **自動輝度**（v1.1+）
- **APDS9960 対応**: 周囲光センサー（4 ピン接続、割り込み不要）
- **滑らかな遷移**: フェード時間とステップ数を設定可能（既定 800ms、12 ステップ）
- **非線形応答**: 照度に応じて見やすい明るさへマッピング
- **自動/手動切り替え**: センサー制御と手動調整を切り替え（タッチモード）
- **アクティビティ連動減光**: キーボードがアイドルになると 5% まで低下（v2.0）

### v2.0 の新機能

- 🎯 **タッチパネル対応**: 任意の CST816S タッチで設定操作
- 🔌 **USB 表示修正**: USB 接続時に "> USB" を表示
- 🔒 **スレッドセーフ**: ジェスチャー中のランダムフリーズを解消
- ⏱️ **タイムアウト減光**: キーボードの無操作で 5% まで自動減光
- 📊 **信号更新**: 受信強度は安定した 1Hz 更新
- 🌞 **4 ピン APDS9960**: 割り込み無しで周囲光センサー対応

---

## タッチモード vs 非タッチモード

v2.0 は **2 つのビルド構成**（タッチモード / 非タッチモード）をサポートします。

### 簡易比較

| 機能 | 非タッチモード | タッチモード |
|---------|---------------|------------|
| **Display** | Waveshare 1.69" タッチ LCD | 同じディスプレイ |
| **配線** | ディスプレイ 6 ピン + 電源（タッチピンは **未接続**） | **+4 タッチピン**（TP_SDA/SCL/INT/RST） |
| **設定** | Kconfig のみ（変更には再ビルドが必要） | デバイス上で対話的に調整 |
| **ジェスチャー** | 非対応 | 4 方向スワイプ |
| **ファームサイズ** | 約 900KB | 約 920KB（+20KB） |
| **設定方法** | `prospector_scanner.conf` を編集 | `prospector_scanner.conf` を編集 + タッチを有効化 |

**補足**: 両モードとも **同じ Waveshare 1.69" タッチ LCD** を使用します。非タッチモードはタッチ用 4 ピンを未接続にするだけ（元の Prospector と同様）です。

### どちらを選ぶ？

#### **非タッチモード**が向いている人
- ✅ 配線を簡単にしたい（ディスプレイ 6 ピンのみ、タッチピン不要）
- ✅ デバイス上で設定を変える必要がない
- ✅ ファームをできるだけシンプルにしたい
- ✅ 元の Prospector の配線構成に合わせたい

#### **タッチモード**が向いている人
- ✅ タッチ用 4 ピンを配線して操作したい
- ✅ 再ビルドせずに設定を変更したい
- ✅ 今後の機能拡張（ジェスチャー等）を活用したい
- ✅ +4 本の配線に抵抗がない

### タッチモード詳細

👉 **[タッチモード完全ガイド →](docs/TOUCH_MODE.md)**

タッチモードで必要なもの:
- Waveshare 1.69" Round LCD（CST816S タッチコントローラ付き）
- **追加の 4 接続**: TP_SDA, TP_SCL, TP_INT, TP_RST
- 設定変更: `prospector_scanner.conf` で `CONFIG_PROSPECTOR_TOUCH_ENABLED=y`

**このガイドは非タッチモード**（標準 Prospector 配線）に焦点を当てています。タッチ固有のセットアップはタッチモードガイドを参照してください。

---

## クイックスタート

### 前提条件

- **Hardware**: Seeeduino XIAO BLE + Waveshare 1.69" Round LCD
- **Keyboard**: ステータス Advertisement を有効化した ZMK キーボード（[キーボード統合](#キーボード統合)を参照）

### Step 1: ファームウェアを入手

#### Option A: GitHub Actions（推奨）

1. **このリポジトリを Fork**
   - https://github.com/t-ogura/zmk-config-prospector を開く
   - 右上の "Fork" をクリック

2. **GitHub Actions を有効化**
   - Fork 先の "Actions" タブへ
   - "I understand my workflows, enable them" をクリック

3. **ビルドを実行**
   - "Actions" タブへ
   - "Build" ワークフローを選択
   - "Run workflow" → "Run workflow"
   - 約 5〜10 分待つ

4. **ファームウェアをダウンロード**
   - "Artifacts" セクションから
   - "firmware" zip をダウンロード
   - `prospector_scanner-seeeduino_xiao_ble-zmk.uf2` を取り出す
   - **既定**: 非タッチモード（ディスプレイ 6 ピン配線）
   - **タッチモードの場合**: Fork 先でタッチを有効化する手順は [Touch Mode Guide](docs/TOUCH_MODE.md) を参照

#### Option B: ローカルビルド

```bash
# Clone and setup
git clone https://github.com/YOUR_USERNAME/zmk-config-prospector.git
cd zmk-config-prospector
python3 -m venv .venv
source .venv/bin/activate
pip install west

# Initialize workspace
west init -l config
west update

# Build (non-touch mode)
west build -b seeeduino_xiao_ble -s zmk/app -- \
  -DSHIELD=prospector_scanner \
  -DZMK_CONFIG="$(pwd)/config"

# Output: build/zephyr/zmk.uf2
```

### Step 2: ハードウェアを配線

ピン配置は [ハードウェア & 配線](#ハードウェア--配線) を参照してください。

**最小接続**（ディスプレイ 6 ピン + 電源）:
```
LCD_DIN  → Pin 10 (MOSI)
LCD_CLK  → Pin 8  (SCK)
LCD_CS   → Pin 9  (CS)
LCD_DC   → Pin 7  (Data/Command)
LCD_RST  → Pin 3  (Reset)
LCD_BL   → Pin 6  (Backlight PWM)
VCC      → 3.3V
GND      → GND
```

**任意**: APDS9960 センサー（4 ピン、割り込み不要）
```
SDA → D4 (P0.04)
SCL → D5 (P0.05)
VCC → 3.3V
GND → GND
```

### Step 3: ファームウェアを書き込み

1. **ブートローダーへ入る**:
   - XIAO BLE を USB 接続
   - RESET ボタンを **素早く 2 回** 押す（0.5 秒以内）
   - `XIAO-SENSE` ドライブが出現

2. **書き込み**:
   - `.uf2` ファイルを `XIAO-SENSE` ドライブへドラッグ
   - ドライブは自動的に切断
   - Scanner が再起動

### Step 4: キーボードを設定

キーボードの `.conf` に追加:

```conf
# Enable status advertisement
CONFIG_ZMK_STATUS_ADVERTISEMENT=y
CONFIG_ZMK_STATUS_ADV_KEYBOARD_NAME="MyKeyboard"

# Power optimization (10Hz active, 0.03Hz idle)
CONFIG_ZMK_STATUS_ADV_ACTIVITY_BASED=y
CONFIG_ZMK_STATUS_ADV_ACTIVE_INTERVAL_MS=100
CONFIG_ZMK_STATUS_ADV_IDLE_INTERVAL_MS=30000

# Split keyboard battery monitoring (if applicable)
CONFIG_ZMK_SPLIT_BLE_CENTRAL_BATTERY_LEVEL_FETCHING=y
```

キーボードのファームウェアを再ビルドして書き込んでください。

### Step 5: 動作確認

1. Scanner の電源を入れる（"Waiting for keyboards..." と表示されるはず）
2. キーボードの電源を入れる
3. 1〜2 秒で Scanner がキーボードを検出
4. 画面にデバイス名、レイヤー、バッテリー等が表示される

**成功！** Scanner が動作しています。

---

## ハードウェア & 配線

### 必須部品

| 部品 | 仕様 | Link |
|-----------|--------------|------|
| **MCU** | Seeeduino XIAO BLE (nRF52840) | [Seeed Studio](https://www.seeedstudio.com/Seeed-XIAO-BLE-nRF52840-p-5201.html) |
| **Display** | Waveshare 1.69" Round LCD (ST7789V) | [Waveshare](https://www.waveshare.com/1.69inch-lcd-module.htm) |

### 任意部品

| 部品 | 用途 | Link |
|-----------|---------|------|
| **APDS9960** | 周囲光センサー（自動輝度） | [Adafruit](https://www.adafruit.com/product/3595) |
| **LiPo Battery** | 携帯運用（400〜600mAh） | Generic 3.7V LiPo |

### ピン割り当て

#### ディスプレイ接続（必須）

| Display Pin | XIAO BLE Pin | nRF52840 GPIO | 役割 |
|------------|--------------|---------------|----------|
| LCD_DIN | Pin 10 | P1.15 | SPI MOSI（データ） |
| LCD_CLK | Pin 8 | P1.13 | SPI クロック |
| LCD_CS | Pin 9 | P1.14 | チップセレクト |
| LCD_DC | Pin 7 | P1.12 | Data/Command 切り替え |
| LCD_RST | Pin 3 | P0.03 | ディスプレイリセット |
| LCD_BL | Pin 6 | P1.11 | バックライト PWM |
| VCC | 3.3V | - | 電源（3.3V） |
| GND | GND | - | GND |

#### APDS9960 センサー（任意）

| Sensor Pin | XIAO BLE Pin | nRF52840 GPIO | 役割 |
|-----------|--------------|---------------|----------|
| VCC | 3.3V | - | 電源（3.3V） |
| GND | GND | - | GND |
| SDA | D4 | P0.04 | I2C データ |
| SCL | D5 | P0.05 | I2C クロック |

**Note**: v2.0 は 4 ピン接続（ポーリング）をサポートし、割り込みピンは不要です。

#### バッテリー（任意）

| Battery Wire | XIAO BLE Pad | 説明 |
|-------------|--------------|-------------|
| + (Red) | BAT+ | 正極 |
| - (Black) | BAT- | GND |

### 配線図

```
Waveshare 1.69" LCD             Seeeduino XIAO BLE
┌─────────────────┐             ┌──────────────┐
│                 │             │              │
│  Display Pins   │             │  3.3V ───────┼─── VCC
│  ├─ LCD_DIN ────┼─────────────┤  Pin 10      │
│  ├─ LCD_CLK ────┼─────────────┤  Pin 8       │
│  ├─ LCD_CS ─────┼─────────────┤  Pin 9       │
│  ├─ LCD_DC ─────┼─────────────┤  Pin 7       │
│  ├─ LCD_RST ────┼─────────────┤  Pin 3       │
│  ├─ LCD_BL ─────┼─────────────┤  Pin 6       │
│  ├─ VCC ────────┼─────────────┤  3.3V        │
│  └─ GND ────────┼─────────────┤  GND         │
│                 │             │              │
└─────────────────┘             └──────────────┘

Optional: APDS9960 Sensor
┌─────────────┐
│  APDS9960   │
│  VCC ───────┼─────────────────┤  3.3V        │
│  GND ───────┼─────────────────┤  GND         │
│  SDA ───────┼─────────────────┤  D4          │
│  SCL ───────┼─────────────────┤  D5          │
└─────────────┘                 └──────────────┘

Optional: LiPo Battery
             ┌─ BAT+ (Red wire)
     Battery ┤
             └─ BAT- (Black wire)
```

### 配線のコツ

1. **まずディスプレイだけテスト**: ディスプレイ配線のみで動作確認してからセンサーを追加
2. **配線は短く**: I2C は配線 10cm 未満が安定
3. **内蔵プルアップ**: XIAO BLE は D4/D5 に I2C プルアップがあるため外付け抵抗不要
4. **極性確認**: 電源投入前に VCC/GND を再確認
5. **はんだ品質**: はんだ不良は断続的な表示不良の原因

---

## 設定ガイド

設定ファイル: `config/prospector_scanner.conf`

### 必須設定

#### スキャナーモード（必須）

```conf
# Enable scanner mode
CONFIG_PROSPECTOR_MODE_SCANNER=y

# Multi-keyboard support
CONFIG_PROSPECTOR_MULTI_KEYBOARD=y
CONFIG_PROSPECTOR_MAX_KEYBOARDS=5     # Monitor up to 5 keyboards
```

#### Display & LVGL

```conf
# Display subsystem
CONFIG_ZMK_DISPLAY=y
CONFIG_DISPLAY=y
CONFIG_LVGL=y

# Required LVGL widgets
CONFIG_LV_USE_BTN=y
CONFIG_LV_USE_SLIDER=y
CONFIG_LV_USE_SWITCH=y

# Enable fonts (REQUIRED for widgets to compile)
CONFIG_LV_FONT_MONTSERRAT_12=y
CONFIG_LV_FONT_MONTSERRAT_16=y
CONFIG_LV_FONT_MONTSERRAT_18=y
CONFIG_LV_FONT_MONTSERRAT_20=y        # Default font
CONFIG_LV_FONT_MONTSERRAT_22=y
CONFIG_LV_FONT_MONTSERRAT_24=y
CONFIG_LV_FONT_MONTSERRAT_28=y
CONFIG_LV_FONT_UNSCII_8=y             # Pixel fonts
CONFIG_LV_FONT_UNSCII_16=y

# Set default font
CONFIG_LV_FONT_DEFAULT_MONTSERRAT_20=y
```

**なぜフォントが重要？**: フォントが有効になっていないと、`'lv_font_montserrat_XX' undeclared` エラーでビルドが失敗します。上記のフォントは **すべて** 有効化してください。

### レイヤー表示設定

```conf
# Number of layer indicators shown on screen
CONFIG_PROSPECTOR_MAX_LAYERS=7        # Range: 4-10

# Visual effect:
# - 4 layers: Wide spacing, large indicators
# - 7 layers: Medium spacing (default)
# - 10 layers: Tight spacing, maximum capacity
```

**キーボードに合わせる**: 見た目を最適にするため、キーボードのレイヤー数に合わせて設定してください（例: 5 レイヤーなら 5）。

### 輝度制御

#### 固定輝度（簡単）

```conf
# Disable ambient light sensor
CONFIG_PROSPECTOR_USE_AMBIENT_LIGHT_SENSOR=n

# Set fixed brightness (0-100%)
CONFIG_PROSPECTOR_FIXED_BRIGHTNESS=85
```

APDS9960 が無い場合、または手動制御が好みの場合に適しています。

#### 自動輝度（上級）

```conf
# Enable ambient light sensor (requires APDS9960 hardware)
CONFIG_PROSPECTOR_USE_AMBIENT_LIGHT_SENSOR=y

# Brightness range
CONFIG_PROSPECTOR_ALS_MIN_BRIGHTNESS=20       # Minimum (dark rooms)
CONFIG_PROSPECTOR_ALS_MAX_BRIGHTNESS_USB=100  # Maximum (USB power)

# Smooth fade transitions
CONFIG_PROSPECTOR_BRIGHTNESS_FADE_DURATION_MS=800  # 800ms fade
CONFIG_PROSPECTOR_BRIGHTNESS_FADE_STEPS=12         # 12 steps
```

**動作**: Scanner が数秒ごとに周囲光を読み取り、800ms フェードで 20〜100% に滑らかに調整します。

**ハード要件**: APDS9960 を D4/D5 に配線（4 ピン接続、割り込み不要）。

### タイムアウト輝度（v2.0 新機能）

```conf
# Auto-dim when no keyboard activity
CONFIG_PROSPECTOR_SCANNER_TIMEOUT_MS=480000      # 8 minutes (0=disabled)
CONFIG_PROSPECTOR_SCANNER_TIMEOUT_BRIGHTNESS=5   # Dim to 5%
```

**何をする？**:
1. 8 分間キーボードデータを受信しない → 輝度を 5% に低下
2. 再び受信したら → 直前の輝度へ戻す
3. USB/バッテリー運用どちらでも動作

**用途**: バッテリー運用の Scanner で、キーボードがオフのときに消費を抑える。

**無効化**: `CONFIG_PROSPECTOR_SCANNER_TIMEOUT_MS=0`。

### Scanner バッテリー対応

```conf
# Enable scanner's own battery monitoring
CONFIG_PROSPECTOR_BATTERY_SUPPORT=y   # Requires LiPo connected to XIAO BLE
CONFIG_ZMK_BATTERY_REPORTING=y        # ZMK battery subsystem
```

**表示内容**: 右上にバッテリーアイコン（🔋）と％、充電中表示。

**ハード要件**: LiPo を XIAO BLE の BAT+/BAT- に接続。

**バッテリー無し**: `CONFIG_PROSPECTOR_BATTERY_SUPPORT=n`（バッテリー表示は出ません）。

### USB ログ（開発用）

```conf
# Enable USB serial logging
CONFIG_LOG=y
CONFIG_ZMK_LOG_LEVEL_DBG=y

# Reduce BT noise
CONFIG_BT_LOG_LEVEL_WRN=y
CONFIG_LOG_DEFAULT_LEVEL=4
```

**使い方**:
1. これらを有効化
2. 再ビルド
3. USB 接続
4. シリアルモニタを開く（例: `screen /dev/ttyACM0 115200`）
5. デバッグログを確認

**本番**: `CONFIG_LOG=n` で無効化（サイズ削減）。

### 設定例（全文）

```conf
# ===== SCANNER MODE =====
CONFIG_PROSPECTOR_MODE_SCANNER=y
CONFIG_PROSPECTOR_MULTI_KEYBOARD=y
CONFIG_PROSPECTOR_MAX_KEYBOARDS=5

# ===== DISPLAY =====
CONFIG_ZMK_DISPLAY=y
CONFIG_DISPLAY=y
CONFIG_LVGL=y
CONFIG_PROSPECTOR_MAX_LAYERS=7

# ===== BRIGHTNESS =====
# Option 1: Fixed brightness (simple)
CONFIG_PROSPECTOR_USE_AMBIENT_LIGHT_SENSOR=n
CONFIG_PROSPECTOR_FIXED_BRIGHTNESS=85

# Option 2: Auto-brightness (requires APDS9960)
# CONFIG_PROSPECTOR_USE_AMBIENT_LIGHT_SENSOR=y
# CONFIG_PROSPECTOR_ALS_MIN_BRIGHTNESS=20
# CONFIG_PROSPECTOR_ALS_MAX_BRIGHTNESS_USB=100
# CONFIG_PROSPECTOR_BRIGHTNESS_FADE_DURATION_MS=800

# ===== TIMEOUT =====
CONFIG_PROSPECTOR_SCANNER_TIMEOUT_MS=480000  # 8 min (0=disabled)
CONFIG_PROSPECTOR_SCANNER_TIMEOUT_BRIGHTNESS=5

# ===== BATTERY =====
CONFIG_PROSPECTOR_BATTERY_SUPPORT=n  # Enable if LiPo connected

# ===== FONTS (REQUIRED) =====
CONFIG_LV_FONT_MONTSERRAT_12=y
CONFIG_LV_FONT_MONTSERRAT_16=y
CONFIG_LV_FONT_MONTSERRAT_18=y
CONFIG_LV_FONT_MONTSERRAT_20=y
CONFIG_LV_FONT_MONTSERRAT_22=y
CONFIG_LV_FONT_MONTSERRAT_24=y
CONFIG_LV_FONT_MONTSERRAT_28=y
CONFIG_LV_FONT_UNSCII_8=y
CONFIG_LV_FONT_UNSCII_16=y
CONFIG_LV_FONT_DEFAULT_MONTSERRAT_20=y

# ===== LOGGING (optional) =====
# CONFIG_LOG=y
# CONFIG_ZMK_LOG_LEVEL_DBG=y
```

このテンプレートをコピーして必要に応じて変更してください。

---

## 🏗️ アーキテクチャ概要

### システム設計

**スキャナーモード設計**（独立監視）:
- キーボード → 複数デバイス（通常の BLE 接続で最大 5 台）
- キーボード → Scanner（BLE Advertisement のブロードキャストのみ、接続なし）
- Scanner は接続スロットを消費せず独立動作

### 構成要素

```
┌─────────────┐    BLE Adv     ┌──────────────┐
│   Keyboard  │ ──────────────→│   Scanner    │
│             │    26-byte     │   Display    │
│ (10Hz/0.03Hz)│   Protocol    │  (USB/Battery)│
└─────────────┘                └──────────────┘
       │
       ├── Device 1 (PC)
       ├── Device 2 (Tablet)
       ├── Device 3 (Phone)
       ├── Device 4 (...)
       └── Device 5 (...)
```

**ポイント**:
- キーボードは BLE Advertisement（observer mode）で状態を送信
- Scanner は **受信するだけ**（接続しない）
- キーボードは最大 5 台の接続を維持
- Scanner は USB でもバッテリーでも駆動可能（任意）

### 通信フロー

```
Keyboard                           Scanner
────────                          ────────
[Keypress detected]
    │
    ├─→ Update internal state
    │   (layer, modifiers, WPM)
    │
    ├─→ Package into 26-byte
    │   advertisement payload
    │
    └─→ Broadcast BLE Adv ─────→  [BLE Observer Mode]
        (100ms interval when           │
         typing, 30s when idle)        ├─→ Parse advertisement
                                       │   (battery, layer, etc.)
                                       │
                                       ├─→ Update LVGL widgets
                                       │   (battery bars, layer
                                       │    indicators, WPM, etc.)
                                       │
                                       └─→ Display to screen
                                           (YADS-style UI)
```

---

## 📊 プロトコル仕様

### BLE Advertisement フォーマット（26 bytes）

キーボードは独自の BLE Advertisement ペイロードで状態をブロードキャストします。Scanner は observer mode（接続不要）で受信します。

| Offset | Field | Size | 説明 | 例 |
|--------|-------|------|-------------|---------|
| 0-1 | Manufacturer ID | 2 bytes | `0xFF 0xFF`（カスタム/ローカル用途） | `FF FF` |
| 2-3 | Service UUID | 2 bytes | `0xAB 0xCD`（Prospector Protocol ID） | `AB CD` |
| 4 | Protocol Version | 1 byte | プロトコルバージョン（現在: `0x01`） | `01` |
| 5 | Battery Level | 1 byte | 主バッテリー 0-100%（分割の場合は Central 側） | `5A`（90%） |
| 6 | Active Layer | 1 byte | 現在レイヤー 0-15 | `02`（Layer 2） |
| 7 | Profile Slot | 1 byte | BLE プロファイル 0-4 | `01`（Profile 1） |
| 8 | Connection Count | 1 byte | 接続中 BLE デバイス数 0-5 | `03`（3 台） |
| 9 | Status Flags | 1 byte | USB/BLE/Charging/Caps Lock ビット | `05`（USB+Caps） |
| 10 | Device Role | 1 byte | `0`=Standalone, `1`=Central, `2`=Peripheral | `01`（Central） |
| 11 | Device Index | 1 byte | 分割キーボードのインデックス（0=左、1=右） | `00` |
| 12-14 | Peripheral Batteries | 3 bytes | 左/右/補助バッテリー 0-100% | `52 00 00`（82%, none, none） |
| 15-18 | Layer Name | 4 bytes | ASCII レイヤー識別子（任意） | `4C30...`（"L0"） |
| 19-22 | Keyboard ID | 4 bytes | マルチデバイス用のキーボード名ハッシュ | `12345678` |
| 23 | Modifier Flags | 1 byte | L/R Ctrl,Shift,Alt,GUI 状態 | `05`（LCtrl+LShift） |
| 24 | WPM Value | 1 byte | WPM 0-255 | `3C`（60 WPM） |
| 25 | Reserved | 1 byte | 将来拡張 | `00` |

### Status Flags（Offset 9）

```
Bit 7 6 5 4 3 2 1 0
    │ │ │ │ │ │ │ └─ USB Connected (1=yes, 0=no)
    │ │ │ │ │ │ └─── BLE Active (1=yes, 0=no)
    │ │ │ │ │ └───── Charging (1=yes, 0=no)
    │ │ │ │ └─────── Caps Lock (1=on, 0=off)
    │ │ │ └───────── Reserved
    │ │ └─────────── Reserved
    │ └───────────── Reserved
    └─────────────── Reserved
```

### Modifier Flags（Offset 23）

```
Bit 7 6 5 4 3 2 1 0
    │ │ │ │ │ │ │ └─ Left Ctrl
    │ │ │ │ │ │ └─── Left Shift
    │ │ │ │ │ └───── Left Alt
    │ │ │ │ └─────── Left GUI (Win/Cmd)
    │ │ │ └───────── Right Ctrl
    │ │ └─────────── Right Shift
    │ └───────────── Right Alt
    └─────────────── Right GUI
```

### ブロードキャスト間隔

キーボードはバッテリー節約のため、活動に応じてブロードキャスト頻度を変えます。

- **Active Mode**（入力あり）: 100ms 間隔（10 Hz）
  - どのキー入力でもトリガー
  - WPM とレイヤーをリアルタイムに反映

- **Idle Mode**（入力なし）: 30000ms 間隔（0.03 Hz）
  - 10 秒間キー入力がないと移行
  - 長時間アイドルで省電力
  - 監視用の状態送信は継続

---

## 🎨 ディスプレイ機能

### ウィジェットレイアウト（v2.0）

```
┌─────────────────────────────┐
│ MyKeyboard            🔋 85%│ ← デバイス名（左）、Scanner バッテリー（右、v1.1+）
│ > USB                 [P0] │ ← 接続状態（左: > USB または BLE アイコン、右: プロファイル）
│                             │
│ WPM                    RX   │ ← WPM（左）、信号情報（右）
│ 045             -45dBm 1.0Hz│   （RSSI + 受信レート）
│                             │
│          Layer              │ ← レイヤータイトル
│    0  1  2  3  4  5  6      │ ← パステル色のレイヤー表示（0-15 対応）
│                             │
│        󰘴  󰘶  󰘵              │ ← 修飾キー表示（NerdFont アイコン）
│                             │   󰘴=Ctrl, 󰘵=Shift, 󰘶=Alt, =GUI
│                             │
│    ████████████ 85%         │ ← キーボード主バッテリー（main/central）
│    ████████████ 78%         │ ← 分割キーボードの周辺バッテリー（該当時）
└─────────────────────────────┘
```

### 表示要素

#### バッテリーの色分け（5 段階）

バッテリーバーは色分けで状態が分かりやすいようになっています。

| Range | Color | Hex Code | 見え方 |
|-------|-------|----------|--------|
| **80-100%** | Green | `#00CC66` | 🟢 ほぼ満充電/良好 |
| **60-79%** | Light Green | `#66CC00` | 🟢 良好 |
| **40-59%** | Yellow | `#FFCC00` | 🟡 中程度 |
| **20-39%** | Orange | `#FF9900` | 🟠 少ない |
| **0-19%** | Red | `#FF3333` | 🔴 危険 |

#### レイヤー表示

- **パステルカラー**: レイヤーごとに 7〜15 種類のパステル色
- **表示数を設定可能**: `CONFIG_PROSPECTOR_MAX_LAYERS`（既定 7）
- **動的センタリング**: レイヤー数に応じて間隔を自動調整
  - 4 レイヤー = 広い
  - 10 レイヤー = 詰めて表示
  - 常に中央寄せ
- **アクティブ表示**: 現在レイヤーは明るい色で強調

#### 接続状態

- **USB モード**: "> USB" と表示（v2.0 で Issue #5 を修正）
- **BLE モード**: Bluetooth アイコン
- **プロファイル**: `[P0]` 〜 `[P4]`

#### WPM 表示（Words Per Minute）

- **リアルタイム計算**: 入力中は減衰アルゴリズムで更新
- **3 桁表示**: `000` 〜 `255` WPM
- **アイドル減衰**: 入力が止まると徐々に低下
- **ウィンドウ**: `CONFIG_ZMK_STATUS_ADV_WPM_WINDOW_SECONDS` で設定（10s/30s/60s）

#### 信号表示（右下）

- **RSSI**: dBm 表示（例: `-45dBm` は強い、`-80dBm` は弱い）
- **受信レート**: Hz 表示（例: `10Hz` は入力中、`0.03Hz` はアイドル）

#### Scanner バッテリー（右上、v1.1+）

- **表示**: `🔋 85%`
- **充電表示**: USB 充電中はアイコンが変化
- **任意**: `CONFIG_PROSPECTOR_BATTERY_SUPPORT=y` のときのみ表示

### ディスプレイ輝度（v2.0）

#### 自動輝度（APDS9960 センサー）

- **センサー**: APDS9960（4 ピン、割り込み無し）
- **非線形応答**: センサー値を見やすい輝度にマッピング
- **滑らかな遷移**: フェード（既定 800ms、12 ステップ）
- **USB/バッテリープロファイル**: 電源別に最大輝度が異なる
- **アクティビティ連動減光**: タイムアウト後に 5% へ（設定可）

#### 手動輝度（タッチモード）

- **スワイプ**: 下方向スワイプで Display Settings へ
- **スライダー**: 0〜100% をタッチスライダーで調整
- **永続化**: 設定はフラッシュに保存
- **オート輝度切り替え**: センサー制御の ON/OFF をデバイス上で変更

#### タイムアウト減光（v2.0）

- **自動**: キーボード操作がない場合に 5% へ低下
- **タイムアウト設定**: `CONFIG_PROSPECTOR_SCANNER_TIMEOUT_MS`（既定 8 分）
- **活動で復帰**: キーボードの操作が再開すると通常輝度に戻る

---

## キーボード統合

ZMK キーボード側が BLE Advertisement でステータスをブロードキャストできる必要があります。

### Step 1: キーボードにモジュールを追加

キーボード側の `config/west.yml` を編集:

```yaml
manifest:
  remotes:
    - name: zmkfirmware
      url-base: https://github.com/zmkfirmware
    - name: prospector
      url-base: https://github.com/t-ogura

  projects:
    - name: zmk
      remote: zmkfirmware
      revision: v0.3
      import: app/west.yml

    # Add this:
    - name: prospector-zmk-module
      remote: prospector
      revision: v2.0.0  # Use specific version tag
      path: modules/prospector-zmk-module
```

### Step 2: ステータス Advertisement を有効化

キーボード側の `.conf` に追加:

```conf
# Enable status advertisement
CONFIG_ZMK_STATUS_ADVERTISEMENT=y
CONFIG_ZMK_STATUS_ADV_KEYBOARD_NAME="MyKeyboard"  # Shown on scanner

# Activity-based intervals (battery optimization)
CONFIG_ZMK_STATUS_ADV_ACTIVITY_BASED=y
CONFIG_ZMK_STATUS_ADV_ACTIVE_INTERVAL_MS=100      # 10Hz when typing
CONFIG_ZMK_STATUS_ADV_IDLE_INTERVAL_MS=30000      # 0.03Hz when idle

# Split keyboard battery monitoring (if applicable)
CONFIG_ZMK_SPLIT_BLE_CENTRAL_BATTERY_LEVEL_FETCHING=y

# (Optional) Central side selection for split keyboards
# CONFIG_ZMK_STATUS_ADV_CENTRAL_SIDE="LEFT"  # or "RIGHT" (default)
```

### Step 3: キーボードのファームウェアを再ビルド

```bash
# In your keyboard config directory
west update
west build -b your_board -- -DSHIELD=your_shield

# Flash to keyboard
# (Copy .uf2 to bootloader drive)
```

### Step 4: 確認

1. キーボードの電源を入れる
2. Scanner の電源を入れる
3. 1〜2 秒で検出される
4. デバイス名が `CONFIG_ZMK_STATUS_ADV_KEYBOARD_NAME` と一致していること

---

## ドキュメント

### ガイド
- **[Touch Mode Guide](docs/TOUCH_MODE.md)** - タッチパネルのセットアップと使い方
- **[v2.0 Release Notes](docs/RELEASES/v2.0.0/release_notes.md)** - 変更履歴と移行ガイド
- **[Architecture Design](SCANNER_RECONSTRUCTION_DESIGN.md)** - 技術アーキテクチャ（ローカル開発用ファイル）

### リリース履歴
- **[All Releases](docs/RELEASES.md)** - バージョン履歴とリリースノート
- **v2.0.0**（2025-11-20）: タッチ対応、USB 表示修正、スレッドセーフ化
- **v1.1.1**（2025-08-29）: ハードウェア互換性、10 レイヤー対応
- **v1.1.0**（2025-08-26）: 周囲光センサー、電源最適化
- **v1.0.0**（2025-08-01）: YADS 風 UI の初回リリース

### GitHub リソース
- **[Issues](https://github.com/t-ogura/zmk-config-prospector/issues)** - バグ報告・要望
- **[Actions](https://github.com/t-ogura/zmk-config-prospector/actions)** - 自動ビルド
- **[Releases](https://github.com/t-ogura/zmk-config-prospector/releases)** - ビルド済みファーム

### コミュニティ & 関連プロジェクト
- **[ZMK Discord](https://zmk.dev/community/discord/invite)** - ZMK 全般のサポート
- **[Original Prospector (Dongle Mode)](https://github.com/carrefinho/prospector)** - carrefinho によるハードウェア
- **[Original Prospector Firmware](https://github.com/carrefinho/prospector-zmk-module)** - ドングルモード実装

---

## トラブルシューティング

### Scanner が "Waiting for keyboards..." のまま

**問題**: Scanner がキーボードを検出できない。

**対処**:
1. キーボードで `CONFIG_ZMK_STATUS_ADVERTISEMENT=y` を有効化しているか確認
2. モジュール追加後にキーボードを再ビルド/書き込みしたか確認
3. キーボードの電源 ON、BLE 有効を確認
4. 両方を再起動

### ビルドエラー: `'lv_font_montserrat_XX' undeclared`

**問題**: LVGL フォントが有効になっていない。

**対処**: `prospector_scanner.conf` でフォントを **すべて** 有効化:
```conf
CONFIG_LV_FONT_MONTSERRAT_12=y
CONFIG_LV_FONT_MONTSERRAT_16=y
CONFIG_LV_FONT_MONTSERRAT_18=y
CONFIG_LV_FONT_MONTSERRAT_20=y
CONFIG_LV_FONT_MONTSERRAT_22=y
CONFIG_LV_FONT_MONTSERRAT_24=y
CONFIG_LV_FONT_MONTSERRAT_28=y
CONFIG_LV_FONT_UNSCII_8=y
CONFIG_LV_FONT_UNSCII_16=y
```

### 画面が真っ黒 / 何も表示されない

**問題**: ディスプレイ初期化に失敗。

**対処**:
1. 配線（特に VCC/GND）を確認
2. バックライトピン（LCD_BL → Pin 6）を確認
3. 設定リセット済みファームでテスト
4. XIAO BLE が通電しているか確認（LED）
5. ディスプレイモジュール自体を確認（タッチ版なら非タッチファームで試す等）

### USB 接続なのに表示が "BLE" になる

**問題**: USB 接続が検出されていない（本来は "> USB"）。

**これは v1.x の既知問題**で、**v2.0 で修正**済み。

**対処**: Scanner とキーボードを v2.0 へアップグレード。

### APDS9960 センサーが動かない

**問題**: 照明を変えても輝度が変わらない。

**対処**:
1. 4 ピン配線（VCC/GND/SDA(D4)/SCL(D5)）を確認
2. `CONFIG_PROSPECTOR_USE_AMBIENT_LIGHT_SENSOR=y` が有効か確認
3. センサーがケース等で遮られていないか確認
4. `CONFIG_PROSPECTOR_ALS_MIN_BRIGHTNESS` を上げる（暗すぎる可能性）

**Note**: v2.0 は割り込みピン不要で、4 ピン接続で動作します。

### 使用中にフリーズする

**問題**: 画面更新が止まり、操作不能になる。

**これは v1.x の既知問題**で、**v2.0 で修正**済み（LVGL mutex 保護）。

**対処**: v2.0 へアップグレード。

### バッテリー表示が出ない

**問題**: 右上にバッテリーアイコンが表示されない。

**対処**:
1. LiPo が BAT+/BAT- に接続されているか確認
2. `CONFIG_PROSPECTOR_BATTERY_SUPPORT=y` が有効か確認
3. バッテリー対応を有効化後に再ビルドしたか確認

**バッテリー無し**: `CONFIG_PROSPECTOR_BATTERY_SUPPORT=n` の場合は表示されません（正常）。

---

## 高度なトピック

### 監視キーボード数

Scanner は最大 5 台まで監視できます。`prospector_scanner.conf` で設定:

```conf
CONFIG_PROSPECTOR_MAX_KEYBOARDS=5  # Increase if needed (max: 5)
```

ブロードキャストしているキーボードは自動で表示されます。ペアリングは不要です。

### WPM 計算ウィンドウ

WPM の反応性を調整:

```conf
# Ultra-responsive (10s window, 6x multiplier)
# CONFIG_ZMK_STATUS_ADV_WPM_WINDOW_SECONDS=10

# Balanced (30s window, 2x multiplier) - DEFAULT
# CONFIG_ZMK_STATUS_ADV_WPM_WINDOW_SECONDS=30

# Stable (60s window, 1x multiplier)
# CONFIG_ZMK_STATUS_ADV_WPM_WINDOW_SECONDS=60
```

短いほど反応は良いが変動が大きく、長いほど安定するが反映が遅くなります。

### 分割キーボードの Central 側指定

分割キーボードで中央（BLE プロファイルを持つ側）を指定:

```conf
# In keyboard's .conf file
CONFIG_ZMK_STATUS_ADV_CENTRAL_SIDE="LEFT"  # or "RIGHT" (default)
```

これにより、Scanner がどちらを "main" として接続状態表示に使うか判断します。

### デバッグウィジェット（開発用）

開発用のデバッグオーバーレイを有効化:

```conf
CONFIG_PROSPECTOR_DEBUG_WIDGET=y
```

画面上に技術情報を重ねて表示します。**本番では無効化**してください（`=n`）。

---

## ライセンス & クレジット

### ライセンス

本プロジェクトは **MIT License** です。詳細は `LICENSE` を参照してください。

### サードパーティ

#### ZMK Firmware
- **License**: MIT License
- **Source**: https://github.com/zmkfirmware/zmk

#### YADS（Yet Another Dongle Screen）
- **License**: MIT License
- **Source**: https://github.com/janpfischer/zmk-dongle-screen
- **Attribution**: UI ウィジェット設計と NerdFont 修飾キー記号

#### NerdFonts
- **License**: MIT License
- **Source**: https://www.nerdfonts.com/
- **Usage**: 修飾キー記号

### 元の Prospector

本プロジェクトは、carrefinho による Prospector ハードウェアプラットフォームを基にしています。
- **Original Project**: [prospector](https://github.com/carrefinho/prospector) by carrefinho
- **Original Firmware**: [prospector-zmk-module](https://github.com/carrefinho/prospector-zmk-module)
- **Hardware Design**: Seeeduino XIAO BLE + Waveshare 1.69" Round LCD（タッチパネル付き）
- **3D Case Design**: 3D プリント用 STL（オープンソース）
- **License**: MIT License

**違い**: 元の Prospector はドングル用途（キーボードが Prospector に接続）ですが、本プロジェクトは独立モニター用途（キーボードは独立したまま）です。どちらも優れたハードウェアプラットフォームの有効な活用方法です。

---

## Contributing

貢献歓迎です。以下の手順でお願いします。

1. リポジトリを Fork
2. feature ブランチを作成（`git checkout -b feature/amazing-feature`）
3. 変更を加える
4. 十分にテスト
5. コミット（`git commit -m 'Add amazing feature'`）
6. Push（`git push origin feature/amazing-feature`）
7. Pull Request を作成

大きな変更は、まず issue を立てて相談してください。

---

**質問がある場合**: [issue](https://github.com/t-ogura/zmk-config-prospector/issues) を開くか、[ZMK Discord](https://zmk.dev/community/discord/invite) に参加してください。

**Prospector Scanner v2.0.0** - ZMK ステータス表示デバイス

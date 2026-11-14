# SONIC COLLAPSE

> 音は、世界を創り、壊し、再構築する。
> *An interactive audio-visual piece where sound builds, breaks, and rebuilds a 3D world — in the browser.*

音楽をリアルタイムに解析し、その特徴を3D空間の形・粒子・光・世界の構造そのものへ変換するオーディオビジュアル作品です。
曲の展開に合わせて世界が **静寂 → 構築 → 崩壊 → 再生** と移り変わり、鑑賞者は自分の手で世界を叩き、エネルギーをためて壊し、円を描いて組み直すことができます。

**▶ デモ：** `https://<your-github-id>.github.io/sonic-collapse/` （GitHub Pagesを有効にすると公開されます）

<!-- スクリーンショットを docs/screenshot.png に置いたら、下の行のコメントを外す -->
<!-- ![screenshot](docs/screenshot.png) -->

---

## 特徴

| | |
|---|---|
| 🎧 **リアルタイム音響解析** | FFT（2048点）で低域・中域・高域に分け、RMS（音量）、低域のスペクトル変化によるビート検出、BPM推定を毎フレーム計算 |
| 🌌 **4フェーズの世界** | 相対音量・ビート密度・音量の急上昇（ドロップ）から、静寂／構築／崩壊／再生を自動で判定 |
| ✋ **体験できる** | クリックで叩く、長押しでためて崩壊させる、円を描いて再構築する、キーボードで演奏する |
| 🎛 **マッピング編集** | 「どの音の特徴を、映像のどの要素に割り当てるか」をユーザーが変更できる |
| 💾 **プリセット** | 設定を3スロットに保存。JSONでの書き出し・読み込みにも対応 |
| 🎵 **権利フリーのデモ曲** | ページ内のプログラムで合成したオリジナル曲（約64秒）を同梱。外部音源ファイルは不要 |

## 操作方法

| 操作 | 内容 |
|---|---|
| **クリック** | 核を叩く。画面の高さで音程、左右の位置で定位が変わる（Aマイナーペンタトニック） |
| **長押し → 離す** | エネルギーをためて世界を崩壊させる。長くためるほど崩壊が長く続く |
| **円を描く**（再生フェーズ中） | 粒子を中心に集め、2.5周で世界を再構築する |
| `Q` 〜 `I` | メロディ |
| `Z` `X` `C` `V` | キック／スネア／ハット／クラッシュ |
| `Space` | 再生／一時停止 |
| `1` 〜 `4` | フェーズを手動で切り替え |
| `H` | UIを隠す（紹介動画の録画用） |
| `F` | 全画面 |

スタート画面では「デモ曲で体験する」「音源ファイルを選ぶ（WAV推奨・MP3/M4A/OGG可）」「音源なしで遊ぶ」の3つから始められます。

## 仕組み

```
音源（ファイル / デモ曲 / 演奏）
   │  Web Audio API
   ▼
AnalyserNode ── FFT 2048点 ──► LOW 20–250Hz / MID 250–2kHz / HIGH 2k–12kHz
   │                         ──► RMS（音量）
   │                         ──► 低域のスペクトルフラックス → 適応しきい値 → ビート候補 → BPM推定
   ▼
自動ゲイン調整（曲ごとの音量差を吸収） → 平滑化
   ▼
フェーズ判定（相対音量・ビート密度・音量の急上昇）
   ▼
マッピング（音の特徴 → 映像パラメータ）
   ▼
Three.js / GLSL：虹色の殻・内側の光・粒子・魔法陣の輪・衝撃波 → 残像・発光・RGBずれ
```

詳しくは [docs/TECHNICAL.md](docs/TECHNICAL.md) を参照してください。

## 技術構成

| 領域 | 使用技術 |
|---|---|
| 音声の入力・再生・合成 | Web Audio API（AudioBufferSourceNode, GainNode, BiquadFilterNode, StereoPannerNode, OscillatorNode） |
| 解析 | AnalyserNode, FFT, RMS, スペクトルフラックス |
| 3D描画 | Three.js r160 / WebGL2 |
| シェーダー | GLSL（3Dシンプレックスノイズによる頂点変形、フレネル、薄膜干渉風の色、GPU粒子） |
| ポストエフェクト | UnrealBloomPass, AfterimagePass, RGBShift |
| 画面 | HTML / CSS / JavaScript（ES Modules、ビルド不要の1ファイル構成） |
| 保存 | localStorage, JSON |

外部APIやサーバーには依存せず、ブラウザだけで動きます（Three.jsとWebフォントはCDNから読み込み）。

## ローカルで動かす

ES Modulesを使っているため、ファイルを直接開くのではなくローカルサーバー経由で開いてください。

```bash
git clone https://github.com/<your-github-id>/sonic-collapse.git
cd sonic-collapse
python3 -m http.server 8000
# → http://localhost:8000 を開く
```

動作確認環境：Chrome / Edge の最新版（WebGL2 と Web Audio API が必要）。

## GitHub Pagesで公開する

1. リポジトリの **Settings → Pages** を開く
2. **Source** を「Deploy from a branch」、ブランチを `main`、フォルダを `/ (root)` にして保存
3. 数分後に `https://<your-github-id>.github.io/sonic-collapse/` で公開される

## 検証

| 項目 | 方法 | 状況 |
|---|---|---|
| BPM推定 | 128 BPMのデモ曲で推定値を確認 | 約129 BPMと推定（誤差1 BPM程度） |
| フェーズ判定 | デモ曲の展開（イントロ→ビルド→ドロップ→ブレイク→ドロップ）とフェーズの切り替えを比較 | 曲の展開どおりに遷移することを確認 |
| シェーダー | 全シェーダーをWebGL2でコンパイル | エラーなし |
| 操作フロー | ヘッドレスブラウザで、開始→崩壊→再構築→フリープレイを自動操作 | エラーなし |
| 既知のテスト信号でのFFT・RMS確認 | 正弦波・ホワイトノイズで計算値を確認 | 未実施 |
| フレームレート・遅延の測定 | 端末別に計測 | 未実施 |
| ユーザーテスト | 少人数で印象・操作性を評価 | 未実施 |

## 既知の制約

- ビート検出は低域の変化を使っているため、低音の少ない曲（アコースティック、クラシックなど）では検出されにくい。感度スライダーで調整できる。
- フェーズ判定はヒューリスティック（経験則によるルール）で、すべての曲で意図どおりに動くとは限らない。
- 映像の書き出し機能はまだない。`H` でUIを隠し、OSの画面録画機能で録画する。
- マイク入力には未対応。

## 今後の予定

- [ ] 既知のテスト信号による解析精度の検証ページ
- [ ] 映像の録画・書き出し（MediaRecorder）
- [ ] マイク入力（許可を得たうえで）
- [ ] TypeScript + Vite への移行とモジュール分割
- [ ] スマートフォンのタッチ操作の最適化
- [ ] ユーザーテストと改善

## 更新履歴

| バージョン | 内容 |
|---|---|
| v0.3 | 核を虹色の半透明の殻に変更。内側の光、後光、魔法陣の輪を追加 |
| v0.2 | 体験要素を追加（叩く、ためて崩壊、円を描いて再構築、キーボード演奏、音源なしモード） |
| v0.1 | 最小構成（音源入力、FFT解析、ビート検出、4フェーズ、マッピング編集、プリセット） |

## 開発について

企画・仕様・体験設計は作者によるものです。実装には AI（Claude）を開発補助として利用し、設計の判断、動作確認、調整は作者が行っています。
<!-- 自分で書いた部分・判断した部分・修正した部分を具体的に追記してください -->

## クレジット

- [Three.js](https://threejs.org/)（MIT License）
- フォント：[Orbitron](https://fonts.google.com/specimen/Orbitron)、[Noto Sans JP](https://fonts.google.com/noto/specimen/Noto+Sans+JP)、[JetBrains Mono](https://fonts.google.com/specimen/JetBrains+Mono)（SIL Open Font License）
- 3Dシンプレックスノイズ：Ashima Arts / Stefan Gustavson によるGLSL実装（MIT License）
- デモ曲：本プログラム内で合成したオリジナル

## ライセンス

[MIT License](LICENSE)

# SP40 キーボード: カスタマイズおよび書き込みガイド

本ドキュメントでは、SP40キーボード（ZMK Firmware）におけるキーマップのカスタマイズ（レイヤー追加などの方法）と、ファームウェアのビルドからXIAO nRF52840へ実際に書き込む（焼く）までの詳細な手順を解説します。

---

## 1. キーマップのカスタマイズとレイヤーの追加

SP40のキー配列や動作は、`config/sp40.keymap` ファイルを編集することで自由に変更できます。

### 1.1. レイヤー構成の基本
ZMKでは複数のレイヤー（層）を定義でき、基本となる「デフォルトレイヤー（0番目）」から、必要に応じて「記号レイヤー」や「数字レイヤー」へ切り替えて入力します。

`config/sp40.keymap` 内の `default_layer` などのブロックが、各レイヤーを定義しています。

### 1.2. レイヤーの追加方法
新しいレイヤーを追加するには、キーマップの先頭にレイヤーの見出し（マクロ）を定義し、新しい `layer_name` ブロックを追加します。親指の機能などに合わせて自由に定義してください。

```dts
#include <behaviors.dtsi>
#include <dt-bindings/zmk/keys.h>
#include <dt-bindings/zmk/bt.h>

// レイヤー番号を分かりやすい名前に定義しておく
#define DEFAULT 0
#define NUM_SYM 1
#define NAV_BT  2

/ {
    keymap {
        compatible = "zmk,keymap";

        // --- レイヤー 0: デフォルト ---
        default_layer {
            // 親指部分にレイヤー切り替え（&mo）や長押しレイヤー（&lt）を配置します
            bindings = <
                &kp Q      &kp W      &kp E      ... // ここは上3段のキー設定
                &kp A      &kp S      &kp D      ... 
                &kp Z      &kp X      &kp C      ... 
                &kp LCTRL  &kp LALT   &kp LGUI   &mo NUM_SYM  &kp SPACE    &kp RET  &mo NAV_BT ... 
            >;
        };

        // --- レイヤー 1: 数字・記号レイヤー ---
        num_sym_layer {
            // "trans" は下のレイヤー（デフォルト）のキー本来の動作を維持・透過させる設定です
            bindings = <
                &kp N1     &kp N2     &kp N3     &kp N4     &kp N5         &kp N6     &kp N7     &kp N8     &kp N9     &kp N0
                &kp EXCL   &kp AT     &kp HASH   &kp DLLR   &kp PRCNT      &kp CARET  &kp AMPS   &kp STAR   &kp LPAR   &kp RPAR
                &trans     &trans     &trans     &trans     &trans         &trans     &trans     &trans     &trans     &trans
                &trans     &trans     &trans     &trans     &trans         &trans     &trans     &trans     &trans     &trans
            >;
        };

        // --- レイヤー 2: ナビゲーション・Bluetooth制御 ---
        nav_bt_layer {
            // ここに矢印キーやBluetoothクリアなどを配置します
            bindings = <
                &bt BT_CLR &bt BT_SEL 0 &bt BT_SEL 1 &bt BT_SEL 2 &trans    &trans     &trans     &kp UP     &trans     &trans
                &trans     &trans       &trans       &trans       &trans    &trans     &kp LEFT   &kp DOWN   &kp RIGHT  &trans
                &trans     &trans       &trans       &trans       &trans    &trans     &trans     &trans     &trans     &trans
                &trans     &trans       &trans       &trans       &trans    &trans     &trans     &trans     &trans     &trans
            >;
        };
    };
};
```

### 1.3. 頻繁に利用する動作（Behaviors）
ZMKには非常に強力な「Behavior（挙動）」があらかじめ豊富に用意されています。

- **`&kp` (Key Press):** 最も基本的な単一キーの入力用。 例: `&kp A`, `&kp ESC`
- **`&mo` (Momentary Layer):** 押している間だけ指定レイヤーに切り替わる。 例: `&mo NUM_SYM`
- **`&tg` (Toggle Layer):** 押すたびに指定レイヤーのON/OFFが切り替わる（Caps Lock状態のような切り替え）。 例: `&tg 1`
- **`&mt` (Mod-Tap):** 長押しで装飾キー（ShiftやCtrlなど）、短押しで文字入力。 例: `&mt LSHFT A`（長押しで左Shift、単押しで「a」）
- **`&lt` (Layer-Tap):** 長押しでレイヤー切り替え、短押しで文字・記号入力。親指キーやSpaceに最適です。 例: `&lt 1 SPACE`（長押しでレイヤー1、単押しでSpace）
- **`&bt` (Bluetooth):** ペアリング先の切り替えやクリア。 例: `&bt BT_CLR`, `&bt BT_SEL 0`


---

## 2. ファームウェアの自動ビルド（GitHub Actions）

ローカル環境に複雑なCコンパイラを構築しなくても、GitHubを利用すればクラウドで自動的に `.uf2`（ファームウェアファイル）を作成してくれます。

1. ご自身のGitHubアカウントで、本プロジェクトの構成ファイル一式（`build.yaml` や `configディレクトリ` など）を含むリポジトリをPushします（ZMK公式のテンプレートを利用することもできます）。
2. GitHubリポジトリの **[Actions]** タブを開き、ワークフローが有効になっていることを確認します。
3. コードをコミット（修正）するたびに、自動的に「Build」ワークフローが走り出します。
4. ビルドが完了（緑色のチェックマーク）したら、そのビルドのページを開きます。
5. ページの一番下にある **Artifacts** という項目に `firmware`（または `firmware.zip`）というファイルが現れるため、これをダウンロードして解凍します。
6. ZIPファイル内には `sp40_left-....uf2` と `sp40_right-....uf2` という2つのファームウェアが生成されています。

---

## 3. XIAO nRF52840 へのファームウェア書き込み手順

作成された `.uf2` ファイルを手に入れたら、左右の基板（XIAO nRF52840）にそれぞれ書き込みます（これを「フラッシュする / 焼く」と表現します）。

### 🚨 最も重要な前準備（分割キーボードのルール）
- **右（Right）のXIAOには、絶対に `sp40_right....uf2` を書き込んでください。**
- **左（Left）のXIAOには、絶対に `sp40_left....uf2` を書き込んでください。**

ZMKの分割構成では、左側が親玉の「セントラル（PCやスマホへ無線送信を行うハブ）」となり、右側が子機の「ペリフェラル（押されたキー情報を左に送るだけ）」として振る舞う設計です。反対に書き込むと動作しません。

### 書き込み手順

1. **XIAO をPC（Mac等）にUSB接続する**
   まずは書き込みたい片方のXIAO（例：左手側）をUSB Type-CケーブルでMacに接続します。

2. **ブートローダーモード（Bootloader Mode）へ入る**
   - XIAOのボード上（裏面のType-Cコネクタ付近など）にある「**極小のリセットボタン（RST / RESET）**」を探します。
   - ピンセット等を用いて、このリセットボタンを**「カチッ・カチッ」と素早く2回（ダブルクリック）**押します。
   - 成功すると、Mac側で XIAO が新しい「USBフラッシュメモリ（ドライブ名は `XIAO-SENSE` など）」としてマウント（認識）されます。

3. **.uf2 ファイルのドラッグ＆ドロップ**
   - マウンタ（USBドライブ）が表示されたら、左手用のXIAOであれば先ほどダウンロードした **`sp40_left...uf2` ファイルを、そのドライブの中にドラッグ＆ドロップしてコピー**します。
   - コピー作業が完了した瞬間にXIAOは自動的に再起動し、USBドライブが画面上から勝手にアンマウント（消滅）します。
   - これで左手側の書き込み・再起動は無事に完了です。

4. **反対側（右手側）も同様に行う**
   - 右手側のXIAOをUSB接続し、同じくリセットボタンをダブルクリックしてドライブを表示させます。
   - 今度は **`sp40_right...uf2`** ファイルをドラッグ＆ドロップします。

### 初回のペアリングと動作確認
- 左右の基板に書き込みが完了した後、両方の基板に電源を通します（バッテリー使用時、または両方をUSBで繋いだ状態）。
- 数秒待つと、左右の基板間で専用の無線通信接続（BLE）が自動的に確立します。
- その後、MacやスマートフォンのBluetooth設定画面を開き、デバイス名「**SP40 Left**」（`Kconfig.defconfig` で指定した名称）というキーボードを検索し、ペアリング機能で接続すれば、分割キーボードとして文字入力が可能になります！
- **※有線USB接続での利用**：片側だけUSBで繋いで使う場合は、セントラルである「左手側」とMacをUSB接続してください。右手側から左へ無線でキー情報が飛び、左がUSB経由でMacに出力します。

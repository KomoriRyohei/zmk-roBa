# PMW3610 Ball Action 機能ガイド

## 概要

Ball Action は、PMW3610 トラックボールドライバーの拡張機能で、トラックボールの動きを特定のキー入力やマクロに変換できます。特定のレイヤーでトラックボールを使って矢印キー操作、ページスクロール、ブラウザナビゲーションなどを実現できます。

**追加時期**: 2025年1月末（コミット 3196146）

**要件**:
- ZMK v0.3-branch 以降
- zmk-pmw3610-driver main ブランチ

## 基本的な仕組み

1. **動き検出**: トラックボールの X/Y 方向の動きを蓄積
2. **閾値判定**: 蓄積した動きが `tick` 値を超えたら発動
3. **方向判定**: 4方向（右、左、上、下）のうち、どの方向に動いたかを判定
4. **アクション実行**: 対応するキーバインディングを実行
5. **リセット**: 蓄積値をリセットして次の動きを待機

### 方向とバインディングの対応

```
bindings = <&kp RIGHT>, <&kp LEFT>, <&kp UP>, <&kp DOWN>;
            ↑           ↑           ↑         ↑
            0           1           2         3
            右          左          上        下
```

## デバイスツリー設定例

### 基本設定（矢印キー）

```dts
&trackball {
    status = "okay";
    compatible = "pixart,pmw3610";
    reg = <0>;
    spi-max-frequency = <2000000>;
    irq-gpios = <&gpio0 2 (GPIO_ACTIVE_LOW | GPIO_PULL_UP)>;

    // レイヤー3で矢印キー操作
    arrows {
        layers = <3>;
        bindings =
            <&kp RIGHT_ARROW>,  // 右方向の動き
            <&kp LEFT_ARROW>,   // 左方向の動き
            <&kp UP_ARROW>,     // 上方向の動き
            <&kp DOWN_ARROW>;   // 下方向の動き

        tick = <10>;  // 感度調整（小さいほど敏感）
    };
};
```

### 応用例1: ブラウザナビゲーション

```dts
&trackball {
    // ... 基本設定 ...

    // レイヤー5でブラウザ操作
    browser_nav {
        layers = <5>;
        bindings =
            <&kp C_AC_FORWARD>,     // 右: 進む
            <&kp C_AC_BACK>,        // 左: 戻る
            <&kp PG_UP>,            // 上: ページアップ
            <&kp PG_DN>;            // 下: ページダウン

        tick = <15>;
        wait-ms = <10>;  // キー押下間の待機時間
        tap-ms = <5>;    // キープレス時間
    };
};
```

### 応用例2: テキスト編集（Vim風）

```dts
&trackball {
    // ... 基本設定 ...

    // レイヤー4でVim風操作
    vim_nav {
        layers = <4>;
        bindings =
            <&kp L>,  // 右: カーソル右
            <&kp H>,  // 左: カーソル左
            <&kp K>,  // 上: カーソル上
            <&kp J>;  // 下: カーソル下

        tick = <8>;
    };
};
```

### 応用例3: 音量・明るさ調整

```dts
&trackball {
    // ... 基本設定 ...

    // レイヤー6でメディアコントロール
    media_control {
        layers = <6>;
        bindings =
            <&kp C_VOL_UP>,         // 右: 音量アップ
            <&kp C_VOL_DN>,         // 左: 音量ダウン
            <&kp C_BRI_UP>,         // 上: 明るさアップ
            <&kp C_BRI_DN>;         // 下: 明るさダウン

        tick = <20>;     // 大きめの値で誤動作防止
        wait-ms = <50>;  // 連続入力を緩やかに
    };
};
```

### 応用例4: マクロとの組み合わせ

```dts
/ {
    macros {
        // Ctrl+C マクロ
        copy: copy {
            compatible = "zmk,behavior-macro";
            #binding-cells = <0>;
            bindings = <&kp LC(C)>;
        };

        // Ctrl+V マクロ
        paste: paste {
            compatible = "zmk,behavior-macro";
            #binding-cells = <0>;
            bindings = <&kp LC(V)>;
        };

        // Ctrl+Z マクロ
        undo: undo {
            compatible = "zmk,behavior-macro";
            #binding-cells = <0>;
            bindings = <&kp LC(Z)>;
        };

        // Ctrl+Shift+Z マクロ
        redo: redo {
            compatible = "zmk,behavior-macro";
            #binding-cells = <0>;
            bindings = <&kp LC(LS(Z))>;
        };
    };
};

&trackball {
    // ... 基本設定 ...

    // レイヤー7で編集操作
    edit_ops {
        layers = <7>;
        bindings =
            <&paste>,  // 右: 貼り付け
            <&copy>,   // 左: コピー
            <&redo>,   // 上: やり直し
            <&undo>;   // 下: 元に戻す

        tick = <12>;
        tap-ms = <10>;
    };
};
```

## パラメータ詳細

### tick（必須）

トラックボールの動きを蓄積して、何ピクセル分の動きでアクションを発動するか。

```dts
tick = <10>;  // 10ピクセル相当の動きで発動
```

- **小さい値（5-10）**: 敏感、わずかな動きで発動
- **中程度（10-20）**: バランス型、標準的な操作感
- **大きい値（20-40）**: 鈍感、大きな動きが必要、誤動作防止

### wait-ms（オプション）

連続したキー入力の間隔（ミリ秒）。デフォルト: 0

```dts
wait-ms = <50>;  // 50ms待機してから次のキー入力
```

**用途**:
- メディアキー（音量調整など）で連続入力を緩やかにする
- システムが処理できる速度に合わせる

### tap-ms（オプション）

キーを押している時間（ミリ秒）。デフォルト: 0

```dts
tap-ms = <10>;  // 10msキーを押し続ける
```

**用途**:
- 一部のアプリケーションで確実に入力を認識させる
- マクロの実行タイミング調整

## 複数のBall Actionを組み合わせる

同じトラックボールに複数のBall Actionを定義して、レイヤーごとに異なる動作を設定できます：

```dts
&trackball {
    status = "okay";
    compatible = "pixart,pmw3610";
    reg = <0>;
    spi-max-frequency = <2000000>;
    irq-gpios = <&gpio0 2 (GPIO_ACTIVE_LOW | GPIO_PULL_UP)>;

    // 通常レイヤー（レイヤー0）は通常のマウス操作

    // レイヤー3: 矢印キー
    arrows {
        layers = <3>;
        bindings = <&kp RIGHT>, <&kp LEFT>, <&kp UP>, <&kp DOWN>;
        tick = <10>;
    };

    // レイヤー5: ページスクロール
    page_scroll {
        layers = <5>;
        bindings = <&kp PG_DN>, <&kp PG_UP>, <&kp HOME>, <&kp END>;
        tick = <15>;
        wait-ms = <20>;
    };

    // レイヤー7: Vim操作
    vim {
        layers = <7>;
        bindings = <&kp L>, <&kp H>, <&kp K>, <&kp J>;
        tick = <8>;
    };
};
```

## 実装のヒント

### 1. 感度調整

最初は `tick = <10>` から始めて、使用感に応じて調整：

- **反応が遅い**: tick値を小さくする（8, 6など）
- **誤動作が多い**: tick値を大きくする（15, 20など）

### 2. レイヤー設計

Ball Actionを活用するレイヤー構成の例：

```
レイヤー0: 通常のタイピング（マウスモード）
レイヤー1: ファンクションキー
レイヤー2: 数字・記号
レイヤー3: 矢印キー（Ball Action）
レイヤー4: マウスクリック
レイヤー5: スクロール・ページ操作（Ball Action）
レイヤー6: メディア・システム（Ball Action）
```

### 3. 通常のマウス操作との切り替え

Ball Actionを使うレイヤーと通常のマウスモードを明確に分ける：

```dts
&trackball {
    // ... 基本設定 ...

    scroll-layers = <4>;      // レイヤー4はスクロールモード
    automouse-layer = <8>;    // レイヤー8は自動マウスレイヤー

    // レイヤー3では矢印キー操作
    arrows {
        layers = <3>;
        bindings = <&kp RIGHT>, <&kp LEFT>, <&kp UP>, <&kp DOWN>;
        tick = <10>;
    };
};
```

### 4. Kconfig設定

Ball Action機能を使う場合、`.conf`ファイルに以下を追加：

```conf
CONFIG_PMW3610=y
CONFIG_PMW3610_BALL_ACTION_TICK=20  # デフォルトのtick値
```

## トラブルシューティング

### 反応しない

1. レイヤーが正しくアクティブか確認
2. `tick` 値が大きすぎないか確認
3. `bindings` の構文が正しいか確認（カンマ区切り、4つのバインディング）

### 誤動作が多い

1. `tick` 値を大きくする（15-25程度）
2. `wait-ms` を追加して連続入力を制御
3. レイヤーの切り替えタイミングを見直す

### 特定の方向だけ動かない

バインディングの順序を確認：
```
<RIGHT>, <LEFT>, <UP>, <DOWN>
```
この順序は固定です。

## 参考資料

- [zmk-pmw3610-driver GitHub](https://github.com/kumamuk-git/zmk-pmw3610-driver)
- [Ball Action実装PR](https://github.com/kumamuk-git/zmk-pmw3610-driver/pull/3)
- [ZMK公式ドキュメント](https://zmk.dev/)

## まとめ

Ball Action機能を使えば、トラックボールを単なるポインティングデバイスとしてではなく、多機能な入力装置として活用できます。レイヤーごとに異なる動作を設定することで、キーボードの機能性が大幅に向上します。

**推奨ワークフロー**:
1. まず矢印キーのような基本的な設定から始める
2. 感度（tick値）を調整して好みの操作感を見つける
3. よく使う操作をBall Actionに割り当てる
4. 複数のレイヤーで異なる動作を設定して効率化

**注意点**:
- Auto mouse layer機能とBall Actionは同時に使わない方が良い
- 通常のマウス操作が必要なレイヤーとBall Actionレイヤーは分ける
- 初期設定は控えめな感度から始める

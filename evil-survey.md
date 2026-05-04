# UXを著しく低下させて適当な回答をさせないPower Apps選手権

ギミックの説明を自然言語で受け取り、Canvas Apps MCP を使って悪いアンケートアプリを自動生成するスキル。

```
/evil-survey ボタンを押すたびに「はい」と「いいえ」の位置がランダムに入れ替わる
```

---

## 前提条件（実行前に確認）

1. Canvas Apps MCP（`canvas-authoring`）が `.mcp.json` に設定・接続済みであること
2. Power Apps Studio で**横長（ランドスケープ）**の空キャンバスアプリを作成済みであること
3. Studio で「共同編集を有効にする」を実行済みであること
4. MCP の作業フォルダが `.mcp.json` に正しく設定されていること

---

## 選手権レギュレーション v1.0

### 概要

「適当さに適当さで勝負してみる」

Power Apps で 7 問の yes/no アンケートを作り、**UX を意図的に悪化させるギミック**を仕込むことで  
ユーザーが雑にクリックできない状態を作り出す。その工夫と技術力を競う選手権。

### ベースライン仕様（全作品共通）

| 項目 | 内容 |
|------|------|
| プラットフォーム | Power Apps キャンバスアプリ |
| 画面方向 | 横長（ランドスケープ、1366×768） |
| 設問数 | 7問（固定） |
| 回答形式 | 各設問に「はい」「いいえ」の2択ボタン |
| データソース | なし（コレクション変数のみ） |
| 最終画面 | 全問回答後に「はい」「いいえ」の集計を表示 |

### 必須条件

- 回答自体は**最終的に可能であること**（詰みにしない）

---

## 実行手順

### Step 1: ギミック設計

`$ARGUMENTS` を読んで以下を決定する。設計結果はユーザーに提示して確認を取ること。

1. **ギミック名**（短い日本語。例：「シャッフルボタン」「縮小ボタン」）
2. **必要な変数一覧**：ギミック制御用の `_` プレフィックス変数と初期値
3. **BtnHai / BtnIie の OnSelect ロジック**：ギミックの挙動 + 回答確定の処理
4. **ボタンの X, Y, Width, Height 式**：固定値か変数を使った式か
5. **LblInstruction のテキスト**：ユーザーへのヒントメッセージ
6. **Timer 等の追加コントロール**：必要であれば

**回答確定時に必ず含める処理：**
```
Collect(gAnswers, {question: gQuestionIndex, answer: "はい"});  // or "いいえ"
If(gQuestionIndex >= 7, Navigate(Screen3, ScreenTransition.Fade), Set(gQuestionIndex, gQuestionIndex + 1));
// ← ここでギミック変数を初期値にリセットする
```

### Step 2: 作業フォルダの確認

カレントディレクトリに `evil-survey/` フォルダがあればそれを使う。なければ作成する。

```bash
mkdir -p evil-survey
```

### Step 3: App.pa.yaml を書き込む

以下のテンプレートを元に `evil-survey/App.pa.yaml` を書き込む。

`OnStart` の末尾にギミック変数の初期化（`Set(_変数名, 初期値);`）を追加すること。

```yaml
App:
  Properties:
    OnStart: |-
      =ClearCollect(
          gQuestions,
          {id: 1, text: "あなたは朝型人間ですか？"},
          {id: 2, text: "週に3回以上運動していますか？"},
          {id: 3, text: "料理をするのが好きですか？"},
          {id: 4, text: "新しいことを学ぶのが楽しいですか？"},
          {id: 5, text: "一人の時間が大切だと思いますか？"},
          {id: 6, text: "読書を習慣にしていますか？"},
          {id: 7, text: "チャレンジすることが好きですか？"}
      );
      Clear(gAnswers);
      Set(gQuestionIndex, 1);
      [GIMMICK_VARS_INIT]
    Theme: =PowerAppsTheme
```

`[GIMMICK_VARS_INIT]` をStep 1で設計した変数初期化コードに置き換える。

### Step 4: Screen1.pa.yaml を書き込む

以下のテンプレートを元に `evil-survey/Screen1.pa.yaml` を書き込む。

変更箇所：
- `LblSubtitle.Text`：ギミック名を反映した第N弾タイトルに変更
- `BtnStart.OnSelect`：ギミック変数のリセット行を追加

```yaml
Screens:
  Screen1:
    Properties:
      Fill: =RGBA(15, 23, 42, 1)
    Children:
      - LblTitle1:
          Control: Label
          Properties:
            Align: =Align.Center
            Color: =RGBA(250, 204, 21, 1)
            FontWeight: =FontWeight.Bold
            Height: =65
            Size: =42
            Text: ="UXを著しく低下させて"
            Width: =1366
            Y: =110
      - LblTitle2:
          Control: Label
          Properties:
            Align: =Align.Center
            Color: =RGBA(250, 204, 21, 1)
            FontWeight: =FontWeight.Bold
            Height: =65
            Size: =42
            Text: ="適当な回答をさせない"
            Width: =1366
            Y: =178
      - LblTitle3:
          Control: Label
          Properties:
            Align: =Align.Center
            Color: =RGBA(255, 255, 255, 1)
            FontWeight: =FontWeight.Bold
            Height: =90
            Size: =56
            Text: ="Power Apps 選手権"
            Width: =1366
            Y: =248
      - LblSubtitle:
          Control: Label
          Properties:
            Align: =Align.Center
            Color: =RGBA(148, 163, 184, 1)
            Height: =50
            Size: =26
            Text: ="[GIMMICK_TITLE]"
            Width: =1366
            Y: =345
      - LblDescription:
          Control: Label
          Properties:
            Align: =Align.Center
            Color: =RGBA(203, 213, 225, 1)
            Height: =50
            Size: =20
            Text: ="7つの質問に答えてください。ただし、ボタンが素直に言うことを聞くとは限りません。"
            Width: =1100
            X: =133
            Y: =405
      - BtnStart:
          Control: Button
          Properties:
            BasePaletteColor: =RGBA(250, 204, 21, 1)
            BorderRadiusBottomLeft: =12
            BorderRadiusBottomRight: =12
            BorderRadiusTopLeft: =12
            BorderRadiusTopRight: =12
            FontColor: =RGBA(15, 23, 42, 1)
            FontSize: =28
            FontWeight: =FontWeight.Bold
            Height: =80
            OnSelect: |-
              =Clear(gAnswers);
              Set(gQuestionIndex, 1);
              [GIMMICK_VARS_RESET];
              Navigate(Screen2, ScreenTransition.Fade)
            Text: ="スタート"
            Width: =300
            X: =533
            Y: =510
```

`[GIMMICK_TITLE]` と `[GIMMICK_VARS_RESET]` を実際の値に置き換える。

### Step 5: Screen2.pa.yaml を書き込む（ギミックの核心）

`evil-survey/Screen2.pa.yaml` を以下の構造で生成する。

**固定部分（変更しない）：**

```yaml
Screens:
  Screen2:
    Properties:
      Fill: =RGBA(15, 23, 42, 1)
    Children:
      - LblProgress:
          Control: Label
          Properties:
            Align: =Align.Right
            Color: =RGBA(148, 163, 184, 1)
            Size: =22
            Text: ="問 " & gQuestionIndex & " / 7"
            Width: =240
            X: =1096
            Y: =28
      - RectProgressBG:
          Control: Rectangle
          Properties:
            Fill: =RGBA(30, 41, 59, 1)
            Height: =10
            Width: =900
            X: =50
            Y: =35
      - RectProgressBar:
          Control: Rectangle
          Properties:
            Fill: =RGBA(250, 204, 21, 1)
            Height: =10
            Width: =If(gQuestionIndex > 1, 900 * (gQuestionIndex - 1) / 7, 0)
            X: =50
            Y: =35
      - LblQuestion:
          Control: Label
          Properties:
            Align: =Align.Center
            Color: =RGBA(255, 255, 255, 1)
            FontWeight: =FontWeight.Bold
            Height: =100
            Size: =38
            Text: =LookUp(gQuestions, id = gQuestionIndex).text
            Width: =1200
            X: =83
            Y: =190
```

**ギミックに合わせて変更する部分：**

```yaml
      - LblInstruction:
          Control: Label
          Properties:
            Align: =Align.Center
            Color: =[ギミック状態に応じたColor式]
            FontWeight: =[ギミック状態に応じたFontWeight式]
            Height: =45
            Size: =22
            Text: =[ギミックのヒントメッセージ式]
            Width: =1366
            Y: =330
      - BtnHai:
          Control: Button
          Properties:
            BasePaletteColor: =RGBA(34, 197, 94, 1)
            BorderRadiusBottomLeft: =12
            BorderRadiusBottomRight: =12
            BorderRadiusTopLeft: =12
            BorderRadiusTopRight: =12
            FontColor: =RGBA(255, 255, 255, 1)
            FontSize: =30
            FontWeight: =FontWeight.Bold
            Height: =[高さ式（固定なら =90）]
            OnSelect: |-
              =[ギミックロジック + 回答確定処理]
            Text: =[ボタンテキスト式]
            Width: =[幅式（固定なら =220）]
            X: =[X座標式]
            Y: =[Y座標式]
      - BtnIie:
          Control: Button
          Properties:
            BasePaletteColor: =RGBA(239, 68, 68, 1)
            BorderRadiusBottomLeft: =12
            BorderRadiusBottomRight: =12
            BorderRadiusTopLeft: =12
            BorderRadiusTopRight: =12
            FontColor: =RGBA(255, 255, 255, 1)
            FontSize: =30
            FontWeight: =FontWeight.Bold
            Height: =[高さ式（固定なら =90）]
            OnSelect: |-
              =[ギミックロジック + 回答確定処理]
            Text: =[ボタンテキスト式]
            Width: =[幅式（固定なら =220）]
            X: =[X座標式]
            Y: =[Y座標式]
```

Timer など追加コントロールが必要なギミックは `BtnIie` の後に追記する。

**座標の安全域：**
- X: `Min(Max(基準X + オフセット, 20), 1126)` でボタンが画面外に出ないようにする
- Y: `Min(Max(基準Y + オフセット, 100), 658)` で同様

### Step 6: Screen3.pa.yaml を書き込む

以下のテンプレートを元に `evil-survey/Screen3.pa.yaml` を書き込む。

変更箇所：`BtnRestart.OnSelect` にギミック変数のリセット行を追加する。

```yaml
Screens:
  Screen3:
    Properties:
      Fill: =RGBA(15, 23, 42, 1)
    Children:
      - LblComplete:
          Control: Label
          Properties:
            Align: =Align.Center
            Color: =RGBA(250, 204, 21, 1)
            FontWeight: =FontWeight.Bold
            Height: =100
            Size: =60
            Text: ="全問回答完了！"
            Width: =1366
            Y: =140
      - LblMessage:
          Control: Label
          Properties:
            Align: =Align.Center
            Color: =RGBA(203, 213, 225, 1)
            Height: =55
            Size: =28
            Text: ="お疲れ様でした。適当に押せましたか？"
            Width: =1366
            Y: =255
      - LblHaiCount:
          Control: Label
          Properties:
            Align: =Align.Center
            Color: =RGBA(34, 197, 94, 1)
            FontWeight: =FontWeight.Bold
            Height: =55
            Size: =28
            Text: ="「はい」と答えた回数：" & CountIf(gAnswers, answer = "はい") & " 回"
            Width: =700
            X: =333
            Y: =360
      - LblIieCount:
          Control: Label
          Properties:
            Align: =Align.Center
            Color: =RGBA(239, 68, 68, 1)
            FontWeight: =FontWeight.Bold
            Height: =55
            Size: =28
            Text: ="「いいえ」と答えた回数：" & CountIf(gAnswers, answer = "いいえ") & " 回"
            Width: =700
            X: =333
            Y: =430
      - BtnRestart:
          Control: Button
          Properties:
            BasePaletteColor: =RGBA(250, 204, 21, 1)
            BorderRadiusBottomLeft: =12
            BorderRadiusBottomRight: =12
            BorderRadiusTopLeft: =12
            BorderRadiusTopRight: =12
            FontColor: =RGBA(15, 23, 42, 1)
            FontSize: =24
            FontWeight: =FontWeight.Bold
            Height: =80
            OnSelect: |-
              =Clear(gAnswers);
              Set(gQuestionIndex, 1);
              [GIMMICK_VARS_RESET];
              Navigate(Screen1, ScreenTransition.Fade)
            Text: ="もう一度挑戦する"
            Width: =400
            X: =483
            Y: =560
```

### Step 7: コンパイルとエラー修正

`compile_canvas` を実行する。エラーがあれば該当箇所を修正して再実行。エラーが解消するまで繰り返す。

よくあるエラー：
- `=` プレフィックスの抜け（全プロパティ値の先頭に `=` が必要）
- セミコロンの位置（Power Fx は `;` 区切り）
- 変数名のタイポ

### Step 8: Studio に同期

`sync_canvas` を実行する。

完了したら以下をユーザーに伝える：
- Power Apps Studio でアプリを確認してください
- F5 またはプレビューボタンで動作確認してください
- 参加作品として X（旧Twitter）に投稿するときは **#低UxPowerApps選手権** タグをつけてください

---

## ギミックアイデア集（参考）

| ギミック名 | 概要 |
|----------|------|
| ムニュボタン | ボタンを押すと逃げる。もう一度押して確定（第1弾・実装済み） |
| シャッフルボタン | Timer で一定時間ごとに「はい」「いいえ」の X 座標が入れ替わる |
| 縮小ボタン | 回答するたびにボタンの Width / Height が小さくなる |
| 分身ボタン | ダミーボタンを複数表示し、本物だけ OnSelect に回答処理を入れる |
| 逆転ボタン | `Mod(gQuestionIndex, 2)` で「はい」「いいえ」の左右が毎問入れ替わる |
| 透明ボタン | Timer で経過時間に応じて Opacity が下がる |
| ダブルタップ必須 | 1回目クリックで確認フラグを立て、2回目で確定 |
| 揺れるボタン | Timer + Sin/Cos で X/Y をアニメーション（近似） |

---

*利用は自己責任でお願いします。MIT License。*

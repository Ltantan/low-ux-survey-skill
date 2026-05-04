# UXを著しく低下させて適当な回答をさせないPower Apps選手権 — Claude Code スキル

Power Apps で **意地悪なアンケートアプリ** を自動生成する Claude Code スキルです。  
ギミック（仕掛け）の説明を自然言語で入力するだけで、Canvas App の YAML を生成して Power Apps Studio に反映します。

> Skills を作ってみたかったので、やってみました。

例えばこんな使い方
```
/evil-survey ボタンを押すたびに「はい」と「いいえ」の位置がランダムに入れ替わる
```

---

## 選手権について

**「適当さに適当さで勝負してみる」**

Power Apps で 7 問の yes/no アンケートを作り、UX を意図的に悪化させる「ギミック」を仕込むことで  
ユーザーが雑にクリックできない状態を作り出す。その工夫と技術力を競う選手権です。

作品を Xに投稿するときは **#低UxPowerApps選手権** タグをつけてください。
（つけなくても良いですし、なんならアプリ開発しなくても大丈夫です。）

> 主催：[@kama_bizdev](https://x.com/kama_bizdev)（参加者随時募集中）

---

## セットアップ

### 必要なもの

| 必要なもの | 確認方法 |
|-----------|---------|
| Claude Code（CLI） | `claude --version` |
| Canvas Apps MCP（`canvas-authoring`） | `.mcp.json` に設定済みであること |
| Power Apps 環境 | make.powerapps.com にアクセスできること |

### スキルのインストール

> ⚠️ スキルファイルは配置前に内容をご確認ください

 `evil-survey.md` を `~/.claude/commands/` にコピーしてください。

### Power Apps アプリの準備

スキルを実行する前に以下を完了させてください。

1. [make.powerapps.com](https://make.powerapps.com) で**空のキャンバスアプリを新規作成**（横長・ランドスケープ）
2. Studio のメニューから「**共同編集を有効にする**」をクリック
3. URL から環境 ID とアプリ ID を取得して `.mcp.json` に設定

```json
{
  "mcpServers": {
    "canvas-authoring": {
      "command": "...",
      "args": ["--environment-id", "YOUR_ENV_ID", "--app-id", "YOUR_APP_ID"]
    }
  }
}
```

4. Claude Code を起動（または再起動）して MCP 接続を確認

---

## 使い方

```bash
# Claude Code を起動
claude

# ギミックを自然言語で説明して実行
/evil-survey ボタンを押すたびに縮んでいく

/evil-survey 奇数問目と偶数問目で「はい」「いいえ」の位置が左右逆になる

/evil-survey 一定時間が経つとボタンが透明になる
```

スキルが自動で：
1. ギミックを設計・提案
2. 4つの YAML ファイルを生成（App / Screen1 / Screen2 / Screen3）
3. `compile_canvas` でエラーチェック・修正
4. `sync_canvas` で Power Apps Studio に反映

---

## レギュレーション v1.0

全作品が守るべき共通仕様：

| 項目 | 内容 |
|------|------|
| プラットフォーム | Power Apps キャンバスアプリ |
| 画面方向 | 横長（ランドスケープ） |
| 設問数 | 7問（固定） |
| 回答形式 | 各設問に「はい」「いいえ」の2択ボタン |
| データソース | なし（コレクション変数のみ） |
| 必須条件 | 回答が**最終的に可能**であること（詰みにしない） |

---

## 歴代作品

| 弾 | ギミック | 作者 |
|----|---------|------|
| 第1弾 | ムニュボタン（ボタンが逃げる） | [@kama_bizdev](https://x.com/kama_bizdev/status/2050953084881489976?s=20) |
| 第2弾 | ボタンが増えるやつ | [@kama_bizdev](https://x.com/kama_bizdev/status/2050956522646368429?s=20) |

---

## ライセンス

MIT License — 利用は自己責任でお願いします。

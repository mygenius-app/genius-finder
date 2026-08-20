# GENIUS FINDER — Claude Code 引き継ぎ資料

## プロジェクト概要

**プロダクト名**: GENIUS FINDER  
**開発会社**: 株式会社すみなす  
**目的**: TIB PITCHイベント向けプロトタイプ。アート作品を選んで対話することで「MY GENIUS CARD（自分らしさの仮説）」を生成するアプリ。  
**テスト目標**: 2026年9月1日までに動作確認できる状態にする。20〜30名のテスト対象、70%以上の自己発見実感を目指す。

---

## 現在の状態

### 完成ファイル
- **`genius_finder.html`**（単一ファイルHTML、約650KB）
  - ジーニアスの作品9点をbase64で埋め込み済み
  - Anthropic APIキーをユーザーが入力して動作する
  - サーバー不要、ブラウザで直接開ける

### 次のやること
1. **GitHubリポジトリにアップロードしてGitHub Pagesで公開**
   - GitHubユーザー名: `kansho52-boop`
   - リポジトリ名: `genius-finder`（まだ作成していない可能性あり）
   - ファイルを `index.html` としてアップ
   - Settings → Pages → Deploy from main branch で公開
   - 公開URL: `https://kansho52-boop.github.io/genius-finder/`

2. **スマホで動作確認**

---

## アプリの構成

### 画面フロー（8画面）
```
0. APIキー入力
1. スタート（ニックネーム入力）
2. アート選択（9作品から3つ選ぶ）
3. リフレクション（選んだ作品について質問に答える）
4. テーマ選択
5. 創作（自分の作品をアップロード）
6. 仮説生成（AIがMY GENIUS仮説を提示）
7. MY GENIUS CARD（結果カード表示）
```

### 主要な技術仕様
- **単一ファイルHTML** — CSS・JS・画像すべてインライン
- **Anthropic API** — ブラウザから直接呼び出し（`anthropic-dangerous-direct-browser-access: true` ヘッダー必須）
- **モデル**: `claude-sonnet-4-5`
- **画像**: 9作品をbase64 JPEGとして埋め込み（CORS問題を回避）

### APIコール関数
```javascript
async function callClaude(prompt, useSystem = false, systemPrompt = '', content = null) {
  const body = {
    model: 'claude-sonnet-4-5',
    max_tokens: 1024,
    messages: [{ role: 'user', content: content || prompt }]
  };
  if (useSystem && systemPrompt) { body.system = systemPrompt; }
  const res = await fetch('https://api.anthropic.com/v1/messages', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'x-api-key': state.apiKey,
      'anthropic-version': '2023-06-01',
      'anthropic-dangerous-direct-browser-access': 'true'
    },
    body: JSON.stringify(body)
  });
  if (!res.ok) { const err = await res.json(); throw new Error(err.error?.message || 'API error'); }
  const data = await res.json();
  return data.content[0].text;
}
```

### 状態管理
```javascript
const state = {
  apiKey: '',
  nickname: '',
  selectedArts: [],    // 選択した作品IDの配列（3つ）
  reflectionData: {},  // Q&Aの回答
  selectedTheme: '',
  uploadedImage: null, // base64データURL
  hypothesis: null,    // AIが生成した仮説オブジェクト
  reaction: '',        // 'agree'|'pending'|'disagree'
  reactionNote: ''
};
```

### AIが生成する仮説のJSON構造
```json
{
  "geniusName": "あなたらしさを表す一言（15文字以内）",
  "hypothesis": "2〜3文で仮説を説明",
  "wants": "内側から求めていること（1〜2文）",
  "being": "自然でいられる状態・条件（1〜2文）",
  "geniusMoves": ["動詞フレーズ1", "動詞フレーズ2", "動詞フレーズ3"],
  "quest": "今日から試せる小さな行動（1文）",
  "evidence": ["根拠1", "根拠2", "根拠3"]
}
```

---

## GITHUBアップロード手順（Claude Codeで実行）

```bash
# 1. リポジトリ作成
gh repo create kansho52-boop/genius-finder --public

# 2. フォルダ作成してファイルを配置
mkdir genius-finder && cd genius-finder
cp /path/to/genius_finder.html index.html

# 3. Git初期化してプッシュ
git init
git add index.html
git commit -m "GENIUS FINDER initial commit"
git remote add origin https://github.com/kansho52-boop/genius-finder.git
git push -u origin main

# 4. GitHub Pages を有効化
gh api repos/kansho52-boop/genius-finder/pages \
  --method POST \
  --field source[branch]=main \
  --field source[path]=/
```

公開まで1〜2分待つと以下のURLでアクセス可能：  
**https://kansho52-boop.github.io/genius-finder/**

---

## GENIUSメソッドの背景

4ステップのフレームワーク：  
`ひらく → 見出す → 磨く → つながる`

**MY GENIUS CARDの要素**:
- **GENIUS NAME**: あなたらしさを一言で
- **WANTS**: 内側から求めていること
- **BEING**: 自然でいられる状態
- **GENIUS MOVES**: 得意な動詞3つ
- **CURRENT QUEST**: 今日から試せる行動
- **ENERGY SOURCE**: エネルギーの源
- **EVIDENCE**: 仮説の根拠

**コーチのペルソナ（AIシステムプロンプト用）**:  
親しみやすい女性の先生。一人ひとりのユニークな資質を見出すのが得意。押しつけがましくなく、問いかけで導くスタイル。

---

## 既知の問題・注意点

- `anthropic-dangerous-direct-browser-access: true` ヘッダーがないとAPIコールがCORSエラーになる
- 画像はbase64埋め込みのため、作品を追加・変更する場合は再度base64変換が必要
- APIキーはブラウザに入力する形式のため、GitHub Pagesで公開してもキーは露出しない（ユーザーが自分のキーを入力する）
- 現時点では作品は9点（仕様では10点を推奨しているが動作上は問題なし）

---

## 連絡先
- **担当**: 雄一郎（株式会社すみなす）
- **メール**: kansho52@gmail.com
- **GitHub**: kansho52-boop

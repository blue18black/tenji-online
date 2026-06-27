# tenji-online

点字（日本語点字）を学ぶためのクイズアプリです。`index.html` 1枚だけの静的サイトで、ビルド不要です。

- **ひとりで練習**: 点字訓練・単語・文章・数字・読みの5モード。点字マスをタップして回答します。
- **みんなで対戦**: ルームを作って（パスワード任意）コードを共有し、同じ問題に複数人で挑戦。回答が早いほど高得点（500〜1000点、不正解・未回答は0点）。タイマーバー上に誰がいつ回答したかがリアルタイムで表示され、結果発表で正解/不正解の色に変わります。

オンライン対戦は Firebase Realtime Database + 匿名認証を使っています。

## デプロイ

`index.html` をそのまま GitHub Pages や Netlify に置くだけで動きます（ビルドステップなし）。

## Firebase の設定（オンライン対戦を使う場合に必要）

`index.html` の `<script type="module">` 内、`firebaseConfig` のところに自分の Firebase プロジェクトの設定を貼り付けます（既存の `tenji-game` プロジェクトをそのまま使う場合はこの手順は不要です）。

1. [Firebase Console](https://console.firebase.google.com/) でプロジェクトを作成（または既存のものを使う）。
2. **Authentication** → Sign-in method で **匿名（Anonymous）** を有効化。
3. **Realtime Database** を作成。
4. プロジェクト設定 → マイアプリ → ウェブアプリを追加し、表示される `firebaseConfig` を `index.html` に貼り付け。

### Realtime Database のセキュリティルール

Realtime Database → ルール タブに、以下を貼り付けて公開してください（**重要**: これを設定しないと初期状態のテストモードルール期限切れ等で書き込みができなくなります）。

```json
{
  "rules": {
    "rooms": {
      "$roomCode": {
        ".read": "auth != null",
        ".write": "auth != null && (!data.exists() || data.child('hostUid').val() === auth.uid)",

        "players": {
          "$uid": {
            ".write": "auth != null && auth.uid === $uid"
          }
        },
        "questionsProgress": {
          "$qIndex": {
            "answers": {
              "$uid": {
                ".write": "auth != null && auth.uid === $uid && root.child('rooms/'+$roomCode+'/currentQuestionIndex').val() === $qIndex",
                ".validate": "newData.child('uid').val() === auth.uid"
              }
            }
          }
        }
      }
    }
  }
}
```

このルールの考え方:

- ルーム本体（`status`/`currentQuestionIndex`/`questionStartedAt`/`questions` など）はホストの uid だけが書き込めます。
- 各プレイヤーは自分の `players/{uid}`（名前・スコア）と、自分の回答（`questionsProgress/{問題番号}/answers/{uid}`）だけを書き込めます。正解判定・スコア計算は各プレイヤーが自分のブラウザで行い、自分のデータとして書き込む方式なので、「ホストだけが採点する」ための特別な権限は不要です。
- ルームを読めるのは「ルームコードを知っている人」全員です。

### パスワードについて（重要な注意）

ルームのパスワードは、合言葉が漏れることへの保険程度のものです。Realtime Database のルール上「ルームコードを知っていれば誰でもルームの中身（パスワードのハッシュ含む）を読める」ため、暗号学的に強固な保護ではありません。実際の絞り込みは、ランダムな6桁のルームコードそのものが担っています。授業や友人同士などカジュアルな利用を想定してください。

## ファイル構成

- `index.html` — アプリ本体（HTML・CSS・JS すべて1ファイル）。

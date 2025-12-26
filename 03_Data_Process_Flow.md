# 🔄 3. 詳細処理フロー図 (Flowchart)

## 3.1 ゲームループ処理 (Main Loop)
`requestAnimationFrame` によって毎フレーム（約1/60秒ごと）実行されるメインループのフローチャートです。

```mermaid
flowchart TD
    Start([開始: 1フレーム処理]) --> Input[1. 入力・時間取得<br/>(DeltaTime, マウス/骨格座標)]
    Input --> Sync[2. 楽曲同期 TextAlive<br/>現在の再生位置を取得]

    Sync --> LyricCheck{歌詞データはある？}
    LyricCheck -- No --> MoveBubbles
    LyricCheck -- Yes --> Spawn[歌詞バブル生成<br/>(BubblePoolから取得)]
    Spawn --> MoveBubbles[バブル位置更新]

    MoveBubbles --> HitCheck{当たり判定}
    HitCheck -- "接触なし" --> Render
    HitCheck -- "接触あり" --> CalcScore[3. スコア計算<br/>ホールド進行度加算/コンボ更新]

    CalcScore --> Render[4. 描画レンダリング<br/>DOM / Three.js / Canvas]
    Render --> End([終了: 次のフレームへ])

    End -.-> Start
```
---

## 3.2 スコア送信フロー (Score Submission)
ゲーム終了後、スコアがデータベースに保存されるまでの判定フローチャートです。

```mermaid
flowchart TD
    GameEnd([ゲーム終了]) --> Token[1. 認証トークン取得<br/>GET /api/token]
    Token --> Captcha[2. Turnstile認証<br/>(Bot対策)]
    Captcha --> Post[3. データ送信<br/>POST /api/score]

    Post --> ServerVal{4. サーバー検証}
    
    subgraph ServerSide [Cloudflare Workers]
        ServerVal -- "頻度過多" --> Error429[エラー 429<br/>Too Many Requests]
        ServerVal -- "署名不正" --> Error403[エラー 403<br/>Forbidden]
        ServerVal -- "Bot判定" --> ErrorBot[エラー 403<br/>Bot Detected]
        
        ServerVal -- "検証OK" --> LogicCheck{スコア妥当性チェック}
        
        LogicCheck -- "異常値 (100万超)" --> Flag[不正フラグ付与<br/>is_suspicious=true]
        LogicCheck -- "正常値" --> Safe[正常ステータス]
    end

    Flag --> Save
    Safe --> Save
    
    Save[(5. DB保存<br/>Supabase)] --> Response[レスポンス返却]
    Response --> RankRef[ランキング更新]
    RankRef --> Finish([完了])
```
### 処理のポイント
1.  **ゲームループ**: 「入力→同期→生成→判定→描画」の順で処理が進みます。ループの最後で次のフレームを予約します。
2.  **スコア送信**: クライアントから送信されたデータは、サーバー側で**4段階のチェック**（レート制限、署名、Bot、異常値）を通過したものだけが正規の記録として扱われます。

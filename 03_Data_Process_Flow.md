# 🔄 3. 詳細処理フロー図

## 3.1 ゲームループ処理 (Main Loop)
`requestAnimationFrame` によって毎フレーム（約1/60秒ごと）実行される処理の流れです。

```mermaid
flowchart TD
    Start([START FRAME]) --> Input[1. センシング・入力取得<br/>経過時間/マウス/骨格座標]
    Input --> Sync[2. 楽曲同期 TextAlive<br/>再生位置取得]
    
    Sync --> CheckLyrics{歌詞データはある？}
    CheckLyrics -- No --> HitCheck
    CheckLyrics -- Yes --> Spawn[歌詞バブル生成 Spawn]
    Spawn --> HitCheck
    
    HitCheck{当たり判定 Hit Check} -->|手首・カーソルとの距離| ScoreCalc[3. スコア・コンボ計算<br/>ホールド進行度更新]
    
    ScoreCalc --> Render[4. 描画 Render<br/>DOM / Three.js]
    Render --> End([END FRAME])
    
    End -.->|Next Frame| Start
```
---

## 3.2 スコア送信フロー (Score Submission)
ゲーム終了後、クライアントからサーバーへスコアを送信し、検証を経て保存するまでのプロセスです。

```mermaid
flowchart TD
    Finish([GAME FINISHED]) --> Token[1. トークン取得<br/>GET /api/token]
    Token --> Captcha[2. CAPTCHA認証<br/>Turnstile]
    Captcha --> Submit[3. スコア送信<br/>POST /api/score]
    
    Submit --> Verify[4. サーバー側検証]
    
    subgraph ServerSide [Server Verification]
        Verify --> RateLimit{レート制限}
        RateLimit -- NG --> Err429[Error 429]
        RateLimit -- OK --> SignCheck{署名 HMAC 検証}
        
        SignCheck -- NG --> Err403[Error 403]
        SignCheck -- OK --> BotCheck{Turnstile検証}
        
        BotCheck -- NG --> Err403Bot[Error 403]
        BotCheck -- OK --> ScoreCheck{スコアは正常値？}
    end
    
    ScoreCheck -- Yes --> Save
    ScoreCheck -- "No (不正疑い)" --> Flag[不正フラグ付与<br/>is_suspicious=true]
    Flag --> Save
    
    Save[(5. DB保存<br/>Supabase)] --> RankUpdate[ランキング更新]
    RankUpdate --> End([END])
```
### 処理詳細補足
* **ゲームループ**: ユーザー入力（マウス、タッチ、骨格）と楽曲再生位置を毎フレーム同期させ、判定と描画を行います。
* **スコア送信**: 不正なスコア送信を防ぐため、サーバー側で「レート制限」「署名検証」「Bot認証」「スコア妥当性チェック」の4重の検証を行います。

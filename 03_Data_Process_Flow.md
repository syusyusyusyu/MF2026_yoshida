# 🔄 3. 詳細処理フロー図

## 3.1 ゲームループ処理 (Main Loop)

`requestAnimationFrame` によって毎フレーム実行される処理の流れです。

```mermaid
sequenceDiagram
    participant Loop as GameLoop
    participant GM as GameManager
    participant Player as TextAlivePlayer
    participant Lyrics as LyricsRenderer
    participant Pose as MediaPipe

    Loop->>GM: onUpdate(delta, elapsed)

    rect rgb(30,30,30)
    note over GM: 🎵 ① 再生位置同期
    GM->>Player: getTimer().position
    Player-->>GM: currentTime (ms)

    note over GM: 📝 ② 歌詞更新
    GM->>GM: updateLyrics(currentTime)
    alt 歌詞タイミング到来
        GM->>Lyrics: displayLyric(lyricData)
    end

    note over GM: 👆 ③ ホールド・入力判定
    GM->>GM: updateHoldStates(delta)

    note over GM: 📷 ④ ボディ検知 (Body Mode)
    opt Body Mode
        Pose-->>GM: onResults(landmarks)
        GM->>GM: handlePoseResults(landmarks)
        note right of GM: 手首座標とバブルの衝突判定を実行
    end
    end
```

---

## 3.2 スコア送信フロー (Score Submission)

クライアントからサーバーへの安全なスコア送信プロセスです。

1. **トークン取得**: クライアントは `GET /api/token` でHMAC署名付きトークンを取得。
2. **CAPTCHA認証**: Turnstileウィジェットを実行し、認証トークンを取得。
3. **データ送信**: スコア、コンボ、ランク等のデータを `POST /api/score` へ送信。
4. **サーバー検証**:
    - **Rate Limit**: IPごとの頻度制限チェック。
    - **Signature**: トークンの改ざん検証。
    - **Turnstile**: Cloudflare APIで人間であることを確認。
    - **Cheat Check**: 異常なハイスコア（>1,000,000）を検知。
5. **DB保存**: 全ての検証を通過した場合のみ、Supabaseへ保存。

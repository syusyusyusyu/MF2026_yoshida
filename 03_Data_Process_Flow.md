# 3. 詳細処理フロー図

## 3.1 ゲームループ処理 (Main Loop)
`requestAnimationFrame` によって毎フレーム実行される処理の流れです。

```text
+----------+                                +-----------+
| GameLoop |                                | TextAlive |
+-----+----+                                +-----+-----+
      |                                           |
      +--- onUpdate(delta) --> [GameManager]      |
                                     |            |
      +------------------------------+<--getPos---+
      | 1. 再生位置同期              |
      |                              |
      +------------------------------+---spawn---> [LyricsRenderer]
      | 2. 歌詞更新 (Spawn)          |                  |
      |                              |            [DOM Update]
      |                              |
      +------------------------------+<--input---- [MediaPipe/Mouse]
      | 3. ホールド・入力判定        |
      |    (Distance Check)          |
      |                              |
      +------------------------------+
      | 4. スコア・コンボ加算        |
      +------------------------------+
```

---

## 3.2 スコア送信フロー (Score Submission)
クライアントからサーバーへの安全なスコア送信プロセスです。

```text
[Client / Browser]                       [Server / Cloudflare Workers]
        |                                              |
        |---(1) トークン取得 (GET /api/token)--------->|
        |                                              |
        |<--(2) 署名トークン & Nonce返却---------------|
        |                                              |
    [Turnstile]                                        |
        |                                              |
        |---(3) CAPTCHA認証 -------------------------->|
        |                                              |
        |---(4) スコア送信 (POST /api/score)---------->|
        |       { score, rank, token... }              |
        |                                              |
        |                                      [Validation]
        |                                      + Rate Limit Check
        |                                      + HMAC Signature Verify
        |                                      + Turnstile Verify
        |                                      + Cheat Check
        |                                              |
        |                                       [Supabase DB]
        |                                              |
        |<--(5) 200 OK (保存完了)----------------------|
        |                                              |
```

### 補足説明
1. **トークン取得**: クライアントは `GET /api/token` でHMAC署名付きトークンを取得。
2. **CAPTCHA認証**: Turnstileウィジェットを実行し、認証トークンを取得。
3. **データ送信**: スコア、コンボ、ランク等のデータを `POST /api/score` へ送信。
4. **サーバー検証**: レート制限、署名検証、Botチェック、不正スコア検知を実行。
5. **DB保存**: 全ての検証を通過した場合のみ、Supabaseへ保存。

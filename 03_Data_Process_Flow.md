sequenceDiagram
    participant GL as GameLoop
    participant GM as GameManager
    participant TA as TextAlivePlayer
    participant LR as LyricsRenderer
    participant MP as MediaPipe

    GL->>GM: onUpdate(delta, elapsed)

    rect rgb(30, 30, 30)
        note over GM: 🎵 再生位置同期
        GM->>TA: getTimer().position
        TA-->>GM: currentTime (ms)

        note over GM: 📝 歌詞更新
        GM->>GM: updateLyrics(currentTime)
        alt 歌詞タイミング到来
            GM->>LR: displayLyric(lyricData)
        end

        note over GM: 👆 ホールド・入力判定
        GM->>GM: updateHoldStates(delta)

        note over GM: 📷 ボディ検知 (Body Mode)
        opt Body Mode
            MP-->>GM: onResults(landmarks)
            GM->>GM: handlePoseResults(landmarks)
            note right of GM: 手首座標とバブルの衝突判定を実行
        end
    end

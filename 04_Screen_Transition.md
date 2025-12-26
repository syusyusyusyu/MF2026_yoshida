# 📱 4. 画面・状態遷移図

## 4.1 ゲームライフサイクル

ユーザー操作とゲーム内部状態の遷移定義です。

```mermaid
stateDiagram-v2
    [*] --> Initializing : アクセス

    state Initializing {
        TextAliveInit : API接続
        MediaPipeInit : カメラ起動 (Bodyモード)
    }

    Initializing --> Ready : ロード完了
    
    state Ready {
        [*] --> CountDown : Bodyモード開始時
        CountDown --> Playing
        [*] --> Playing : その他モード
    }

    state Playing {
        Running : 楽曲再生中
        Paused : 一時停止
        Running --> Paused : ユーザー操作 / フォーカスロスト
        Paused --> Running : 再開
    }

    Playing --> Finished : 楽曲終了 (onFinish)
    
    state Finished {
        Results : リザルト表示
        Submitting : スコア送信中
        Reported : 送信完了
    }

    Finished --> Initializing : リプレイ
    Finished --> [*] : タイトルへ戻る
```

---

## 4.2 ルーティング (URL設計)

| パス | コンポーネント | 説明 |
| :--- | :--- | :--- |
| `/` | `IndexPage.tsx` | **タイトル画面**<br>モード選択、ランキング閲覧、ヘルプ表示が可能。 |
| `/game` | `GamePage.tsx` | **ゲーム画面**<br>クエリパラメータ `?mode=cursor|body|mobile` で挙動を変化させる。 |
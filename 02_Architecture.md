# 2. アーキテクチャ詳細

## 2.1 クラス図
`GameManager`を中心としたクラス構成と依存関係は以下の通り。

```mermaid
classDiagram
    note "Singleton"
    class GameManager {
        +songId: string
        +score: number
        +combo: number
        +player: TextAlivePlayer
        +initGame()
        +playMusic()
        +handleGameLoopUpdate(delta)
        +submitScore()
    }

    class GameLoop {
        -frameId: number
        +start()
        +stop()
    }

    class BodyDetectionManager {
        +isReady(): boolean
        +evaluateLandmarks(landmarks)
        -startCountdown()
    }

    class LyricsRenderer {
        +displayLyric(lyricData)
        +displayViewerLyric(text)
    }

    class BubblePool {
        -pool: HTMLElement[]
        +acquire(): HTMLElement
        +release(element)
    }

    class InputManager {
        +setupEvents()
        -handleMove(x, y)
    }

    GameManager "1" *-- "1" GameLoop : manages
    GameManager "1" *-- "1" BodyDetectionManager : uses
    GameManager "1" *-- "1" LyricsRenderer : delegates
    GameManager "1" *-- "1" InputManager : delegates
    LyricsRenderer "1" *-- "1" BubblePool : uses

## 2.2 ディレクトリ構造
src/
├── game/                # ゲームコアロジック
│   ├── GameManager.ts       # メインコントローラー
│   ├── GameLoop.ts          # ループ管理
│   ├── BubblePool.ts        # オブジェクトプール
│   ├── TimerManager.ts      # タイマー管理
│   ├── gameLoader.ts        # 楽曲設定
│   └── events.ts            # イベント定義
├── components/game/     # ゲームUIコンポーネント
│   ├── RankingModal.tsx     # ランキング画面
│   ├── RankingPanel.tsx     # リスト表示
│   └── ModeTabs.tsx         # モード切替
└── worker/              # バックエンド (Cloudflare)
    ├── routes/
    │   └── score.ts         # スコアAPI
    ├── middleware/
    │   └── session.ts       # セッション管理
    └── rateLimiter.ts       # レート制限 (Durable Object)

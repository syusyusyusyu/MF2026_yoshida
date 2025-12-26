# 📦 6. モジュール詳細仕様書

## 6.1 GameManager (`src/game/GameManager.ts`)
ゲーム全体の状態を管理するシングルトンクラス。

### 定数定義
- `DEFAULT_SONG_ID`: `'HmfsoBVch26BmLCm'`
- `TIMER_KEYS`: タイマー管理キー
  - `ComboReset`: コンボ切れ判定 (30秒)
  - `ResultCheck`: リザルト遷移監視

### 主要メソッド仕様
| メソッド名 | 引数 | 戻り値 | 処理概要 |
| :--- | :--- | :--- | :--- |
| `initGame` | - | void | 3Dステージ、歌詞データ、タイマーの初期化を行う。 |
| `playMusic` | - | Promise | TextAliveの再生を開始。失敗時はフォールバックタイマーを起動。 |
| `updateLyrics`| `pos: number` | void | 現在の再生位置(`pos`)に基づき、表示すべき歌詞を検索して表示。 |
| `handlePoseResults` | `landmarks` | void | AIから受け取った骨格座標と、画面上のバブル座標の距離計算を行う。 |

---

## 6.2 RateLimiter (`worker/rateLimiter.ts`)
Cloudflare Durable Objects を利用した分散レート制限クラス。

### エンドポイント
- `/limit?key={IP}&limit={N}&window={SEC}`
  - 指定IPのアクセス数をカウントし、ウィンドウ時間内の上限を超えたら `429` を返す。
- `/nonce?val={NONCE}`
  - リプレイ攻撃防止用。一度使用されたNonceを保存し、再使用時は `409` を返す。

---

## 6.3 RankingPanel (`src/components/game/RankingPanel.tsx`)
ランキング表示用Reactコンポーネント。

### キャッシュ戦略
- **スコープ**: グローバル変数 `rankingCache`
- **キー**: `songId` + `mode` + `period`
- **有効期限**: 30秒 (`CACHE_TTL`)
- **動作**: コンポーネントマウント時にキャッシュを確認し、有効ならAPIリクエストをスキップする。
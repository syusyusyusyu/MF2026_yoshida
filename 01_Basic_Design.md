# 1. 基本設計書

**プロジェクト名**: Cross Stage
**バージョン**: 1.0.0

## 1.1 システム概要
本システムは、TextAlive App APIによる楽曲解析情報と、MediaPipe Poseによる身体検知を用いたWebブラウザ上で動作するリズムゲームである。
専用デバイスを使用せず、PCおよびスマートフォンのカメラ入力のみで全身操作を行うことを主機能とする。

## 1.2 技術スタック

### Frontend
* **Framework**: React 18.2 / Vite 5.1
* **Language**: TypeScript 5.2
* **Visual/3D**: Three.js 0.161
* **Sensing**: MediaPipe Pose (WASM)
* **Audio/Analysis**: TextAlive App API 0.3

### Backend / Infrastructure
* **Runtime**: Cloudflare Workers (Honoフレームワーク)
* **Consistency**: Durable Objects (レート制限、整合性管理)
* **Database**: Supabase (PostgreSQL)
* **Security**: Cloudflare Turnstile

## 1.3 機能要件一覧

### ゲームプレイ機能
* **楽曲同期**: TextAliveよりビート、サビ情報を取得し、演出を同期させる。
* **歌詞表示**: 楽曲進行に合わせた歌詞バブルの生成とCSSアニメーション制御。
* **操作モード**:
    1.  **Cursor Mode**: マウス/タッチ操作によるプレイ。
    2.  **Mobile Mode**: スマートフォン向けのタップ＆ホールド操作。
    3.  **Body Mode**: Webカメラ映像からの骨格検知による操作。

### スコア・ランキング機能
* **スコア計算**: 判定ロジックに基づくスコア算出およびランク判定（SS〜D）。
* **スコア送信**: プレイ終了後のサーバー送信処理。
* **ランキング**: 全期間/週間/24時間別の集計と表示。

### セキュリティ機能
* **リクエスト検証**: HMAC SHA-256を用いたAPI署名検証。
* **レート制限**: 短時間での多重アクセス検知と遮断。
* **不正検知**: スコア論理値チェックによる不正フラグ管理。

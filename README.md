## ぬりみち
### 概要
- 位置情報を利用したリアルタイム陣地取り対戦ゲーム。MapboxとFirebaseを用いた2人対戦システム
### 機能
- 歩いた軌跡の取得と保存
- ルーム作成・参加(6桁のコード)
- グリッドの陣地化
- 勝敗判定
---
### 使用技術
| 分類       | 技術                      |
| -------------- | ------------------------------- |
| プラットフォーム       | Android                         |
| プログラミング言語       | Kotlin                          |
| ユーザインターフェース             | Jetpack Compose                 |
| 地図表示           | Mapbox                          |
| 位置情報       | Android Fused Location Provider |
| 認証 | Firebase Authentication         |
| データベース      | Cloud Firestore                 |
| リアルタイム通信    | Firestore Snapshot Listener     |


### 対戦ルール
- 制限時間は30分
- ゲーム範囲は600m x 600mの正方形であり各グリッドは30m x 30m
- 2人でルームを共有する
- スタートボタンを押したのち範囲内を自由に歩くことでグリッドを取得していく
- 各グリッド内に3~6秒ほど滞在することでグリッドを取得可能
- 相手のグリッドを取得することも可能

---
### データベース構造
- ユーザ
```
users/
└── {uid}
    ├── uid
    ├── createdAt
    └── currentRoom
```
- ルーム
```
rooms/
└── {roomCode}    #ルームのID
    ├── hostUid    #作成者のUID
    ├── guestUid    #参加者のUID
    ├── status    #waiting状態かmatched状態
    ├── createdAt    #ルーム作成日時
    ├── centerLat    #ゲーム範囲の中心点
    ├── centerLng    #ゲーム範囲の中心点
    ├── radius    #ゲーム範囲の半径
    ├── startTime    #ゲーム開始時間
    ├── endTime    #ゲーム終了時間
    │
    ├── routes/    #歩いた記録
    │   └── {uid}/    #歩いた人のUID
    │       └── walks/
    │           └── {walkId}    #散歩のID
    │               ├── uid    #歩いた人のUID
    │               ├── points    #GPS取得時の座標
    │               └── createdAt    #作成日時
    │
    └── grids/    #グリッド記録
        └── {uid}    #グリッド保持者のUID
            ├── uid    #グリッド保持者のUID
            ├── grids    #保持しているグリッド
            │   ├── row    #グリッドの座標情報
            │   └── col    #グリッドの座標情報
            └── updatedAt    #更新日時
```
--- 
### システムの構造図

```text
┌──────────────────────────────┐
│          Android App         │
│                              │
│  Jetpack Compose             │
│          │                   │
│          ▼                   │
│      Mapbox Map              │
│          │                   │
│          ▼                   │
│   GPS / Location Provider    │
└──────────────┬───────────────┘
               │
               │ Location
               ▼
        ┌──────────────┐
        │ Game Logic   │
        │              │
        │ Gridcheck    │
        │ Territory    │
        │ Route        │
        │ Game Timer   │
        └──────┬───────┘
               │
               ▼
       ┌────────────────┐
       │ Firebase       │
       │                │
       │ Authentication │
       │ Firestore      │
       └───────┬────────┘
               │
        Real-time Sync
               │
       ┌───────▼────────┐
       │ Opponent       │
       │                │
       │ Route          │
       │ Territory      │
       └────────────────┘
```
---

### ゲームの流れ
A(ホスト)
B(ゲスト)
1. Aがルーム作成ボタンを押す
2. Aの画面に表示された6桁のコードをBの画面上に入力する
3. A,Bともにスタートボタンを押し範囲内を歩き始める
4. ルーム作成から30分後ゲームが終了して勝敗結果が画面に表示される

---
### グリッド取得の流れ

```text
GPS位置を取得
      ↓
現在のグリッドを計算
      ↓
新しいグリッド？
   ┌──┴──┐
  Yes    No
   ↓      ↓
滞在時間を   滞在時間を確認
記録
          ↓
       3秒以上？
          ↓
       マスを取得
          ↓
     相手のマスなら奪取
```

---
### 開発環境
* Android Studio version：Quail 3 | 2026.1.3
* Kotlin version：2.2.10
* Android SDK version：
  * Compile SDK / Target SDK：37
  * Min SDK：26
* Gradle version：AGP 9.3.1 に対応するバージョン
* Jetpack Compose version：BOM 2026.02.01
* Mapbox SDK version：11.5.0
* Firebase SDK version：Firebase BOM 34.17.0


---
### 今後の改善
- 同じグリッドを何度も取得することでグリッドの色を濃くするなどの独自性の作成
- 制限時間表示
- ゲーム範囲の設定方法を指定型へ変更

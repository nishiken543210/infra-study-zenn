---
title: "自宅ホームラボの全体構成を整理する ── 物理サーバー1台でどこまでできるか"
emoji: "🏗️"
type: "tech"
topics: ["proxmox", "homelab", "cloudflare", "truenas", "インフラ"]
published: true
---

これまでの連載で Proxmox を入れた話、TrueNAS で iSCSI を構成した話、Cloudflare Worker を繋いだ話を書いてきました。

「で、全体として何を作っているの？」という疑問に、一度きちんと答えておきたいと思います。

---

## 何を作っているか

一言でいうと、**自宅サーバー上で動く Linux 学習プラットフォーム**です。

ブラウザさえあれば、受講者がどこからでも Linux やネットワークの実機演習をできる環境を作ろうとしています。AWS や VirtualBox ではなく、物理サーバーにこだわっている理由は[この連載の最初の記事](https://zenn.dev/infra_study_ze/articles/why-homelab-not-cloud)に書いたとおりです。

---

## 全体構成図

```
【受講者のブラウザ】
        │
        │ HTTPS
        ▼
【Cloudflare Pages】 infra-study.org
  テキスト / 図解 / AI相談
        │
        │ Cloudflare Tunnel
        ▼
【自宅 LAN】
┌───────────────────────────────────────────────────┐
│                                                   │
│  Raspberry Pi 3B+  ← Tunnel エージェント         │
│                                                   │
│  HPE ML30 Gen10（Proxmox VE）                     │
│  ┌───────────────────────────────────────────┐   │
│  │ TrueNAS SCALE                              │   │
│  │   └─ iSCSI Target / ZFS / 2.93TB          │   │
│  │                                            │   │
│  │ 受講者VM                                  │   │
│  │   └─ 受講者が操作する Ubuntu               │   │
│  │      iSCSI Initiator                       │   │
│  │                                            │   │
│  │ Guacamole                                  │   │
│  │   └─ ブラウザ ↔ SSH の中継               │   │
│  │                                            │   │
│  │ ルーターVM（FRRouting）                    │   │
│  │   └─ L3 演習用ルーター VM                 │   │
│  │                                            │   │
│  │ 踏み台VM                                    │   │
│  │   └─ 受講者踏み台 VM                     │   │
│  └───────────────────────────────────────────┘   │
│                                                   │
│  TL-SG108E（L2+ スイッチ / VLAN 設定中）          │
└───────────────────────────────────────────────────┘

【Cloudflare Workers】 AI 相談バックエンド
  POST {question, lesson} → {answer}
```

---

## 各レイヤーの役割

### Web 公開層（Cloudflare）

サイト本体は Cloudflare Pages でホストしています。静的コンテンツなので、`git push` すれば自動でデプロイされます。

AI 相談機能は別の Cloudflare Worker として動いています。各章の「AI に相談」ボタンを押すと、その Worker に質問が飛んで回答が返ってきます。「[git 履歴を辿ったら AI が生きていた](https://zenn.dev/infra_study_ze/articles/ai-worker-revival)」で書いた Worker です。

### 自宅 LAN への入口（Cloudflare Tunnel）

自宅サーバーに外からアクセスするために Cloudflare Tunnel を使っています。Raspberry Pi 3B+ 上でトンネルエージェントが常時稼働していて、ポートの穴あけは一切していません。

### 仮想化基盤（Proxmox）

HPE ML30 Gen10 上に Proxmox VE を入れて、複数の VM を動かしています。CPU は Xeon E-2100 シリーズ、RAM は 32GB。ストレージは SSD（OS 用）と HDD × 3 本の RAID ボリューム（TrueNAS 用）の 2 系統です。

### ストレージ（TrueNAS）

TrueNAS SCALE が、iSCSI で受講者 VM にブロックデバイスを提供しています。受講者が `fdisk` や `mkfs` を打てるのは、この iSCSI LUN があるからです。「仮想ディスクを物理ディスクのように扱う」体験を作りたかったので、[iSCSI が開通するまでの話](https://zenn.dev/infra_study_ze/articles/truenas-iscsi-struggle)で書いた構成はかなりこだわっています。

### 受講者の動線

受講者がブラウザで `infra-study.org` を開く → 章の内容を読む → 「演習を始める」ボタンを押す → Guacamole が SSH 端末をブラウザ上に開く → 受講者は `踏み台VM` に入り、そこから `ssh 受講者VM` でジャンプして演習できる、という流れです。

ターミナルソフト不要、SSH 設定不要。ブラウザだけで始められるようにしたかったので、Guacamole を選んでいます。

### ネットワーク演習用の隔離セグメント

L3 演習（ルーティング / OSPF）のために、仮想ブリッジを独立させています。VLAN 10 と VLAN 20 に論理分離する予定で、物理スイッチ（TL-SG108E）の VLAN 機能を使って分離します。「リアルなスイッチの操作」も演習に含まれます。ここはまだ配線中です。

---

## 物理サーバー 1 台でやっていることのまとめ

| 機能 | 担当 VM |
|------|--------|
| iSCSI ストレージ | TrueNAS SCALE |
| 受講者演習環境 | 受講者VM |
| ブラウザ端末 | Guacamole |
| 踏み台サーバー | 踏み台VM |
| L3 ルーター演習 | ルーターVM |

電気代節約のため、使わないときはサーバーごと落としています。「VM 全停止 = 異常」ではなく「VM 全停止 = 節電中」という運用です。

---

## 現在地と残り作業

コンテンツ（テキスト・図解）は Part 1（Linux 基礎）8章がほぼ完成している状態です。演習環境の接続（Guacamole 経由のシェル）と AI 相談は動いています。

残っているのは:

- **ネットワーク演習（物理ケーブルの接続）** — TL-SG108E とサーバーを繋いで VLAN を検証する
- **受講者 VM のテンプレート化** — 複数人が同時に演習できるよう VM をスケールアウトできる仕組みを作る
- **認証 / 予約管理** — 受講者ごとのセッション管理・進捗トラッキング

公開目標は 2026 年 7 月 12 日です。間に合わせます。

---

*この連載では、自宅ホームラボで Linux 学習プラットフォームを作りながら経験したことを書いています。*

---

## ご紹介

自分は、自宅に物理サーバーを置いて、インフラを学べる場所を作っています。

教材は登録なしで無料で読めます。実機のサーバーに接続する演習は構築中で、まだ全章には対応していません。

**全章インデックス:** [infra-study.org/curriculum](https://infra-study.org/curriculum)

**感想フォーム:** [感想フォーム（Tally）](https://tally.so/r/eqvX8l)

---

X（@taro3_01）で更新を告知しています。フォローすると次回の通知が届きます。

@[card](https://x.com/taro3_01)

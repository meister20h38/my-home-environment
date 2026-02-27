# 🚀 Home-Lab & Hybrid Remote Access Architecture
外出先や大学から、自宅の計算リソース（AI開発・競技プログラミング用マシン）へ安全かつシームレスにアクセスするためのプライベートネットワーク環境です。

## 🌐 Network Diagram
```mermaid
graph TD
    %% 外部ネットワーク
    subgraph Internet_Zone ["インターネット (Public Network)"]
        direction TB
        ExtTitle[" "]
        Internet((Internet))
        style Internet_Zone fill:#f5f5f5,stroke:#ccc,stroke-dasharray: 5 5
    end

    %% Tailscale Mesh Network
    subgraph Tailnet ["Tailscale Private Mesh Network"]
        direction TB

        %% ノートPCサーバー（出口）
        Server["<b>Gateway Server (Laptop)</b><br/>OS: Ubuntu Server<br/>CPU: Core i3 7th Gen<br/>Role: Exit Node / Jump Host / Nextcloud"]
        
        %% メインPC
        MainPC["<b>Main Workstation</b><br/>OS: Windows 11 Pro<br/>CPU: Ryzen 7 7800X3D<br/>GPU: Radeon RX 7800XT<br/>Role: Competitive Programming / Gaming"]
        
        %% AI PC
        AIPC["<b>AI / Dev Machine</b><br/>OS: Windows 11 Pro<br/>CPU: Ryzen 9 5950X<br/>GPU: RTX 3060 (12GB)<br/>Role: Local LLM / RAG Dev / Stable Diffusion"]

        %% 接続関係（Mesh）
        MainPC <--> Server
        AIPC <--> Server
        MainPC <-.-> AIPC
    end

    %% 外部との通信
    Internet <-->|Secure Tunnel| Server
    
    %% デザインのカスタマイズ
    style Server fill:#ff9100,stroke:#333,stroke-width:2px,color:#fff
    style Tailnet fill:#e3f2fd,stroke:#1565c0,stroke-width:2px
    style MainPC fill:#ffffff,stroke:#333
    style AIPC fill:#ffffff,stroke:#333
    style ExtTitle fill:none,stroke:none
```

## 📋 System Design & Features
### 1. セキュアなリモートアクセスと出口ノード (Exit Node)
- Tailscale (WireGuardベース) を採用することで、複雑なルーターのポート開放設定を排除し、大学の学内LANや公衆Wi-Fiなどの制限された環境下でも、自宅環境へのL3レベルのセキュアな接続を実現しています。
- Ubuntu Server (Laptop) を出口ノードとして設定。安全性の低いネットワークを利用する際、すべての通信を自宅の暗号化されたトンネル経由（固定回線）でルーティングし、セキュリティを担保しています。

### 2. コンピュートリソースの最適配置
- AI / Dev Machine: ローカルLLMの推論やRAG（Retrieval-Augmented Generation）の研究開発、Stable Diffusionを用いた画像生成に特化。RTX 3060 (12GB) の豊富なVRAMを活かした開発環境を構築しています。
- Main Workstation: Ryzen 7 7800X3Dの高いシングルスレッド性能を活用し、競技プログラミング（AtCoder/Paiza）のコード作成や日常的な開発業務を担当。
- Server (Sustainable Hardware): 旧世代のノートPCを再利用することで、消費電力を抑えつつ24時間稼働のファイルサーバー（Nextcloud）および踏み台サーバーとして運用しています。

### 3. 主な用途
- 大学からのリモート開発: 大学の講義中や研究室から、自宅のGPUリソースをSSH経由で利用。
- プライベートクラウド: Nextcloudによるデバイス間でのファイル同期およびバックアップ。
- 安全性確保: 出先からのインターネット利用時におけるプライバシー保護。


## 🔋 Remote Power Management (Wake-on-LAN)

電力効率を最適化するため、電力消費の大きいメインマシンおよびAI PCは常時起動せず、24時間稼働のGateway Serverから必要に応じてリモート起動する運用を行っています。

### 構成のポイント
- **省電力運用**: 常時稼働は低消費電力なノートPC（Server）のみに限定。
- **オンデマンド起動**: 外出先からTailscale経由でサーバーにSSH接続し、マジックパケットを送信してデスクトップPCを起動。

### 起動用スクリプト (wake_my_pc.sh)
```bash
wakeonlan AA:BB:CC:DD:EE:01
```

### 設定の注意点
- 各PCのBIOS/UEFIにて「Wake on Magic Packet」を有効化済み。
- Windowsのネットワークアダプター設定にて、高速スタートアップの無効化およびWoL設定を適用。

## 🛠 Service Management & Automation

サーバー上のサービス（Nextcloud 等）はすべて Docker コンテナとして管理しており、ディレクトリ構成の整理とメンテナンスの自動化を行っています。

### Directory Structure
```text
/datashare/
├── docker-compose.yml  # インフラ構成定義 (IaC)
└── update.sh           # メンテナンス・自動更新スクリプト
```

### 1. Dockerによるインフラ管理
docker-compose.yml を用いることで、環境の再現性とポータビリティを確保しています。
疎結合な設計: 各サービスをコンテナ化し、ホストOSを汚さない運用。
永続データの分離: /datashare 下にデータを集約し、バックアップを容易に。

### 2. メンテナンスの自動化 (update.sh)
サーバーの可用性を維持するため、以下の手順をワンコマンド（または cron）で実行できるスクリプトを自作・運用しています。

```Bash
docker compose pull      # 最新イメージの取得
docker compose up -d     # コンテナの再起動（ダウンタイムの最小化）
docker image prune -f    # 不要な古いイメージの削除
```

■ Data Pipeline & Integration
- Centralized Storage: 収集した論文データや学習資料をNextcloudに集約。
- AI Pipeline: AI PC上のRAGシステムが、Tailscale経由でNextcloudのWebDAV/共有フォルダにアクセスし、知識ベースを動的に更新する構成を構築中。
このように、属人性を排除した「メンテナンスの仕組み化」を意識しています。

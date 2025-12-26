# Host-Oriented Ansible for Raspberry Pi Homelab

Raspberry Piホームラボ環境を構築・管理するためのAnsibleプレイブック集です。
このプロジェクトは、「Host-oriented（ホスト指向）」な構成管理を実証するためのデモ・説明用リポジトリです。

> ⚠️ 注意 / Warning
> 本リポジトリは解説を目的としているため、APIトークンやパスワード等の機密情報が意図的に平文で記述されています。
> 実運用環境で使用する場合は、必ず [Ansible Vault](https://docs.ansible.com/ansible/latest/vault_guide/index.html) を使用して機密情報を暗号化してください。

## 📖 コンセプト: Host-Oriented Approach

一般的なAnsible構成では `group_vars` や `roles` への複雑な依存関係が発生しがちですが、本プロジェクトでは「各ホストが自身の構成を知っている」という設計思想を採用しています。

1. `site.yml` はシンプルに: 全ホストに対して共通のタスクを実行するのではなく、変数を読み込むだけの薄いラッパーとして機能します。
2. `host_vars` が主役: 各ホストのYAMLファイル（例: `host_vars/raspberrypi4.lan/main.yml`）内で `apply_roles` リストを定義し、そのホストに必要な役割（Role）を宣言します。
3. 汎用的なDockerロール: `docker_apps` ロールは、個別のアプリケーションごとにタスクを書くのではなく、定義されたリストに基づいて設定ファイルとコンテナを動的にデプロイします。

## ✨ 特徴

* Raspberry Pi 最適化:
* Docker実行に必要な `cgroup` 設定（`cmdline.txt`）の自動化
* Wi-Fi省電力設定の無効化
* Systemd Watchdogの設定


* Dockerアプリの宣言的管理:
* `docker-compose.yml` や設定ファイル（`.env`, `config.alloy` 等）をテンプレートとして配布
* Cloudflared, Dockge, Alloy などの導入例


* 可観測性（Observability）:
* Grafana Alloy を用いたメトリクス・ログの収集
* Node Exporter, Docker Metrics, Syslog (rsyslog) の統合


## 📂 ディレクトリ構造

```text
.
├── ansiblg.cfg
├── inventory.ini           # インベントリ定義
├── site.yml                # メインプレイブック（動的ロール読み込み）
├── host_vars/              # ★ この構成の核心
│   ├── raspberrypi4.lan/  # ホストごとの設定ディレクトリ
│   │   └── main.yml        # 適用するロールや変数を定義
│   └── ...
└── roles/
    ├── docker_apps/        # 汎用Dockerデプロイロール
    ├── raspberrypi_setup/  # RPiハードウェア設定
    ├── system_bootstrap/   # 基本パッケージ導入・ロケール設定
    └── rsyslog/            # ログ転送設定

```

## 🚀 使い方

### 1. 前提条件

* Ansible がインストールされたコントローラーマシン
* SSH接続可能な Raspberry Pi（Debian/Raspbian系）

### 2. セットアップ

必要なコレクションとロールをインストールします。

```bash
ansible-galaxy install -r requirements.yml

```

### 3. ホスト定義（`host_vars`）

`host_vars/<hostname>/main.yml` に、そのホストに適用したいロールと設定を記述します。

```yaml
# 例: host_vars/my-pi.lan/main.yml
---
apply_roles:
  - raspberrypi_setup       # 基本設定
  - name: geerlingguy.docker # Dockerのインストール
    become: true
  - docker_apps             # コンテナのデプロイ

# docker_apps でデプロイするアプリのリスト
docker_apps:
  - dockge
  - cloudflared

# Dockerの設定（必要に応じて上書き）
docker_daemon_options:
  log-driver: "journald"

```

### 4. 実行

```bash
ansible-playbook -i inventory.ini site.yml

```

## 🛠 主要ロールの解説

### `roles/docker_apps`

このロールは `docker_apps` 変数にリストされたアプリケーションをループ処理します。
アプリケーション名に対応するディレクトリ（`host_vars/<host>/<app_name>` または `templates/<app_name>`）から `docker-compose.yml` や設定ファイルを自動検出し、ターゲットホストへ配置・起動します。

### `roles/raspberrypi_setup`

Raspberry Pi で Docker を安定稼働させるための設定を行います。特に `/boot/firmware/cmdline.txt` を編集し、`cgroup_enable=memory cgroup_memory=1` を付与してコンテナのリソース制限を有効化します。

## 📝 License

MIT License

Copyright (c) 2025 Yoshio HANAWA

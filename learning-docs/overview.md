### 章1: GCP基盤構築と堅牢なGKEクラスタセットアップ

*   **目的**: 本格的なKubernetesワークロードをデプロイするためのGCP基盤とGKEクラスタを、セキュリティと運用性を考慮して構築します。
*   **学習内容**:
    *   GCPプロジェクトと課金の設定（複数プロジェクト運用を想定した設計）
    *   GCPネットワーク設計（Shared VPC、Private GKEクラスタ、ファイアウォールルール、Cloud NATによるセキュアな外部通信制御）
    *   GCP IAMとサービスアカウント管理（最小権限の原則に基づいたIAM設計、Workload Identity）
    *   GKEクラスタの構築（StandardモードでのNode Pool設計、Autoscaling設定、ネットワークポリシー）
    *   TerraformによるIaCの実践（GKEクラスタと関連GCPリソースのコード管理）
*   **成果物**: Terraformコードによって完全に定義され、プライベートなGKEクラスタがセキュアに稼働しているGCPプロジェクト。

---

### 章2: コンテナ化とArtifact Registryへのデプロイ

*   **目的**: Goで開発されたAPIをコンテナ化し、GCPのセキュアなコンテナレジストリであるArtifact Registryにデプロイします。
*   **学習内容**:
    *   Go APIのDockerfile作成（効率的なマルチステージビルド、軽量イメージとセキュリティベストプラクティス）
    *   コンテナイメージのビルドと管理（Cloud Buildを用いた自動ビルドパイプライン、Artifact Registryへのイメージプッシュとバージョン管理）
    *   環境変数とシークレット管理の準備（GCP Secret Managerを活用した機密情報の管理）
*   **成果物**: Artifact RegistryにプッシュされたGo APIのDockerイメージ。

---

### 章3: Kubernetesアプリケーションのデプロイと基礎

*   **目的**: Kubernetesのコア概念をGKE上で実践的に理解し、Go APIアプリケーションをデプロイします。
*   **学習内容**:
    *   Go APIのKubernetesマニフェスト作成（Deployment、Service、GKE IngressとGCP Load Balancerを用いた公開）
    *   Kubernetesにおける設定管理（ConfigMapを用いた設定情報の外部化、Secretの安全な管理）
    *   永続化ストレージの利用（PersistentVolume、PersistentVolumeClaim、Cloud Storage FUSE CSI Driver、Cloud SQL Auth Proxyを利用した安全な接続）
    *   自動スケーリング（Horizontal Pod Autoscaler (HPA) によるPodの自動スケーリング）
    *   ヘルスチェック（Liveness ProbeとReadiness Probeの設定）
    *   TerraformによるKubernetesリソースの管理（Terraform Kubernetes Providerを用いたコード管理）
*   **成果物**: GKEクラスタ上で稼働するGo APIアプリケーションと、Terraformで管理されたKubernetesマニフェスト。

---

### 章4: 高度なKubernetesデプロイと環境管理

*   **目的**: 複数環境におけるKubernetesアプリケーションの管理と、実践的なデプロイ手法を習得します。異なるGCP環境に安全かつ効率的にアプリケーションをデプロイする技術と、セキュリティ強化のためのアクセス制御を確立します。
*   **学習内容**:
    *   Kustomizeによる環境ごとのマニフェスト管理（base/とoverlays/の構造を用いた効率的な設定管理）
    *   Workload Identityを用いたセキュアなIAMアクセス制御（Pod単位でのGCPサービスアカウント割り当てと最小権限の原則）
    *   External Secrets Operatorによるシークレット管理（GCP Secret Managerの機密情報をKubernetes Secretに自動同期）
    *   GKE IngressとGoogle Managed SSL/TLS証明書（外部からのHTTPS通信の自動化と証明書管理）
*   **成果物**: KustomizeとExternal Secrets Operatorを活用し、異なるGCP環境に安全かつ効率的にデプロイ可能なKubernetesアプリケーション。Workload Identityにより安全にGCPリソースにアクセスできるGo API、およびGoogle Managed SSL/TLS証明書によりHTTPS通信が提供された状態。

---

### 章5: CI/CDパイプラインの構築と自動化

*   **目的**: GitOpsの原則に基づき、完全に自動化されたCI/CDパイプラインを構築し、効率的なデプロイを実現します。
*   **学習内容**:
    *   GitHub Actionsを用いたCI/CDパイプラインの構築（Go APIのビルド、ユニットテスト、インテグレーションテスト、Artifact Registryへのコンテナイメージプッシュ、Kubernetesマニフェストのlintと検証）
    *   Argo CDの導入とGitOpsの実践（Argo CDのGKEクラスタへのデプロイ、Gitリポジトリとの連携によるアプリケーション自動同期、Argo CD Image UpdaterによるCI/CDの自動化）
*   **成果物**: GitOpsワークフローに基づき、GitHub ActionsとArgo CDによって完全に自動化されたCI/CDパイプライン。

---

### 章6: 可観測性と運用

*   **目的**: 本番環境に耐えうるロギング、メトリクス、アラートシステムを構築し、運用知識を深めます。
*   **学習内容**:
    *   ログとメトリクスの統合監視（Fluent Bitによるコンテナログ収集、Prometheusによるメトリクス収集、Grafanaによるダッシュボード構築とアラート設定）
    *   GCP Cloud Monitoringとの連携（アラートポリシーと通知設定）
    *   K8sの運用とデバッグ（PodやNodeの障害時のデバッグ手法、イベントログの活用）
*   **成果物**: Cloud Logging、Prometheus、Grafanaを用いた統合監視システムが構築されたGKE環境、および運用に必要なダッシュボードとアラート設定。

---

### 章7: マイクロサービスとサービスメッシュ

*   **目的**: マイクロサービスアーキテクチャをGKE上で実現し、サービスメッシュの利点を理解・活用します。
*   **学習内容**:
    *   Go APIによるマイクロサービスの設計と実装（複数のGo APIサービスの連携と通信設計）
    *   Istioの導入と活用（GKEクラスタへのIstioデプロイ、Istio GatewayとVirtualServiceによるL7ルーティング、mTLSによるサービス間通信暗号化、サービストポロジー可視化とメトリクス収集）
    *   GKE IngressとIstioの連携パターン（GCP Load BalancerからIstio Ingress Gatewayへのルーティング構成）
*   **成果物**: Istioによって管理され、高度なトラフィックルーティング（カナリアリリース、A/Bテストなど）が可能なGo APIベースのマイクロサービス群。

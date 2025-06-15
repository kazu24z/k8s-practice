## GCP実践Kubernetes: マイクロサービス運用カリキュラム

このカリキュラムは、GCPを主軸とし、Kubernetesを中心とした、実務で通用する本格的な学習カリキュラム案である。インフラの基礎知識があり、GCP、Terraform、コンテナ、Kubernetes、GitHub Actionsの経験がある者を対象とする。特に、Terraformの基本的な使い方やCloud RunへのCI/CD構築経験があることを前提とし、より実践的で応用的な内容に焦点を当てている。

**カリキュラムの進め方の方針:**
*   **アウトプット主体** : 各章で具体的なインフラやアプリケーションの構築物を明確に設定する。
*   **ステップ・バイ・ステップ** : 基礎から応用へと段階的にスキルを習得できるよう構成する。
*   **実務的アプローチ** : 実際の開発現場で使われるテクニックや知識を盛り込む。
*   **Kubernetes中心** : GKEをプラットフォームとし、関連ツールを積極的に活用する。
*   **Go API** : マイクロサービス構成の章では、APIをGoで実装する。

---

### 章1: GCP基盤構築と堅牢なGKEクラスタセットアップ

**目的** : 本章の目的は、本格的なKubernetesワークロードをデプロイするためのGCP基盤とGKEクラスタを、セキュリティと運用性を考慮して構築することである。これは、実環境での堅牢なシステム運用を見据えた、基盤構築の要である。

**学習内容** : 以下の要素を体系的に習得し、実際のプロジェクトで応用可能なスキルを確立する。
*   **GCPプロジェクトと課金の設定** : 複数プロジェクトでの運用を想定した設計を理解し、設定する。
*   **GCPネットワーク設計** :
    *   Shared VPCの概念と実装。
    *   Private GKEクラスタの構築と、Control Planeへのプライベートアクセス設定。
    *   ファイアウォールルール とCloud NAT によるセキュアな外部通信制御。
*   **GCP IAMとサービスアカウント管理** :
    *   最小権限の原則に基づいたIAM設計。
    *   Workload Identity の概念とGKEクラスタでの設定。
*   **GKEクラスタの構築** :
    *   StandardモードでのGKEクラスタ構築 (Node Poolの設計、Autoscaling設定)。
    *   ネットワークポリシーによるPod間通信の制御。
*   **TerraformによるIaCの実践** : GKEクラスタと関連するGCPリソース（VPC、サブネット、ファイアウォールなど）をTerraformコードで管理する。

---

#### 1.1 GCPプロジェクトと課金の設定

**概要** : GCPにおけるリソース管理の最小単位がプロジェクトである。複数プロジェクト運用は、開発・ステージング・本番環境の分離や、組織内の部門ごとのリソース分離、コスト管理といった観点から必須となる。プロジェクトと課金アカウントの適切な設計は、将来的な運用負荷を軽減し、セキュリティを強化する基盤となる。

**具体例** :
`my-company-dev` (開発環境), `my-company-stg` (ステージング環境), `my-company-prod` (本番環境) のように、環境ごとにプロジェクトを分離するケースが一般的である。各プロジェクトは共通の課金アカウントに紐付けられつつ、各プロジェクト内のリソース利用状況を個別に把握できるように設定する。また、組織ポリシーを用いて、特定のリージョンやサービスの使用を制限することも考慮する。

**課題** :
あなたの会社が「Webサービスを開発しており、開発・ステージング・本番の3環境をGCPで構築する」というシナリオを想定せよ。
1.  **設計:** 各環境のGCPプロジェクト名とID、それらを紐付ける課金アカウントの構造を設計せよ。また、将来的に別のチームがこのGCP環境を利用する際のプロジェクト追加方針も考慮し、ドキュメントにまとめよ。
2.  **実装:** 設計に基づき、Terraformを用いて3つのGCPプロジェクトを作成し、それぞれを既存の課金アカウントに紐付けるコードを作成せよ。プロジェクトの作成後、Terraformによってステートファイルが管理されることを確認せよ。

---

#### 1.2 GCPネットワーク設計

**概要** : GCPにおけるネットワーク設計は、GKEクラスタのセキュリティと接続性を担保する上で極めて重要である。特に大規模な環境では、Shared VPCの適切な利用が求められ、Private GKEクラスタによるセキュリティ強化、そしてファイアウォールとCloud NATによる通信制御が必須となる。

##### 1.2.1 Shared VPCの概念と実装

**概要** : Shared VPCは、複数のプロジェクト（サービスプロジェクト）が共通のVPCネットワーク（ホストプロジェクト）を共有できる仕組みである。これにより、ネットワークの一元管理、IPアドレスの重複回避、異なるプロジェクト間のセキュアな通信が容易になる。従来のAWSにおけるVPC、Subnet、Route Table、Internet Gatewayに相当するGCPの概念として捉え、大規模環境でのネットワーク設計において基盤となる。

**具体例** :
`shared-vpc-host-project` というホストプロジェクトを作成し、その中に `prod-network` というVPCと `prod-subnet-a`、`prod-subnet-b` といったサブネットを定義する。そして、アプリケーションをデプロイする `my-service-prod` というサービスプロジェクトをこのホストプロジェクトにアタッチする。これにより、`my-service-prod` プロジェクト内のGKEクラスタが `prod-network` のサブネットを利用できるようになる。

**課題** :
上記シナリオに基づき、以下のタスクを実施せよ。
1.  **ホストプロジェクトの作成と有効化:** `shared-vpc-host-project` というGCPプロジェクトを作成し、Shared VPCホストプロジェクトとして有効化するTerraformコードを作成し、実行せよ。
2.  **VPCとサブネットの定義:** ホストプロジェクト内に`prod-network`というカスタムモードVPCを作成し、`prod-subnet-a` (例: 10.0.1.0/24)と`prod-subnet-b` (例: 10.0.2.0/24)という2つのサブネットを定義するTerraformコードを追加せよ。
3.  **サービスプロジェクトのアタッチ:** 前節で作成した`my-company-prod`プロジェクトを、`shared-vpc-host-project`にサービスプロジェクトとしてアタッチするTerraformコードを作成し、実行せよ。

---

##### 1.2.2 Private GKEクラスタの構築

**概要** : Private GKEクラスタは、KubernetesのControl Plane（マスター）へのアクセスをプライベートIPアドレスのみに制限することで、クラスタのセキュリティ境界を大幅に強化する。Control Planeがインターネットに公開されないため、外部からの不正アクセスリスクを低減できる。

**具体例** :
GKEクラスタ作成時に、`--enable-private-nodes`と`--enable-private-endpoint`オプション（Terraformでは`private_nodes`と`private_endpoint`）を設定する。これにより、GKEノードはプライベートIPアドレスのみを持ち、Control PlaneもプライベートIPアドレス経由でしかアクセスできなくなる。管理者はVPC内部から、またはCloud VPN/Interconnect経由でアクセスする。

**課題** :
前節で構築したShared VPC上に、Private GKEクラスタをデプロイする以下のタスクを実施せよ。
1.  **GKEクラスタの定義:** `my-company-prod`サービスプロジェクト内のShared VPCサブネット上に、StandardモードのPrivate GKEクラスタを構築するTerraformコードを作成せよ。Control Planeへのアクセスがプライベートエンドポイント経由のみになるように設定を含めよ。
2.  **アクセス経路の確認:** クラスタ作成後、外部から`kubectl`コマンドでクラスタにアクセスできないことを確認せよ。その後、Cloud ShellなどVPC内部からアクセス可能であることを確認せよ。

---

##### 1.2.3 ファイアウォールルールとCloud NAT

**概要** : ファイアウォールルールは、VPCネットワーク内のインスタンスへのインバウンド/アウトバウンドトラフィックを制御するための基本機能である。Cloud NATは、プライベートIPアドレスを持つインスタンスが、単一のグローバルIPアドレスを使ってインターネットへセキュアにアクセスするためのサービスであり、意図しない通信を防ぎ、セキュリティリスクを低減する。

**具体例** :
Private GKEクラスタのノードが外部のコンテナレジストリからイメージをプルする必要がある場合、GKEノードからのアウトバウンドトラフィックを許可するファイアウォールルールと、Cloud NATを設定する。例えば、`gke-egress-allow-all`といったファイアウォールルールで特定のポート（例: 443/TCP）へのアウトバウンドを許可し、Cloud NATゲートウェイを配置することで、ノードのプライベートIPがNAT変換されて外部に通信する。

**課題** :
構築中のGKEクラスタに対し、以下のセキュリティ要件を満たすよう設定せよ。
1.  **アウトバウンド通信の制御:** Private GKEクラスタのノードからインターネットへのアクセスを、**Cloud NAT経由でのみ** 許可するTerraformコードを作成せよ。具体的には、特定のGCPサービス（例: Google Container Registry, Artifact Registryなど）への通信は許可し、それ以外のインターネットへの直接接続をブロックするファイアウォールルールも考慮せよ。
2.  **インバウンド通信の制限:** GKEクラスタのノードへ、SSH (ポート22) およびHTTPS (ポート443) のみが、特定のソースIPアドレス範囲（例: あなたのオフィスIP）からのみ許可されるファイアウォールルールをTerraformで記述し、適用せよ。クラスタのヘルスチェックやモニタリングに必要なGCP内部からの通信は妨げないように考慮せよ。

---

#### 1.3 GCP IAMとサービスアカウント管理

**概要** : GCP IAM (Identity and Access Management) は、GCPリソースへのアクセスをきめ細かく制御するための仕組みである。セキュリティを確保するためには、「最小権限の原則」に基づいたIAM設計が不可欠である。特にKubernetes PodからGCPリソースへアクセスする際には、Workload Identityの適切な利用が推奨される。

##### 1.3.1 最小権限の原則に基づいたIAM設計

**概要** : 「最小権限の原則」とは、ユーザーやサービスアカウントに、その職務を遂行するために必要最小限の権限のみを付与するというセキュリティの基本原則である。これにより、誤操作や不正アクセスによる被害範囲を最小限に抑えることができる。過剰な権限付与は、潜在的なセキュリティホールとなる。

**具体例** :
GKEクラスタを管理するサービスアカウントには「Kubernetes Engine 管理者」ロールを付与するが、アプリケーションがGCP Secret Managerからシークレットを読み取るだけであれば、「Secret Manager のシークレット アクセサー」ロールのみを付与し、「Secret Manager の管理者」といったより広範なロールは付与しない。

**課題** :
あなたのGCPプロジェクトで、以下のタスクを実施せよ。
1.  **カスタムロールの定義:** GKEクラスタの特定のワークロードが、Google Cloud Storage (GCS) の特定のバケットに対してオブジェクトの読み取り (`storage.objects.get`) のみを許可するカスタムIAMロールをTerraformで定義せよ。
2.  **サービスアカウントの作成と権限付与:** そのカスタムロールを付与する専用のGCPサービスアカウントを作成し、特定のGCSバケットにそのサービスアカウントがアクセスできるようにIAMポリシーをTerraformで記述せよ。

---

##### 1.3.2 Workload Identityの概念とGKEクラスタでの設定

**概要** : Workload Identityは、KubernetesのサービスアカウントとGCPのサービスアカウントを紐付けることで、Kubernetes PodからGCPリソースへ安全かつ簡潔にアクセスできるようにする仕組みである。これにより、GCPサービスアカウントキーをKubernetes Secretとして管理するなどの非推奨な方法を避け、認証情報の漏洩リスクを低減できる。

**具体例** :
Kubernetesの`my-app-sa`というサービスアカウントに、`gcp-my-app-sa@<project-id>.iam.gserviceaccount.com`というGCPサービスアカウントを紐付ける。Podの定義では`serviceAccountName: my-app-sa`を指定するだけで、Pod内のアプリケーションは自動的にGCPサービスアカウントの権限でGCPリソースにアクセスできるようになる。

**課題** :
GKEクラスタとWorkload Identityを活用し、以下のタスクを実施せよ。
1.  **Workload Identityの有効化:** 構築済みのGKEクラスタでWorkload Identityが有効になっていることを確認するか、有効になっていない場合はTerraformで設定を変更せよ。
2.  **サービスアカウントの紐付け:**
    *   Kubernetes上に`my-api-sa`という名前のサービスアカウントを作成せよ。
    *   GCP上に`api-access-sa`という名前のGCPサービスアカウントを作成し、このサービスアカウントに「Secret Manager のシークレット アクセサー」ロールを付与せよ。
    *   `my-api-sa`と`api-access-sa`を紐付けるTerraformコードを作成し、実行せよ。
3.  **検証用Podのデプロイ:** `my-api-sa`サービスアカウントを利用するPodをデプロイし、そのPodの中から`gcloud secret access <your-secret-name>`コマンドが成功し、GCP Secret Managerにアクセスできることを確認せよ。

---

#### 1.4 GKEクラスタの構築

**概要** : GKEクラスタの構築は、アプリケーションの要件に応じた適切なリソース計画が重要である。Standardモードでのクラスタ構築では、Node Poolの設計、Autoscaling設定がパフォーマンスとコスト効率に直結する。また、Kubernetesネットワークポリシーは、Pod間の通信をきめ細かく制御し、マイクロサービス間のセキュリティを担保する上で不可欠である。

##### 1.4.1 StandardモードでのGKEクラスタ構築

**概要** : GKE Standardモードでは、ユーザーがノード、ノードプール、Podの配置などを詳細に制御できる。これにより、特定のワークロード要件（例: GPU利用、高性能CPU、スポットインスタンス）に合わせた柔軟な構成が可能である。Node Poolの設計（マシンタイプ、ノード数、ディスクタイプ、イメージタイプなど）とAutoscaling設定（CPU/メモリ使用率に基づいたノード数の自動調整）は、クラスタの効率的な運用に直結する。

**具体例** :
`n1-standard-4`マシンタイプを使用し、最小1ノード、最大5ノードまでオートスケーリングするノードプールを定義する。ノードの自動アップグレードも有効にして、セキュリティパッチの適用や機能改善を自動化する。

**課題** :
構築中のPrivate GKEクラスタに、以下の要件を満たすNode Poolを追加せよ。
1.  **Node Pool設計:** あなたのWebアプリケーションがCPUバウンドな処理を多く含むと仮定し、適切なマシンタイプ（例: `e2-standard-8`）、ディスクタイプ、初期ノード数を決定せよ。
2.  **Autoscaling設定:** ワークロードの変動に対応するため、CPU使用率が50%を超えた場合に自動的にノードが増加し、2ノードから最大10ノードまでスケールするようなオートスケーリング設定をTerraformで記述せよ。
3.  **ノードの配置戦略:** ゾーンまたはリージョン内の配置、ノードの自動修復、自動アップグレード設定も考慮し、Terraformコードに含めよ。

---

##### 1.4.2 ネットワークポリシーによるPod間通信の制御

**概要** : Kubernetesネットワークポリシーは、Pod間のトラフィックをIPアドレスやポート、ラベルに基づいて制御するための仕組みである。これにより、マイクロサービスアーキテクチャにおいて、サービスAはサービスBにのみ通信を許可し、サービスCには許可しないといった、きめ細やかなセキュリティ境界を定義し、マイクロサービス間のセキュリティを担保できる。

**具体例** :
`backend`ラベルを持つPodに対して、`frontend`ラベルを持つPodからのTCPポート8080への通信のみを許可するネットワークポリシーを定義する。他のPodや外部からのアクセスは拒否される。

**課題** :
GKEクラスタ上で、以下のマイクロサービス構成を想定し、ネットワークポリシーを適用せよ。
1.  **アプリケーションのデプロイ:** 以下の2つのシンプルなアプリケーションをGKEクラスタの同一Namespace（例: `my-app-ns`）にデプロイせよ。（仮のDeploymentとServiceマニフェストを作成せよ）
    *   `frontend-app`: `app: frontend`ラベルを持つPod
    *   `backend-api`: `app: backend`ラベルを持つPod (ポート8080でAPIを提供)
2.  **ネットワークポリシーの作成と適用:**
    *   `frontend-app`から`backend-api`のPodへの通信（ポート8080）のみを許可し、他のPod（例: `debug-pod`など、あなたが別途作成したPod）からの`backend-api`へのアクセスは拒否するKubernetesネットワークポリシーのマニフェストを作成せよ。
    *   作成したマニフェストを`kubectl apply`でGKEクラスタに適用せよ。
3.  **動作検証:**
    *   `frontend-app`のPodから`backend-api`のPodへアクセスが成功することを確認せよ。
    *   `debug-pod`など、ポリシーで許可されていない別のPodから`backend-api`のPodへアクセスを試み、失敗することを確認せよ。

---

#### 1.5 TerraformによるIaCの実践

**概要** : TerraformによるIaC (Infrastructure as Code) は、GCPインフラストラクチャをコードとして管理し、バージョン管理、再現性、自動化を可能にする。GKEクラスタとその関連リソース（VPC、サブネット、ファイアウォールなど）をコードで完全に定義することで、手動での設定ミスを排除し、環境構築のプロセスを標準化できる。

**具体例** :
`main.tf`にVPC、サブネット、GKEクラスタのリソース定義を記述し、`variables.tf`でリージョンやプロジェクトIDを変数化する。`terraform plan`で変更内容を確認し、`terraform apply`で実際にリソースをプロビジョニングする。

**課題** :
これまでの学習内容で設計・実装してきたすべてのGCPインフラ（GCPプロジェクト、Shared VPC、VPCネットワーク、サブネット、ファイアウォールルール、Cloud NAT、GKEクラスタ、Node Pool、IAMサービスアカウント、Workload Identity設定など）を、単一のTerraformプロジェクトとして統合し、以下のタスクを実施せよ。
1.  **Terraformプロジェクトの構成:** 適切なディレクトリ構造（例: `modules`、`environments`）とファイル分割（`main.tf`、`variables.tf`、`outputs.tf`など）を考慮し、Terraformコードを整理せよ。
2.  **完全なIaC:** これまでに手動で設定した項目も含め、すべてのGCPリソースがTerraformコードで定義されていることを確認せよ。
3.  **計画と適用:** Terraformコードの変更が意図通りであることを`terraform plan`で確認し、問題がなければ`terraform apply`を実行して、ゼロから全ての環境が構築できることを検証せよ。

---

**成果物** : Terraformコードによって完全に定義され、プライベートなGKEクラスタがセキュアに稼働しているGCPプロジェクトが完成する。これは、以降のアプリケーションデプロイと運用を行う上での強固な基盤となる。

---

### 章2: コンテナ化とArtifact Registryへのデプロイ

**目的** : 本章の目的は、Goで開発されたAPIをコンテナ化し、Google Cloud Platform（GCP）のセキュアなコンテナレジストリであるArtifact Registryにデプロイすることである。これは、マイクロサービスとしてアプリケーションを運用する上での最初の重要なステップであり、効率性、再現性、セキュリティを確保するための基盤となる。

**学習内容** : 以下の要素を体系的に習得し、実際の開発・運用に適用できるスキルを確立する。
*   **Go APIのDockerfile作成** : 効率的なマルチステージビルドの実践、軽量イメージ作成とセキュリティベストプラクティスを理解し、適用する。
*   **コンテナイメージのビルドと管理** : Cloud Build を用いた自動ビルドパイプラインの構築、Artifact Registry へのイメージプッシュとバージョン管理を学ぶ。
*   **環境変数とシークレット管理の準備** : GCP Secret Manager を活用した機密情報の管理方法を習得し、アプリケーションにセキュアに連携させる準備をする。

---

#### 2.1 Go APIのDockerfile作成

**概要** : コンテナ化におけるDockerfileの記述は、単にアプリケーションを動かすだけでなく、ビルド時間、イメージサイズ、そして最も重要なセキュリティに直結する。本節では、Go言語で開発されたAPIの特性を理解し、実運用に耐えうる効率的かつセキュアなDockerfileを作成する技術を習得する。特に、マルチステージビルドによるイメージサイズの削減、および最小限のコンポーネントのみを含む軽量イメージの採用は、デプロイ時間の短縮と攻撃対象領域の削減に不可欠である。

**具体例** : GoアプリケーションのDockerfileでは、一般的にマルチステージビルドを採用する。
1.  **ビルドステージ** : `golang:<version>-alpine` などのビルド環境が揃ったイメージを基盤とし、Goアプリケーションのコンパイルを行う。このステージでは、アプリケーションの実行に必要なバイナリと、必要に応じて静的ファイルなどが生成される。開発に必要なツールや中間ファイルはこのステージにのみ存在し、最終イメージには含まれない。
2.  **最終ステージ** : `alpine` や `scratch`、またはGoogleの提供する `gcr.io/distroless/static` のような最小限のベースイメージを使用する。このイメージには、前ステージでビルドされたGoバイナリのみをコピーする。これにより、不要なOSパッケージやシェルを含まず、攻撃対象領域を大幅に削減した軽量なイメージが作成される。例えば、Goの静的リンクを行うことで、`scratch` イメージ（OSレイヤーが全くない状態）に直接バイナリを配置することも可能である。セキュリティベストプラクティスとしては、常に最新の公式ベースイメージを使用し、root ユーザーではなく専用の非特権ユーザーでプロセスを実行する `USER` 命令の利用が挙げられる。

**課題** : あなたのGo APIアプリケーション（シンプルなHTTPサーバーを想定）に対して、以下の要件を満たすDockerfileを作成せよ。
1.  **マルチステージビルドの実践** :
    *   Goアプリケーションのビルドステージと、最終的な実行ステージを明確に分離せよ。
    *   ビルドステージでは、必要な依存関係のダウンロードとGoバイナリのコンパイルを行え。
    *   最終ステージでは、非常に軽量なベースイメージ（例: `gcr.io/distroless/static` または `alpine`）を選択し、ビルドされたGoバイナリのみをコピーせよ。
2.  **軽量イメージの追求** :
    *   最終イメージのサイズを最小限に抑えるよう工夫せよ。Goの静的リンクや、`CGO_ENABLED=0` の設定を検討せよ。
3.  **セキュリティベストプラクティス** :
    *   `USER` 命令を使用して、アプリケーションを非特権ユーザーとして実行するように設定せよ。
    *   機密情報（APIキーなど）をDockerfileにハードコードしないように注意せよ。これは後のセクションでGCP Secret Managerを使って管理する。
4.  **動作確認** :
    *   作成したDockerfileを使用してローカルでDockerイメージをビルドし、正しくアプリケーションが起動することを確認せよ。
    *   `docker images` コマンドで最終イメージのサイズを確認し、不要なファイルが含まれていないことを検証せよ。

---

#### 2.2 コンテナイメージのビルドと管理

**概要** : コンテナイメージのビルドプロセスは、手動で行うと再現性の問題やエラーの温床となる。本節では、Cloud Buildを活用してビルドプロセスを自動化し、CI/CDパイプラインの一部として統合する方法を習得する。また、ビルドされたイメージをGCPのプライベートコンテナレジストリであるArtifact Registryにセキュアにプッシュし、バージョン管理を行うことで、デプロイ可能な状態を維持する。これは、開発から本番までのイメージの一貫性と追跡可能性を保証するために不可欠である。

**具体例** : Cloud Buildは、`cloudbuild.yaml` というファイルに記述されたステップに基づいてビルドを実行する。一般的なステップとしては、Goモジュールのダウンロード、テストの実行、Goバイナリのコンパイル、Dockerイメージのビルド、そしてArtifact Registryへのプッシュが含まれる。

```yaml
# cloudbuild.yaml の例
steps:
  # Goモジュールダウンロード
  - name: 'gcr.io/cloud-builders/go'
    args: ['mod', 'download']
    env: ['GOPATH=.']
  # Goテスト実行
  - name: 'gcr.io/cloud-builders/go'
    args: ['test', './...']
    env: ['GOPATH=.']
  # Dockerイメージビルド
  - name: 'gcr.io/cloud-builders/docker'
    args: ['build', '-t', 'asia-northeast1-docker.pkg.dev/$PROJECT_ID/my-go-api-repo/go-api:$COMMIT_SHA', '.']
  # Artifact Registryへプッシュ
  - name: 'gcr.io/cloud-builders/docker'
    args: ['push', 'asia-northeast1-docker.pkg.dev/$PROJECT_ID/my-go-api-repo/go-api:$COMMIT_SHA']
images:
  - 'asia-northeast1-docker.pkg.dev/$PROJECT_ID/my-go-api-repo/go-api:$COMMIT_SHA'
```


Artifact Registryは、Dockerイメージだけでなく様々なアーティファクトを管理できる統合レジストリであり、GCPのIAMと連携してアクセス制御を細かく設定できる。イメージのバージョン管理には、Gitコミットハッシュやセマンティックバージョニングを活用する。

**課題** : 前節で作成したGo APIとDockerfileを基に、以下のタスクを実施せよ。
1.  **Artifact Registryリポジトリの作成** :
    *   GCPプロジェクト内に、Dockerイメージを格納するためのArtifact Registryリポジトリ（例: `my-go-api-repo`）をTerraformを用いて作成せよ。場所（リージョン）とモード（Docker）を適切に設定せよ。
2.  **Cloud Build設定ファイルの作成** :
    *   Go APIのソースコードリポジトリのルートに、`cloudbuild.yaml` ファイルを作成せよ。
    *   このファイルには、Goアプリケーションのビルド（テストを含む）と、Dockerfileを用いたDockerイメージのビルド、そしてArtifact Registryへのイメージプッシュを行うステップを定義せよ。
    *   イメージタグには、`$COMMIT_SHA` や `$BUILD_ID` といったCloud Buildの組み込み変数を利用し、バージョン管理が行われるようにせよ。
3.  **Cloud Buildトリガーの設定** :
    *   GitHub（または利用しているSaaS Gitリポジトリ）にプッシュされた際に自動的にCloud Buildが実行されるよう、Cloud Buildトリガーを設定せよ。
4.  **パイプラインの実行と確認** :
    *   ソースコードをGitリポジトリにプッシュし、Cloud Buildが自動的に実行され、イメージがArtifact Registryにプッシュされることを確認せよ。
    *   Artifact RegistryのUIから、プッシュされたイメージとそのタグ、およびメタデータが正しいことを検証せよ。

---

#### 2.3 環境変数とシークレット管理の準備

**概要** : アプリケーションの設定情報や機密情報（データベース接続文字列、APIキーなど）は、コードにハードコードせず、外部から安全に注入する必要がある。本節では、GCP Secret Managerを活用してこれらの機密情報を一元的に管理し、アプリケーションがセキュアにアクセスできる状態を準備する。これにより、セキュリティリスクを低減し、設定変更時のデプロイプロセスの簡素化を図る。

**具体例** : GCP Secret Managerは、機密情報をバージョン管理し、IAMと連携してアクセス制御を行うことができる。アプリケーションからは、GCP SDKを通じてSecret Managerからシークレットを取得する。例えば、アプリケーションが必要とするDB接続文字列を`projects/my-project/secrets/db-connection-string/versions/latest`としてSecret Managerに保存し、アプリケーションの起動時にこれを取得するように設計する。GCPリソースへのアクセスは、最小権限の原則に基づき、特定のSecretを読み取る権限のみを持つサービスアカウントを割り当てる。Workload Identityは、Kubernetes PodからGCP Secret Managerに安全にアクセスするための主要なメカニズムとなる。

**課題** : あなたのGo APIアプリケーションに、外部から設定を注入する準備として、以下のタスクを実施せよ。
1.  **GCP Secret Managerへのシークレット登録** :
    *   ダミーのAPIキー（例: `my-api-key`）とダミーのデータベース接続文字列（例: `db-connection-string`）をGCP Secret Managerに登録せよ。これらのシークレットには複数バージョンを持たせて、バージョニングの概念を理解せよ。
2.  **GCPサービスアカウントとIAM権限の付与** :
    *   このGo APIがSecret Managerからシークレットを読み取るための専用GCPサービスアカウント（例: `go-api-secret-reader-sa`）を作成せよ。
    *   このサービスアカウントに、`my-api-key` と `db-connection-string` という特定のシークレットの「Secret Manager のシークレット アクセサー」ロール（`secretmanager.secrets.accessors`）を付与せよ。他のシークレットやより広範な権限は与えないこと。これは最小権限の原則に基づいている。
    *   これらの設定はTerraformを用いてコードで管理せよ。
3.  **Go APIアプリケーションの修正（概念のみ）** :
    *   **コードの検討** : Go APIが起動時に環境変数またはGCP Secret Managerからこれらのシークレットを読み込むようなコードの修正箇所を検討し、どのようなSDKやライブラリを使用するか設計をドキュメント化せよ。（実際のコード実装は本章では不要だが、次の章で必要となる。）
    *   **Kubernetesサービスアカウントとの連携準備** : Go APIがKubernetes Podとしてデプロイされた際に、前章で学習したWorkload Identity を用いて、このGCPサービスアカウントの権限でSecret Managerにアクセスできるようにするための、KubernetesサービスアカウントとGCPサービスアカウントの紐付けの概念を確認せよ。

---

**成果物** : 本章の完了時には、以下の成果物が得られる。
*   効率的かつセキュアに記述されたGo APIのDockerfile。
*   Cloud Buildの自動ビルドパイプラインが設定され、Artifact Registryに最新バージョンのGo API Dockerイメージがプッシュされた状態。
*   GCP Secret Managerにアプリケーションの機密情報が安全に格納され、最小権限の原則に基づいたIAMアクセス制御が設定された状態。

---

### 章3: Kubernetesアプリケーションのデプロイと基礎

**目的** : 本章の目的は、GKE (Google Kubernetes Engine) 上でKubernetesのコア概念を実践的に理解し、前章でコンテナ化したGo APIアプリケーションを堅牢かつ効率的にデプロイすることである。これにより、マイクロサービスアーキテクチャの基盤となるアプリケーションデプロイのスキルを確立する。

**学習内容** : 以下の要素を体系的に習得する。
*   **Go APIのKubernetesマニフェスト作成** : Deployment によるアプリケーションのデプロイと管理、Service (ClusterIP, LoadBalancer) を用いたサービス公開、GKE Ingress とGCP Load Balancer (GCLB) を用いた外部からのアクセス設定 。
*   **Kubernetesにおける設定管理** : ConfigMap を用いた設定情報の外部化、Secret の安全な管理とアプリケーションでの利用。
*   **永続化ストレージの利用** : PersistentVolume とPersistentVolumeClaim の概念と、Cloud Storage FUSE CSI Driverを利用した永続化。Cloud SQL を利用する場合のAuth Proxyを活用した安全な接続。
*   **自動スケーリング** : Horizontal Pod Autoscaler (HPA) によるCPU/メモリに基づいたPodの自動スケーリング。
*   **ヘルスチェック** : Liveness Probe とReadiness Probe の設定とチューニング。
*   **TerraformによるKubernetesリソースの管理** : Terraform Kubernetes Providerを用いて、Kubernetesリソースもコードで管理する。

---

#### 3.1 Go APIのKubernetesマニフェスト作成

**概要** : Kubernetesにおけるアプリケーションのデプロイは、主にDeployment、Service、Ingressといったリソースを組み合わせて行われる。これらはアプリケーションのライフサイクル管理、内部・外部からのアクセス制御の要であり、実運用ではそれぞれの特性を理解し、適切に組み合わせることが不可欠である。GKEにおいては、Ingressを通じてGCP Load Balancer (GCLB) がプロビジョニングされ、堅牢なL7トラフィック管理を可能にする。

**具体例** : シンプルなGo APIアプリケーションを例に、これらのマニフェストの基本構造を示す。
1.  **Deployment** : Podの複製数（レプリカ）、更新戦略、コンテナイメージ、ポートなどを定義する。
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: go-api-deployment
      labels:
        app: go-api
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: go-api
      template:
        metadata:
          labels:
            app: go-api
        spec:
          containers:
          - name: go-api
            image: asia-northeast1-docker.pkg.dev/your-project-id/my-go-api-repo/go-api:latest # Artifact Registryのイメージパス
            ports:
            - containerPort: 8080
    ```
   
2.  **Service (ClusterIP)** : クラスタ内部からのアクセスを可能にするServiceである。
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: go-api-service-internal
    spec:
      selector:
        app: go-api
      ports:
        - protocol: TCP
          port: 80
          targetPort: 8080
      type: ClusterIP
    ```
   
3.  **Service (LoadBalancer)** : 外部からのアクセスをGCP Load Balancer経由で可能にするServiceである。これはGKE環境において限定的に利用し、通常はIngressを優先する。
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: go-api-service-lb
    spec:
      selector:
        app: go-api
      ports:
        - protocol: TCP
          port: 80
          targetPort: 8080
      type: LoadBalancer
    ```
   
4.  **Ingress (GKE Ingress)** : HTTP(S)トラフィックをルーティングするためのKubernetesリソースである。GKEでは、これを定義することでGCP Load Balancer（GCLB）が自動的にプロビジョニングされ、外部からのアクセスポイントとなる。
    ```yaml
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: go-api-ingress
      annotations:
        kubernetes.io/ingress.global-static-ip-name: "your-static-ip-name" # 事前にGCPで取得した静的IPの名前
        # ingress.gcp.kubernetes.io/pre-shared-cert: "your-managed-cert-name" # 必要に応じてGoogle Managed SSL/TLS証明書
    spec:
      defaultBackend:
        service:
          name: go-api-service-internal
          port:
            number: 80
      rules:
      - http:
          paths:
          - path: /*
            pathType: ImplementationSpecific
            backend:
              service:
                name: go-api-service-internal
                port:
                  number: 80
    ```
   
**注** : `go-api-service-internal` はClusterIPタイプのServiceであり、Ingressはこの内部Serviceをターゲットにする。

**課題** : 前章でArtifact RegistryにプッシュしたGo APIイメージをGKEクラスタにデプロイせよ。
1.  **Deploymentの作成** : Go APIをデプロイするためのDeploymentマニフェストを作成し、最低2つのレプリカを指定せよ。コンテナイメージには、前章で作成したArtifact Registryのイメージパスとタグを使用せよ。
2.  **Serviceの作成** :
    *   クラスタ内部からアクセス可能なClusterIPタイプのServiceを作成せよ。
    *   （任意）外部からアクセス可能なLoadBalancerタイプのServiceも作成し、グローバルIPアドレスが割り当てられることを確認せよ。ただし、このLoadBalancer Serviceは最終的なアクセス経路としては使用せず、Ingressを使用する前提とする。
3.  **GKE Ingressの設定** :
    *   事前にGCPで静的外部IPアドレス（リージョンではなくグローバル）をプロビジョニングせよ。
    *   Go API Service（ClusterIP）をバックエンドとするIngressマニフェストを作成し、プロビジョニングした静的IPアドレスを使用するように設定せよ。
    *   Ingressが正常にGCLBをプロビジョニングし、APIが外部からアクセス可能であることを検証せよ。
    *   **検証** : `curl`コマンドやブラウザからIngress経由でGo APIにアクセスし、正常なレスポンスが返ってくることを確認せよ。GCPコンソールでGCLBのステータスを確認し、バックエンドサービスが正常であることを確認せよ。

---

#### 3.2 Kubernetesにおける設定管理 (ConfigMap, Secret)

**概要** : アプリケーションの設定情報と機密情報をコードから分離することは、セキュリティ、移植性、運用容易性の観点から必須である。Kubernetesでは、非機密情報にはConfigMap、機密情報にはSecret を用いてこれらを管理する。SecretはBase64エンコードされるが、これは暗号化ではないため、適切なアクセス制御（IAMなど）が重要である。GCP Secret Managerとの連携は後の章で扱うが、ここではKubernetesネイティブなSecretの扱い方を習得する。

**具体例** : ConfigMapとSecretをPodの環境変数として注入する例。
1.  **ConfigMap** : 環境固有の設定、ログレベル、外部サービスのエンドポイントなど。
    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: go-api-config
    data:
      API_VERSION: "1.0"
      EXTERNAL_SERVICE_URL: "http://external.example.com"
      LOG_LEVEL: "INFO"
    ```
   
    DeploymentでConfigMapを環境変数として参照する方法:
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: go-api-deployment
    spec:
      template:
        spec:
          containers:
          - name: go-api
            image: asia-northeast1-docker.pkg.dev/your-project-id/my-go-api-repo/go-api:latest
            envFrom:
            - configMapRef:
                name: go-api-config
    ```
   
2.  **Secret** : データベース認証情報、APIキー、TLS証明書など。
    ```yaml
    apiVersion: v1
    kind: Secret
    metadata:
      name: go-api-secret
    type: Opaque
    data:
      DB_CONNECTION_STRING: "YmY5NGE1NjcxZTY1YzBhMDczMzE1MmFhZTRkODQzMjk=" # base64エンコードされた値
      API_KEY: "c2VjcmV0QVBJS2V5MjAyMw=="
    ```
   
    DeploymentでSecretを環境変数として参照する方法:
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: go-api-deployment
    spec:
      template:
        spec:
          containers:
          - name: go-api
            image: asia-northeast1-docker.pkg.dev/your-project-id/my-go-api-repo/go-api:latest
            envFrom:
            - secretRef:
                name: go-api-secret
    ```
   

**課題** : 前節でデプロイしたGo APIアプリケーションに、設定情報と機密情報を注入せよ。
1.  **ConfigMapの作成** : Go APIが利用する非機密の設定情報（例: APIのバージョン、外部サービスのエンドポイントURL、環境名など）を定義したConfigMapマニフェストを作成せよ。
2.  **Secretの作成** : Go APIが利用する機密情報（例: ダミーのDB接続文字列、APIキーなど）をBase64エンコードし、Secretマニフェストを作成せよ。
3.  **Deploymentの更新** : 作成したConfigMapとSecretを、Go APIのDeploymentのPodに環境変数として注入するようにマニフェストを更新せよ。
4.  **検証** :
    *   Podにログインし、`env` コマンドで環境変数が正しく注入されていることを確認せよ。
    *   Go APIアプリケーション内部でこれらの環境変数が正しく読み取られていることを確認できるような簡単なエンドポイントを追加し、検証せよ（例: `/config` エンドポイントで設定情報を返す）。

---

#### 3.3 永続化ストレージの利用 (PersistentVolume, PersistentVolumeClaim, Cloud Storage FUSE CSI Driver, Cloud SQL Auth Proxy)

**概要** : KubernetesのPodは一時的なものであり、Podが再起動または再スケジュールされるとデータは失われる。ステートフルなアプリケーションには永続化ストレージが必須である。KubernetesではPersistentVolume (PV) とPersistentVolumeClaim (PVC) を用いてストレージを抽象化し、動的なプロビジョニングを可能にする。GCP環境では、Cloud Storage FUSE CSI Driverを利用してCloud StorageをPodにマウントしたり、Cloud SQL Auth Proxyをサイドカーとして利用してCloud SQLに安全に接続 したりすることが一般的である。

**具体例** :
1.  **PersistentVolumeClaim (PVC) と動的プロビジョニング** : StorageClassを定義していれば、PVCを作成するだけでPVが自動的にプロビジョニングされる。
    ```yaml
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: my-pvc
    spec:
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 5Gi
    ```
   
    DeploymentでPVCをマウントする方法:
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: go-api-deployment
    spec:
      template:
        spec:
          containers:
          - name: go-api
            image: asia-northeast1-docker.pkg.dev/your-project-id/my-go-api-repo/go-api:latest
            volumeMounts:
            - name: my-storage
              mountPath: /data
          volumes:
          - name: my-storage
            persistentVolumeClaim:
              claimName: my-pvc
    ```
   
2.  **Cloud Storage FUSE CSI Driver** : Cloud Storageバケットをファイルシステムとしてマウントする例。
    *   まず、Cloud Storage FUSE CSI Driverのデプロイが必要である（通常、Helmなどでインストール）。
    *   StorageClassを定義する。
    ```yaml
    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
      name: gcs-fuse-csi
    provisioner: gcs-fuse.csi.storage.gke.io
    parameters:
      bucketName: "your-gcs-bucket-name"
      # dynamicProvisioning: "true" # 動的プロビジョニングを有効にする場合
    ```
    *   PVCを作成する（`storageClassName: gcs-fuse-csi` を指定）。
    ```yaml
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: my-gcs-pvc
    spec:
      accessModes:
        - ReadWriteMany # GCS CSI DriverはReadWriteManyをサポート
      storageClassName: gcs-fuse-csi
      resources:
        requests:
          storage: 1Gi # 実際にGCSの容量を制限するわけではないが、K8sのPV要件として必要
    ```
    *   DeploymentでPVCをマウントする。
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: go-api-deployment
    spec:
      template:
        spec:
          containers:
          - name: go-api
            image: asia-northeast1-docker.pkg.dev/your-project-id/my-go-api-repo/go-api:latest
            volumeMounts:
            - name: gcs-volume
              mountPath: /gcs-data
          volumes:
          - name: gcs-volume
            persistentVolumeClaim:
              claimName: my-gcs-pvc
    ```
3.  **Cloud SQL Auth Proxy** : Cloud SQLインスタンスへの安全な接続を提供する。サイドカーコンテナとしてPodにデプロイするのが一般的である。
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: go-api-deployment
    spec:
      template:
        spec:
          serviceAccountName: go-api-ksa # Workload Identityが紐付けられたSA
          containers:
          - name: go-api
            image: asia-northeast1-docker.pkg.dev/your-project-id/my-go-api-repo/go-api:latest
            ports:
            - containerPort: 8080
            env:
            - name: DB_HOST
              value: "127.0.0.1" # Cloud SQL Auth ProxyがListenするIP
            - name: DB_PORT
              value: "5432" # Cloud SQL Auth ProxyがListenするポート
            # ... その他のDB接続情報
          - name: cloudsql-proxy
            image: gcr.io/cloudsql-docker/go-cloudsql-proxy:latest
            command: ["/cloud_sql_proxy"]
            args:
              - "-instances=your-project-id:your-region:your-instance-name=tcp:5432"
              - "-enable_iam_login" # IAM認証を利用する場合
            securityContext:
              runAsNonRoot: true
            # 必要に応じてリソース制限とヘルスチェック
            resources:
              requests:
                cpu: "10m"
                memory: "32Mi"
    ```
   
Go APIコンテナのService Accountには、Cloud SQLにアクセスするための適切なIAMロール（例: `cloudsql.client`）が付与されている必要がある（Workload Identity経由）。

**課題** : Go APIアプリケーションが永続化ストレージまたはデータベースに接続できるように設定せよ。
1.  **Cloud Storage FUSE CSI Driverの利用** :
    *   GCSバケットを新規作成せよ。
    *   Cloud Storage FUSE CSI DriverをGKEクラスタにインストールせよ（ドキュメントを参照し、必要に応じてHelmやkubectl applyでデプロイせよ）。
    *   GCSバケットをマウントするStorageClassとPersistentVolumeClaimを作成せよ。
    *   Go APIのPodにこのPVCをマウントし、Go APIがマウントされたパスにファイルを書き込み、そのファイルがGCSバケットに存在することを確認できる簡単な機能を追加せよ。
    *   **検証** : ファイルを書き込み、Podを削除して再作成後もファイルが残っていること、GCSバケットから直接ファイルを確認できることを検証せよ。
2.  **Cloud SQL Auth Proxyの利用** :
    *   Cloud SQLインスタンス（例: PostgreSQLまたはMySQL）をGCPにプロビジョニングせよ。
    *   Go APIアプリケーションのPodにCloud SQL Auth Proxyをサイドカーコンテナとして追加するようにDeploymentマニフェストを修正せよ。
    *   Go APIのKubernetes Service Accountに、Cloud SQLへのアクセス権限を付与するGCPサービスアカウントをWorkload Identityで紐付けよ（章1で学習した内容を適用）。
    *   Go APIアプリケーション内部でCloud SQLへの接続を試み、簡単なデータ操作（例: テーブル作成、データ挿入、読み取り）ができるように機能を拡張せよ。
    *   **検証** : Go APIのエンドポイントを叩くことで、Cloud SQLへのデータ登録・取得が正常に行われることを検証せよ。

---

#### 3.4 自動スケーリングとヘルスチェック (HPA, Liveness Probe, Readiness Probe)

**概要** : Kubernetesにおける自動スケーリングは、アプリケーションの負荷に応じてPod数を自動調整し、リソースの最適化と安定稼働を実現する上で不可欠である。Horizontal Pod Autoscaler (HPA) は、CPU使用率やメモリ使用率、カスタムメトリクスに基づいてPodのレプリカ数を調整する。また、アプリケーションの健全性監視は運用の基本であり、Liveness Probe はPodが正常に動作しているか（停止すべきか）、Readiness Probe はPodがリクエストを受け入れる準備ができているか（トラフィックをルーティングすべきか）を判断する。これらを適切に設定することで、サービスの可用性が向上する。

**具体例** :
1.  **Liveness Probe** : アプリケーションがハングした場合などにPodを再起動させる。
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: go-api-deployment
    spec:
      template:
        spec:
          containers:
          - name: go-api
            image: asia-northeast1-docker.pkg.dev/your-project-id/my-go-api-repo/go-api:latest
            livenessProbe:
              httpGet:
                path: /healthz
                port: 8080
              initialDelaySeconds: 15
              periodSeconds: 10
              timeoutSeconds: 5
              failureThreshold: 3
    ```
   
2.  **Readiness Probe** : アプリケーションがトラフィックを受け入れ可能になるまで待機する。
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: go-api-deployment
    spec:
      template:
        spec:
          containers:
          - name: go-api
            image: asia-northeast1-docker.pkg.dev/your-project-id/my-go-api-repo/go-api:latest
            readinessProbe:
              httpGet:
                path: /ready
                port: 8080
              initialDelaySeconds: 5
              periodSeconds: 5
              timeoutSeconds: 3
              failureThreshold: 1
    ```
   
3.  **Horizontal Pod Autoscaler (HPA)** : CPU使用率に基づいてPod数を自動調整する。
    ```yaml
    apiVersion: autoscaling/v2
    kind: HorizontalPodAutoscaler
    metadata:
      name: go-api-hpa
    spec:
      scaleTargetRef:
        apiVersion: apps/v1
        kind: Deployment
        name: go-api-deployment
      minReplicas: 2
      maxReplicas: 5
      metrics:
      - type: Resource
        resource:
          name: cpu
          target:
            type: Utilization
            averageUtilization: 50
    ```
   

**課題** : Go APIアプリケーションにヘルスチェックを設定し、自動スケーリングを導入せよ。
1.  **ヘルスチェックエンドポイントの追加** : Go APIアプリケーションに、以下のエンドポイントを実装せよ。
    *   `/healthz`: アプリケーションが生存している場合に200 OKを返す（データベース接続など、必須サービスの健全性も確認すること）。
    *   `/ready`: アプリケーションがリクエストを受け入れる準備ができている場合に200 OKを返す（起動時の初期化処理完了後など）。
2.  **Probeの設定** : Go APIのDeploymentマニフェストに、Liveness ProbeとReadiness Probeを設定せよ。上記のヘルスチェックエンドポイントを使用し、`initialDelaySeconds`、`periodSeconds`、`failureThreshold` などのパラメータを適切に設定せよ。
3.  **HPAの導入** :
    *   Go APIのDeploymentに、リソースリクエスト（`resources.requests.cpu`, `resources.requests.memory`）を設定せよ。HPAはこれらの値に基づいてメトリクスを評価する。
    *   Horizontal Pod Autoscaler (HPA) マニフェストを作成し、Go APIのDeploymentをターゲットに設定せよ。CPU使用率が50%を超えた場合にPod数が2から最大5にスケールするように設定せよ。
4.  **動作検証** :
    *   `kubectl get pods` や `kubectl describe pod` コマンドでProbeの状態を確認せよ。
    *   Go APIに意図的に高負荷をかける（例: Apache Benchやlocustなどのツールを使用）ことで、Pod数がHPAによって自動的にスケールアウトすることを確認せよ。負荷が下がった後にスケールインすることも確認せよ。

---

#### 3.5 TerraformによるKubernetesリソースの管理

**概要** : これまでにKubernetesマニフェストを手動で適用してきたが、実運用ではインフラストラクチャと同様にKubernetesリソースもコードとして管理し、バージョン管理、レビュー、自動デプロイメントを可能にすることが求められる。TerraformのKubernetes Providerを利用することで、Deployment、Service、IngressなどのKubernetesリソースをTerraformコードで定義し、IaC (Infrastructure as Code) の原則を徹底できる。これは、GitOpsへの次のステップに繋がる重要なスキルである。

**具体例** : Terraform Kubernetes Providerを用いたDeploymentとServiceの定義例。
```terraform
resource "kubernetes_deployment_v1" "go_api_deployment" {
  metadata {
    name = "go-api-deployment"
    labels = {
      app = "go-api"
    }
  }
  spec {
    replicas = 2
    selector {
      match_labels = {
        app = "go-api"
      }
    }
    template {
      metadata {
        labels = {
          app = "go-api"
        }
      }
      spec {
        container {
          name  = "go-api"
          image = "asia-northeast1-docker.pkg.dev/${google_project.current.project_id}/my-go-api-repo/go-api:latest"
          port {
            container_port = 8080
          }
        }
      }
    }
  }
}

resource "kubernetes_service_v1" "go_api_service_internal" {
  metadata {
    name = "go-api-service-internal"
  }
  spec {
    selector = {
      app = kubernetes_deployment_v1.go_api_deployment.metadata.labels.app
    }
    port {
      protocol    = "TCP"
      port        = 80
      target_port = 8080
    }
    type = "ClusterIP"
  }
}
```


**課題** : これまでの章で手動で作成・適用してきたKubernetesマニフェスト（Deployment, Service, Ingress, ConfigMap, Secret, HPA）を、全てTerraformコードに変換し、Terraformで管理できるようにせよ。
1.  **Terraformプロジェクトの構造化** : 新しいTerraformプロジェクトディレクトリを作成し、Kubernetesリソースを管理するための適切なファイル分割と構成（例: `main.tf`, `variables.tf`, `versions.tf`など）を行え。
2.  **Kubernetes Providerの設定** : GKEクラスタに接続するためのTerraform Kubernetes Providerを設定せよ。GCPのGKEクラスタ情報（エンドポイント、認証情報）をデータソースとして取得し、プロバイダに渡すようにせよ。
3.  **既存マニフェストの変換** : これまでに作成したGo APIのDeployment、Service、Ingress、ConfigMap、Secret、HPAの各マニフェストをTerraformの `kubernetes_` リソースブロックとして定義し直せ。
4.  **状態のインポート（推奨）** : 既にクラスタにデプロイされているリソースがある場合、Terraformでそれらのリソースをインポートし、Terraformのstateに含めよ。これにより、既存のリソースを破壊せずにTerraform管理下に移行できる。
5.  **デプロイと検証** :
    *   `terraform plan` を実行し、Terraformが認識する変更内容を確認せよ。意図しない変更がないことを厳しく確認せよ。
    *   `terraform apply` を実行し、KubernetesリソースがTerraformによってプロビジョニングまたは変更されることを確認せよ。
    *   変更後もGo APIアプリケーションが正常に稼働していること、および各種設定（ConfigMap, Secret）が反映されていることを検証せよ。
    *   `terraform destroy` を実行する前に、KubernetesリソースがTerraformによって削除されることを計画 (`plan`) で確認し、必要であれば実際に一部を削除して動作を理解せよ（ただし、本番環境での実行は厳禁）。

---

**成果物** : GKEクラスタ上で稼働するGo APIアプリケーションと、Terraformによってコードで完全に管理されたKubernetesマニフェストが成果物となる。これにより、アプリケーションのデプロイと管理の再現性と自動化が実現される。

---

### 章4: 高度なKubernetesデプロイと環境管理

**目的** : 本章では、複数環境におけるKubernetesアプリケーションの管理と、実践的なデプロイ手法を習得する。具体的には、開発、ステージング、本番といった異なるGCP環境に安全かつ効率的にアプリケーションをデプロイするための技術と、セキュリティを強化するためのアクセス制御を確立することを目的とする。この章で習得するスキルは、実際のサービス運用におけるデプロイプロセスの標準化とセキュリティ強化に直結する。

**学習内容** : 以下の要素を体系的に習得する。
*   **Kustomizeによる環境ごとのマニフェスト管理** : `base/`と`overlays/`の構造を用いて、開発、ステージング、本番といった異なる環境の設定を効率的に管理する。
*   **Workload Identityを用いたセキュアなIAMアクセス制御** : Pod単位でGCPのサービスアカウントを割り当て、最小権限の原則でGCPリソース（Secret Manager、Cloud SQLなど）へアクセスさせる 。
*   **External Secrets Operatorによるシークレット管理** : GCP Secret Manager に格納された機密情報をKubernetesのSecretリソースに自動的に同期させる。
*   **GKE IngressとGoogle Managed SSL/TLS証明書** : 外部からのHTTPS通信をGCP Load Balancer で受け、証明書の管理を自動化する 

---

#### 4.1 Kustomizeによる環境ごとのマニフェスト管理

**概要** : Kubernetesアプリケーションを開発、ステージング、本番といった複数の環境にデプロイする際、各環境で異なる設定（例: レプリカ数、リソース制限、Ingressのホスト名、ConfigMapの値）を管理する必要がある。これらの設定を手動で編集したり、環境ごとにマニフェストファイルを複製したりすることは、非効率的であり、エラーの原因となる。Kustomizeは、Kubernetesネイティブな方法で、ベースとなるマニフェストから環境固有の差分を適用し、最終的なマニフェストを生成するためのツールである。これにより、マニフェストの重複を避け、環境間の差分管理を効率化し、可読性と保守性を向上させる。

**具体例** : Kustomizeは通常、`base/` ディレクトリに共通のマニフェストを配置し、`overlays/` ディレクトリに環境ごとの差分（パッチ）を定義する。

```yaml
# base/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: go-api-deployment
  labels:
    app: go-api
spec:
  replicas: 2
  selector:
    matchLabels:
      app: go-api
  template:
    metadata:
      labels:
        app: go-api
    spec:
      containers:
      - name: go-api
        image: asia-northeast1-docker.pkg.dev/your-project-id/my-go-api-repo/go-api:v1.0.0
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "200m"
            memory: "256Mi"

# base/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- deployment.yaml
- service.yaml # 必要に応じてService, Ingressなども追加

# overlays/production/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
bases:
- ../../base
patchesStrategicMerge:
- production-patch.yaml # または patches: - path: production-patch.yaml で JSON Patch/Strategic Merge Patch

# overlays/production/production-patch.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: go-api-deployment
spec:
  replicas: 5
  template:
    spec:
      containers:
      - name: go-api
        image: asia-northeast1-docker.pkg.dev/your-project-id/my-go-api-repo/go-api:v1.0.0-prod # 本番用イメージ
        resources:
          requests:
            cpu: "500m"
            memory: "512Mi"
          limits:
            cpu: "1000m"
            memory: "1024Mi"
```

上記の例では、`base/` のデプロイメントがデフォルトのレプリカ2、特定のリソース設定を持ち、`overlays/production/` ではレプリカ数を5に、リソースを本番向けに引き上げている。

**課題** : 前章でデプロイしたGo APIアプリケーションのマニフェストをKustomizeで管理せよ。
1.  **Kustomizeプロジェクトの初期化** :
    *   アプリケーションのKubernetesマニフェストを格納する新しいGitリポジトリ（または既存リポジトリ内のサブディレクトリ）を作成せよ。
    *   `base/` ディレクトリを作成し、そこにGo APIのDeployment、Service、Ingress、ConfigMap、Secret（ダミー値で可）などの共通マニフェストを配置せよ。これらのマニフェストは環境に依存しない汎用的な内容にせよ。
    *   `base/kustomization.yaml` を作成し、`resources` フィールドにすべてのベースマニフェストをリストせよ。
2.  **環境ごとのオーバーレイ作成** :
    *   `overlays/` ディレクトリの下に `development/` と `production/` というサブディレクトリを作成せよ。
    *   `overlays/development/kustomization.yaml` を作成し、開発環境用の設定（例: `replicas: 1`、低リソース要求、開発環境特有のConfigMap値など）をパッチとして定義せよ。
    *   `overlays/production/kustomization.yaml` を作成し、本番環境用の設定（例: `replicas: 3`以上、高リソース要求、本番環境特有のIngressホスト名、本番イメージタグなど）をパッチとして定義せよ。`base` のリソースを参照することを忘れるな。
3.  **マニフェストの生成と適用** :
    *   `kubectl kustomize overlays/development` コマンドを実行し、開発環境用の最終マニフェストが正しく生成されることを確認せよ。
    *   `kubectl kustomize overlays/production` コマンドを実行し、本番環境用の最終マニフェストが正しく生成されることを確認せよ。
    *   Terraformの`kubernetes_manifest`リソース（または`kubernetes_all`データソースと`kubernetes_manifest`リソースの組み合わせ）を使用して、Kustomizeで生成したマニフェストをGKEクラスタにデプロイするTerraformコードを作成せよ。異なる環境にデプロイできるように、Terraformのワークスペースや変数を利用せよ。
4.  **動作検証** :
    *   開発環境のクラスタと本番環境のクラスタ（または同一クラスタ内の異なるNamespace）に、それぞれのKustomizeで生成したマニフェストを適用せよ。
    *   各環境でGo APIアプリケーションが期待通りの設定（レプリカ数、ConfigMapの値など）で稼働していることを `kubectl get` や `kubectl describe` コマンドで確認せよ。

---

#### 4.2 Workload Identityを用いたセキュアなIAMアクセス制御

**概要** : Kubernetes PodがGCPリソース（Secret Manager、Cloud SQL、Cloud Storageなど）にアクセスする際、GCPサービスアカウントの認証情報が必要である。GCPサービスアカウントキーをKubernetes SecretとしてPodに直接マウントする方法はセキュリティリスクが高く、非推奨である。Workload Identityは、KubernetesのサービスアカウントとGCPのサービスアカウントをセキュアに紐付けることで、PodがGCPサービスアカウントの権限を透過的に利用できるようにするメカニズムである。これにより、認証情報の漏洩リスクを低減し、最小権限の原則に基づいたきめ細かいアクセス制御をPod単位で実現する。

**具体例** : Workload Identityを使用するには、GKEクラスタでWorkload Identityを有効化し、KubernetesサービスアカウントとGCPサービスアカウントの間でIAMバインディングを設定する。
1.  **GCPサービスアカウントの作成** :
    ```bash
    gcloud iam service-accounts create my-api-gsa --project=your-gcp-project
    ```
   
2.  **IAMポリシーのバインディング** : GCPサービスアカウントがKubernetesサービスアカウントのIDを借用できるようにするIAMポリシーを設定する。
    ```bash
    gcloud iam service-accounts add-iam-policy-binding my-api-gsa@your-gcp-project.iam.gserviceaccount.com \
        --role="roles/iam.workloadIdentityUser" \
        --member="serviceAccount:your-gcp-project.svc.id.goog[your-namespace/my-api-ksa]"
    ```
   
3.  **Kubernetesサービスアカウントのアノテーション** : KubernetesサービスアカウントにGCPサービスアカウントを紐付けるアノテーションを追加する。
    ```yaml
    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: my-api-ksa
      annotations:
        iam.gke.io/gcp-service-account: my-api-gsa@your-gcp-project.iam.gserviceaccount.com
    ```
   
4.  **Podでのサービスアカウントの利用** : Podの定義で`serviceAccountName: my-api-ksa`を指定することで、Pod内のアプリケーションは`my-api-gsa`の権限でGCPリソースにアクセスできる。
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: go-api-deployment
    spec:
      template:
        spec:
          serviceAccountName: my-api-ksa # ここでSAを指定
          containers:
          - name: go-api
            image: asia-northeast1-docker.pkg.dev/your-project-id/my-go-api-repo/go-api:latest
            # ...
    ```

**課題** : Go APIアプリケーションがWorkload Identityを用いてGCP Secret Managerにセキュアにアクセスできるように設定せよ。
1.  **GKEクラスタのWorkload Identity有効化確認** :
    *   現在利用しているGKEクラスタでWorkload Identityが有効になっていることを確認せよ。有効でない場合は、既存のGKEクラスタ設定をTerraformで修正し、Workload Identityを有効化せよ。
2.  **GCPサービスアカウントの作成と権限付与** :
    *   前章で作成したGCP Secret Managerのシークレット（例: `my-api-key`, `db-connection-string`）にアクセスするための専用GCPサービスアカウント（例: `go-api-secret-reader-gsa`）をTerraformで作成せよ。
    *   このGCPサービスアカウントに対し、**必要最小限の権限**、具体的にはこれらのシークレットに対する「Secret Manager のシークレット アクセサー」ロール（`roles/secretmanager.secretAccessor`）のみを付与せよ。Terraformの`google_secret_manager_secret_iam_member`リソースを活用せよ。
3.  **Kubernetesサービスアカウントの作成と紐付け** :
    *   Kubernetes上に、Go APIアプリケーションが使用するサービスアカウント（例: `go-api-ksa`）をKustomizeの`base`マニフェストとして作成せよ。
    *   このKubernetesサービスアカウントに、先ほど作成したGCPサービスアカウントを紐付けるアノテーションを追加せよ。この設定もKustomizeを通じて管理されるようにせよ。
    *   GCPサービスアカウントとKubernetesサービスアカウントの間の`iam.workloadIdentityUser`ロールのバインディングをTerraformで設定せよ。
4.  **Go APIアプリケーションの修正とデプロイ** :
    *   Go APIアプリケーションのDeploymentマニフェストを更新し、`serviceAccountName: go-api-ksa` を指定するようにせよ。
    *   Go APIアプリケーションのコードを修正し、GCP Secret Manager SDK（例: `cloud.google.com/go/secretmanager/apiv1`）を使用して、先ほど登録したシークレットを読み込む機能を追加せよ。**注意** : 実際のアプリケーションコード修正は、このカリキュラムの範囲外だが、概念として理解し、どのように実装するかを設計せよ。
5.  **動作検証** :
    *   Kustomizeでマニフェストを生成し、GKEクラスタにデプロイせよ。
    *   デプロイされたGo API Podにログインし、GCP Secret Managerへのアクセスを試み、`gcloud secret access <secret-name> --project=<your-project>` コマンド（またはGo API内の機能）が成功し、シークレットが正しく取得できることを確認せよ。
    *   `kubectl describe sa go-api-ksa` コマンドで、アノテーションが正しく適用されていることを確認せよ。
    *   `kubectl get serviceaccount go-api-ksa -o yaml` でService AccountにGCP Service Accountが正しく紐付けられていることを確認せよ。
    *   意図的に権限を持たないGCPサービスアカウントを紐付けた場合や、権限を付与しない場合にアクセスが失敗することを確認し、Workload Identityの機能とIAM権限の重要性を理解せよ。

---

#### 4.3 External Secrets Operatorによるシークレット管理

**概要** : 前節でWorkload Identityを用いてGCP Secret Managerからシークレットを直接読み込む方法を学習したが、Kubernetesネイティブな方法でシークレットを扱う場合、Secretリソースとしてクラスタ内に存在させたいことがある。しかし、Secretリソースに直接機密情報を記述することは非推奨である。External Secrets Operator (ESO) は、GCP Secret Managerのような外部のシークレット管理システムに格納された機密情報を、Kubernetesクラスタ内のSecretリソースに自動的に同期させるためのコントローラーである。これにより、シークレットの一元管理は外部システムで行い、Kubernetesアプリケーションは標準的なSecretリソースとして機密情報にアクセスできるようになる。これにより、より堅牢なシークレット運用と、GitOpsワークフローとの連携が容易になる。

**具体例** : External Secrets Operatorを導入し、ExternalSecretリソースを定義することで、GCP Secret ManagerのシークレットがKubernetesのSecretリソースに自動同期される。
1.  **External Secrets Operatorのインストール** : Helmなどを用いてクラスタにESOをデプロイする。
2.  **SecretStore の定義** : 外部シークレットプロバイダー（GCP Secret Manager）への接続方法を定義する。Workload Identityを使用する場合、ここでもWorkload Identityを介したGCPサービスアカウントの指定が必要である。
    ```yaml
    apiVersion: external-secrets.io/v1beta1
    kind: SecretStore
    metadata:
      name: gcp-secret-manager
    spec:
      provider:
        gcpsm:
          projectID: "your-gcp-project-id"
          auth:
            workloadIdentity:
              serviceAccountRef:
                name: go-api-ksa # Workload Identityが紐付けられたK8s SA
    ```
   
3.  **ExternalSecret の定義** : どのGCP Secret Managerのシークレットを、どのKubernetes Secretに同期するかを定義する。
    ```yaml
    apiVersion: external-secrets.io/v1beta1
    kind: ExternalSecret
    metadata:
      name: go-api-external-secret
    spec:
      refreshInterval: "1h" # 同期間隔
      secretStoreRef:
        name: gcp-secret-manager
        kind: SecretStore
      target:
        name: go-api-secret # 作成するKubernetes Secretの名前
        creationPolicy: Owner
      data:
        - secretKey: API_KEY # Kubernetes Secretのキー
          remoteRef:
            key: my-api-key # GCP Secret Managerのシークレット名
            version: latest # 最新バージョン
        - secretKey: DB_CONNECTION_STRING
          remoteRef:
            key: db-connection-string
            version: latest
    ```
   
`go-api-secret`というKubernetes Secretが自動生成され、Podから環境変数やボリュームとして利用できる。

**課題** : Go APIアプリケーションの機密情報をExternal Secrets Operatorを用いてGCP Secret ManagerからKubernetes Secretに同期せよ。
1.  **External Secrets Operatorのデプロイ** :
    *   Helm（または公式ドキュメントの方法）を用いて、GKEクラスタにExternal Secrets Operatorをデプロイせよ。
2.  **SecretStore の定義とWorkload Identity連携** :
    *   Workload Identityを通じてGCP Secret ManagerにアクセスするためのSecretStoreマニフェストを作成せよ。
    *   このSecretStoreが、前節でWorkload Identity用に設定したKubernetesサービスアカウント（例: `go-api-ksa`）を使用するように設定せよ。
    *   このSecretStoreマニフェストをKustomizeの`base`ディレクトリに追加し、Terraformでデプロイできるようにせよ。
3.  **ExternalSecret の定義** :
    *   GCP Secret Managerに登録されている `my-api-key` と `db-connection-string` をKubernetesの`go-api-secret`という名前のSecretリソースに同期させるExternalSecretマニフェストを作成せよ。
    *   このExternalSecretマニフェストもKustomizeの`base`ディレクトリに追加せよ。
4.  **Go APIアプリケーションのDeployment更新** :
    *   Go APIのDeploymentマニフェストを更新し、`go-api-secret`というKubernetes Secretから環境変数を読み込むように修正せよ（章3で学習した方法と同様）。これにより、アプリケーションコードは直接GCP Secret Manager SDKを呼び出す必要がなくなる。
5.  **動作検証** :
    *   Kustomizeで生成したマニフェストをGKEクラスタにデプロイせよ。
    *   `kubectl get secret go-api-secret -o yaml` を実行し、Kubernetes Secretが自動的に作成され、期待するデータがBase64エンコードされた形式で含まれていることを確認せよ。
    *   Go API Podにログインし、環境変数が正しく注入されていること、およびアプリケーションがシークレットを正しく読み取れていることを確認せよ。
    *   GCP Secret Managerでシークレットの値を更新し、External Secrets Operatorがそれを検知してKubernetes Secretが自動的に更新されることを確認せよ。

---

#### 4.4 GKE IngressとGoogle Managed SSL/TLS証明書

**概要** : ウェブアプリケーションが外部から安全にアクセスされるためには、HTTPS通信が不可欠である。GKE環境では、Ingressリソースを通じてGCP Load Balancer (GCLB) がプロビジョニングされ、外部からのHTTP(S)トラフィックをKubernetesサービスにルーティングする。Google Managed SSL/TLS証明書は、GCPが提供するフルマネージドな証明書サービスであり、証明書の取得、更新、デプロイ、そして失効までを自動的に処理する。これにより、SSL/TLS証明書の管理にかかる運用コストと手間を大幅に削減し、常にセキュアな通信を維持できる。

**具体例** : GKE IngressとGoogle Managed SSL/TLS証明書を組み合わせるには、ManagedCertificateリソース（またはGKE Ingressのアノテーション）を使用する。
1.  **静的IPアドレスのプロविजन** : 外部からのアクセス用に静的IPアドレスが必要である。
    ```bash
    gcloud compute addresses create go-api-external-ip --global
    ```
2.  **ManagedCertificate の定義** : ドメイン名と紐付ける。
    ```yaml
    apiVersion: networking.gke.io/v1
    kind: ManagedCertificate
    metadata:
      name: go-api-cert
    spec:
      domains:
        - "your-subdomain.your-domain.com"
    ```
3.  **Ingressの定義** : ManagedCertificateを参照するアノテーションを追加し、静的IPアドレスを割り当てる。
    ```yaml
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: go-api-ingress
      annotations:
        kubernetes.io/ingress.global-static-ip-name: "go-api-external-ip"
        networking.gke.io/managed-certificates: "go-api-cert"
    spec:
      tls:
      - secretName: go-api-cert # ManagedCertificateの名前と同じにする
      rules:
      - host: "your-subdomain.your-domain.com"
        http:
          paths:
          - path: /*
            pathType: ImplementationSpecific
            backend:
              service:
                name: go-api-service-internal
                port:
                  number: 80
    ```
   
**注意** : 実際のドメインとDNS設定が必要である。

**課題** : Go APIアプリケーションへの外部からのHTTPSアクセスを、GKE IngressとGoogle Managed SSL/TLS証明書を用いて実現せよ。
1.  **ドメインとDNS設定の準備** :
    *   テスト用に利用可能なドメイン（例: `your-subdomain.your-domain.com`）を用意せよ。または、Google DomainsやCloud DNSで新しいドメインを購入・設定せよ。
    *   このドメインを解決するために、DNSのAレコードをGCPの静的グローバルIPアドレス（前章で取得済み、または新規取得）に向けるよう設定せよ。
2.  **ManagedCertificate リソースの作成** :
    *   用意したドメイン名（例: `your-subdomain.your-domain.com`）を指定したManagedCertificateマニフェストを作成せよ。
    *   このマニフェストをKustomizeの`overlays/production`ディレクトリに追加せよ。これは本番環境のみでHTTPSを有効化する想定である。
3.  **GKE Ingressの設定更新** :
    *   既存のGo APIのIngressマニフェストを更新し、HTTPSを有効にする`tls`セクションを追加せよ。`secretName`はManagedCertificateリソースの名前と一致させよ。
    *   Ingressのアノテーションに、`networking.gke.io/managed-certificates`と`kubernetes.io/ingress.global-static-ip-name`を追加し、それぞれ作成したManagedCertificateと静的IPアドレスの名前を指定せよ。
    *   このIngressマニフェストの変更もKustomizeの`overlays/production`ディレクトリにパッチとして定義せよ。
4.  **動作検証** :
    *   Kustomizeで本番環境用のマニフェストを生成し、GKEクラスタにデプロイせよ。
    *   `kubectl get managedcertificate go-api-cert -o yaml` コマンドで証明書の状態を確認し、`Status: Active`になっていることを確認せよ（DNS伝播と証明書プロビジョニングには時間がかかる場合がある）。
    *   ブラウザや `curl -v https://your-subdomain.your-domain.com/` コマンドでGo APIにHTTPSでアクセスし、SSL/TLS証明書が正しく適用されていること、および通信が暗号化されていることを確認せよ。
    *   GCPコンソールでGCLBのフロントエンド設定を確認し、SSL証明書が正しく関連付けられていることを確認せよ。

---

**成果物** : KustomizeとExternal Secrets Operatorを活用し、異なるGCP環境に安全かつ効率的にデプロイ可能なKubernetesアプリケーションが構築される。また、Workload IdentityによりGCPリソースに安全にアクセスできるGo APIが構築され、Google Managed SSL/TLS証明書によりHTTPS通信が提供される。これにより、実運用に耐えうる複数環境管理とセキュリティ基盤が完成する。

---

### 章5: CI/CDパイプラインの構築と自動化

**目的** : 本章の目的は、GitOpsの原則に基づき、完全に自動化されたCI/CDパイプラインを構築し、効率的なデプロイを実現することである。これは、継続的なデリバリーを可能にし、開発サイクルを高速化するために不可欠なプロセスである。

**学習内容** : 以下の要素を体系的に習得する。
*   **GitHub Actionsを用いたCI/CDパイプラインの構築** :
    *   Go APIのビルド、ユニットテスト、インテグレーションテスト。
    *   Artifact Registry へのコンテナイメージのプッシュ。
    *   Kubernetesマニフェストのlintと検証。
*   **Argo CDの導入とGitOpsの実践** :
    *   Argo CDのGKEクラスタへのデプロイと初期設定。
    *   Gitリポジトリ（Kustomizeで管理されたマニフェスト）をArgo CDに連携させ、アプリケーションの自動同期を構成する。
    *   Argo CD Image UpdaterによるCI/CDの自動化: Artifact Registry 上の新しいタグを自動で検知し、Gitリポジトリのイメージタグを更新（自動コミット）させる。これにより、Argo CDがその差分を検知してKubernetesリソースを自動的に更新する仕組みを構築する。

---

#### 5.1 GitHub Actionsを用いたCI/CDパイプラインの構築

**概要** : 開発プロセスにおいて、コードの変更が自動的にテストされ、ビルドされ、デプロイ可能な成果物として生成されるCI（継続的インテグレーション）は不可欠である。GitHub Actionsは、Gitリポジトリと密接に連携し、これらのワークフローを効率的に自動化するための強力なツールである。本節では、Go APIのビルドとテスト、そしてコンテナイメージのArtifact Registryへのプッシュ、さらにKubernetesマニフェストの品質チェックをGitHub Actionsで自動化する実務的な方法を習得する。

##### 5.1.1 Go APIのビルド、ユニットテスト、インテグレーションテスト

**概要** : Go APIの品質と安定性を保証するためには、自動化されたテストが不可欠である。ユニットテストは個々の機能の検証、インテグレーションテストは複数のコンポーネント間の連携検証を目的とする。これらをCIパイプラインに組み込むことで、コード変更による回帰バグを早期に発見し、開発コストを削減する。

**具体例** : Goアプリケーションのビルドとテストを実行するGitHub Actionsワークフローの例を示す。
```yaml
# .github/workflows/go-ci.yaml
name: Go CI
on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - name: Set up Go
      uses: actions/setup-go@v4
      with:
        go-version: '1.21'
    - name: Download Go modules
      run: go mod download
    - name: Run unit tests
      run: go test ./...
    # - name: Run integration tests (optional)
    #   run: |
    #     # Example for integration tests with a local database
    #     docker-compose up -d db
    #     go test -tags=integration ./...
```

インテグレーションテストが含まれる場合は、外部サービス（例えばダミーデータベース）への接続をモックするか、あるいはテスト環境をセットアップするステップを追加する。例えば、Docker Composeでテスト用DBを立ち上げるなどの手法が考えられる。

**課題** : 前章までに作成したGo APIリポジトリに、GitHub Actionsのワークフローを追加せよ。
1.  **CIワークフローの作成** : Go APIのソースコードが存在するリポジトリの `.github/workflows/` ディレクトリに、新しいYAMLファイルを作成せよ。
2.  **ビルドとユニットテストの自動化** :
    *   プッシュまたはプルリクエスト時に、GoモジュールのダウンロードとGo APIのビルドが行われるように設定せよ。
    *   全てのGoパッケージのユニットテストが自動的に実行され、結果がGitHub Actionsのログに表示されることを確認せよ。
3.  **インテグレーションテストの考慮** : Go APIにインテグレーションテストが含まれる場合、テストに必要な環境（例: ローカルDBコンテナ）をGitHub Actions内でセットアップし、テストを実行するステップを検討・追加せよ。もしテストが存在しない場合、インテグレーションテストの骨子（例: ダミーDBへの疎通確認）を実装し、CIで実行されることを確認せよ。
4.  **動作検証** : コードをプッシュし、GitHub Actionsが正常に実行され、ビルドとテストが全てパスすることを確認せよ。意図的にテストを失敗させて、CIがエラーを検出することも確認せよ。

---

##### 5.1.2 Artifact Registry へのコンテナイメージのプッシュ

**概要** : ビルドされたGo APIはコンテナイメージとしてパッケージ化され、GCPのArtifact Registry へプッシュされる。これにより、バージョン管理されたデプロイ可能な成果物が作成され、セキュリティと配布の効率性が向上する。GitHub ActionsからArtifact Registryへセキュアに認証を行う方法も習得する。

**具体例** : GitHub ActionsでGo APIのDockerイメージをビルドし、Artifact Registryにプッシュするワークフローの例を示す。GCPへの認証には、Workload Identity Federation for GitHub Actionsを利用する。
```yaml
# .github/workflows/go-build-publish.yaml (一部抜粋)
name: Go Build and Publish
on:
  push:
    branches:
      - main
jobs:
  build-and-publish:
    runs-on: ubuntu-latest
    permissions:
      contents: 'read'
      id-token: 'write' # For Workload Identity Federation
    steps:
    - uses: actions/checkout@v3
    - name: 'Authenticate to Google Cloud'
      id: 'auth'
      uses: 'google-github-actions/auth@v1'
      with:
        workload_identity_provider: 'projects/your-project-number/locations/global/workloadIdentityPools/your-pool-id/providers/your-provider-id'
        service_account: 'your-artifact-registry-pusher-sa@your-project-id.iam.gserviceaccount.com'
    - name: 'Set up Cloud SDK'
      uses: 'google-github-actions/setup-gcloud@v1'
    - name: 'Docker Login to Artifact Registry'
      run: |-
        gcloud auth configure-docker asia-northeast1-docker.pkg.dev
    - name: Build and push Docker image
      run: |
        docker build -t asia-northeast1-docker.pkg.dev/your-project-id/my-go-api-repo/go-api:${{ github.sha }} .
        docker push asia-northeast1-docker.pkg.dev/your-project-id/my-go-api-repo/go-api:${{ github.sha }}
```


**課題** : 前節でビルド・テストを自動化したワークフローに、Artifact Registryへのコンテナイメージのプッシュ機能を追加せよ。
1.  **GCP認証の設定** :
    *   GitHub ActionsがGCPリソースにアクセスできるよう、Workload Identity Federation for GitHub Actionsを設定せよ。GCP側でWorkload Identity Pool、Provider、そしてArtifact Registryへのプッシュ権限を持つ専用のサービスアカウントを作成し、GitHub Actionsリポジトリとサービスアカウントを紐付けよ。
2.  **Dockerイメージのビルドとプッシュ** :
    *   Go APIのDockerfile（章2で作成済み）を使用してDockerイメージをビルドし、GCP Artifact Registryにプッシュするステップを追加せよ。
    *   イメージタグには、GitコミットのSHA値やビルドIDなど、一意で追跡可能な値を自動で付与するように設定せよ。
3.  **動作検証** : コードをプッシュし、GitHub Actionsが正常に実行され、Artifact Registryに新しいタグの付いたGo APIイメージがプッシュされていることを確認せよ。

---

##### 5.1.3 Kubernetesマニフェストのlintと検証

**概要** : Kubernetesマニフェストの記述ミスやベストプラクティスからの逸脱は、デプロイ失敗や運用上の問題を引き起こす。CIパイプラインでマニフェストのlint（構文チェック）と検証（意味チェック、Kustomize適用チェック）を行うことで、これらの問題をデプロイ前に特定し、品質を向上させる。特に、Kustomizeで管理されたマニフェストは、最終的な適用形をCIで確認することが重要である。

**具体例** : Kubernetesマニフェストのlintと検証を行うGitHub Actionsワークフローの例を示す。kubevalやkonstraint、Kustomizeのビルド確認などを組み合わせる。
```yaml
# .github/workflows/k8s-manifest-validation.yaml
name: K8s Manifest Validation
on:
  pull_request:
    branches:
      - main
    paths:
      - 'kubernetes/**' # Kustomizeマニフェストが格納されているパス
jobs:
  validate-manifests:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - name: Set up Kustomize
      uses: kodermax/setup-kustomize@v1
      with:
        kustomize-version: "4.x"
    - name: Build Kustomize manifests for dev
      run: kustomize build kubernetes/overlays/development > manifests-dev.yaml
    - name: Build Kustomize manifests for prod
      run: kustomize build kubernetes/overlays/production > manifests-prod.yaml
    - name: Validate Kubernetes manifests (kubeval)
      uses: instrumenta/kubeval-action@master
      with:
        files: |
          manifests-dev.yaml
          manifests-prod.yaml
        kubeval_version: 0.16.1
        # Add other options like --strict, --schema-location
    # - name: Run OPA Conftest (Optional)
    #   uses: instrumenta/conftest-action@v1
    #   with:
    #     policy: ./policy # Your OPA policies
    #     files: |
    #       manifests-dev.yaml
    #       manifests-prod.yaml
```


**課題** : 前章でKustomizeを用いて管理しているKubernetesマニフェストのリポジトリに、GitHub Actionsのワークフローを追加せよ。
1.  **マニフェスト検証ワークフローの作成** : Kubernetesマニフェストが格納されているリポジトリの `.github/workflows/` ディレクトリに、新しいYAMLファイルを作成せよ。
2.  **Kustomizeビルドと静的解析** :
    *   Kustomizeの `base` および `overlays` ディレクトリから、各環境（開発、本番）ごとの最終的なKubernetesマニフェストを `kustomize build` コマンドで生成するステップを追加せよ。
    *   生成されたマニフェストに対し、`kubeval` などのツールを用いてスキーマ検証を実行するステップを追加せよ。
    *   （任意）より高度な検証として、`konstraint` や OPA Gatekeeper の`conftest` などを利用し、カスタムルールに基づいたベストプラクティスチェックを組み込むことを検討せよ。
3.  **動作検証** : マニフェストを修正し、意図的に構文エラーや不正なフィールドを導入し、CIがエラーを検出することを確認せよ。その後、修正してCIがパスすることを確認せよ。

---

#### 5.2 Argo CDの導入とGitOpsの実践

**概要** : GitOpsは、Gitリポジトリを「信頼できる唯一の情報源（Single Source of Truth）」として、インフラストラクチャとアプリケーションのデプロイを自動化する運用モデルである。これにより、構成の履歴管理、変更の追跡、デプロイの自動化と可視化が実現される。Argo CDは、KubernetesネイティブなGitOpsツールとして、Gitリポジトリとクラスタの状態を常に同期させ、自動化された継続的デリバリーを可能にする。

##### 5.2.1 Argo CDのGKEクラスタへのデプロイと初期設定

**概要** : Argo CDをKubernetesクラスタにデプロイし、初期設定を行うことは、GitOpsワークフローの基盤を築く最初のステップである。GKEクラスタにArgo CDを導入することで、宣言的な方法でアプリケーションの状態を管理し、自動同期によるデプロイを実現できる。

**具体例** : Helmを利用してArgo CDをGKEクラスタにデプロイする一般的な手順を示す。
```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
helm install argocd argo/argo-cd -n argocd --create-namespace -f values.yaml
```

`values.yaml` の例（Ingressを有効にする場合など）:
```yaml
# values.yaml (一部抜粋)
server:
  ingress:
    enabled: true
    hosts:
      - argocd.your-domain.com
    tls:
      - hosts:
          - argocd.your-domain.com
        secretName: argocd-ingress-tls # Google Managed Certificatedによって作成されたSecret名
    annotations:
      kubernetes.io/ingress.global-static-ip-name: "your-argocd-static-ip"
      networking.gke.io/managed-certificates: "your-argocd-managed-cert"
```

初期パスワードの取得とログイン:
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```


**課題** : 構築済みのGKEクラスタにArgo CDをデプロイし、初期設定を行え。
1.  **Argo CDのデプロイ** : Helm Chart（推奨）または公式マニフェストを用いて、Argo CDをGKEクラスタにデプロイせよ。専用のNamespace (`argocd`) にデプロイするようにせよ。
2.  **Argo CD UIへのアクセス** :
    *   外部からArgo CD UIにアクセスできるよう、Ingressを設定し、静的IPアドレスとGoogle Managed SSL/TLS証明書を適用せよ（章4の知識を応用）。
    *   UIに管理者としてログインし、ダッシュボードが正常に表示されることを確認せよ。初期パスワードの取得方法も確認せよ。
3.  **セキュリティ設定** : Argo CDのパスワードをセキュアなものに変更し、不要な機能（例: RBACの制限）を無効にするなど、初期セキュリティ設定を検討せよ。

---

##### 5.2.2 GitリポジトリとArgo CDの連携、自動同期の構成

**概要** : Argo CDの核心機能は、Gitリポジトリで定義された宣言的なアプリケーション状態をKubernetesクラスタに自動的に同期させることである。前章でKustomizeを用いて管理したKubernetesマニフェストのリポジトリをArgo CDに登録し、Argo CD Applicationリソースとしてデプロイすることで、GitOpsによる自動デプロイを実現する。

**具体例** : Gitリポジトリ（Kustomizeを含む）をArgo CDに連携させ、アプリケーションの自動同期を構成するArgo CD Applicationリソースの例を示す。
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: go-api-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/your-org/your-k8s-manifests.git # Kustomizeマニフェストのリポジトリ
    targetRevision: HEAD
    path: kubernetes/overlays/production # Kustomizeのオーバーレイパス
    kustomize: {} # Kustomizeを使用する場合に指定
  destination:
    server: https://kubernetes.default.svc
    namespace: my-app-prod # デプロイ先のK8s Namespace
  syncPolicy:
    automated: # 自動同期を有効化
      prune: true # Gitに存在しないK8sリソースを削除
      selfHeal: true # K8sクラスタの手動変更をGitの状態に自動修復
    syncOptions:
      - CreateNamespace=true # Namespaceが存在しない場合に自動作成
```


**課題** : Kustomizeで管理しているGo APIのKubernetesマニフェストをArgo CDで管理し、自動同期を構成せよ。
1.  **Gitリポジトリの登録** : Argo CDのUIまたはCLI/APIを通じて、Go APIのKubernetesマニフェストを管理しているGitリポジトリをArgo CDに登録せよ。プライベートリポジトリの場合はSSHキーやHTTPSトークンなどの認証情報を設定せよ。
2.  **Argo CD Applicationの作成** :
    *   Go APIアプリケーションをデプロイするためのArgo CD Applicationマニフェスト（YAML）を作成せよ。
    *   このマニフェスト内で、Kustomizeのパス（例: `kubernetes/overlays/production`）を指定し、ターゲットとなるGKEクラスタのNamespace (`my-app-prod` など) を設定せよ。
    *   `syncPolicy` を `automated` に設定し、`prune: true` と `selfHeal: true` を有効にして、GitOpsの原則に従った自動デプロイと状態の同期を構成せよ。
3.  **動作検証** :
    *   作成したApplicationマニフェストを `kubectl apply` でGKEクラスタにデプロイせよ。
    *   Argo CD UIでアプリケーションの状態が `Synced` となり、Go APIのPodが指定されたNamespaceにデプロイされ、正常に稼働していることを確認せよ。
    *   Kubernetesリソースをクラスタ上で手動で変更し、Argo CDがその変更を検知して自動的にGitの状態に修正（自己修復）することを確認せよ。

---

##### 5.2.3 Argo CD Image UpdaterによるCI/CDの自動化

**概要** : Argo CD Image Updaterは、CIパイプラインによってArtifact Registry にプッシュされた新しいコンテナイメージのタグを自動的に検知し、Gitリポジトリ内のKubernetesマニフェスト（Kustomizeの設定ファイルなど）のイメージタグを自動的に更新（Gitコミット）する機能を提供する。これにより、CI（ビルド、テスト、イメージプッシュ）とCD（デプロイ）が完全に自動連携し、手動でのマニフェスト変更が不要な、真のGitOpsパイプラインが完成する。

**具体例** : Argo CD Image Updaterの導入と、Applicationリソースへのアノテーション追加の例を示す。
1.  **Argo CD Image Updaterのインストール**
2.  **Applicationリソースにアノテーションを追加** Kustomizeの `kustomization.yaml` 内でイメージタグを管理している場合、以下のようにArgo CD Applicationにアノテーションを設定する。
    ```yaml
    apiVersion: argoproj.io/v1alpha1
    kind: Application
    metadata:
      name: go-api-app
      namespace: argocd
      annotations:
        argocd-image-updater.argoproj.io/image-list: go-api=asia-northeast1-docker.pkg.dev/your-project-id/my-go-api-repo/go-api
        argocd-image-updater.argoproj.io/go-api.update-strategy: latest
        argocd-image-updater.argoproj.io/write-back-method: git # gitに書き戻す
        argocd-image-updater.argoproj.io/git-branch: main # 更新対象ブランチ
    spec:
      project: default
      source:
        repoURL: https://github.com/your-org/your-k8s-manifests.git
        targetRevision: HEAD
        path: kubernetes/overlays/production
        kustomize:
          images:
            - name: asia-northeast1-docker.pkg.dev/your-project-id/my-go-api-repo/go-api # 更新対象イメージ名
              newTag: "v1.0.0" # Image Updaterが最初に参照するタグ（初期値）
      destination:
        server: https://kubernetes.default.svc
        namespace: my-app-prod
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
    ```
   
Gitへの書き込みに必要な認証情報（SSHキーまたはHTTPSトークン）は、Argo CDのSecretとして別途登録しておく必要がある。

**課題** : Argo CD Image Updaterを導入し、CI/CDパイプラインを完全に自動化せよ。
1.  **Argo CD Image Updaterのデプロイ** : Argo CDと同じNamespaceにArgo CD Image Updaterをデプロイせよ。
2.  **Git書き込み権限の設定** : Argo CD Image UpdaterがKubernetesマニフェストリポジトリにコミットできるように、Gitへの書き込み権限を持つSSHキーまたはHTTPSトークンをArgo CDにSecretとして登録せよ。
3.  **Applicationマニフェストの更新** :
    *   前節で作成したArgo CD Applicationマニフェストに、Argo CD Image Updaterが利用するためのアノテーションを追加せよ。
    *   `argocd-image-updater.argoproj.io/image-list` にGo APIのArtifact Registryパスとイメージ名を指定せよ。
    *   `argocd-image-updater.argoproj.io/<image-name>.update-strategy: latest` を設定し、Artifact Registryの最新タグを追従するようにせよ。
    *   `kustomize.images` フィールドで、Image Updaterが更新するイメージ名と初期タグを設定せよ。
4.  **動作検証** :
    *   Go APIのソースコードを修正し、GitHub Actionsによって新しいコンテナイメージがArtifact Registryにプッシュされることを確認せよ（章5.1.2）。
    *   数分後、Argo CD Image UpdaterがArtifact Registryの新しいタグを検知し、Kubernetesマニフェストリポジトリ（Git）に自動でコミットが生成されることを確認せよ。
    *   Argo CDがそのGitコミットの差分を検知し、Go APIアプリケーションが新しいイメージタグで自動的に更新（デプロイ）されることを確認せよ。
    *   Argo CD UIでアプリケーションの同期ステータスを確認し、Podのイメージが更新されていることを検証せよ。

---

**成果物** : GitOpsワークフローに基づき、GitHub ActionsとArgo CDによって完全に自動化されたCI/CDパイプラインが成果物となる。これにより、コードの変更から本番環境へのデプロイまでが自動化され、迅速かつ信頼性の高いソフトウェアデリバリーが可能となる。

---

### 章6: 可観測性と運用

**目的** : 本章の目的は、本番環境に耐えうるロギング、メトリクス、アラートシステムを構築し、運用知識を深めることである。これにより、システムの健全性を継続的に監視し、インシデント発生時に迅速な対応が可能となる基盤を確立する。

**学習内容** : 以下の要素を体系的に習得する。
*   **ログとメトリクスの統合監視** :
    *   Fluent Bitによるコンテナログ収集: 各ノードにDaemonSet としてデプロイし、コンテナログを収集してCloud Logging へ送信する。
    *   Prometheusによるメトリクス収集: StatefulSet として構築し、PodのCPU使用率、メモリ使用量、APIレイテンシーなどのメトリクスを収集する。必要に応じてGKE Managed Service for Prometheusの利用も検討する。
    *   Grafanaによるダッシュボード構築とアラート設定: Prometheusで収集したメトリクスをPromQL を使って可視化し、SLO/SLAに基づくアラートを設定する。
*   **GCP Cloud Monitoringとの連携** : Cloud Monitoring のアラートポリシーと通知設定。
*   **K8sの運用とデバッグ** : PodやNodeの障害時のデバッグ手法、イベントログの活用。

---

#### 6.1 ログとメトリクスの統合監視

##### 6.1.1 Fluent Bitによるコンテナログ収集

**概要** : GKE環境において、各コンテナから出力されるログは、アプリケーションの挙動把握、問題特定、デバッグに不可欠である。Fluent Bitは軽量なログプロセッサ・フォワーダであり、Kubernetes環境ではDaemonSet として各ノードにデプロイすることで、ノード上の全コンテナログを一元的に収集し、Cloud Logging のような中央集約型ログシステムへ転送することが標準的な手法である。これにより、分散環境におけるログ管理の複雑性を軽減し、検索・分析を可能にする。

**具体例** : Fluent BitをDaemonSetとしてデプロイする際のマニフェスト例を示す。これは、Kubernetesメタデータを付与し、Cloud Loggingへログを送信するための基本的な構成である。GCP上の認証には、適切なService AccountとIAM権限（例: `roles/logging.logWriter`）の付与が必要となる。
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluent-bit
  namespace: logging
  labels:
    k8s-app: fluent-bit
spec:
  selector:
    matchLabels:
      k8s-app: fluent-bit
  template:
    metadata:
      labels:
        k8s-app: fluent-bit
    spec:
      serviceAccountName: fluent-bit-sa
      tolerations:
      - key: node-role.kubernetes.io/master
        operator: Exists
        effect: NoSchedule
      containers:
      - name: fluent-bit
        image: fluent/fluent-bit:2.1.0 # 最新の安定バージョンを使用
        args:
          - -c
          - /fluent-bit/etc/fluent-bit.conf
        env:
          - name: GCP_PROJECT_ID
            value: your-gcp-project-id
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
        - name: fluentbit-config
          mountPath: /fluent-bit/etc/fluent-bit.conf
          subPath: fluent-bit.conf
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
      - name: fluentbit-config
        configMap:
          name: fluent-bit-config
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: fluent-bit-config
  namespace: logging
data:
  fluent-bit.conf: |
    [SERVICE]
        Flush        1
        Log_Level    info
        Daemon       off
        Parsers_File parsers.conf
        HTTP_Server  On
        HTTP_Listen  0.0.0.0
        HTTP_Port    2020

    [INPUT]
        Name             tail
        Path             /var/log/containers/*.log
        Parser           docker
        Tag              kube.*
        Exclude_Path     /var/log/containers/fluent-bit*.log,/var/log/containers/cloud-sql-proxy*.log

    [OUTPUT]
        Name        stackdriver
        Match       kube.*
        # resource allows setting the MonitoredResource for log entries.
        # Valid options are: kubernetes (default), gce_instance, global
        resource    kubernetes
        # labels adds additional labels to the log entries.
        # This is useful for adding custom metadata.
        labels      logging_agent=fluentbit
        # autoformat_stackdriver_trace allows Stackdriver Trace IDs
        # to be automatically extracted from log entries.
        autoformat_stackdriver_trace On
        # autoformat_stackdriver_severity allows Stackdriver Severity
        # to be automatically extracted from log entries.
        autoformat_stackdriver_severity On
        # project_id specifies the GCP project ID to which logs will be sent.
        # If not set, it will automatically detect from the environment.
        # project_id ${GCP_PROJECT_ID}
    [PARSER]
        Name        docker
        Format      json
        Time_Key    time
        Time_Format %Y-%m-%dT%H:%M:%S.%L%z
```


**課題** :
1.  **Fluent Bitのデプロイ** : 既存のGKEクラスタに、上記例を参考にFluent BitをDaemonSetとしてデプロイせよ。`logging` Namespaceを作成し、Fluent Bit用のKubernetes Service Account、およびCloud Loggingへの書き込み権限を持つGCP Service AccountをTerraformで作成し、Workload Identity を利用してこれらを紐付けよ。
2.  **ログの確認** : デプロイ後、Cloud LoggingコンソールにFluent Bitによって収集されたコンテナログが表示されることを確認せよ。特に、前章までにデプロイしたGo APIアプリケーションのログが正しく収集されているか検証し、エラーログなどが適切に識別されているか確認せよ。
3.  **カスタムログパース** : Go APIのログ出力形式がJSONの場合、`parsers.conf`を調整し、Fluent BitがJSON形式のログを適切にパースし、Cloud Loggingに構造化ログとして取り込むように設定せよ。これにより、Cloud Loggingでの検索・分析が容易になることを確認せよ。

---

##### 6.1.2 Prometheusによるメトリクス収集

**概要** : システムの健全性やパフォーマンスを数値的に把握するためには、メトリクス収集が不可欠である。Prometheusはオープンソースの監視システムであり、Pull型でターゲットからメトリクスを収集する。Kubernetes環境では、Prometheus Operatorを用いてデプロイすることが一般的であり、ServiceMonitorやPodMonitorといったCRD (Custom Resource Definition) を活用することで、Kubernetesリソースのメトリクスを効率的に収集できる。PodのCPU使用率、メモリ使用量、ネットワークI/O、アプリケーションが公開するカスタムメトリクス（例: APIレイテンシー、エラーレート）などを収集することで、システム全体の状態を詳細に把握することが可能となる。GKE環境においては、GKE Managed Service for Prometheusの利用も検討することで、Prometheusサーバ自体の運用負荷を軽減できる。

**具体例** : Prometheus Operatorをデプロイし、Go APIアプリケーションのメトリクスを収集するためのServiceMonitorを定義する。Go APIには`/metrics`エンドポイントでPrometheus形式のメトリクスを公開するよう実装が必要となる。
Go APIのServiceMonitorマニフェスト例:
```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: go-api-servicemonitor
  namespace: my-app-ns # Go APIがデプロイされているNamespace
  labels:
    release: prometheus-stack # Prometheus OperatorのHelmリリース名に合わせる
spec:
  selector:
    matchLabels:
      app: go-api # Go APIのPodのラベル
  endpoints:
  - port: http-metrics # ServicePort名
    path: /metrics # Prometheusメトリクスエンドポイント
    interval: 30s
  namespaceSelector:
    matchNames:
      - my-app-ns
```

PrometheusをStatefulSet として構築し、PersistentVolume とPersistentVolumeClaim を用いて時系列データを永続化する設定も必要となるが、ここではPrometheus Operatorがそれらを管理する。

**課題** :
1.  **Prometheus Operatorのデプロイ** : 既存のGKEクラスタにPrometheus Operatorをデプロイせよ。Helm Chartを利用することが推奨される。Prometheus本体の永続化設定（ストレージクラス、容量）を適切に定義せよ。
2.  **Go APIのメトリクス公開** : 前章までに作成したGo APIアプリケーションに、Prometheus互換のメトリクスエンドポイント（例: `/metrics`）を実装せよ。APIのリクエスト数、レイテンシー、エラーレート、Podのメモリ使用量などをメトリクスとして公開できるよう、GoのPrometheusクライアントライブラリを利用せよ。
3.  **ServiceMonitorの作成** : Go APIアプリケーションからメトリクスを収集するためのServiceMonitorリソースを作成し、Prometheusがメトリクスを収集していることをPrometheus UI (通常は`http://<Prometheus_IP>:9090/graph`) の「Status」→「Targets」ページで確認せよ。また、PromQLを用いて基本的なメトリクスをクエリできることを検証せよ。
4.  **GKE Managed Service for Prometheusの検討** : Prometheusの運用・管理コストを削減するため、GKE Managed Service for Prometheusへの移行可能性について調査し、そのメリット・デメリットを整理せよ。特に、既存のPromQLクエリやGrafanaダッシュボードとの互換性に焦点を当てて評価せよ。

---

##### 6.1.3 Grafanaによるダッシュボード構築とアラート設定

**概要** : 収集されたメトリクスは、Grafanaのような可視化ツールを用いることで、システムの状況を一目で把握できるダッシュボードとして表現され、運用における意思決定を支援する。GrafanaはPrometheusをデータソースとして利用し、PromQL を使って柔軟なクエリを記述することで、多角的な視点からメトリクスを分析できる。さらに、SLO (Service Level Objective) やSLA (Service Level Agreement) に基づくアラートを設定することで、サービスの異常を早期に検知し、対応を自動化または通知することが可能となる。

**具体例** : GrafanaをGKEクラスタにデプロイし、Prometheusをデータソースとして追加する。Ingressを通じてGrafana UIを公開することも可能である。
GrafanaのデプロイとPrometheusデータソース設定 (ConfigMap経由):
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      containers:
      - name: grafana
        image: grafana/grafana:9.4.7 # 最新の安定バージョンを使用
        ports:
        - containerPort: 3000
        env:
        - name: GF_PATHS_CONFIG
          value: /etc/grafana/grafana.ini
        volumeMounts:
        - name: grafana-storage
          mountPath: /var/lib/grafana
        - name: grafana-config
          mountPath: /etc/grafana/provisioning/datasources/prometheus.yaml
          subPath: prometheus.yaml
      volumes:
      - name: grafana-storage
        persistentVolumeClaim:
          claimName: grafana-pvc
      - name: grafana-config
        configMap:
          name: grafana-datasource-config
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: grafana-pvc
  namespace: monitoring
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: grafana-datasource-config
  namespace: monitoring
data:
  prometheus.yaml: |
    apiVersion: 1
    datasources:
      - name: Prometheus
        type: prometheus
        url: http://prometheus-operated.monitoring.svc.cluster.local:9090 # Prometheus OperatorがデプロイするPrometheus Service
        isDefault: true
        access: proxy
        version: 1
        editable: true
```


**課題** :
1.  **Grafanaのデプロイと公開** : 既存のGKEクラスタにGrafanaをデプロイせよ。Helm Chartを利用するか、上記マニフェストを参考にデプロイし、永続化ストレージ（PersistentVolumeClaim）も設定せよ。章3で学習したIngress とGCLB を用いて、Grafana UIを安全に外部公開せよ。
2.  **Prometheusデータソースの設定** : GrafanaからPrometheusにアクセスできるよう、データソースを設定せよ。上記ConfigMapを利用し、設定がコード化されていることを確認せよ。
3.  **ダッシュボードの作成** : Go APIアプリケーションの主要なメトリクス（リクエスト数、成功率、レイテンシー、エラーレート、CPU/メモリ使用率など）を可視化するGrafanaダッシュボードを作成せよ。SLO/SLAを満たしているかを一目で判断できるようなグラフをPromQLを用いて作成し、複数のパネルで構成せよ。
4.  **アラート設定** : Go APIのエラーレートが閾値（例: 5%を5分間継続）を超過した場合や、レイテンシーが特定の値（例: P99レイテンシーが200msを1分間継続）を超過した場合に通知が発報されるよう、Grafanaアラートを設定せよ。通知先として、SlackやPagerDutyなどの通知チャネルを設定し、実際にアラートが発報されることを検証せよ。

---

#### 6.2 GCP Cloud Monitoringとの連携

**概要** : GCP Cloud Monitoring は、GCPリソースやアプリケーションから収集されたメトリクス、ログ、トレースを一元的に管理し、可視化、モニタリング、アラート機能を提供するフルマネージドサービスである。K8s環境においては、Fluent BitによってCloud Loggingに転送されたログや、Prometheusのメトリクス（GKE Managed Service for Prometheusを利用した場合）がCloud Monitoringに統合され、より広範なGCPインフラ全体の監視と組み合わせることが可能となる。Cloud Monitoringのアラートポリシーと通知設定を活用することで、GCPネイティブなアラートシステムを構築し、インシデント対応プロセスに組み込むことができる。

**具体例** : Cloud Monitoringでカスタムメトリクスに基づくアラートポリシーを設定し、通知チャネルとしてSlackやPagerDutyを設定する例。
例えば、Cloud Loggingに特定のキーワード（`severity: ERROR`や`message:"failed to connect DB"`など）を含むログが一定時間内に指定回数以上発生した場合にアラートを発報するログベースのメトリクスとアラートポリシーをTerraformで設定できる。

Terraformでのアラートポリシー定義例:
```terraform
resource "google_monitoring_metric_descriptor" "go_api_error_count" {
  metric_kind = "GAUGE"
  value_type  = "INT64"
  type        = "custom.googleapis.com/go_api/error_count"
  description = "Count of Go API application errors from logs."
  display_name = "Go API Error Count"
  labels {
    key         = "severity"
    value_type  = "STRING"
    description = "Log severity"
  }
}

resource "google_monitoring_alert_policy" "go_api_error_alert" {
  display_name = "Go API High Error Rate Alert"
  combiner     = "OR" # ORまたはAND
  conditions {
    display_name = "High error logs detected"
    condition_threshold {
      filter     = "resource.type=\"k8s_container\" AND metric.type=\"logging.googleapis.com/user/go_api_error_count\" AND resource.labels.project_id=\"${google_project.current.project_id}\""
      duration   = "300s" # 5分間
      comparison = "COMPARISON_GT" # Greater than
      threshold_value = 5 # 5回以上
      aggregations {
        alignment_period   = "60s"
        per_series_aligner = "ALIGN_RATE" # ログをレートとしてカウント
      }
    }
  }
  notification_channels = [
    google_monitoring_notification_channel.slack_channel.id # Slackチャンネルなど
  ]
  enabled = true
}

resource "google_monitoring_notification_channel" "slack_channel" {
  display_name = "Slack Notifications for Alerts"
  type         = "slack"
  labels = {
    channel_name = "#your-slack-channel"
    auth_token = "your-slack-webhook-url" # 実際の認証トークンはSecret Managerなどから取得
  }
  description = "Slack channel for critical alerts."
}
```


**課題** :
1.  **ログベースのメトリクス作成** : Go APIアプリケーションが特定のキーワード（例: `ERROR`レベルのログ、`Database connection failed`メッセージ）で出力するログを検知し、Cloud Monitoring上でカスタムメトリクスとしてカウントアップするログベースのメトリクスをTerraformで作成せよ。
2.  **アラートポリシーの作成と通知設定** : 作成したログベースのメトリクス、またはPrometheusで収集しGKE Managed Service for Prometheus経由でCloud Monitoringに連携されたメトリクス（例: `kubernetes.io/container/cpu/utilization`）を元に、Cloud MonitoringでアラートポリシーをTerraformで定義せよ。通知先として、Slack、メール、またはPagerDutyなどのチャネルを設定し、実際にアラートが発報されることを検証せよ。アラートがクリアされた際の通知も確認すること。
3.  **SLA/SLOとの連携** : 前章で定義したSLO/SLA（例: 99.9%の可用性、P99レイテンシー200ms以内）を満たしているかをCloud Monitoringのダッシュボードで確認できるよう、カスタムダッシュボードをTerraformで作成し、アラートポリシーがこれらの目標と連動するように調整せよ。特に、SLA違反に直結するような重要メトリクスをアラート条件に含めること。

---

#### 6.3 K8sの運用とデバッグ

**概要** : GKEクラスタでマイクロサービスを運用する上で、障害発生時の迅速なデバッグと原因特定は不可欠である。PodやNodeの障害は多様な原因で発生し、その診断にはKubernetesの組み込みツールやGCPのモニタリングツールを効果的に活用する知識が求められる。特に、`kubectl describe`、`kubectl logs`、`kubectl events`などのコマンドや、Cloud Loggingで収集されたイベントログは、コンテナの再起動ループ、Podスケジューリングの問題、リソース枯渇、ネットワークポリシーの問題などを特定する上で重要な情報源となる。これらの運用知識は、SRE (Site Reliability Engineering) の実践に直結する。

**具体例** : 特定のPodが起動しない、または頻繁に再起動を繰り返す場合のデバッグ手順の例。この手順は、障害の根本原因を特定し、解決策を導き出すための体系的なアプローチである。
1.  **Podのステータス確認** : `kubectl get pod <pod-name> -n <namespace>` でステータスを確認する。`CrashLoopBackOff`、`Evicted`、`ImagePullBackOff`、`Pending`などの状態は、それぞれ異なる問題を示唆する。
2.  **Podの詳細情報確認** : `kubectl describe pod <pod-name> -n <namespace>` で詳細情報を確認する。特に以下の項目に注目する。
    *   **Events** : Podのライフサイクル中に発生した重要なイベント（スケジューリング、コンテナ起動・停止、ボリュームマウントなど）が表示される。エラーや警告がここに示されることが多い。
    *   **Containers** : 各コンテナの状態（Ready, Running, Terminated）、再起動回数、終了コード、リソース要求・制限を確認する。
    *   **Controlled By** : 親リソース（Deployment, DaemonSet, StatefulSetなど）を確認する。
3.  **コンテナログの確認** : `kubectl logs <pod-name> -n <namespace> -c <container-name>` でアプリケーションログやコンテナの起動ログを確認する。`--previous`オプションで、コンテナが再起動した場合の直前のログを確認できる。
4.  **Kubernetesイベントログの確認** : `kubectl get events -n <namespace> --sort-by='.lastTimestamp'` で特定のNamespace内のイベントログを時系列で確認する。クラスタ全体のイベントは `kubectl get events --all-namespaces`。これにより、Podの再起動理由やNodeの状態変化、PersistentVolumeClaim の問題などを把握できる。
5.  **ノードのリソース確認** : `kubectl describe node <node-name>` でノードのリソース状況（CPU, Memory, Diskの使用量と空き容量）、Podの割り当てなどを確認する。ノードがリソース不足の場合、Podが`Evicted`される可能性がある。Cloud MonitoringでノードのCPU/メモリ使用率をグラフで確認することも有効である。
6.  **ネットワークポリシーの確認** : `kubectl get networkpolicy -n <namespace>` で適用されているネットワークポリシー を確認し、`kubectl describe networkpolicy <policy-name>` で詳細を見る。アプリケーション間の通信が意図せずブロックされていないか確認する。
7.  **Cloud Loggingでの詳細分析** : クラスタから収集されたログはCloud Loggingに集約されているため、より広範な期間でのログ検索、フィルタリング、エクスポートが可能である。特定のPodやコンテナのログを詳細に分析し、アプリケーションレベルの問題を特定する。

**課題** :
1.  **Podの障害シミュレーションとデバッグ** :
    *   意図的にGo APIアプリケーションのPodに障害を発生させるシミュレーションを実施せよ。例として、以下のいずれかを実行し、その後元の状態に戻せることを確認せよ:
        *   アプリケーションに無限ループやパニックを発生させるコードを注入し、`CrashLoopBackOff`状態を誘発する。
        *   DeploymentマニフェストのConfigMap名を誤ったものに変更し、アプリケーションが起動に失敗する状況を作り出す。
        *   Podのリソースリミットを極端に低く設定し、アプリケーションがOOMKilledされる状況を再現する。
    *   これらの状況において、上記「具体例」で示した`kubectl`コマンドとCloud Loggingを駆使し、障害の根本原因を特定する手順を詳細に記述せよ。
    *   特定した原因に基づき、問題を解決する手順を記述し、実際に修正を適用してPodが正常に稼働することを確認せよ。
2.  **Nodeの障害シミュレーションと影響分析** :
    *   GKEクラスタの特定のNodeを手動でドレイン (`kubectl drain <node-name>`) または停止するシミュレーションを行い、そのNode上で稼働していたGo API Podがどのように他のNodeに再スケジュールされるか、サービスの可用性に影響がないかを確認せよ。
    *   この過程で発生するKubernetesイベントやCloud Loggingの監査ログを分析し、GKEの自動復旧メカニズムと、アプリケーションのHA (High Availability) 設定（Horizontal Pod Autoscaler (HPA) やPod Disruption Budget (PDB) など）がどのように機能したかをレポートせよ。
3.  **リソース枯渇の検知と対策** :
    *   Go APIアプリケーションのPodに意図的に低いリソースリクエスト/リミットを設定し、高負荷時にPodがOOMKilled（Out Of Memory Killed）される状況を再現せよ。これには、負荷テストツール（例: Apache Bench, locust）を用いてGo APIに高負荷をかけよ。
    *   この現象をCloud Monitoringのメトリクス（例: `kubernetes.io/container/memory/usage`）とKubernetesイベントから検知し、適切なリソースリクエスト/リミットの値を決定し、Deploymentマニフェストを修正せよ。修正後、再度負荷テストを行い、OOMKilledが発生しないことを確認せよ。

---

**成果物** : Cloud Logging、Prometheus、Grafanaを用いた統合監視システムが構築されたGKE環境、および運用に必要なダッシュボードとアラート設定。

---

### 章7: マイクロサービスとサービスメッシュ

**目的** : 本章の目的は、GKE（Google Kubernetes Engine）上でマイクロサービスアーキテクチャを実際に構築し、サービスメッシュの利点を理解し活用することである。これにより、複雑な分散システムにおける通信制御、セキュリティ、可観測性を高めるための実践的なスキルを習得する。

**学習内容** : 以下の要素を体系的に習得する。
*   **Go APIによるマイクロサービスの設計と実装** : 複数のGo APIサービスを連携させ、マイクロサービス間の効率的かつ堅牢な通信を設計・実装する。
*   **Istioの導入と活用** :
    *   GKEクラスタへのIstio のデプロイ。
    *   Istio GatewayとVirtualServiceを用いたL7ルーティングを実践し、ドメイン、パス、ヘッダーに基づくきめ細かいトラフィック制御を学ぶ。具体的には、カナリアリリースやA/Bテストといった高度なデプロイ戦略を実現する。
    *   Istioを用いたmTLSによるサービス間通信の暗号化。
    *   Istioによるサービストポロジーの可視化とメトリクス収集。
*   **GKE IngressとIstioの連携パターン** : 外部からのリクエストをGCP Load Balancer で受け、Istio Ingress Gateway へルーティングする構成を構築する (GCP Load Balancer とIstioの高度なトラフィック管理機能を連携)。

---

#### 7.1 Go APIによるマイクロサービスの設計と実装

**概要** : 現代のウェブサービス開発において、モノリシックなアプリケーションからマイクロサービスアーキテクチャへの移行は、スケーラビリティ、開発速度、耐障害性の向上に寄与する。本節では、複数のGo APIサービスを連携させる基本的なマイクロサービス設計を実践する。各サービスは単一の責任を持ち、明確なAPIインターフェースを通じて通信する。これにより、独立した開発・デプロイが可能となり、システム全体の柔軟性が向上する。

**具体例** : ここでは、シンプルなECサイトのバックエンドを想定し、「Product Service (商品情報管理)」と「Order Service (注文処理)」の2つのGo APIサービスを例に挙げる。Order Serviceが注文を作成する際に、Product Serviceから商品の詳細情報を取得するシナリオを実装する。
**Product Service (product-api)** `main.go` (一部抜粋):
```go
package main

import (
	"log"
	"net/http"
	"encoding/json"
	"fmt"
)

type Product struct {
	ID    string `json:"id"`
	Name  string `json:"name"`
	Price int    `json:"price"`
}

var products = map[string]Product{
	"p001": {ID: "p001", Name: "Laptop", Price: 1200},
	"p002": {ID: "p002", Name: "Mouse", Price: 25},
}

func getProductHandler(w http.ResponseWriter, r *http.Request) {
	id := r.URL.Path[len("/products/"):]
	product, ok := products[id]
	if !ok {
		http.Error(w, "Product not found", http.StatusNotFound)
		return
	}
	json.NewEncoder(w).Encode(product)
}

func healthCheckHandler(w http.ResponseWriter, r *http.Request) {
    w.WriteHeader(http.StatusOK)
    fmt.Fprint(w, "OK")
}

func main() {
	http.HandleFunc("/products/", getProductHandler)
    http.HandleFunc("/healthz", healthCheckHandler)
    http.HandleFunc("/ready", healthCheckHandler)
	log.Fatal(http.ListenAndServe(":8080", nil))
}
```

**Order Service (order-api)** `main.go` (一部抜粋):
```go
package main

import (
	"bytes"
	"encoding/json"
	"fmt"
	"log"
	"net/http"
)

type Order struct {
	OrderID    string `json:"orderId"`
	ProductID  string `json:"productId"`
	Quantity   int    `json:"quantity"`
	TotalPrice int    `json:"totalPrice"`
}

type Product struct {
	ID    string `json:"id"`
	Name  string `json:"name"`
	Price int    `json:"price"`
}

const productServiceURL = "http://product-api-service-internal:80/products/" # Kubernetes Internal Service Name

func createOrderHandler(w http.ResponseWriter, r *http.Request) {
	if r.Method != "POST" {
		http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
		return
	}

	var orderRequest struct {
		ProductID string `json:"productId"`
		Quantity  int    `json:"quantity"`
	}
	if err := json.NewDecoder(r.Body).Decode(&orderRequest); err != nil {
		http.Error(w, err.Error(), http.StatusBadRequest)
		return
	}

	// Call Product Service to get product details
	resp, err := http.Get(productServiceURL + orderRequest.ProductID)
	if err != nil {
		http.Error(w, fmt.Sprintf("Failed to get product details: %v", err), http.StatusInternalServerError)
		return
	}
	defer resp.Body.Close()

	if resp.StatusCode != http.StatusOK {
		http.Error(w, "Product service returned an error", resp.StatusCode)
		return
	}

	var product Product
	if err := json.NewDecoder(resp.Body).Decode(&product); err != nil {
		http.Error(w, fmt.Sprintf("Failed to decode product details: %v", err), http.StatusInternalServerError)
		return
	}

	newOrder := Order{
		OrderID:    fmt.Sprintf("order-%d", len(orderHistory)+1), // Simple order ID generation
		ProductID:  product.ID,
		Quantity:   orderRequest.Quantity,
		TotalPrice: product.Price * orderRequest.Quantity,
	}
	orderHistory = append(orderHistory, newOrder)

	w.WriteHeader(http.StatusCreated)
	json.NewEncoder(w).Encode(newOrder)
}

var orderHistory []Order

func healthCheckHandler(w http.ResponseWriter, r *http.Request) {
    w.WriteHeader(http.StatusOK)
    fmt.Fprint(w, "OK")
}

func main() {
	http.HandleFunc("/orders", createOrderHandler)
    http.HandleFunc("/healthz", healthCheckHandler)
    http.HandleFunc("/ready", healthCheckHandler)
	log.Fatal(http.ListenAndServe(":8080", nil))
}
```

両サービスはポート8080でリッスンするように設定する。Product Serviceは `/products/{id}` で商品情報を提供し、Order Serviceは `/orders` へのPOSTリクエストで注文を作成し、その際に内部でProduct Serviceを呼び出す。

**課題** :
1.  **マイクロサービスの設計と実装** :
    *   上記例を参考に、Go言語で2つ以上の独立したマイクロサービスを設計し、実装せよ。各サービスは最低1つのHTTPエンドポイントを持ち、少なくとも1つのサービスが他のサービスのエンドポイントを呼び出すようにせよ。例えば、`user-api`と`auth-api`、または`payment-api`と`invoice-api`といった組み合わせでも良い。
    *   各サービスのGoモジュールを初期化し、必要に応じて依存関係を管理せよ。
2.  **Dockerfileの作成** :
    *   章2で学習したマルチステージビルドとセキュリティベストプラクティス に基づき、各Goマイクロサービス用のDockerfileを作成せよ。軽量なベースイメージ（`gcr.io/distroless/static` または `alpine`）を使用し、最終イメージサイズを最小限に抑えよ。
    *   作成したDockerfileを使って、各マイクロサービスのDockerイメージをローカルでビルドし、正しく動作することを確認せよ。
3.  **Kubernetesマニフェストの準備** :
    *   各Goマイクロサービス（例: `product-api`, `order-api`）のDeploymentとClusterIP ServiceのマニフェストをKustomizeの`base`ディレクトリに用意せよ。ポートは8080を使用すること。
    *   章3で学習したLiveness ProbeとReadiness Probeを適切に設定せよ。`healthz`エンドポイントなどを活用せよ。

---

#### 7.2 Istioの導入と活用

**概要** : Istioは、サービスメッシュとしてKubernetes上で動作するアプリケーションのトラフィック管理、セキュリティ、可観測性を向上させるためのオープンソースプラットフォームである。サイドカーパターンを用いてEnvoyプロキシを各Podに自動的にインジェクトし、サービス間の通信を透過的に制御する。本節では、Istioのデプロイ、トラフィックルーティング、セキュリティ機能（mTLS）、そして可観測性機能の活用方法を学ぶ。

##### 7.2.1 GKEクラスタへのIstioのデプロイ

**概要** : Istioは、KubernetesクラスタにControl Plane（Pilot, Citadel, Galley, Mixerなど）とData Plane（Envoyプロキシ）を展開することで機能する。Control PlaneはルーティングルールやポリシーをEnvoyプロキシに配布し、EnvoyプロキシはPodのサイドカーとしてすべてのネットワークトラフィックをインターセプト・処理する。本節では、`istioctl` CLIツールを用いてGKEクラスタにIstioをデプロイする。

**具体例** : Istioのインストールは、公式推奨の`istioctl`コマンドが最も確実である。 まず、Istioのリリースをダウンロードして展開する。
```bash
curl -L https://istio.io/downloadIstio | sh -
cd istio-*-*
export PATH=$PWD/bin:$PATH
```

次に、GKEクラスタにIstioをインストールする。本番環境を想定したプロファイル`default`を使用する。
```bash
istioctl install --set profile=default -y
```

アプリケーションをデプロイするNamespaceにIstioのサイドカー自動注入を有効にする。
```bash
kubectl label namespace my-app-ns istio-injection=enabled --overwrite
```

このラベルが付与されたNamespaceにPodをデプロイすると、自動的にEnvoyプロキシのサイドカーコンテナがPodに追加される。

**課題** :
1.  **Istioのデプロイ** :
    *   既存のGKEクラスタに、`istioctl`コマンドを用いてIstioをデプロイせよ。プロファイルは`default`を使用すること。
    *   デプロイ後、`kubectl get pods -n istio-system`コマンドでIstio Control PlaneのPodがすべて`Running`状態であることを確認せよ。
    *   `istioctl verify-install`コマンドを実行し、Istioのインストールが正常に完了していることを確認せよ。
2.  **Namespaceへの自動注入有効化** :
    *   章7.1で準備したGoマイクロサービスをデプロイするNamespace（例: `my-app-ns`）を作成し、Istioのサイドカー自動注入を有効にするラベルを付与せよ。
    *   このNamespaceにGoマイクロサービス（`product-api`, `order-api`）のDeploymentとServiceをデプロイせよ。
    *   デプロイ後、`kubectl get pods -n my-app-ns`コマンドで各アプリケーションPodに2つのコンテナ（アプリケーションコンテナとEnvoyサイドカー）があることを確認せよ。
    *   `kubectl describe pod <pod-name> -n my-app-ns`で、Envoyプロキシが注入されていることを詳細情報から確認せよ。

---

##### 7.2.2 Istio GatewayとVirtualServiceによるL7ルーティングと高度なデプロイ戦略

**概要** : Istio Gatewayは、外部からのトラフィックをサービスメッシュに導入するためのエントリポイントであり、KubernetesのIngressとは異なり、メッシュ内の詳細なルーティング制御と連携する。VirtualServiceは、指定されたホスト名やパス、HTTPヘッダー、トラフィックの重み付けなど、レイヤー7（L7）レベルのきめ細かいルーティングルールを定義するために使用される。これにより、カナリアリリース、A/Bテスト、ミラーリングといった高度なデプロイ戦略を容易に実装できる。

**具体例** : **Go API v1のデプロイ** : 章7.1で作成したGo APIマイクロサービス（`product-api`, `order-api`）をデプロイする。
`product-api-v1-deployment.yaml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: product-api-v1
  namespace: my-app-ns
spec:
  replicas: 2
  selector:
    matchLabels:
      app: product-api
      version: v1
  template:
    metadata:
      labels:
        app: product-api
        version: v1
    spec:
      containers:
      - name: product-api
        image: asia-northeast1-docker.pkg.dev/your-project-id/my-go-api-repo/product-api:v1.0.0
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: product-api-service
  namespace: my-app-ns
spec:
  selector:
    app: product-api
  ports:
  - port: 80
    targetPort: 8080
```

`order-api`も同様にデプロイする。

**Istio Gatewayの定義** : 外部からのトラフィックを受け入れるエントリポイントを定義する。
`gateway.yaml`:
```yaml
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: my-app-gateway
  namespace: my-app-ns
spec:
  selector:
    istio: ingressgateway # デフォルトのIstio Ingress Gatewayを使用
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*" # 任意のホスト名を受け入れる（または特定のドメインを指定）
  - port:
      number: 443
      name: https
      protocol: HTTPS
    hosts:
    - "*"
    tls:
      mode: SIMPLE # または MUTUAL, ISTIO_MUTUAL
      credentialName: my-app-cert # GKE Ingressと連携する場合、ここでGCP LBが終端するため不要な場合が多い
```


**VirtualServiceによるルーティング** :`product-api`サービスへのトラフィックをルーティングする。
`product-api-virtualservice.yaml`:
```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: product-api-vs
  namespace: my-app-ns
spec:
  hosts:
  - "*" # または実際のドメイン名
  gateways:
  - my-app-gateway # 上で定義したGateway
  http:
  - match:
    - uri:
        prefix: /products
    route:
    - destination:
        host: product-api-service # product-apiのService名
        subset: v1 # DestinationRuleで定義するサブセット
      weight: 100
```

同様に`order-api`用のVirtualServiceも定義する。

**カナリアリリース（重み付けルーティング）** :`product-api`のv1とv2が存在し、v2に少しずつトラフィックを流す。
`product-api-v2-deployment.yaml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: product-api-v2
  namespace: my-app-ns
spec:
  replicas: 1
  selector:
    matchLabels:
      app: product-api
      version: v2
  template:
    metadata:
      labels:
        app: product-api
        version: v2
    spec:
      containers:
      - name: product-api
        image: asia-northeast1-docker.pkg.dev/your-project-id/my-go-api-repo/product-api:v2.0.0
        ports:
        - containerPort: 8080
```

`product-api-virtualservice-canary.yaml` (VirtualServiceの更新):
```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: product-api-vs
  namespace: my-app-ns
spec:
  hosts:
  - "*"
  gateways:
  - my-app-gateway
  http:
  - match:
    - uri:
        prefix: /products
    route:
    - destination:
        host: product-api-service
        subset: v1
      weight: 90 # v1に90%のトラフィック
    - destination:
        host: product-api-service
        subset: v2
      weight: 10 # v2に10%のトラフィック
```

このVirtualServiceを適用するためには、DestinationRuleで`product-api`サービスのサブセットを定義する必要がある。
`product-api-destinationrule.yaml`:
```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: product-api-destination
  namespace: my-app-ns
spec:
  host: product-api-service
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
```


**A/Bテスト（ヘッダーベースルーティング）** : 特定ヘッダーを含むリクエストのみをv2にルーティング。
`product-api-virtualservice-abtest.yaml` (VirtualServiceの更新):
```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: product-api-vs
  namespace: my-app-ns
spec:
  hosts:
  - "*"
  gateways:
  - my-app-gateway
  http:
  - match: # ヘッダー "x-test-user: true" を持つリクエストをv2に
    - uri:
        prefix: /products
      headers:
        x-test-user:
          exact: true
    route:
    - destination:
        host: product-api-service
        subset: v2
  - match: # それ以外のリクエストは全てv1に
    - uri:
        prefix: /products
    route:
    - destination:
        host: product-api-service
        subset: v1
```


**課題** :
1.  **Istio GatewayとVirtualServiceの初期設定** :
    *   章7.1でデプロイしたGoマイクロサービスに対し、Istio GatewayとVirtualServiceをそれぞれ作成し、適用せよ。Ingress Gateway経由で各サービスに外部からアクセスできることを確認せよ。
    *   `kubectl get gateway -n my-app-ns`, `kubectl get virtualservice -n my-app-ns`でリソースが作成されていることを確認せよ。
    *   Istio Ingress Gatewayの外部IPアドレス（`kubectl get svc istio-ingressgateway -n istio-system`）を確認し、`curl`コマンドでGo APIにアクセスできることを検証せよ。
2.  **カナリアリリース（重み付けルーティング）の実践** :
    *   `product-api`（または他の任意のサービス）のバージョンv2を新たに作成し、Dockerfileのイメージタグを`v2.0.0`などに更新してArtifact Registryにプッシュせよ。
    *   `product-api`のv1とv2両方のDeploymentが稼働している状態で、DestinationRuleを定義し、各バージョンのサブセットを明確にせよ。
    *   VirtualServiceを更新し、v1に90%、v2に10%のトラフィックをルーティングするカナリアリリースを実装せよ。
    *   ループでアクセスツール（例: `for i in $(seq 1 100); do curl http://<Ingress_Gateway_IP>/products/p001; done`）を用いて繰り返しアクセスし、v1とv2からのレスポンスの比率が設定通りになっていることを確認せよ。
3.  **A/Bテスト（ヘッダーベースルーティング）の実践** :
    *   VirtualServiceを更新し、`x-test-user: true`ヘッダーを持つリクエストのみをv2にルーティングし、それ以外のトラフィックは全てv1にルーティングするA/Bテストを設定せよ。
    *   ヘッダーを付与した場合としない場合で、それぞれ期待通りのバージョンのサービスにアクセスされることを`curl -H "x-test-user: true" http://<Ingress_Gateway_IP>/products/p001`などで検証せよ。

---

##### 7.2.3 Istioを用いたmTLSによるサービス間通信の暗号化

**概要** : サービスメッシュにおけるセキュリティは極めて重要である。Istioは、サービス間通信のmTLS（相互TLS認証）を透過的に提供することで、メッシュ内のすべてのトラフィックを暗号化し、サービス間の認証を強化する。これにより、中間者攻撃や認証情報の漏洩リスクを大幅に低減できる。mTLSは、PeerAuthenticationリソースを用いてNamespace全体またはサービス単位で設定でき、段階的に導入できるよう`PERMISSIVE`（両方許可）と`STRICT`（mTLS必須）のモードが用意されている。

**具体例** : **mTLSをPermissiveモードで有効化** : これは、既存のサービスがmTLSをサポートしていない場合でも通信を許可しつつ、mTLSをサポートするサービス間ではmTLSを使用する設定である。移行期間中に有用。
```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: my-app-ns # アプリケーションがデプロイされているNamespace
spec:
  mtls:
    mode: PERMISSIVE
```

**mTLSをStrictモードで有効化** : このモードでは、すべてのサービス間通信がmTLSを使用することを強制する。これにより、メッシュ内のセキュリティが最大化される。
`peerauthentication-strict.yaml`:
```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: my-app-ns
spec:
  mtls:
    mode: STRICT
```

DestinationRuleを使用して、クライアントサイドのmTLS設定を制御することもできる。例えば、特定のサービスに対してmTLSを有効にする。
```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: product-api-destination
  namespace: my-app-ns
spec:
  host: product-api-service
  trafficPolicy:
    tls:
      mode: ISTIO_MUTUAL # このサービスへの通信はmTLSを強制
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
```


**課題** :
1.  **mTLSのPermissiveモード導入** :
    *   Goマイクロサービスがデプロイされている`my-app-ns`Namespaceに対し、PeerAuthenticationリソースを用いてmTLSを`PERMISSIVE`モードで有効化せよ。
    *   この状態で`order-api`から`product-api`へのサービス間通信が正常に行われることを検証せよ。同時に、非Istio Pod（Envoyサイドカーが注入されていないPod）から`product-api`へのアクセスも可能であることを確認せよ。
2.  **mTLSのStrictモード強制** :
    *   PeerAuthenticationを更新し、mTLSモードを`STRICT`に設定せよ。
    *   再度、`order-api`から`product-api`へのサービス間通信が正常に行われることを検証せよ。
    *   非Istio Podからのアクセスがブロックされることを確認し、mTLSが強制されていることを理解せよ。
3.  **mTLS状態の確認** :
    *   Istioが提供するKialiダッシュボード（次の章でデプロイ）などの可視化ツールを用いて、サービス間の通信がmTLSによって保護されていることを視覚的に確認せよ。または、`istioctl proxy-status`やEnvoyのメトリクスを調査し、mTLS接続が確立されていることを確認せよ。

---

##### 7.2.4 Istioによるサービストポロジーの可視化とメトリクス収集

**概要** : Istioは、サービスメッシュ内のすべてのトラフィックをEnvoyプロキシを通じてルーティングするため、自動的に豊富なテレメトリーデータを収集する。これにより、サービス間の依存関係、トラフィックの流れ、レイテンシー、エラーレートといったサービストポロジーとメトリクスを可視化できる。Kialiのようなツールは、Istioが収集したデータを基にサービスグラフを生成し、PrometheusやGrafanaといった監視ツールと連携して、システム全体の健全性とパフォーマンスを詳細に把握することを可能にする。

**具体例** : KialiはIstioと共にデプロイされることが多く、サービスメッシュの可視化に特化している。
Kialiのインストール例（`istioctl`経由）：
```bash
istioctl dashboard kiali
```

Kiali UIへのアクセス例:
```bash
kubectl port-forward svc/kiali -n istio-system 20001:20001
```

Kialiダッシュボードでは、以下のような可視化が可能となる。
*   **サービスグラフ** : サービス間の呼び出し関係、トラフィック量、エラー率、レイテンシーなどを視覚的に表示。
*   **ワークロード** : 各DeploymentやPodの状態、バージョンごとのトラフィック比率。
*   **アプリケーション** : 特定のアプリケーションに焦点を当てたメトリクスとグラフ。
*   **トレース** : Jaegerとの連携により、リクエストが複数のサービスをまたいでどのように処理されたかを確認。

**課題** :
1.  **Kialiのデプロイ** :
    *   既存のGKEクラスタにKialiをデプロイせよ（`istioctl install --set profile=demo`でIstioと一緒にデプロイ済みであれば不要。そうでなければ`istioctl dashboard kiali`で一時的に起動するか、公式ドキュメントに従ってデプロイせよ）。
    *   Kiali UIにアクセスし、ログインできることを確認せよ。
2.  **サービストポロジーの確認** :
    *   Kialiの「Graph」ビューに移動し、`my-app-ns` Namespaceを選択してサービスグラフを表示せよ。
    *   `product-api`と`order-api`間のトラフィックフロー、Istio Ingress Gatewayからのトラフィックが視覚的に表示されていることを確認せよ。
    *   章7.2.2で設定したカナリアリリースやA/Bテストのルーティングがグラフ上で反映されていることを確認せよ（例: v1とv2へのトラフィック比率）。
3.  **メトリクスの確認** :
    *   Kiali上で各サービスの詳細ビューを開き、Istioが収集したメトリクス（リクエスト数、成功率、レイテンシー、CPU/メモリ使用率など）がグラフとして表示されていることを確認せよ。
    *   章6で構築したPrometheusとGrafanaが連携されている場合、Prometheus UIでIstioが収集したメトリクス（例: `istio_requests_total`など）が確認できることを検証し、Grafanaで基本的なダッシュボードを作成して可視化せよ。

---

#### 7.3 GKE IngressとIstioの連携パターン

**概要** : 外部からのトラフィックをGKEクラスタに導入する際、GCPのフルマネージドなロードバランサ（GKE IngressでプロビジョニングされるGCLB）とIstio Ingress Gatewayを連携させる構成は、多くの利点がある。GCP Load BalancerはDDoS防御、グローバル負荷分散、高度なSSL/TLS終端といった機能を提供し、Istio Ingress Gatewayはサービスメッシュ内の高度なルーティング、ポリシー適用、可観測性を担当する。この連携により、両者の強みを活かした堅牢かつ柔軟なシステムを構築できる。

**具体例** : **GCP Static IPのプロビジョニング** : 章3で学習した通り、Ingressに静的IPアドレスを割り当てる。
```bash
gcloud compute addresses create my-app-external-ip --global
```

**Istio Ingress GatewayのService確認** : Istioインストール時に`istio-ingressgateway`というServiceが自動的にプロビジョニングされ、外部IPアドレスが割り当てられているはずである。このServiceをGKE Ingressのバックエンドとして利用する。
```bash
kubectl get svc istio-ingressgateway -n istio-system
```

**GKE Ingressの定義** : GKE Ingressのバックエンドに`istio-ingressgateway`サービスを指定し、静的IPアドレスとManaged SSL/TLS証明書（章4で学習）を連携させる。
`gke-ingress-for-istio.yaml`:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: istio-external-ingress
  annotations:
    kubernetes.io/ingress.global-static-ip-name: "my-app-external-ip"
    networking.gke.io/managed-certificates: "my-app-managed-cert" # Google Managed Certificateの名前
spec:
  tls:
  - secretName: my-app-managed-cert # ManagedCertificateの名前と一致させる
  rules:
  - host: "api.your-domain.com" # 実際のドメインに合わせる
    http:
      paths:
      - path: /*
        pathType: ImplementationSpecific
        backend:
          service:
            name: istio-ingressgateway # istio-system Namespaceのingressgateway
            port:
              number: 80 # istio-ingressgatewayのHTTPポート
```

**ManagedCertificateの定義** : 章4で学習したManagedCertificate を使用する。
`my-app-managed-cert.yaml`:
```yaml
apiVersion: networking.gke.io/v1
kind: ManagedCertificate
metadata:
  name: my-app-managed-cert
spec:
  domains:
    - "api.your-domain.com"
```


**課題** :
1.  **静的IPアドレスとDNS設定の準備** :
    *   GCPでグローバル静的外部IPアドレスをプロビジョニングせよ。
    *   このIPアドレスに対し、テスト用のドメイン名（例: `api.your-domain.com`）を解決するDNS Aレコードを設定せよ。
2.  **Istio Ingress GatewayのService名の確認** :
    *   `istio-system` Namespaceにある`istio-ingressgateway` ServiceのClusterIPとポートを確認せよ。
3.  **GKE Ingressの定義とデプロイ** :
    *   上記具体例を参考に、GKE Ingressリソースを定義せよ。
    *   `backend.service.name`には`istio-ingressgateway`を、`backend.service.port.number`にはIstio Ingress GatewayのHTTPポート（通常80）を指定せよ。
    *   定義したGKE Ingressを`my-app-ns` Namespaceにデプロイせよ。
4.  **動作検証** :
    *   GKE IngressがGCP Load Balancerをプロビジョニングするまで待機せよ（数分かかる場合がある）。
    *   DNSが解決された後、設定したドメイン名（例: `http://api.your-domain.com/products/p001`）にアクセスし、Istioが管理するGo APIマイクロサービスに到達できることを確認せよ。
    *   （任意）Google Managed SSL/TLS証明書を定義し、GKE Ingressと連携させてHTTPSアクセス（例: `https://api.your-domain.com/products/p001`）が正常に行われることを確認せよ。

---

**成果物** : Istioによって管理され、高度なトラフィックルーティング（カナリアリリース、A/Bテストなど）が可能なGo APIベースのマイクロサービス群が成果物となる。

---

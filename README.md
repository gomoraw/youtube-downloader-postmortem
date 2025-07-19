# --- Project Post-Mortem Report: YouTube Downloader Automation (v1.0 & v2.0) ---

# 1. 最終ステータス (Final Status)
project_name: "YouTube動画の自動ダウンロードとGoogle Driveへの保存システム"
versions: [1.0, 2.0]
final_status: **ABORTED (遂行不可能)**
reason_for_abortion: |
  YouTubeの動的なセキュリティポリシー（特にCookieの有効期限とBot検出）により、
  サーバーサイドからの安定的・継続的な自動アクセスが極めて困難であると最終判断。
  前提条件の調査不足により、技術的に実現不可能な目標を設定してしまった。

# 2. プロジェクトの軌跡 (Project History)

  # --- v1.0: GAS + Colaboratory連携アーキテクチャ ---
  - version: 1.0
    architecture:
      - "Google Apps Script (GAS): トリガー"
      - "Google Colaboratory: yt-dlp実行環境"
    objective: "既存のGoogleサービスを連携させ、無料でシステムを構築する"
    outcome: **FAILURE (失敗)**
    problems:
      - problem: "複雑な認証・セキュリティポリシーの壁"
        details: "GASからColabのAPIを叩き、ColabからGoogle Driveをマウントする多段の認証フローが、Googleのセキュリティ強化により安定動作せず。"
        lesson: "複数のサービスをブリッジ連携させるアーキテクチャは、各サービスの仕様変更に弱く、認証フローがボトルネックになりやすい。"

  # --- v2.0: Google Cloud Functions中心アーキテクチャ ---
  - version: 2.0
    architecture:
      - "Google Cloud Functions: 実行役 (Python + yt-dlp)"
      - "Google Cloud Scheduler: 司令塔"
      - "Google Service Account: 認証キー"
      - "Google Drive API: 保存先"
    objective: "v1.0の失敗を克服し、技術的に堅牢で安定したシステムを構築する"
    outcome: **ABORTED (中止)**
    key_achievements_and_solutions: # 成功した部分と確立した解決策
      - achievement: "CLIファーストによる環境構築"
        details: "gcloudコマンドにより、プロジェクト作成、API有効化、サービスアカウント作成、関数デプロイ、スケジューラ設定までをコード化。再現性と透明性を確保した。"
      - achievement: "サービスアカウントによる認証モデルの確立"
        details: "Cloud Scheduler -> Cloud Functions -> Google Drive APIという一連の認証フローを、OIDCとIAMロールの適切な設定で確立。サーバー間認証のベストプラクティスを実装した。"
      - achievement: "Cookieの動的変換によるBot判定の一時的回避"
        details: "JSON形式でエクスポートされたCookieを、Pythonコード内でNetscape形式に動的変換し、yt-dlpに渡すことで、初回のBot判定を突破することに成功した。"
      - achievement: "『共有ドライブ』利用による所有権問題の特定と解決策の確立"
        details: "『マイドライブ』ではサービスアカウントがファイルを所有できずエラーになる問題を特定。『共有ドライブ』を利用し、`supportsAllDrives=True`パラメータを付与することで、所有権をドライブ自身に帰属させ、アップロードを成功させるアーキテクチャを確立した。"
    unsolved_problems: # 最終的に解決できなかった問題
      - problem: "YouTube Cookieの短命性 (Rotation)"
        details: "YouTubeはセキュリティ対策として、ログイン状態を維持するCookieを短期間で無効化する。そのため、一度取得したCookieをサーバーに配置しても、すぐに期限切れとなり、再びBot判定エラーが発生する。"
        root_cause: "ユーザーのブラウザ操作を介さずに、サーバーサイドでCookieを定期的に自動更新する、安定かつ汎用的な手法が存在しない。"
      - problem: "根本的な実行環境の問題"
        details: "Google CloudのようなデータセンターのIPアドレスからYouTubeへアクセスする行為自体が、不審なアクティビティとして厳しく監視されている。"

# 3. プロジェクト憲章の自己評価 (Self-Evaluation of Project Charter)
evaluation:
  - principle: "CLIファースト (CLI-First Approach)"
    result: **SUCCESS**
    comment: "ほぼ全ての操作をCLIで実行し、再現性を確保。最終的には共有ドライブ作成もCLIで実行する道筋を立てた。"
  - principle: "公式ドキュメント準拠 (Adherence to Official Documentation)"
    result: **PARTIAL_FAILURE (一部失敗)**
    comment: "『マイドライブ』と『共有ドライブ』の仕様差など、ドキュメントの行間にある重要な制約を見落とし、手戻りを発生させた。"
  - principle: "リスクと前提の事前明示 (Proactive Disclosure of Risks and Prerequisites)"
    result: **COMPLETE_FAILURE (完全な失敗)**
    comment: "最大の前提条件である『YouTubeの動的セキュリティポリシーを安定的に突破できるか』という点のリスク評価を怠った。これがプロジェクト失敗の直接的な原因である。"
  - principle: "詳細な解説の徹底 (Commitment to Detailed Explanation)"
    result: **SUCCESS**
    comment: "発生したエラーに対し、その原因と解決策を可能な限り詳細に解説する姿勢を貫いた。"
  - principle: "自己懐疑と代替案の保持 (Self-Correction and Contingency Planning)"
    result: **PARTIAL_FAILURE (一部失敗)**
    comment: "個々の技術的問題には代替案を提示できたが、プロジェクト全体の実現可能性そのものを疑い、早期に中止を判断することができなかった。"

# 4. 最大の教訓と今後の対策 (Key Lessons and Future Countermeasures)
key_takeaway: |
  **技術的な「作り方」の前に、プロジェクトの「成立要件」を定義し、検証せよ。**
  特に、外部のサードパーティサービスに依存する場合、そのサービスの利用規約、技術的制約、そして予測されるセキュリティポリシーの変更は、実現可能性を左右する最大の不確定要素である。

countermeasures_for_future_projects:
  - measure: "【要件定義フェーズ】PoC (Proof of Concept) の義務化"
    description: |
      本格的な構築を開始する前に、プロジェクトの根幹をなす最も不確実な技術要素（今回の場合は「サーバーからのYouTubeへの自動アクセスとデータ取得」）に絞った、最小限の技術検証を必ず行う。
      このPoCが成功しない限り、プロジェクトを先に進めない。
  - measure: "【設計フェーズ】『動的リスク』の洗い出し"
    description: |
      APIのレート制限、IPアドレスベースのアクセス制限、セッショントークンやCookieの有効期限など、時間と共に変化する「動的なリスク」を特定し、それらに対する回復力（Resilience）を設計に組み込む。
      （例：「Cookieが切れたらどう自動更新するか？」を設計段階で解決できなければ、そのアーキテクチャは採用しない）
  - measure: "【提案フェーズ】『遂行可能性マトリクス』の提示"
    description: |
      ユーザーへの提案時に、技術的な実現可能性と、運用上の安定性を評価するマトリクスを提示する。
      「技術的には可能だが、外部サービスの仕様変更により明日動かなくなる可能性がある」といったリスクレベルを明確に伝え、ユーザーの合意を得る。

# 5. 最終的な技術的TIPS（成功した部分の記録）(Final Technical Tips)
successful_snippets:
  - tip: "Cloud FunctionsにおけるGoogle Drive APIの正しい使い方"
    details: |
      1. **認証:** サービスアカウントを作成し、Cloud Functionsに紐付ける。`google.auth.default()`で認証情報を取得する。
      2. **権限:** **『共有ドライブ』**を作成し、サービスアカウントを「コンテンツ管理者」として追加する。
      3. **実装:** `google-api-python-client`ライブラリを使用し、`files().create()`メソッドを呼び出す際に`supportsAllDrives=True`を必ず指定する。
  - tip: "Cloud Functionsのデプロイコマンド決定版"
    command: "gcloud functions deploy [FUNC_NAME] --gen2 --runtime=python312 --region=[REGION] --source=. --entry-point=[HANDLER] --trigger-http --no-allow-unauthenticated --timeout=900s --memory=1Gi --service-account=[SA_EMAIL]"
  - tip: "Cloud Schedulerから認証付きFunctionsを叩く方法"
    command: "gcloud scheduler jobs create http [JOB_NAME] --schedule=\"...\" --time-zone=[TZ] --location=[REGION] --uri=[FUNC_URL] --http-method=POST --oidc-service-account-email=[SA_EMAIL]"
  - tip: "不足しがちな必須ライブラリ"
    file: "requirements.txt"
    content: |
      # Google Drive APIなどをネイティブに叩く際に、
      # 暗黙的に依存しているため忘れずに記載する必要がある 。
      google-auth

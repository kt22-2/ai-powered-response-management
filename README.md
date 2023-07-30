# ai-powered-response-management
# AIによる感情分析とカテゴライズを用いたMicrosoft　Power Platformによる問い合わせ対応システムの概略

1. **Microsoft Formsのセットアップ**

   - Microsoft Formsを開き、新規のFormを作成します。
   - 与えられたフィールド（名前、ふりがな、郵便番号、住所、電話番号、メールアドレス、宛先、問い合わせ内容）を入力フィールドとして設定します。特に宛先フィールドについては、与えられた選択肢をドロップダウンリストとして設定します。

2. **SharePointリストのセットアップ**

   - SharePointのサイトを開き、新しいリストを作成します。
   - 与えられた列名（名前、ふりがな、郵便番号、住所、電話番号、メールアドレス、宛先、問い合わせ内容、提出日時、タイトル、感情スコア、ネガポジ判定、緊急フラグ、対応ステータス、担当者、エスカレーションフラグ、返信内容、TeamsメッセージID）を作成します。

3. **Power Automateのセットアップ**

   - Power Automateを開き、新しいフローを作成します。トリガーとして「Microsoft Formsからの新しいレスポンスを取得」を選択します。
   - 「新しいレスポンスを取得」アクションの後に「SharePointのリスト項目の作成」アクションを追加します。これにより、新しいFormのレスポンスがSharePointリストに自動的に保存されます。
   - 次に、AzureのCognitive Servicesを使用して感情分析を行い、結果をSharePointの感情スコア列に保存します。このためには、「HTTPアクション」を使用してCognitive Services APIにリクエストを送信し、「問い合わせ内容」フィールドを送信します。次に、レスポンスを解析し、結果をSharePointの感情スコア列に格納します。
   - その後、条件アクションを設定して、感情スコアに基づいて緊急フラグを設定します。
   - 次に、「Teamsのメッセージの作成」アクションを追加します。これにより、新しいリスト項目が作成されると、自動的にTeamsの指定チャネルにアダプティブカードが投稿されます。
   - アダプティブカードは、受付ボタンと差し戻しボタンを含むように設計します。それぞれのボタンは、リスト項目の対応ステータス列を更新するか、指定の担当者にTeamsで通知するアクションをトリガーします。

4. **Power Appsのセットアップ**

   - Power Appsを開き、新しいCanvas Appを作成します。
   - SharePointリストをデータソースとして接続します。これにより、アプリ内でリスト項目を表示および編集できます。
   - 一覧画面と詳細画面を作成します。一覧画面では、すべてのリスト項目を一覧表示し、詳細画面では、選択したリスト項目の詳細を表示し、OpenAI APIを使用して作成した返信を確認して編集できます。
   - メール送信機能を追加します。これには、「Office 365 Outlookのメールを送信」アクションを使用します。メールの本文には、OpenAI APIで生成した返信を使用します。
   - 返信が送信されたら、対応ステータス列を「完了」に更新します。また、該当するTeamsメッセージに@mentionを付けて通知します。

5. **Power BIのセットアップ**

   - Power BI Desktopを開き、新しいレポートを作成します。
   - SharePointリストをデータソースとして接続します。このときに適切なリスト（問い合わせデータが保存されているリスト）を選択します。
   - 必要なデータビジュアルを選択してドラッグ＆ドロップし、各ビジュアルに対応するフィールドを設定します。例えば、問い合わせ数の時間帯別グラフ、問い合わせ内容の感情スコア分布、問い合わせの種類別数などを設定できます。
   - レポートが完成したら、Power BI Serviceにパブリッシュします。Power BI Serviceでは、作成したレポートをダッシュボードとして組み合わせることができます。

6. **カテゴリ分類のセットアップ**

   - Power Automate内で新しいフローを作成します。トリガーとして「SharePointリストアイテムの作成または更新」を選択します。
   - 作成または更新されたリストアイテムの問い合わせ内容を取得し、テキスト分類用のAIサービス（例えばAzureのText AnalyticsまたはCognitive Services）に送信します。
   - AIサービスから返されたカテゴリを取得し、SharePointリストアイテムの新しいフィールド（例えば「問い合わせカテゴリ」）に保存します。


```mermaid
sequenceDiagram
    participant User
    participant Forms as Microsoft Forms
    participant PA as Power Automate
    participant SPList as SharePoint List
    participant PApps as Power Apps
    participant AI as AI Category Classifier
    participant PB as Power BI

    User->>Forms: Submit form (名前, ふりがな, 郵便番号, etc.)
    Forms->>PA: Trigger new response
    PA->>SPList: Create new item with given details
    PA->>AI: Send enquiry content for classification
    AI->>PA: Return classification result
    PA->>SPList: Update item with classification result
    PA->>PApps: Notify new item
    PApps->>User: Display item details
    User->>PApps: Send reply
    PApps->>User: Update status, notify on Teams
    SPList->>PB: Data connection
    PB->>User: Display dashboard

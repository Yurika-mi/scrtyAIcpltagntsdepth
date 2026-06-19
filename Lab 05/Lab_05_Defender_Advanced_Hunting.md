# ラボ 05: Microsoft Defender — AI エージェント インベントリと脅威ハンティング

## 概要

Microsoft Defender for Cloud Apps には、テナント内のすべての Copilot Studio カスタム エージェントを発見し、セキュリティ調査のために公開する専用 AI エージェント インベントリが用意されています。Microsoft Defender XDR の Advanced Hunting テーブル「AIAgentsInfo」と組み合わせることで、セキュリティ チームはエージェント構成をクエリし、設定の誤りを検出し、ガバナンスのギャップを特定し、Defender ポータルを離れることなく、リスクのあるエージェント動作を積極的にハンティングすることができます。

このラボでは、Defender プレビュー機能を有効にし、Copilot Studio AI エージェント インベントリをアクティブ化して、Power Platform に接続します。その後、Patti Fernandes が AI エージェント インベントリを確認し、Zava エージェント構成を調査し、Advanced Hunting KQL クエリを実行して、Zava エージェント資産全体の潜在的なセキュリティ リスクを特定します。

---

## シナリオ

Zava のセキュリティ オペレーション センター (SOC) チームは、デプロイ済みのすべての AI エージェントが Defender ポータルで表示され、セキュリティ チームが設定が誤ったエージェントやリスクのあるエージェントをハンティングするためのツールを備えていることを確認するよう求められています。Patti Fernandes は AI エージェント インベントリを使用して、認証タイプ、知識ソース、所有者割り当てを含む Zava エージェント プロパティを確認し、一連のコミュニティおよびカスタム KQL クエリを実行して、設定リスクを表面化します。すべての検出結果は Day 2 の終わりに CISO レビュー用にドキュメント化されます。

---

## 目標

- Microsoft Defender プレビュー機能を Cloud Apps、Defender for Cloud、および Defender XDR に対して有効にします。
- Defender for Cloud Apps 設定で Copilot Studio AI エージェント インベントリを有効にします。
- Power Platform Admin Center で AI エージェント インベントリのオンボーディングを完了します。
- Defender ポータルで緑色の「接続済み」ステータスを確認します。
- AI エージェント インベントリを確認し、Zava エージェントの詳細を確認します。
- Go hunt を使用して、Advanced Hunting を開き、特定のエージェントに対してプリフィルタリングします。
- AI Agents フォルダーのコミュニティ クエリを実行して、認証されていないエージェントと設定が誤ったエージェントを特定します。
- カスタム KQL クエリを実行して、すべての Zava エージェント構成を単一ビューで確認します。
- Cloud Apps エージェント関連アクティビティの Defender アラート キューを確認します。

---

## ラボ期間

推定時間: **60 分**

---

## 演習 1: Defender プレビュー機能を有効にする

### タスク 1: Microsoft Defender XDR でプレビュー機能を有効にする

1. ブラウザーを開き、以下の URL を使用して **Microsoft Defender** に移動します。

    ```
    https://security.microsoft.com
    ```

2. プロンプトが表示された場合は、**ODL ユーザー**の認証情報でサインインします。

3. 左ナビゲーション ウィンドウで、**System(1)** を展開して **Settings(2)** を選択します。

4. **Settings** ページで、**Microsoft Defender XDR(3)** を選択します。

   ![](./media/l05-e1-t1-s4.png)

5. 左側のサブナビゲーションで、**Preview features** を選択します。

   ![](./media/l05-e1-t1-s5.png)

6. **Preview features** ページで、**Preview features** トグルを **On** に設定します (まだ設定されていない場合)。

   - **Microsoft Defender XDR** と **Microsoft Defender for Cloud Apps** のチェックボックスが有効になっていることを確認します。

      ![](./media/l05-e1-t1-s6.png)

   - **Save preferences** を選択します。

        ![](./media/l05-e1-t1-s7.png)

   - 成功通知が表示されていることを確認します。

        ![](./media/l05-e1-t1-s8.png)

---

## 演習 2: Copilot Studio AI エージェント インベントリを有効にする

### タスク 1: Azure ポータルで新しいアプリ登録を作成する

1. 以下の URL を使用して Azure ポータルに移動し、プロンプトが表示された場合は **ODL ユーザー**の認証情報でサインインします。

    ```
    https://portal.azure.com
    ```

1. ページの右上隅から Cloud Shell アイコンを選択して、Azure Cloud Shell セッションを起動します。

   ![](./media/appid.png)

    >**注**: プロンプトが表示された場合は、続行する前に Cloud Shell の初期化を完了してください。

1. **Manage Files** をクリックして **Upload** を選択します。

   ![](./media/appid2.png)
 
    - C:\ラボFiles\Create-CopilotWebhookApp.ps1 から **Create-CopilotWebhookApp.ps1** をアップロードします。
    
    - 確認ポップアップでスクリプトがアップロードされていることを確認します。

       ![](./media/appid3.png)


1. CloudShell でコマンドを実行します。

    ```
    .\Create-CopilotWebhookApp.ps1 -TenantId "<Paste your TenantId>" -Endpoint "https://mcsaiagents.security.core.microsoft/v1/protection" -DisplayName "Copilot Security Integration - Production" -FICName "ProductionFIC"
    ```

    - Azure ポータルで **Microsoft Entra ID** に移動します。

         ![](./media/appid4.png)

    - コマンドで使用する **Tenant ID** をコピーします。

         ![](./media/appid5.png)

   ![](./media/appid6.png)

1. リンクに移動して、認証を完了するためにコードを貼り付けます。

   ![](./media/appid7.png)

1. Copilot と Defender を接続するタスク時に使用されるため、App ID をコピーします。

   ![](./media/appid8.png)

### タスク 2: Defender と Copilot Studio を接続する

1. **Settings** に戻り、**Security for AI** を選択します。

   ![](./media/l05-e2-t1-s2.png)

4. 下にスクロールして **Copilot Studio** を見つけ、**Connect** をクリックして統合セットアップを開始します。

   ![](./media/l05-e2-t1-s4.png)

5. Copilot Studio リアルタイム保護ペインで、リアルタイム保護が有効になっていることを確認し、生成された Power Platform 統合 URL を確認します。

   ![](./media/l05-e2-t1-s5.png)

6. App ID フィールドに必須の App ID を入力し、**Save** をクリックして Copilot Studio リアルタイム保護構成を完了します。

   ![](./media/l05-e2-t1-s6.png)

   ![](./media/l05-e2-t1-s7.png)

   > **注**: この設定を有効にすると、Defender for Cloud Apps と Copilot Studio 間の接続が開始されます。Power Platform Admin Center での 2 番目のステップを完了してから、緑色の「接続済み」ステータスが表示されます。

---

### タスク 3: Power Platform Admin Center でオンボーディングを完了する

1. 新しいブラウザー タブを開き、以下の URL を使用して **Power Platform** に移動します。

    ```
    https://admin.powerplatform.microsoft.com
    ```

2. プロンプトが表示された場合は、**ODL ユーザー**の認証情報でサインインします。
   - **Email/Username:** <inject key="AzureAdUserEmail"></inject>
   - **Password:** <inject key="AzureAdUserPassword"></inject>

3. 左ナビゲーション ウィンドウで **Security** をクリックして **Threat Detection** を選択し、**Microsoft Defender - Copilot Studio Agents (Preview)** を見つけます。

   ![](./media/l5e2t2s3.png)   

7. **Enable Microsoft Defender - Copilot Studio Agents** トグルを **On** に設定して **Manage** をクリックします。

   ![](./media/l5e2t2s5.png)  

1. **Dev** 環境を選択して **Setup** をクリックします。

   ![](./media/l5e2t2s6.png)

1. **Allow Copilot Studio to share data with a threat detection partner** のチェックボックスを有効にして、**Azure Entra App ID** と **Endpoint link** を入力し、**save** をクリックします。

   ![](./media/l5e2t2s7.png)
   
   >Endpoint link と Entra App ID を Defender ポータルから取得します。
      ![](./media/l05-e2-t1-s6.png)

---

### タスク 3: Defender ポータルで「接続済み」ステータスを確認する

1. **ODL ユーザー**ブラウザー セッションの Defender ポータルに戻ります。

    ```
    https://security.microsoft.com
    ```

2. **Settings** に移動して **Security for AI** を選択します。

   ![](./media/l05-e2-t1-s2.png)

6. **Copilot Studio** に緑色の **Connected** インジケーターが表示されていることを確認します。

   ![](./media/l5e2t3s3.png)

   > **注**: 両方のオンボーディング ステップを完了した後、初期接続ステータスが更新されるまで時間がかかる場合があります。

---

## 演習 3: AI エージェント インベントリを確認する

### タスク 1: AI エージェント インベントリにアクセスする

1. 左ナビゲーション ウィンドウの **Assets** から **AI Agents** を選択します。

   ![](./media/l5e3t1s2.png)

   > **注**: **AI Agents** が Assets の下に表示されない場合は、演習 1 でプレビュー機能が有効化されており、演習 2 でインベントリ接続が完了したことを確認してください。演習 2 の完了後、再試行する前に最大 30 分待機してください。

3. **AI Agents** ページで、Zava テナントで発見されたエージェントの完全なリストを確認します。

4. **Platform (1)** フィルターで **Copilot Studio (2)** を選択して **Apply (3)** をクリックし、ビューを Copilot Studio カスタム エージェントのみにフィルタリングします。

   ![](./media/l5e3t1s4.png)

5. インベントリに以下の 3 つのエージェントが表示されていることを確認します。

   | エージェント名 | ステータス | プラットフォーム |
   |---|---|---|
   | Zava HR Assistant | Published | Copilot Studio |
   | Zava Finance Agent | Published | Copilot Studio |
   | Zava IT Support Agent | Published | Copilot Studio |

   ![](./media/l5e3t1s5.png)

---

### タスク 2: Zava HR Assistant エージェントの詳細を確認する

1. **AI Agents** ページで **Zava HR Assistant** を選択して、その詳細ペインを開きます。

   ![](./media/l5e3t2s1.png)

2. 詳細ペインで、以下のフィールドを確認します。

   - **Agent name**
   - **Status**
   - **Version**
   - **Publish Status**
   - **Model**
   - **Tools**
   - **Channels**
   - **Active alerts**
   - **Entra Agent ID**

   ![](./media/l5e3t2s3.png)

---

### タスク 3: Go Hunt を使用して Zava HR Assistant の Advanced Hunting を開く

1. **Zava HR Assistant** 詳細ペインで **Go hunt** ボタンまたはリンクをクリックします。

   ![](./media/l5e3t3s1.png)

3. ブラウザーが **Investigation & response > Hunting > Advanced hunting** に移動し、Zava HR Assistant エージェントに対してスコープされたクエリが事前に入力されていることを確認します。

   ![](./media/l5e3t3s2.png)

4. 事前に入力されたクエリを確認して、その構造を理解します。

5. **Run query** を選択してクエリを実行します。

   ![](./media/l5e3t3s3.png)

6. クエリ出力パネルで返された結果を確認します。

   ![](./media/l5e3t3s4.png)

---

## 演習 4: AIAgentsInfo に対して Advanced Hunting クエリを実行する

### タスク 1: Patti Fernandes として Defender ポータルにサインインする

1. **InPrivate** または **Incognito** ブラウザー ウィンドウを新たに開きます。

2. 以下の URL を使用して **Defender ポータル**に移動します。

    ```
    https://security.microsoft.com
    ```

3. **Resources** タブから **Patti Fernandes** の認証情報でサインインします。
   - **Email:** <inject key="User 01 UPN"></inject>
   - **Password:** <inject key="User's Password"></inject>

4. 左ナビゲーション ウィンドウで **Investigation & response** を選択します。

5. **Investigation & response** で **Hunting** を選択します。

6. **Advanced hunting** を選択します。

   ![](./media/l5e3t3s2.png)

---

### タスク 2: Zava エージェント構成確認カスタム クエリを実行する

1. **Advanced hunting** ページで **New query** タブを選択して、空白のクエリ エディターを開きます。

   ![](./media/l5e4t2s1.png)

2. クエリ エディターに、以下の KQL クエリを入力します。

   ```
   AgentsInfo
   | summarize arg_max(Timestamp, *) by AgentId
   | where LifecycleStatus != "Deleted"
   | where Name has_any ("Zava HR Assistant", "Zava Finance Agent", "Zava IT Support Agent")
   | project
       CreatedDateTime,
       Name,
       LifecycleStatus,
       PublishedStatus,
       Owners,
       SharedWith,
       Version,
       Model,
       LastUpdatedDateTime,
       Avaiラボility,
       Permissions
   | sort by CreatedDateTime asc
   ```

3. **Run query** を選択します。

   ![](./media/l5e4t2s2.png)

4. 3 つの Zava エージェント全体に対して返された結果を確認します。

   ![](./media/l5e4t2s3.png)

6. **Save** を選択してクエリを保存します。

   ![](./media/l5e4t2s4.png)

7. **Save query** ペインで以下を入力して **Save** をクリックします。

   - **Query name:** `Zava Agent Configuration Review`
   - **Location:** **My queries** を選択します。

   ![](./media/l5e4t2s5.png)

   ![](./media/l5e4t2s6.png)

---

## 演習 5: エージェント関連アクティビティの Defender アラート キューを確認する [オプション]

### タスク 1: Cloud Apps ソースでアラート キューをフィルタリングする

1. Microsoft Defender ポータルで **Patti Fernandes** としてサインイン状態を保ちます。

2. 左ナビゲーション ウィンドウで **Incidents & alerts** を選択します。

3. **Alerts** を選択します。

4. **Alerts** ページで **Add filter** を選択します。

5. フィルター ドロップダウンで **Service source** を選択します。

6. フィルター値として **Microsoft Defender for Cloud Apps** を選択します。

7. **Apply** を選択します。

8. フィルター済みビューで返されたアラートを確認します。

9. アラートが存在する場合は、そのアラートを選択して詳細ペインを開きます。

10. アラート詳細ペインで、以下のフィールドを確認します。

    - **Alert name**
    - **Severity**
    - **Status**
    - **Affected entity**
    - **Detection source**
    - **Activity log**

11. アラート詳細ペインを閉じます。

    > **注**: 新しく構成されたラボ環境では、Cloud Apps アラート キューが空の場合があります。または、コネクタ関連のイベントのみが含まれている場合があります。エージェント関連アラートは、Zava エージェントが起動され、リアルタイム保護シグナルが生成され、Day 2 および Day 3 ラボ全体でポリシー違反が発生すると表示され始めます。このステップにより、Patti が Day 3 でのインシデント調査に使用するアラート キューの習熟度を確立します。

---

### タスク 2: エージェント固有のインシデントを確認する [オプション]

1. 左ナビゲーション ウィンドウで **Incidents & alerts** を選択します。

2. **Incidents** を選択します。

3. **Incidents** ページの検索バーで `Zava` を入力します。

4. Zava エージェント アクティビティを参照するために返されたインシデントを確認します。

5. インシデントが存在する場合は、それを選択してインシデント詳細ページを開きます。

6. インシデント詳細ページで **Alerts** タブを確認して、インシデントにグループ化されたすべてのアラートを確認します。

7. **Evidence and response** タブを確認して、影響を受けたエンティティを確認します。

8. インシデントを閉じて **Incidents** ページに戻ります。

   > **注**: コースのこの段階で Zava 関連のインシデントが表示されない場合、これは予想されています。ここで説明する検索およびフィルター手法に注意してください。これらは、Day 3 でアクティブな脅威調査タスクが導入されたときに使用されます。

---

## 概要

このラボでは、Defender XDR および Defender for Cloud Apps の Microsoft Defender プレビュー機能を有効にしました。これらは Copilot Studio AI エージェント インベントリと `AIAgentsInfo` Advanced Hunting スキーマにアクセスするために必要です。Defender for Cloud Apps 設定で Copilot Studio AI エージェント インベントリを有効にし、データ接続を確立するために Power Platform Admin Center で対応するオンボーディング ステップを完了しました。Defender ポータルで緑色の「接続済み」ステータスを確認しました。Assets の AI エージェント インベントリを確認し、認証タイプ、アクセス制御ポリシー、知識ソース、所有者割り当てを含む Zava HR Assistant エージェントの詳細を確認しました。また、Go hunt アクションを使用して、Advanced Hunting を開き、そのエージェントに対してプリフィルタリングしました。

Patti Fernandes として、AI Agents フォルダーから 2 つのコミュニティ クエリを実行しました。認証なしのエージェントと難しくされた認証情報を持つエージェントを検出し、Zava エージェント資産に対して結果を確認しました。すべての 3 つの Zava エージェントのキー セキュリティ プロパティを単一ビューで表示するために、カスタム Zava エージェント構成確認 KQL クエリを実行し、将来の使用のために保存しました。アクセス制御ポリシーが過度に広いエージェントを特定するために 2 番目のカスタム クエリを実行しました。最後に、Cloud Apps ソースでフィルタリングした Defender アラート キューを確認し、Incidents ページで Zava 関連アクティビティを確認して、Day 3 の調査ベースラインを確立しました。

Day 2 が完了しました。Zava のエージェントは条件付きアクセス ポリシーによって管理されており、機密データは Purview ラベルで分類され、DLP コントロールで保護されています。セキュリティ チームは、Defender AI エージェント インベントリと Advanced Hunting を通じて、エージェント構成とアクティビティの完全な可視性を備えています。

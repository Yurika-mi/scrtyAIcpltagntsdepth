# ラボ 00：環境設定 — Zava Corporation のAIエージェントインフラストラクチャ

## はじめに

**Zava Corporation**は、英国およびEU全域で事業を展開する中規模の金融サービスおよび人事コンサルティング企業です。Zavaは、機密性の高い従業員記録、顧客の財務データ、およびサードパーティベンダーとの契約を管理しています。同社では最近、業務効率の向上を目的として、人事、財務、ITサポートの各部門にAIエージェントを導入しました。

セキュリティ設定を開始する前に、Zava Corporation の環境を完全にプロビジョニングする必要があります。この実習では、Microsoft Entra ID テナントの設定、Microsoft Copilot Studio の有効化、エージェント作成に必要なセキュリティグループの登録、コース全体を通じてガバナンスの対象となる 3 つの AI エージェントの作成、各エージェントを指定された SharePoint ナレッジソースへの接続、および Zava の実データ環境をシミュレートするサンプルビジネス文書のアップロードを行います。

以降のすべての実習は、ここで作成されたエージェント、ID、およびファイルに依存します。ラボ 01に進む前に、3つの演習をすべて順番に完了させてください。

---

   > **注:** 実際の環境では、このような責任は、開発者、IT管理者、セキュリティ管理者、コンプライアンス担当者といった複数の役割に分散され、各役割は「最小権限の原則」および「ゼロトラスト」の原則に沿った範囲限定の権限で運用されます。

---

## 目的

- Microsoft Entra ID でロール割り当て可能なセキュリティ グループを作成し、そのグループに「特権ロール管理者」ロールを割り当てます。
- Power Platform 管理センターで、**copilotagentsecurity** グループを Copilot Studio の承認済み作成者グループとして有効にします。
- 環境レベルで、Copilot Studio 用の Entra エージェント ID を有効にします。
- Power Apps メーカー ポータルで、SharePoint をデータ ソースとして接続します。
- 3 つの Copilot Studio エージェント（Zava HR アシスタント、Zava 財務エージェント、Zava IT サポートエージェント）を作成します。
- 各エージェントを、指定された SharePoint ナレッジソースに接続します。
- 各エージェントを公開し、該当するラボユーザーと共有します。
- Zava のサンプル ビジネス ドキュメントを、人事および財務の SharePoint サイトにアップロードします。
- Microsoft Agent 365 エージェント レジストリで、3 つのエージェントすべてが「アクティブ」と表示されていることを確認します。

---

## ラボ所要時間

推定所要時間：**30分**

---
## 演習 0: Zava HR SharePoint サイトを作成する

1. 新しいブラウザタブを開き、以下のサイト `https://admin.microsoft.com` に移動します。プロンプトが表示された場合は、**ODL_User** の認証情報でサインインしてください。

	- **メールアドレス/ユーザー名:** <inject key="AzureAdUserEmail"></inject>
	- **パスワード:** <inject key="AzureAdUserPassword"></inject>

2. 左側のナビゲーションペインで **[すべてを表示]** をクリックし、**[管理センター]** の下から **[SharePoint]** を選択します。

	![](./media/Lab-00-01.png)

3. SharePoint 管理センターで、左側のナビゲーション ペインから **[サイト (1)]** を展開し、**[アクティブなサイト (2)]** を選択します。その後、**[+ 作成 (3)]** をクリックします。

	![](./media/Lab-00-02.png)

4. **[サイトの作成]** パネルで、**[チームサイト]** を選択します。

	![](./media/Lab-00-03.png)

5. **[テンプレートの選択]** ページで、**[標準チーム]** を選択します。

	![](./media/Lab-00-04.png)

6. **['標準チーム' テンプレートのプレビューと使用]** ページで、**[テンプレートを使用]** を選択します。

	![](./media/Lab-00-05.png)

7. **[チームサイト]** 構成ページで、以下を入力して **[次へ(4)]** をクリックします。

   - **サイト名 (1):** **HR<inject key="Deployment ID" enableCopy="false"></inject>**
   - **サイトアドレス (2):** URL パスが **/sites/HR<inject key="Deployment ID" enableCopy="false"></inject>** であることを確認します
   - **グループの所有者 (3):** ODL_User <inject key="Deployment ID" enableCopy="false"></inject>

	![](./media/Lab-00-06.png)

8. 以下の詳細情報を追加して、**[サイトを作成 (4)]** をクリックします。

   - **プライバシー設定 (1):** **[プライベート - メンバーのみがこのサイトにアクセスできます]** を選択します。
   - **言語を選択 (2):** 日本語
   - **タイムゾーンを選択 (3):** (UTC+09:00) 大阪、札幌、東京

		![](./media/Lab-00-07.png)

9. **[サイト所有者とメンバーの追加]** ページで、**[完了]** をクリックします。

	![](./media/Lab-00-08.png)

10. サイトのプロビジョニングが完了するまで待機します。**[アクティブなサイト]** リストに URL **https://[TenantPrefix].sharepoint.com/sites/HR<inject key="Deployment ID" enableCopy="false"></inject>** が表示されていることを確認します。

   > **注：** このサイトは、ラボ 00の演習 2 で作成した Zava HR アシスタント エージェント用の SharePoint ナレッジソースです。Copilot Studio でのエージェント接続では、**/sites/HR<inject key="Deployment ID" enableCopy="false"></inject>** が明示的に参照されます。これとは異なる URL スラッグは使用しません。

11. 手順 3 から手順 9 までの同じ手順に従い、次のサイトを作成します。

   - **サイト名:** **Operations<inject key="Deployment ID" enableCopy="false"></inject>**
   - **サイトアドレス:** URL パスが **/sites/Operations<inject key="Deployment ID" enableCopy="false"></inject>** であることを確認します
   - **グループ所有者:** ODL_User <inject key="Deployment ID" enableCopy="false"></inject>
   - **プライバシー設定:** **[プライベート]** を選択します
   - **言語を選択:** 日本語
   - **タイムゾーン:** (UTC+09:00) 大阪、札幌、東京
---

## 演習 1: Entra ID の構成と Copilot Studio 作成者の有効化

### タスク 1: サインインと多要素認証の設定

1. ブラウザを開き、`https://entra.microsoft.com` にアクセスします。

2. サインインページで、プロンプトが表示されたら、ラボ環境の **[環境]** タブに記載されている **ODL ユーザー** の認証情報を入力します:
	- **メールアドレス/ユーザー名:** <inject key="AzureAdUserEmail"></inject>

      ![](./media/Lab-00-09.png)

    - **一時アクセスパス:** <inject key="AzureAdUserPassword"></inject>

      ![](./media/Lab-00-10.png)

3. **[サインインの状態を維持しますか？]** と尋ねられた場合は、**[はい]** を選択します。

    ![](./media/Lab-00-11.png)

4. Microsoft Entra 管理センターのウェルカム画面で、**開始**を選択します。

---

### タスク 2: copilotagentsecurity セキュリティ グループを作成する

1. Microsoft Entra 管理センターの左側のナビゲーション ペインで、**[Entra ID]** を展開し、**[グループ]** を選択します。

    ![](./media/Lab-00-12.png)

2. **[概要]** ページで、**[新しいグループ]** を選択します。

	![](./media/Lab-00-13.png)

3. **[新しいグループ]** ページで、以下のフィールドを設定します。

   - **グループの種類:** **セキュリティ** を選択します。
   - **グループ名:** `copilotagentsecurity` を入力します。
   - **グループに Microsoft Entra ロールを割り当てることができる:** **はい** を選択します。このオプションが表示されない場合は、このフィールドをスキップして次に進みます。

        ![](./media/Lab-00-14.png)

4. **[所有者]** の下で、**[所有者は選択されていません]** を選択します。

    ![](./media/Lab-00-15.png)

5. **[所有者の追加]** パネルで、**ODL_USER <inject key="Deployment ID" enableCopy="false"></inject>** を検索して選択します。[**選択**] をクリックして所有者を確認します。

    ![](./media/Lab-00-16.png)

6. **[メンバー]** の下で、**[メンバーが選択されていません]** を選択します。

    ![](./media/Lab-00-17.png)

7. **[メンバーの追加]** パネルで、**[ODL User <inject key="Deployment ID" enableCopy="false"></inject>]** および **[Patti Fernandes]** を検索して選択します。[選択] をクリックしてメンバーを確定します。

    ![](./media/Lab-00-18.png)

8. **[ロール]** の下で、**[ロールは選択されていません]** を選択します。

    ![](./media/Lab-00-19.png)

9. **[ロールの選択]** パネルで、`グローバル管理者` を検索し、**[グローバル管理者]** のロールを選択して、**[選択]** をクリックします。

    ![](./media/Lab-00-20.png)

10. **[作成]**をクリックして、新しいグループを作成します。

    ![](./media/Lab-00-21.png)

11. 確認ダイアログで、**[はい]**を選択します。

    ![](./media/Lab-00-22.png)

12. ページ上部に成功通知が表示されていることを確認します。

    ![](./media/Lab-00-23.png)

---

### タスク 3: Azure リソースのアクセス管理を有効にする

1. Microsoft Entra 管理センターの左側のナビゲーションペインで、**[Entra ID]** を展開し、**[概要]** を選択します。

    ![](./media/Lab-00-24.png)

2. **[概要]** ページで、上部のバーから **[プロパティ]** を選択します。

    ![](./media/Lab-00-25.png)

3. **[プロパティ]** ページで、**[Azureリソースのアクセス管理]** トグルを見つけて、**[はい]** に設定します。

    ![](./media/Lab-00-26.png)

5. **[セキュリティの既定値群の管理]** を選択します。

    ![](./media/Lab-00-27.png)

6. **[セキュリティの既定値群]** パネルで、**[セキュリティの既定値群]** の下にある **[有効]** を選択（まだ有効化されていない場合）してから、 **[保存]** をクリックします。

    ![](./media/Lab-00-28.png)

7. **[プロパティ]** ページに戻り、**[保存]** を選択します。

    ![](./media/Lab-00-29.png)

---

### タスク 4: 特権ロール管理者 ロールを割り当てる

1. Microsoft Entra 管理センターの左側のナビゲーションペインで、**[Entra ID]** を展開し、**[役割と管理者]** を選択します。

    ![](./media/Lab-00-30.png)

2. **[ロールと管理者]** ページの検索バーで、`特権ロール管理者` を検索し、 **[特権ロール管理者]** のロール名を選択します。

    ![](./media/Lab-00-31.png)

3. **[特権ロール管理者]** ページで、**[+ 割り当てを追加]** を選択します。

    ![](./media/Lab-00-32.png)

4. **[割り当ての追加]** パネルで、**copilotagentsecurity** を検索して選択します。**[追加]** を選択して確認します。

    ![](./media/Lab-00-33.png)

5. ロール割り当てが割り当てリストに表示されることを確認します。

    ![](./media/Lab-00-34.png)

---

### タスク 5: Power Platform 管理センターで Copilot Studio 作成者を構成する

1. ブラウザの新しいタブを開き、`https://admin.powerplatform.microsoft.com` にアクセスします。

2. 左側のナビゲーション ペインで、**[管理] (1) > [環境] (2)** を選択します。**[+新規] (3)** をクリックします。

    ![](./media/Lab-00-35.png)

3. [新しい環境] ポップアップで、名前を **DevOne-<inject key="Deployment ID" enableCopy="false"></inject>** と指定します。

    ![](./media/Lab-00-36.png)

4. 画面を下にスクロールし、[既定の設定を変更する] ドロップダウンを展開して、**[Dataverse データ ストアを追加しますか? (1)]** を有効にし、**[次へ (2)]** をクリックします。

    ![](./media/Lab-00-37.png)

5. **Dataverseの追加**ページで、[セキュリティグループ] の下にある **[+選択してください]** をクリックします。

    ![](./media/Lab-00-38.png)

6. 検索結果から **[copilotagentsecurity (1)]** グループを選択します。その後、**[完了 (2)]** を選択します。

    ![](./media/Lab-00-39.png)

7. **[保存]** を選択して設定を適用します。

    ![](./media/Lab-00-40.png)

8. **[管理]** の下にある **[テナント設定]** を選択します。**[テナント設定]** ページで、リストから **[Copilot Studio 作成者]** を見つけて選択します。

    ![](./media/Lab-00-41.png)

9. **Copilot Studio 作成者** パネルで、セキュリティ グループの下にある **[編集]** アイコンを選択します。

    ![](./media/Lab-00-42.png)

10. 検索フィールドに `copilotagentsecurity` と入力します。検索結果から **copilotagentsecurity** グループを選択します。その後、**[完了]** をクリックします。

    ![](./media/Lab-00-43.png)

11. **[保存]** を選択して設定を適用します。

    ![](./media/Lab-00-44.png)

---

### タスク 6: Copilot Studio の Entra エージェント ID を有効にする

1. 左側のナビゲーションペインで、**Copilot** を選択します。

    ![](./media/Lab-00-45.png)

2. **Copilot** ページで、**[設定]** を選択します。

    ![](./media/Lab-00-46.png)

3. 設定リストの **Copilot Studio** セクションで、**[Copilot Studio の Entra エージェント ID]** を選択します。

    ![](./media/Lab-00-47.png)

4. **[Copilot Studio の Entra エージェント ID]** パネルで、環境リストから **DevOne-<inject key="Deployment ID" enableCopy="false"></inject>** 環境を選択し、**[設定の編集]** を選択します。

    ![](./media/Lab-00-48.png)

5. [Copilot Studio の Entra エージェント ID] の設定パネルで、まだ設定されていない場合は **[オン]** を選択し、**[保存]** をクリックします。

    ![](./media/Lab-00-49.png)

1. 保存後、パネルを閉じます。

    ![](./media/Lab-00-50.png)

      >**注:** Entra エージェント ID を有効にすると、Copilot Studio エージェントに Microsoft Entra ID 内で一意の ID が自動的に割り当てられます。これは、今後のラボにおける ID ガバナンス、条件付きアクセス、および Defender for Cloud Apps との統合に必要です。
---

### タスク 7: Power Apps Maker ポータルで SharePoint 接続を追加する

1. ブラウザの新しいタブを開き、`https://make.powerapps.com` にアクセスして、プロンプトが表示される場合は、**ODL_User** の認証情報でサインインします。

	- **メールアドレス/ユーザー名:** <inject key="AzureAdUserEmail"></inject>
    - **パスワード:** <inject key="AzureAdUserPassword"></inject>

2. **[Power Apps へようこそ]** 画面が表示されたら、 **[開始する]** をクリックします。

    ![](./media/Lab-00-51.png)

3. 右上隅の環境スイッチャーで、**DevOne-<inject key="Deployment ID" enableCopy="false"></inject>** 環境が選択されていることを確認します。選択されていない場合は、環境スイッチャーをクリックし、**DevOne-<inject key="Deployment ID" enableCopy="false"></inject>** を選択します。

    ![](./media/Lab-00-52.png)

4. 左側のナビゲーション バーで、**[詳細 (1)]** を展開し、**[接続 (2)]** を選択します。

    ![](./media/Lab-00-53.png)

5. **[接続]** ページで、**[+ 新しい接続]** を選択します。

    ![](./media/Lab-00-54.png)

6. コネクタの検索バーに `SharePoint` と入力し、利用可能なコネクタのリストから **[SharePoint]** を選択します。

    ![](./media/Lab-00-55.png)

7. **[SharePoint に接続する]** のパネルで、**直接接続 (クラウドサービス)** を選択します。**[作成]** を選択します。
 
	![](./media/Lab-00-56.png)

8. プロンプトが表示されたら、**ODL_User** の認証情報を使用してサインインし、接続を承認します。
   
    ![](./media/Lab-00-57.png)

9. [確認が必要です] というポップアップが表示されたら、[このリクエストを確認済みであり、このソースを信頼しています (1)] のチェックボックスをオンにし、[アクセスを許可 (2)] を選択します。

    ![](./media/Lab-00-58.png)

11. **[接続]** リストに SharePoint 接続が表示され、ステータスが **[接続済み]** になっていることを確認します。

    ![](./media/Lab-00-59.png)
---

## 演習 2: Zava Copilot Studio エージェントの作成

この演習では、Microsoft Copilot Studio で 3 つの Zava エージェントすべてを作成します。各エージェントには、名前、説明、操作手順、および SharePoint ナレッジソースが設定されます。公開後、各エージェントは指定されたラボユーザーアカウントと共有されます。これらのエージェントは、ラボ 01 から 07 までの実際のガバナンス対象として機能します。

---

### タスク 1: Zava HR アシスタント を作成する

1. ブラウザの新しいタブを開き、`https://copilotstudio.microsoft.com` にアクセスします。プロンプトが表示されたら、**ODL_User** の認証情報でサインインします。

    - **メールアドレス/ユーザー名:** <inject key="AzureAdUserEmail"></inject>
    - **パスワード:** <inject key="AzureAdUserPassword"></inject>

2. Copilot Studio が読み込まれない場合は、次の手順に従ってください。

    - `https://admin.powerplatform.microsoft.com/` を開きます。**[管理]** > **[環境]** > **[DevOne-<inject key="Deployment ID" enableCopy="false"></inject>]** を選択し、**[環境 ID]** の値をコピーします。
      
		![](./media/Lab-00-60.png)
   
   - [Copilot Studio] タブに戻り、`https://copilotstudio.microsoft.com/environments/<EnvironmentID>` を開きます（`<EnvironmentID>` は、上記でコピーした値に置き換えてください）。

3. **「ようこそ」** 画面で、**[開始する]** をクリックします。

	 ![](./media/Lab-00-61.png)

4. 左側のナビゲーション ペインで、**エージェント**を選択します。**エージェントの作成**ページで、**[空のエージェントを作成する]** を選択します。

     ![](./media/Lab-00-62.png)

5.  **名前** フィールドに `Zava HR アシスタント` と入力し、**[作成]** をクリックします。

     ![](./media/Lab-00-63.png)

6.  **[編集]** をクリックします。

     ![](./media/Lab-00-64.png)

7. **説明**フィールドに `Zava の従業員が HR ポリシー、福利厚生情報、従業員向け手続きを検索することを支援する AI アシスタント` と入力し、**[保存]**を選択します。

     ![](./media/Lab-00-65.png)

8. 画面を下にスクロールして**指示**フィールドまで移動し、**[編集]** をクリックして以下の内容を入力し、**[保存]** を選択します。

    ```
    あなたは、Zava HR アシスタントです。Zava HR SharePoint ナレッジ ベースに掲載されている情報のみを用いて質問に回答してください。推測による回答や、ナレッジ ベース以外の情報を提供しないでください。常にプロフェッショナルな対応を心がけてください。
    ```

     ![](./media/Lab-00-66.png)

9. エージェント設定ページで、**ナレッジ**セクションを見つけます。**[+ ナレッジを追加]** を選択します。

     ![](./media/Lab-00-67.png)

10. **ナレッジの追加** パネルで、**SharePoint** を選択します。

     ![](./media/Lab-00-68.png)

11. **SharePoint URL** フィールドに、SharePoint HR サイトの URL を以下の形式で入力し、**[追加]** を選択します。
    **https://[TenantPrefix].sharepoint.com/sites/HR<inject key="Deployment ID" enableCopy="false"></inject>**

    > **注:** `[TenantPrefix]` を、ラボ環境の **環境** タブに記載されているテナントプレフィックスに置き換えるか、演習 0 の URL を使用してください。

     ![](./media/Lab-00-69.png)

12. **[エージェントに追加する]** を選択し、SharePoint サイトをナレッジ ソースとして接続します。

     ![](./media/Lab-00-70.png)

13. エージェント設定ページの右上隅で、**[公開する]** を選択します。

     ![](./media/Lab-00-71.png)

14. 確認ダイアログで、**公開**を選択して確定します。

     ![](./media/Lab-00-72.png)

15. エージェント設定ページの上部にある**[チャネル]**タブを見つけます（直接表示されていない場合は、**+**を選択してください）。

     ![](./media/Lab-00-73.png)

16. **Microsoft 365 と Microsoft Teams** を選択して、チャネルとして追加します。

     ![](./media/Lab-00-74.png)

17. 次に、**チャネルの追加** を選択します。

     ![](./media/Lab-00-75.png)

18. **利用設定** を選択します。

     ![](./media/Lab-00-76.png)

19. **Microsoft 365 Copilot および Microsoft Teams** ページで、**組織内の全員に表示** を選択します。

     ![](./media/Lab-00-77.png)

20. **組織カタログに送信** を選択します。

     ![](./media/Lab-00-78.png)

21. **「このエージェントへのアクセス権を全員に付与しますか？」**という確認ダイアログで、**「はい」**を選択します。

     ![](./media/Lab-00-79.png)

22. **組織の Teams アプリ ストアに表示** ページにリダイレクトされ、**エージェントが送信され、Teams 管理者による承認待ちです** という通知が表示されます。[**閉じる**] をクリックします。

     ![](./media/Lab-00-80.png)

---

### Task 2: Create the Zava Finance Agent

1. In the left navigation pane, select **Agents**. Then select **Create blank agent**.

	![](./media/pp22.png)

3. In the **Name** field, enter `Zava Finance Agent` and click on **Create**.

	![](./media/pp50.png)

4. In the **Description** field, enter `An AI assistant that helps Zava finance team members retrieve budget information, invoice data, and financial reports.` Then select **Save**.

5. In the **Instructions** field, select **Edit**.

6. Enter the following and select **Save**.

    ```
    You are the Zava Finance Agent. Answer questions using only the information in the Zava Finance SharePoint knowledge base. Do not share financial data with users who have not been granted access to the Finance SharePoint site. Always respond professionally and flag any requests for data outside your knowledge base.
    ```

	![](./media/pp51.png)

7. Scroll down and on the agent configuration page, locate the **Knowledge** section. Select **+ Add knowledge**.

    ![](./media/kn.png) 

8. On the **Add knowledge** panel, select **SharePoint**.

	![](./media/image82.png)

9. In the **SharePoint URL** field, enter the SharePoint Finance site URL in the following format:
    **https://[TenantPrefix].sharepoint.com/sites/Operations<inject key="Deployment ID" enableCopy="false"></inject>**

    > **Note:** Replace `[TenantPrefix]` with your tenant prefix from the **Environment** tab.

10. Select **Add** to connect the SharePoint site as the knowledge source.

	![](./media/pp52.png)

11. Then select **Add to agent**.

    ![](./media/l0e2t2s10.png)

12. On the agent configuration page, locate the **Channels** tab on the top section (select **+** if it is not directly visible).

	![](./media/pp53.png)

13. Select **Microsoft 365 Copilot and Microsoft Teams** to add them as channels.

	![](./media/pp54.png)

14. Then select **Add channel**.

	![](./media/L00-E2-T2-S14.png)

15. In the **Ready to publish?** dialog, select **Publish**. Close the tab.

	![](./media/image89.png)
---

### Task 3: Create the Zava IT Support Agent

1. In the left navigation pane, select **Agents**. Then select **Create blank agent**.

	![](./media/pp22.png)

3. In the **Name** field, enter `Zava IT Support Agent` and click on **Create**.

	![](./media/pp55.png)

4. In the **Description** field, enter `An AI assistant that helps Zava employees resolve common IT issues, submit support requests, and find IT policy documentation.` Then select **Save**.

5. In the **Instructions** field, select **Edit**.

6. Enter the following and select **Save**.

    ```
    You are the Zava IT Support Agent. Help users with common IT questions using publicly available Microsoft support documentation and Zava IT policies. Do not access or share any sensitive financial or HR information. Escalate complex issues to the IT helpdesk.
    ```

	 ![](./media/pp56.png)

7. On the agent configuration page, locate the **Knowledge** section. Select **+ Add knowledge**.

    ![](./media/kn.png) 

8. On the **Add knowledge** panel, select **Public Websites**.

	![](./media/image103.png)

9. In the **URL** field, enter `https://support.microsoft.com/` and select **Add** to connect the site as the knowledge source.

	![](./media/image104.png)

10. Then, select **Add to agent**.

	![](./media/image105.png)

11. In the top-right corner of the agent configuration page, select **Publish**.

	![](./media/L00-E2-T3-S11.png)

12. In the confirmation dialog, select **Publish** to confirm.

	![](./media/image89.png)

13. On the agent configuration page, locate the **Channels** tab on the top section (select **+** if it is not directly visible).

	![](./media/pp58.png)

15. Select **Microsoft 365 Copilot and Microsoft Teams** to add them as channels.

	![](./media/L00-E2-T3-S14.png)

16. Then select **Add channel**.

	![](./media/L00-E2-T3-S15.png)

17. Select **Availability options**.

	![](./media/L00-E2-T3-S16.png)

18. On the **Microsoft 365 Copilot and Microsoft Teams** page, select **Show to everyone in my org**.

	![](./media/L00-E2-T3-S17.png)

19. Select **Submit to org catalog**.

	![](./media/L00-E2-T3-S18.png)

20. On the **Give everyone access to this agent?** confirmation dialog, select **Yes**.

	![](./media/image114.png)

21. You will be redirected to **Show in Teams app store for org** and see a notification: **Your agent is submitted and waiting for approval from your Teams admin**. Close the tab.

---

## Exercise 3: Upload Zava Knowledge Files to SharePoint

In this exercise, you will uploads the Zava sample business documents to the SharePoint HR and Finance sites. These files contain the sensitive data — including employee PII, payroll records, credit card numbers, and financial forecasts — that will trigger security detections and DLP policy matches throughout Labs 04, 05, and 07.

---

### Task 1: Upload Files to the Zava HR SharePoint Site

1. Open a new browser tab and navigate to **https://[TenantPrefix].sharepoint.com/sites/HR<inject key="Deployment ID" enableCopy="false"></inject>**.

   > **Note:** Replace `[TenantPrefix]` with your tenant prefix from the **Environment** tab.

3. From the left navigation menu, click on **Documents (1)** , select **Create or upload (2)**. Then, select **Files upload (3)**.

	![](./media/pp59.png)

4. In the file picker, navigate to the **C:\LabFiles\lab file\HR** folder on your lab VM desktop.

5. Select the following files and then select **Open** to upload them:

   | Filename | Contains |
   |---|---|
   | `Zava_HR_Policy_2024.docx` | Leave and disciplinary policy — no PII |
   | `Zava_Employee_Records.xlsx` | Employee IDs (format: ZVA123456), names, DOB, salary |
   | `Zava_Payroll_Q1_2025.xlsx` | Payroll data with credit card numbers in expense column |
   | `Zava_Onboarding_Guide.docx` | Standard onboarding content |
   | `Zava_Benefits_Summary.pdf` | Insurance and pension details |
   | `Zava_Org_Chart.docx` | Reporting lines and management structure |
   | `Zava_Termination_Checklist.docx` | Departing employee process with names and dates |
   | `Zava_Sick_Leave_Report.xlsx` | Employee names and illness reasons |

6. Wait for all 8 files to finish uploading.

7. On the **Documents** page, confirm that all 8 files appear in the document library.

	![](./media/l0e3t1s6.png)

---

### Task 2: Upload Files to the Zava Finance SharePoint Site

1. Open a new browser tab and navigate to **https://[TenantPrefix].sharepoint.com/sites/Operations<inject key="Deployment ID" enableCopy="false"></inject>**.

   > **Note:** Replace `[TenantPrefix]` with your tenant prefix from the **Environment** tab.

2. From the left navigation menu, click on **Documents (1)** , select **Create or upload (2)**. Then, select **Files upload (3)**.
   
	![](./media/pp60.png)

4. In the file picker, navigate to the **C:\LabFiles\lab file\Operations** folder on your lab VM desktop.

5. Select the following files and then select **Open** to upload them:

   | Filename | Contains |
   |---|---|
   | `Zava_Budget_2025.xlsx` | Department budgets and cost centres |
   | `Zava_Invoice_Log.xlsx` | Vendor invoices with IBAN and account numbers |
   | `Zava_Expense_Report_Alex.xlsx` | Alex Wilber's expenses with Visa credit card number |
   | `Zava_Audit_Report_2024.docx` | Internal audit findings — marked Confidential |
   | `Zava_Contracts_External.docx` | Third-party vendor contract — externally shared |
   | `Zava_Financial_Projections.xlsx` | Revenue forecasts with broad SharePoint permissions |

6. Wait for all 6 files to finish uploading.

7. On the **Documents** page, confirm that all 6 files appear in the document library.

    ![](./media/l0e3t2s6.png)

---

### Task 3: Verify Agents in the Microsoft Agent 365 Agent Registry

1. Open a new browser tab and navigate to `https://admin.cloud.microsoft/`. Sign in with **ODL_User** credentials if prompted.

	- **Email/Username:** <inject key="AzureAdUserEmail"></inject>

	- **Password:** <inject key="AzureAdUserPassword"></inject>

2. In the left navigation pane, expand **Agents** and then select **All agents**.

    ![](./media/l0e3t3s1.png)

3. On this page, confirm that the following three agents appear in the list. You can search for `Zava` in the search box to filter the results.

   | Agent Name | Status |
   |---|---|
   | Zava HR Assistant | Available | 
   | Zava Finance Agent | Available | 
   | Zava IT Support Agent | Available |

	  ![](./media/pp61.png)

	  >**Note:** It may take up to 10 minutes after publishing in Copilot Studio for agents to appear in the Agent Registry. If the agents are not visible, wait 10 minutes and then refresh the page.
	  
---

## Exercise 4: Enable Organizational Setup

1. Navigate to **Exchange Admin Center** using the below URL

    ```
    https://admin.cloud.microsoft/exchange
	```
1. Sign in with **ODL_User** credentials if prompted.

	- **Email/Username:** <inject key="AzureAdUserEmail"></inject>
	- **Password:** <inject key="AzureAdUserPassword"></inject>

1. In the Exchange admin center, select the **Cloud Shell icon** from the upper-right corner of the page to launch an Azure Cloud Shell session

	![](./media/ex-1.png)

	>**Note**: If prompted, complete the Cloud Shell initialization before proceeding.

1. After the Cloud Shell session is ready and displays the PowerShell prompt, run the following command to disconnect the current Exchange Online session

    ```
    Disconnect-ExchangeOnline -Confirm:$false	
	```

	![](./media/ex-2.png)

1. Connect to Exchange Online Using Device Authentication
by running the following command to initiate a new Exchange Online connection using device authentication:

    ```
    Connect-ExchangeOnline -Device
    ```

	![](./media/ex-3.png)

    - Note: A device code and sign-in URL will be displayed. Open the URL and paste the code to complete the authentication

	    ![](./media/ex-4.png)

		![](./media/ex-7.png)

1. After the Exchange Online PowerShell session is successfully connected, run the following command to enable organization customization:

    ```
    Enable-OrganizationCustomization
	```

	![](./media/ex-8.png)

     >**Note**: This command prepares the Exchange Online organization for advanced configuration tasks. If organization customization has already been enabled, the command returns a message indicating that no further action is required. Continue with the next step in the lab. This may take upto 24 hours to get organization custimaztion enabled
	

---

## Summary

In this lab, you completed the full environment baseline for the Zava Corporation AI security course. You created a role-assignable security group in the Microsoft Entra admin center, configured ODL User as owner and member, assigned the Privileged Role Administrator role, and enabled the group as the authorised Copilot Studio Authors group in Power Platform Admin Center. You enabled Entra Agent Identity for Copilot Studio at the environment level, added a SharePoint connection in the Power Apps maker portal, and created three Copilot Studio agents — Zava HR Assistant, Zava Finance Agent, and Zava IT Support Agent — each connected to a designated knowledge source, published across Teams and Microsoft 365 channels. You uploaded 14 sample business documents containing realistic sensitive data across the Zava HR and Finance SharePoint sites, and verified that all three agents are registered and Active in the Microsoft Agent 365 Agent Registry. The environment is now fully prepared for security configuration in Labs 01 through 07.

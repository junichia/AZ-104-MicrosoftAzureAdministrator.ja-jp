---
lab:
  title: 02a - サブスクリプションと RBAC を管理する
  module: Module 02 - Governance and Compliance
---

# <a name="lab-02a---manage-subscriptions-and-rbac"></a>ラボ 02a - サブスクリプションと RBAC を管理する
# <a name="student-lab-manual"></a>受講生用ラボ マニュアル

## <a name="lab-requirements"></a>ラボの要件

## <a name="lab-scenario"></a>ラボのシナリオ

Contoso の Azure リソースの管理を強化するために、次の機能を実装する任務を負いました。

- Contoso のすべての Azure サブスクリプションを含む管理グループを作成する

- granting permissions to submit support requests for all subscriptions in the management group to a designated Azure Active Directory user. That user's permissions should be limited only to: 

    - サポート要求チケットの作成
    - リソース グループの表示 

## <a name="objectives"></a>目標

このラボでは、次のことを行います。

+ タスク 1:管理グループを実装する
+ タスク 2:カスタム RBAC ロールを作成する 
+ タスク 3:RBAC ロールを割り当てる


## <a name="estimated-timing-30-minutes"></a>推定時間:30 分

## <a name="architecture-diagram"></a>アーキテクチャの図

![image](../media/lab02a.png)


## <a name="instructions"></a>Instructions

### <a name="exercise-1"></a>演習 1

#### <a name="task-1-implement-management-groups"></a>タスク 1:管理グループを実装する

このタスクでは、管理グループを作成および構成します。 

1. [**Azure Portal**](http://portal.azure.com) にサインインします。

1. 「**管理グループ**」を検索して選択し、**[管理グループ]** ブレードに移動します。

1. 管理グループブレードの上部に表示されるメッセージを確認します。「You are registered as a directory admin but do not have the permissions to access the root management group」というメッセージが表示されている場合は、次の一連の手順を実行してください。

    1. Azure portal で、 **[Azure Active Directory]** を検索して選択します。
    
    1.  Azure Active Directory テナントのプロパティを表示しているブレードの、左側の垂直方向に配置されたメニュー内で **[管理]** セクションから **[プロパティ]** を選択します。
    
    1.  Azure Active Directory テナントの **[プロパティ]** ブレードの、**[Azure リソースのアクセス管理]** セクションで、**[はい]** を選択し、さらに **[保存]** を選択します。
    
    1.  **「管理グループ」** ブレードに戻り、 **「更新」** をクリックします。

1. **[管理グループ]** ブレードで、**[+ 作成]** をクリックします。

    >**注**: 以前に管理グループを作成していない場合は、**[管理グループの使用を開始]** を選択します

1. 次の設定で管理グループを作成します。

    | 設定 | [値] |
    | --- | --- |
    | 管理グループ ID | **az104-02-mg1** |
    | 管理グループの表示名 | **az104-02-mg1** |

1. Tenant Root Groupの配下に、Azure Passサブスクリプションがあることを確認します。

1. Azure Passサブスクリプションのエントリの右端にある「・・・」をクリックし、「移動」をクリックします。

1. **「az104-02-mg1」** を選択し、「保存」をクリックします。

1. **[az104-02-mg1]** を開き、Azure Passサブスクリプションの横に表示されているサブスクリプションの ID をクリップボードにコピーします。 これは、次のタスクで必要になります。

#### <a name="task-2-create-custom-rbac-roles"></a>タスク 2:カスタム RBAC ロールを作成する

このタスクでは、カスタム RBAC ロールの定義を作成します。

1. ラボのコンピューターから **[\\Allfiles\\Labs\\02\\az104-02a-customRoleDefinition.json](https://github.com/junichia/AZ-104-MicrosoftAzureAdministrator.ja-jp/tree/main/Allfiles/Labs/02)** ファイルをメモ帳で開き、その内容を確認します。

   ```json
   {
      "Name": "Support Request Contributor (Custom)",
      "IsCustom": true,
      "Description": "Allows to create support requests",
      "Actions": [
          "Microsoft.Resources/subscriptions/resourceGroups/read",
          "Microsoft.Support/*"
      ],
      "NotActions": [
      ],
      "AssignableScopes": [
          "/providers/Microsoft.Management/managementGroups/az104-02-mg1",
          "/subscriptions/SUBSCRIPTION_ID"
      ]
   }
   ```
    > **注**:ファイルがラボ環境内のローカルのどこに格納されているかが分からない場合は、インストラクターに確認してください。

1. JSON ファイルの `SUBSCRIPTION_ID` プレースホルダーを、クリップボードにコピーしたサブスクリプション ID に置き換えて、変更を保存します。

1. Azure portal で、**[Cloud Shell]** ウィンドウを開くには、検索テキストボックスの右側にあるツールバー アイコンを直接クリックします。

1. **Bash** または **PowerShell** の選択を求めるメッセージが表示されたら、 **[PowerShell]** を選択します。 

    >**注**: **Cloud Shell** の初回起動時に **"ストレージがマウントされていません"** というメッセージが表示された場合は、このラボで使用しているサブスクリプションを選択し、**[ストレージの作成]** を選択します。 

1. [Cloud Shell] ペインのツールバーで、 **[ファイルのアップロード/ダウンロード]** アイコンをクリックし、ドロップダウン メニューの **[アップロード]** をクリックして、ファイル **\\Allfiles\\Labs\\02\\az104-02a-customRoleDefinition.json** を Cloud Shell のホーム ディレクトリにアップロードします。

1. Cloud Shell ウィンドウから、次の操作を実行して、カスタム ロール定義を作成します。

   ```powershell
   New-AzRoleDefinition -InputFile $HOME/az104-02a-customRoleDefinition.json
   ```

1. [Cloud Shell] ペインを閉じます。

#### <a name="task-3-assign-rbac-roles"></a>タスク 3:RBAC ロールを割り当てる

このタスクでは、Azure Active Directory ユーザーを作成し、前のタスクで作成した RBAC ロールをそのユーザーに割り当て、ユーザーが RBAC ロール定義で指定されたタスクを実行できることを確認します。

1. Azure portal の [Azure Active Directory] ブレードで、「**Azure Active Directory**」を検索して選択し、**[ユーザー]** をクリックして、**[新しいユーザー]** をクリックします。

1. 次の設定で、新しいユーザーを作成します (他の設定は既定値のままにします)。

    | 設定 | 値 |
    | --- | --- |
    | ユーザー名 | **az104-02-aaduser1**|
    | 名前 | **az104-02-aaduser1**|
    | 自分でパスワードを作成する | enabled |
    | 初期パスワード | **セキュリティで保護されたパスワードを指定する** |

    ><bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: <bpt id="p2">**</bpt>Copy to clipboard<ept id="p2">**</ept> the full <bpt id="p3">**</bpt>User name<ept id="p3">**</ept>. You will need it later in this lab.

1. Azure portal で、**az104-02-mg1** 管理グループを表示します。

1. **[アクセス制御 (IAM)]** をクリックし、 **[+ 追加]** 、 **[ロールの割り当て]** の順にクリックして、新しく作成したユーザー アカウントに **Support Request Contributor (Custom)** の役割を割り当てます。

    >**注**: カスタム役割が表示されない場合、作成後にカスタム役割が表示されるまでに最大 10 分かかることがあります。

1. Azure portal の **[InPrivate]** ブラウザー ウィンドウで、**[リソース グループ]** を検索して選択し、az104-02-aaduser1 ユーザーがすべてのリソース グループを表示できることを確認します。

1. Azure portal の **[InPrivate]** ブラウザー ウィンドウで、**[すべてのリソース]** を検索して選択し、az104-02-aaduser1 ユーザーにリソースを表示できないことを確認します。

1. Azure portal の **[InPrivate]** ブラウザー ウィンドウで、**[ヘルプとサポート]** を検索して選択し、**[サポートリクエストの作成]** をクリックします。 

1. 新しいサポートリクエスト の画面で、**問題の種類** として **サービスとサブスクリプションの制限** を選択します。このラボで使用しているサブスクリプションが[Subscription]ドロップダウン リストに表示されていることに注意してください。

    >**注**: このラボで使用しているサブスクリプションが **[サブスクリプション]** ドロップダウン リストに表示されている場合、使用しているアカウントに、サブスクリプション固有のサポート要求を作成するために必要なアクセス許可があることを示しています。

    >**注**: **[サービスとサブスクリプションの制限 (クォータ)]** オプションが表示されない場合は、Azure portal からサインアウトしてログインし直します。

1. サポート リクエストの作成は続けないでください。代わりに、Azure ポータルから az104-02-aaduser1 ユーザーとしてサインアウトし、InPrivate ブラウザー ウィンドウを閉じます。

#### <a name="task-4-clean-up-resources"></a>タスク 4: リソースをクリーンアップする

   ><bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: Remember to remove any newly created Azure resources that you no longer use. Removing unused resources ensures you will not see unexpected charges, although, resources created in this lab do not incur extra cost.

   ><bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: Don't worry if the lab resources cannot be immediately removed. Sometimes resources have dependencies and take a longer time to delete. It is a common Administrator task to monitor resource usage, so just periodically review your resources in the Portal to see how the cleanup is going.

1. Azure portal で **[Azure Active Directory]** を検索して選択し、[Azure Active Directory] ブレードで、**[ユーザー]** をクリックします。

1. **[ユーザー - すべてのユーザー]** ブレードで **[az104-02-aaduser1]** をクリックします。

1. **[az104-02-aaduser1 - プロファイル]** ブレードで、**[オブジェクト ID]** 属性の値をコピーします。

1. Azure portal 内の **[Cloud Shell]** で **PowerShell** セッションを開始します。

1. [Cloud Shell] ペインで、次の手順を実行してカスタム ロール定義の割り当てを削除します (`[object_ID]` プレースホルダーを、このタスクの前半でコピーした **az104-02-aaduser1** の Azure Active Directory ユーザー アカウントの **object ID** 属性の値で置き換えます)。

   ```powershell
   
   $scope = (Get-AzRoleDefinition -Name 'Support Request Contributor (Custom)').AssignableScopes[0]

   Remove-AzRoleAssignment -ObjectId '[object_ID]' -RoleDefinitionName 'Support Request Contributor (Custom)' -Scope $scope
   ```

1. [Cloud Shell] ウィンドウから次の手順を実行して、カスタム ロールの定義を削除します。

   ```powershell
   Remove-AzRoleDefinition -Name 'Support Request Contributor (Custom)' -Force
   ```

1. Azure portal で、**Azure Active Directory** の **[ユーザー - すべてのユーザー]** ブレードに戻り、**az104-02-aaduser1** ユーザー アカウントを削除します。

1. Azure portal で、**[管理グループ]** ブレードに戻ります。 

1. **[管理グループ]** ブレードで、**az104-02-mg1** 管理グループの下のサブスクリプションの横にある**省略記号**アイコンを選択し、**[移動]** を選択してサブスクリプションを **[テナント ルート管理グループ]** に移動します。

   >**注**:このラボを実行する前にカスタム管理グループ階層を作成した場合を除き、ターゲットの管理グループが**テナント ルート管理グループ**であると考えられます。
   
1. **[更新]** を選択して、サブスクリプションが **[テナント ルート管理グループ]** に正常に移動したことを確認します。

1. **[管理グループ]** ブレードに戻り、 **[az104-02-mg1]** 管理グループの右側にある**省略記号**アイコンをクリックし、 **[削除]** をクリックします。
  >管理グループ内のすべてのサブスクリプションに対するサポート要求を、指定された Azure Active Directory ユーザーに提出するアクセス許可を付与する。

#### <a name="review"></a>確認

このラボでは、次のことを行いました。

- 管理グループを作成しました
- カスタム RBAC ロールを作成しました 
- RBAC ロールを割り当てました

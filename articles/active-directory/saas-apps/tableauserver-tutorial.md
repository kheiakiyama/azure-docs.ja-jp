---
title: 'チュートリアル: Azure Active Directory と Tableau Server の統合 | Microsoft Docs'
description: Azure Active Directory と Tableau Server の間でシングル サインオンを構成する方法について説明します。
services: active-directory
documentationCenter: na
author: jeevansd
manager: femila
ms.reviewer: joflore
ms.assetid: c1917375-08aa-445c-a444-e22e23fa19e0
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/12/2018
ms.author: jeedes
ms.openlocfilehash: c727cddf41c269c214b541134cd9f688017ee687
ms.sourcegitcommit: 295babdcfe86b7a3074fd5b65350c8c11a49f2f1
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/27/2018
ms.locfileid: "53789723"
---
# <a name="tutorial-azure-active-directory-integration-with-tableau-server"></a>チュートリアル: Azure Active Directory と Tableau Server の統合

このチュートリアルでは、Tableau Server と Azure Active Directory (Azure AD) を統合する方法について説明します。

Tableau Server と Azure AD の統合には、次の利点があります。

- Tableau Server にアクセスするユーザーを Azure AD で管理できます。
- ユーザーが各自の Azure AD アカウントで Tableau Server に自動的にサインオン (シングル サインオン) するように、設定が可能です。
- 1 つの中央サイト (Azure Portal) でアカウントを管理できます。

SaaS アプリと Azure AD の統合の詳細については、「[Azure Active Directory のアプリケーション アクセスとシングル サインオンとは](../manage-apps/what-is-single-sign-on.md)」をご覧ください。

## <a name="prerequisites"></a>前提条件

Azure AD と Tableau Server の統合を構成するには、次のものが必要です。

- Azure AD サブスクリプション
- Tableau Server のシングル サインオンが有効なサブスクリプション

> [!NOTE]
> このチュートリアルの手順をテストする場合、運用環境を使用しないことをお勧めします。

このチュートリアルの手順をテストするには、次の推奨事項に従ってください。

- 必要な場合を除き、運用環境は使用しないでください。
- Azure AD の評価環境がない場合は、[1 か月の評価版を入手できます](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>シナリオの説明

このチュートリアルでは、テスト環境で Azure AD のシングル サインオンをテストします。 このチュートリアルで説明するシナリオは、主に次の 2 つの要素で構成されています。

1. ギャラリーから Tableau Server を追加する
2. Azure AD シングル サインオンの構成とテスト

## <a name="adding-tableau-server-from-the-gallery"></a>ギャラリーから Tableau Server を追加する

Azure AD への Tableau Server の統合を構成するには、ギャラリーから管理対象 SaaS アプリの一覧に Tableau Server を追加する必要があります。

**ギャラリーから Tableau Server を追加するには、次の手順に従います。**

1. **[Azure Portal](https://portal.azure.com)** の左側のナビゲーション ウィンドウで、**[Azure Active Directory]** アイコンをクリックします。 

    ![Azure Active Directory のボタン][1]

2. **[エンタープライズ アプリケーション]** に移動します。 次に、**[すべてのアプリケーション]** に移動します。

    ![[エンタープライズ アプリケーション] ブレード][2]
    
3. 新しいアプリケーションを追加するには、ダイアログの上部にある **[新しいアプリケーション]** をクリックします。

    ![[新しいアプリケーション] ボタン][3]

4. 検索ボックスに「**Tableau Server**」と入力し、結果ウィンドウで **[Tableau Server]** を選び、**[追加]** をクリックして、アプリケーションを追加します。

    ![結果リストの Tableau Server](./media/tableauserver-tutorial/tutorial-tableauserver-addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Azure AD シングル サインオンの構成とテスト

このセクションでは、"Britta Simon" というテスト ユーザーに基づいて、Tableau Server で Azure AD のシングル サインオンを構成し、テストします。

シングル サインオンを機能させるには、Azure AD ユーザーに対応する Tableau Server ユーザーが Azure AD で認識されている必要があります。 言い換えると、Azure AD ユーザーと Tableau Server の関連ユーザーの間で、リンク関係が確立されている必要があります。

Tableau Server で Azure AD のシングル サインオンを構成してテストするには、次の手順を完了する必要があります。

1. **[Azure AD シングル サインオンの構成](#configure-azure-ad-single-sign-on)** - ユーザーがこの機能を使用できるようにします。
2. **[Tableau Server シングル サインオンの構成](#configure-tableau-server-single-sign-on)** - アプリケーション側でシングル サインオン設定を構成します。
3. **[Azure AD のテスト ユーザーの作成](#create-an-azure-ad-test-user)** - Britta Simon で Azure AD のシングル サインオンをテストします。
4. **[Tableau Server のテスト ユーザーの作成](#create-tableau-server-test-user)** - Cisco Umbrella で Britta Simon に対応するユーザーを作成し、Azure AD の Britta Simon にリンクさせます。
5. **[Azure AD テスト ユーザーの割り当て](#assign-the-azure-ad-test-user)** - Britta Simon が Azure AD シングル サインオンを使用できるようにします。
6. **[シングル サインオンのテスト](#test-single-sign-on)** - 構成が機能するかどうかを確認します。

### <a name="configure-azure-ad-single-sign-on"></a>Azure AD シングル サインオンの構成

このセクションでは、Azure Portal で Azure AD のシングル サインオンを有効にし、Tableau Server アプリケーションでシングル サインオンを構成します。

**Tableau Server で Azure AD シングル サインオンを構成するには、次の手順に従います。**

1. Azure Portal の **Tableau Server** アプリケーション統合ページで、**[シングル サインオン]** をクリックします。

    ![シングル サインオン構成のリンク][4]

2. **[シングル サインオン方式の選択]** ダイアログで、**[SAML]** モードの **[選択]** をクリックして、シングル サインオンを有効にします。

    ![Configure single sign-on](common/tutorial-general-301.png)

3. Tableau Server アプリケーションは、カスタム要求**ユーザー名**を要求します。ユーザー名は次のように定義する必要があります。 これは、一意のユーザー識別子要求の代わりにユーザー識別子として使用されています。 これらの属性の値は、アプリケーション統合ページの **[ユーザー属性と要求]** セクションで管理できます。 **[編集]** ボタンをクリックして、**[ユーザー属性と要求]** ダイアログを開きます。

    ![image](./media/tableauserver-tutorial/tutorial-tableauserver-attribute.png)

4. **[ユーザー属性と要求]** ダイアログの **[ユーザーの要求]** セクションで、上の図のように SAML トークン属性を構成し、次の手順を実行します。
    
    | 属性名 | 属性値 | 名前空間 |
    | ---------------| --------------- | ----------- |   
    | username | user.userprincipalname | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims` |

    a. **[新しい要求の追加]** をクリックして **[ユーザー要求の管理]** ダイアログを開きます。

    ![image](./media/tableauserver-tutorial/tutorial-tableauserver-add-attribute.png)

    ![image](./media/tableauserver-tutorial/tutorial-tableauserver-manage-attribute.png)

    b. **[名前]** ボックスに、その行に対して表示される属性名を入力します。

    c. **[名前空間]** 値を入力します。

    d. [ソース] として **[属性]** を選択します。

    e. **[ソース属性]** の一覧から、その行に表示される属性値を入力します。

    f. **[Save]** をクリックします。

5. **[SAML でシングル サインオンをセットアップします]** ページで、**[編集]** アイコンをクリックして **[基本的な SAML 構成]** ダイアログを開きます。

    ![Configure single sign-on](common/editconfigure.png)

6. **[基本的な SAML 構成]** セクションで、次の手順を実行します。

    a. **[サインオン URL]** ボックスに、`https://azure.<domain name>.link` のパターンを使用して URL を入力します。
    
    b. **[識別子]** ボックスに、`https://azure.<domain name>.link` の形式で URL を入力します。

    c. **[応答 URL]** ボックスに、`https://azure.<domain name>.link/wg/saml/SSO/index.html` のパターンを使用して URL を入力します。

    ![image](./media/tableauserver-tutorial/tutorial-tableauserver-url.png)
     
    > [!NOTE] 
    > 上記の値は、実際の値ではありません。 [Tableau Server Configiuration]\(Tableau Server の構成) ページから入手した実際の URL と識別子で値を更新します。これについてはこのチュートリアルで後ほど説明します。

7. **[SAML 署名証明書]** ページの **[SAML 署名証明書]** セクションで、**[ダウンロード]** をクリックして**フェデレーション メタデータの XML** をダウンロードし、証明書ファイルをコンピューターに保存します。

    ![証明書のダウンロードのリンク](./media/tableauserver-tutorial/tutorial-tableauserver-certificate.png)

### <a name="configure-tableau-server-single-sign-on"></a>Tableau Server のシングル サインオンを構成する 

1. アプリケーションに合わせて SSO を構成するには、管理者として Tableau Server テナントにサインオンする必要があります。

2. **[構成]** タブで、**[User Identity & Access]\(ユーザー ID とアクセス\)** を選択し、**[Authentication Method]\(認証方法\)** タブを選択します。

    ![Configure single sign-on](./media/tableauserver-tutorial/tutorial-tableauserver-auth.png)

3. **[CONFIGURATION]\(構成\)** ページで、次の手順を実行します。

    ![Configure single sign-on](./media/tableauserver-tutorial/tutorial-tableauserver-config.png)

    a. **[Authentication Method]\(認証方法\)** として SAML を選択します。
    
    b. **[Enable SAML Authentication for the server]\(サーバーの SAML 認証を有効にする\)** のチェック ボックスをオンにします。

    c. [Tableau Server return URL]\(Tableau Server の戻り先 URL\): Tableau Server ユーザーがアクセスする URL (http://tableau_server など)。 http://localhost の使用は推奨されません。 末尾にスラッシュが付いている URL (例: http://tableau_server/) はサポートされていません。 **[Tableau Server return URL]\(Tableau Server の戻り先 URL\)** をコピーし、Azure AD の **[Tableau Server のドメインと URL]** セクションにある **[サインオン URL]** ボックスに貼り付けます。

    d. [SAML entity ID]: IdP に対して Tableau Server のインストールを一意に識別するエンティティ ID。 必要に応じてこの欄にも Tableau Server URL を入力できますが、使用する Tableau Server URL にする必要はありません。 **[SAML entity ID]\(SAML エンティティ ID\)** をコピーし、Azure AD の **[Tableau Server のドメインと URL]** セクションにある **[識別子]** ボックスに貼り付けます。

    e. **[Download XML Metadata File]\(XML メタデータ ファイルのダウンロード\)** をクリックし、テキスト エディター アプリケーションで開きます。 Http Post で Index 0 の [Assertion Consumer Service URL] を探し、URL をコピーします。 Azure AD の **[Tableau Server のドメインと URL]** セクションにある **[応答 URL]** ボックスに貼り付けます。

    f. Azure Portal からダウンロードしたフェデレーション メタデータ ファイルを検索し、**[SAML Idp metadata file]\(SAML Idp メタデータ ファイル\)** でアップロードします。

    g. ユーザー名、表示名、メール アドレスを IdP が保持するために使用する属性の名前を入力します。

    h. **[保存]**

    >[!NOTE] 
    >顧客は任意の証明書を Tableau Server の SAML SSO 構成でアップロードする必要があります。SSO フローではその証明書は無視されます。
    >Tableau Server で SAML を構成する方法について不明な点がある場合は、[SAML の構成](https://onlinehelp.tableau.com/v2018.2/server/en-us/saml_config_steps_tsm_ui.htm)に関する記事を参照してください。

### <a name="create-an-azure-ad-test-user"></a>Azure AD のテスト ユーザーの作成

このセクションの目的は、Azure Portal で Britta Simon というテスト ユーザーを作成することです。

1. Azure portal の左側のウィンドウで、**[Azure Active Directory]**、**[ユーザー]**、**[すべてのユーザー]** の順に選択します。

    ![Azure AD ユーザーの作成][100]

2. 画面の上部にある **[新しいユーザー]** を選択します。

    ![Azure AD のテスト ユーザーの作成](common/create-aaduser-01.png) 

3. [ユーザーのプロパティ] で、次の手順を実行します。

    ![Azure AD のテスト ユーザーの作成](common/create-aaduser-02.png)

    a. **[名前]** フィールドに「**BrittaSimon**」と入力します。
  
    b. **[ユーザー名]** フィールドに「**brittasimon@yourcompanydomain.extension**」と入力します  
    たとえば、BrittaSimon@contoso.com のように指定します。

    c. **[プロパティ]** を選択し、**[パスワードを表示]** チェック ボックスをオンにして、[パスワード] ボックスに表示された値を書き留めます。

    d. **作成**を選択します。
  
### <a name="create-tableau-server-test-user"></a>Tableau Server のテスト ユーザーの作成

このセクションの目的は、Tableau Server で Britta Simon というユーザーを作成することです。 Tableau Server 内のすべてのユーザーをプロビジョニングする必要があります。 

また、ユーザーのユーザー名は、Azure AD のカスタム属性 **username** で構成した値と一致する必要があります。 正しい対応付けがあれば、統合で「 [Azure AD シングル サインオンの構成](#configuring-azure-ad-single-sign-on)」が機能します。

>[!NOTE]
>ユーザーを手動で作成する必要がある場合は、組織の Tableau Server 管理者に問い合わせてください。

### <a name="assign-the-azure-ad-test-user"></a>Azure AD テスト ユーザーの割り当て

このセクションでは、Britta Simon に Tableau Server へのアクセスを許可することで、このユーザーが Azure シングル サインオンを使用できるようにします。

1. Azure portal で **[エンタープライズ アプリケーション]** を選択し、**[すべてのアプリケーション]** を選択します。

    ![ユーザーの割り当て][201]

2. アプリケーションの一覧で **[Tableau Server]** を選択します。

    ![Configure single sign-on](./media/tableauserver-tutorial/tutorial-tableauserver-app.png) 

3. 左側のメニューで **[ユーザーとグループ]** をクリックします。

    ![ユーザーの割り当て][202]

4. **[追加]** ボタンをクリックします。 次に、**[割り当ての追加]** ダイアログで **[ユーザーとグループ]** を選択します。

    ![ユーザーの割り当て][203]

5. **[ユーザーとグループ]** ダイアログの [ユーザー] の一覧で **[Britta Simon]** を選択し、画面の下部にある **[選択]** ボタンをクリックします。

6. **[割り当ての追加]** ダイアログで、**[割り当て]** ボタンを選択します。

### <a name="test-single-sign-on"></a>シングル サインオンのテスト

このセクションでは、アクセス パネルを使用して Azure AD のシングル サインオン構成をテストします。

アクセス パネルで [Tableau Server] タイルをクリックすると、Tableau Server アプリケーションに自動的にサインオンします。
アクセス パネルの詳細については、[アクセス パネルの概要](../user-help/active-directory-saas-access-panel-introduction.md)に関するページを参照してください。

## <a name="additional-resources"></a>その他のリソース

* [SaaS アプリと Azure Active Directory を統合する方法に関するチュートリアルの一覧](tutorial-list.md)
* [Azure Active Directory のアプリケーション アクセスとシングル サインオンとは](../manage-apps/what-is-single-sign-on.md)

<!--Image references-->

[1]: common/tutorial-general-01.png
[2]: common/tutorial-general-02.png
[3]: common/tutorial-general-03.png
[4]: common/tutorial-general-04.png

[100]: common/tutorial-general-100.png

[201]: common/tutorial-general-201.png
[202]: common/tutorial-general-202.png
[203]: common/tutorial-general-203.png

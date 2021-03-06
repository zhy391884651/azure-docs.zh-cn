---
title: "Azure Active Directory B2C：使用自定义策略添加 Salesforce SAML 提供者 | Microsoft Docs"
description: "有关 Azure Active Directory B2C 自定义策略的主题"
services: active-directory-b2c
documentationcenter: 
author: gsacavdm
manager: krassk
editor: gsacavdm
ms.assetid: d7f4143f-cd7c-4939-91a8-231a4104dc2c
ms.service: active-directory-b2c
ms.workload: identity
ms.tgt_pltfrm: na
ms.topic: article
ms.devlang: na
ms.date: 04/30/2017
ms.author: gsacavdm
ms.translationtype: Human Translation
ms.sourcegitcommit: 97fa1d1d4dd81b055d5d3a10b6d812eaa9b86214
ms.openlocfilehash: 1d97c75f3130ea6fdacbc6335b6e70677b4d226e
ms.contentlocale: zh-cn
ms.lasthandoff: 05/11/2017


---
# <a name="azure-active-directory-b2c-log-in-using-salesforce-accounts-via-saml"></a>Azure Active Directory B2C：使用 Salesforce 帐户通过 SAML 登录

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

本文介绍如何让用户使用[自定义策略](active-directory-b2c-overview-custom.md)从特定的 Salesforce 组织登录。

## <a name="prerequisites"></a>先决条件

### <a name="azure-ad-b2c-setup"></a>Azure AD B2C 设置
确保已完成有关如何[开始使用自定义策略](active-directory-b2c-get-started-custom.md)的步骤。

这包括：

1. 创建 Azure AD B2C 租户。
1. 创建 Azure AD B2C 应用程序。
1. 注册两个策略引擎应用程序。
1. 设置密钥。
1. 设置初学者包。

### <a name="salesforce-setup"></a>Salesforce 设置
本教程假设已做好以下准备：
1. 已注册一个 Salesforce 帐户。 可以注册[免费的 Developer Edition](https://developer.salesforce.com/signup)。 
1. 为 Salesforce 组织[设置“我的域”](https://help.salesforce.com/articleView?id=domain_name_setup.htm&language=en_US&type=0)。

## <a name="get-the-salesforce-saml-metadata"></a>获取 Salesforce SAML 元数据
>[!NOTE]
> 本教程假设使用 [Salesforce Lighting Experience](https://developer.salesforce.com/page/Lightning_Experience_FAQ)。

1. [登录到 Salesforce](https://login.salesforce.com/) 
1. 在左侧菜单中的“设置”下面展开“标识”，然后单击“标识提供者”
1. 单击“启用标识提供者”
1. 选择在与 Azure AD B2C 通信时希望 Salesforce 使用的**证书**，然后单击“保存”。 可以使用默认证书。
1. 单击现在可用的“下载元数据”按钮，然后保存元数据文件供后面的步骤使用。

## <a name="add-a-saml-signing-certificate-to-azure-ad-b2c"></a>将 SAML 签名证书添加到 Azure AD B2C
需要将一个要在为 Azure AD B2C 租户的请求签名时使用的 SAML 证书上传到该租户。 为此，请按以下步骤操作：

1. 导航到 Azure AD B2C 租户，然后打开“B2C 设置”>“标识体验框架”>“策略密钥”
1. 单击“+添加”
1. 选项：
 * 选择“选项”>“上传”
 * **名称**：> `ContosoIdpSamlCert`。  前缀 B2C_1A_ 将自动添加到密钥名称。 记下完整名称（带有 B2C_1A_），因为稍后将在策略中引用此名称。
 * 使用“上传文件控件”以选择证书，并提供证书的密码（如果适用）。
1. 单击“创建” 
1. 确认已创建密钥：`B2C_1A_ContosoIdpSamlCert`

## <a name="create-the-salesforce-saml-claims-provider-in-your-base-policy"></a>在基本策略中创建 Salesforce SAML 声明提供程序

若要允许用户使用 Salesforce 登录，需将 Salesforce 定义为声明提供程序。 换而言之，需指定要与 Azure AD B2C 通信的终结点。 该终结点将*提供*一组*声明*，Azure AD B2C 使用这些声明来验证特定的用户是否已完成身份验证。 为此，可在策略的扩展文件中添加 Salesforce 的 `<ClaimsProvider>`。

1. 从工作目录打开该扩展文件 (TrustFrameworkExtensions.xml)。
1. 找到 `<ClaimsProviders>` 节。 如果该节不存在，请在根节点的下面添加它。
1. 按如下所示添加新的 `<ClaimsProvider>`：

    ```XML
    <ClaimsProvider>
      <Domain>contoso</Domain>
      <DisplayName>Contoso</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="Contoso">
          <DisplayName>Contoso</DisplayName>
          <Description>Login with your Contoso account</Description>
          <Protocol Name="SAML2"/>
          <Metadata>
            <Item Key="RequestsSigned">false</Item>
            <Item Key="WantsEncryptedAssertions">false</Item>
            <Item Key="PartnerEntity">
    <![CDATA[ <md:EntityDescriptor xmlns:md="urn:oasis:names:tc:SAML:2.0:metadata" entityID="https://contoso.com" validUntil="2026-10-05T23:57:13.854Z" xmlns:ds="http://www.w3.org/2000/09/xmldsig#"><md:IDPSSODescriptor protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol"><md:KeyDescriptor use="signing"><ds:KeyInfo><ds:X509Data><ds:X509Certificate>MIIErDCCA….qY9SjVXdu7zy8tZ+LqnwFSYIJ4VkE9UR1vvvnzO</ds:X509Certificate></ds:X509Data></ds:KeyInfo></md:KeyDescriptor><md:NameIDFormat>urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified</md:NameIDFormat><md:SingleSignOnService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST" Location="https://contoso.com/idp/endpoint/HttpPost"/><md:SingleSignOnService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect" Location="https://contoso.com/idp/endpoint/HttpRedirect"/></md:IDPSSODescriptor></md:EntityDescriptor>]]>
            </Item>
          </Metadata>       
          <CryptographicKeys>
            <Key Id="SamlAssertionSigning" StorageReferenceId="B2C_1A_ContosoIdpSamlCert"/>
            <Key Id="SamlMessageSigning" StorageReferenceId="B2C_1A_ContosoIdpSamlCert "/>
          </CryptographicKeys>
          <OutputClaims>
            <OutputClaim ClaimTypeReferenceId="socialIdpUserId" PartnerClaimType="userId"/>
            <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="given_name"/>
            <OutputClaim ClaimTypeReferenceId="surname" PartnerClaimType="family_name"/>
            <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="email"/>
            <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="name"/>
            <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="externalIdp"/>
            <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="SAMLIdp" />
          </OutputClaims>
          <OutputClaimsTransformations>
            <OutputClaimsTransformation ReferenceId="CreateRandomUPNUserName"/>
            <OutputClaimsTransformation ReferenceId="CreateUserPrincipalName"/>
            <OutputClaimsTransformation ReferenceId="CreateAlternativeSecurityId"/>
            <OutputClaimsTransformation ReferenceId="CreateSubjectClaimFromAlternativeSecurityId"/>
          </OutputClaimsTransformations>
          <UseTechnicalProfileForSessionManagement ReferenceId="SM-Noop"/>
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>
    ```

1. 在 `<ClaimsProvider>` 节点下面，将 `<Domain>` 的值更新为可用于区分其他标识提供者的唯一值。
1. 在 `<ClaimsProvider>` 节点下面，将 `<DisplayName>` 的值更新为声明提供程序的友好名称。 目前不会使用此值。

### <a name="update-the-technical-profile"></a>更新技术配置文件

若要从 Salesforce 获取 SAML 令牌，需要定义 Azure AD B2C 与 Azure AD 通信时应使用的协议。 可在 `<ClaimsProvider>` 的 `<TechnicalProfile>` 元素中执行此操作。

1. 更新 `<TechnicalProfile>` 节点的 ID。 此 ID 用于从策略的其他部分引用此技术配置文件。
1. 更新 `<DisplayName>` 的值。 此值将显示在登录屏幕中的登录按钮上。
1. 更新 `<Description>` 的值。
1. Azure AD 使用 OpenID Connect 协议，因此请确保 `<Protocol>` 是“SAML2”。

需要更新上述 XML 中的 `<Metadata>` 节，以反映特定 Azure AD 租户的配置设置。 在 XML 中，按如下所示更新元数据值：

1. 使用从 Salesforce 下载的 Metadata.xml 的内容更新 `<Item Key="PartnerEntity">` 的值。 **请确保使用 <![CDATA[ …metadata… ]]> 封装该元素**。

1. 在上述 XML 的 `<CryptographicKeys>` 节中，将两个 `StorageReferenceId` 的值更新为定义的证书 ID（例如 ContosoSalesforceCert）。

### <a name="upload-the-extension-file-for-verification"></a>上传扩展文件以进行验证

现已配置策略，因此 Azure AD B2C 知道如何与 Azure AD 目录通信。 请尝试上传该策略的扩展文件，这只是为了确认它到目前为止不会出现任何问题。 为此，请执行以下操作：

1. 转到 Azure AD B2C 租户中的“所有策略”边栏选项卡。
1. 选中“覆盖策略(如果存在)”对应的框。
1. 上传扩展文件 (TrustFrameworkExtensions.xml)，并确保它能够通过验证。

## <a name="register-the-salesforce-saml-claims-provider-to-a-user-journey"></a>将 Salesforce SAML 声明提供程序注册到用户旅程

现在，需要将 Salesforce SAML 标识提供者添加到用户旅程之一。 此时，标识提供者已设置，但不会出现在任何注册/登录屏幕中。 为此，我们需要创建现有模板用户旅程的副本，然后对其进行修改，以便它也包含 Azure AD 标识提供者。

1. 打开策略的基文件（例如 TrustFrameworkBase.xml）
1. 找到 `<UserJourneys>` 元素并复制包含 Id=”SignUpOrSignIn” 的整个 `<UserJourney>`。
1. 打开扩展文件（例如 TrustFrameworkExtensions.xml）并找到 `<UserJourneys>` 元素。 如果该元素不存在，请添加一个。
1. 将复制的整个 `<UserJourney>` 粘贴为 `<UserJourneys>` 元素的子级。
1. 重命名 `<UserJourney>` 的新 ID（例如 SignUpOrSignUsingContoso）。

### <a name="display-the-button"></a>显示“按钮”

`<ClaimsProviderSelection>` 元素类似于注册/登录屏幕上的标识提供者按钮。 为 Salesforce 添加 `<ClaimsProviderSelection>` 元素后，当用户进入页面时，会显示一个新按钮。 为此，请按以下步骤操作：

1. 在刚刚创建的 `<UserJourney>` 中找到采用 `Order="1"` 设置的 `<OrchestrationStep>`。
1. 添加以下内容：

    ```XML
    <ClaimsProviderSelection TargetClaimsExchangeId="ContosoExchange" />
    ```

1. 将 `TargetClaimsExchangeId` 设置为相应的值。 我们建议遵循其他元素使用的相同约定 - *\[ClaimProviderName\]Exchange*。

### <a name="link-the-button-to-an-action"></a>将“按钮”链接到操作

准备好“按钮”后，需将它链接到某个操作。 在本例中，Azure AD B2C 使用该操作来与 Salesforce 通信以接收 SAML 令牌。 为此，可以链接 Salesforce SAML 声明提供程序的技术配置文件。

1. 在 `<UserJourney>` 节点中找到采用 `Order="2"` 设置的 `<OrchestrationStep>`
1. 添加以下内容：

    ```XML
    <ClaimsExchange Id="ContosoExchange" TechnicalProfileReferenceId="ContosoProfile" />
    ```

1. 将 `Id` 更新为上述 `TargetClaimsExchangeId` 的相同值。
1. 将 `TechnicalProfileReferenceId` 更新为前面创建的配置文件的 `Id`（例如 ContosoProfile）。

### <a name="upload-the-updated-extension-file"></a>上传更新的扩展文件

现已完成修改扩展文件。 请保存并上传此文件，确保所有验证成功。

### <a name="update-the-rp-file"></a>更新 RP 文件

现在，需要更新用于启动刚刚创建的用户旅程的 RP 文件。

1. 在工作目录创建 SignUpOrSignIn.xml 的副本并将它重命名（例如 SignUpOrSignInWithAAD.xml）。
1. 打开新文件，并使用唯一值更新 `<TrustFrameworkPolicy>` 的 `PolicyId` 属性。 这将是策略的名称（例如 SignUpOrSignInWithAAD）。
1. 修改 `<DefaultUserJourney>` 中的 `ReferenceId` 属性，使其与创建的新用户旅程的 ID 匹配（例如 SignUpOrSignUsingContoso）。
1. 保存更改并上传文件。

## <a name="create-a-connected-app-in-salesforce"></a>在 Salesforce 中创建连接的应用
需要在 Salesforce 中将 Azure AD B2C 注册为连接的应用。

1. [登录到 Salesforce](https://login.salesforce.com/) 
1. 在左侧菜单中的“设置”下面展开“标识”，然后单击“标识提供者”
1. 在“服务提供程序”部分的底部，单击“现已通过连接的应用创建服务提供程序。请单击此处”。
1. 为连接的应用提供所需的**基本信息**。
1. 现在，请在“Web 应用设置”部分中：
    1. 选中“启用 SAML”
    1. 在“实体 ID”字段中输入以下 URL，并确保替换 `tenantName`。 
    
        ```
        https://login.microsoftonline.com/te/tenantName.onmicrosoft.com/B2C_1A_TrustFrameworkBase
        ```

    1. 在“ACS URL”字段中输入以下 URL，并确保替换 `tenantName`。 
        ```
        https://login.microsoftonline.com/te/tenantName.onmicrosoft.com/B2C_1A_TrustFrameworkBase/samlp/sso/assertionconsumer
        ```

    1. 将其他所有设置保留默认值
1. 一直滚动到底部，单击“保存”按钮


## <a name="troubleshooting"></a>故障排除

测试刚刚上传的自定义策略：打开其边栏选项卡，然后单击“立即运行”。 如果发生失败，请参阅[故障排除](active-directory-b2c-troubleshoot-custom.md)方法。

## <a name="next-steps"></a>后续步骤
 
向 [AADB2CPreview@microsoft.com](mailto:AADB2CPreview@microsoft.com) 提供反馈。



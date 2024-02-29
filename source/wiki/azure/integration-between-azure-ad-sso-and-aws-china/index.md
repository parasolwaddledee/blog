---
layout: wiki
wiki: azure
title: Azure AD SSO 与 AWS China 的集成
date: 2022-12-25
---

## 创建 Azure AD Enterprise Application

{% image /assets/wiki/azure/integration-between-azure-ad-sso-and-aws-china/integration-between-azure-ad-sso-and-aws-china-01.jpg %}

1. 下载 AWS China SAML 元数据文件：[https://signin.amazonaws.cn/static/saml-metadata.xml](https://signin.amazonaws.cn/static/saml-metadata.xml)。
2. 登录 [Azure Portal](https://portal.azure.com/)，激活 Application Administrator 角色（如有必要）。
3. 点击 Azure Active Directory > Enterprise applications > New application。
4. 搜索 AWS Single-Account Access，选中后输入应用名称如 `AWS China`，点击 Create。
5. 等待应用创建完成，点击 Single sign-on > SAML > Upload metadata file > Select a file。
6. 上传刚才下载的 SAML 元数据文件，点击 Add > Save，随即返回 Single sign-on 配置页面。
7. 在 Attributes & Claims 步骤中点击 Edit > Add new claim，输入以下内容并点击 Save：
{% box %}
- Name: `http://schemas.xmlsoap.org/ws/2005/identity/claims/nameidentifier`
Source attribute: `user.userprincipalname`
{% endbox %}
8. 确认以下 Claims 是否存在，如不存在则按照相同的方法创建：
{% box %}
- Name: `https://aws.amazon.com/SAML/Attributes/Role`
Source attribute: `user.assignedroles`
- Name: `https://aws.amazon.com/SAML/Attributes/RoleSessionName`
Source attribute: `user.userprincipalname`
- Name: `https://aws.amazon.com/SAML/Attributes/SessionDuration`
Source attribute: `900`
{% endbox %}
9. 返回 Single sign-on 配置页面，在 SAML Certificates 步骤中下载 Federation Metadata XML。

## 配置 AWS China IAM Provider & Role

{% image /assets/wiki/azure/integration-between-azure-ad-sso-and-aws-china/integration-between-azure-ad-sso-and-aws-china-02.jpg %}

1. 登录 [AWS Console](https://console.amazonaws.cn/)，点击 IAM > Identity providers > Add provider。
2. 输入 Provider name 如 `AzureAD`，点击 Choose file 上传刚才下载的 XML，点击 Add provider。
3. 点击 Roles > Create role > SAML 2.0 federation，选择刚才创建的 `AzureAD` 作为 Provider。
4. 选择 Allow programmatic and Amazon Web Services Management Console access。
5. 点击 Next，选择 Permissions policies 如 `SystemAdministrator`。
6. 点击 Next，输入 Role name 如 `SysAdmin`，点击 Create role。

## 配置 Azure AD Enterprise Application

{% image /assets/wiki/azure/integration-between-azure-ad-sso-and-aws-china/integration-between-azure-ad-sso-and-aws-china-03.jpg %}

1. 返回 Azure Portal，点击 Azure Active Directory > App registrations > All applications。
2. 搜索并打开应用 `AWS China`，点击 App roles > Create app role。
3. 输入 Display name 如 `SysAdmin`，选择 Users/Groups 作为 Allowed member types。
4. 输入 Value，格式为 AWS China IAM Role ARN 逗号 AWS China IAM Provider ARN，如 `arn:aws-cn:iam::123456789012:role/SysAdmin,arn:aws-cn:iam::123456789012:saml-provider/AzureAD`。
5. 输入 Description 如 `SysAdmin`，点击 Apply。

{% box color:blue %}
如需在一个应用中集成多个 AWS China 账户与 IAM Role，则重复上述步骤创建多个 App Role。
{% endbox %}

## 分配应用权限并测试 Azure AD SSO

{% image /assets/wiki/azure/integration-between-azure-ad-sso-and-aws-china/integration-between-azure-ad-sso-and-aws-china-04.jpg %}

1. 返回 Azure Active Directory > Enterprise applications，搜索并打开应用 `AWS China`。
2. 点击 Users and groups > Add user/group，搜索当前用户并点击 Select > Assign。
3. 点击 Single sign-on，在 Test single sign-on with AWS China 步骤中点击 Test > Test sign in。

## Azure AD SSO 与 AWS CLI 的集成

{% tabs %}
<!-- tab Windows -->
1. 下载并安装 Node.js：
{% link https://nodejs.org/ %}
2. 通过 npm 安装 aws-azure-login：
```
npm install -g aws-azure-login
```
3. 创建 AWS CLI 配置文件：
```powershell
notepad "$env:USERPROFILE\.aws\config"
```
4. 复制如下内容并根据实际情况进行修改，随后保存配置文件：
```
[profile sysadmin]
region=cn-north-1
azure_tenant_id=252e61b2-75d3-4c0e-b7ee-0804af693f28
azure_app_id_uri=urn:amazon:webservices:cn-north-1
azure_default_username=parasol@waddledee.com
azure_default_role_arn=arn:aws-cn:iam::123456789012:role/SysAdmin
azure_default_duration_hours=12
azure_default_remember_me=true
```
{% box color:blue %}
- azure_tenant_id 和 azure_app_id_uri 的值可以在 App registrations > Overview 找到。
- 将 azure_default_remember_me 的值设为 true 即可使用浏览器 Cookies 作为登录凭据。
{% endbox %}
5. 首次登录时需要输入密码并批准 MFA 验证：
```
PS C:\> aws-azure-login --profile sysadmin --no-prompt
Logging in with profile 'sysadmin'...
Using AWS SAML endpoint https://signin.amazonaws.cn/saml
? Password: [hidden]
Open your Microsoft Authenticator app and approve the request to sign in​.​
Assuming role arn:aws-cn:iam::123456789012:role/SysAdmin
```
6. 再次登录时即可通过浏览器 Cookies 跳过密码和 MFA 验证：
```
PS C:\> aws-azure-login --profile sysadmin --no-prompt
Logging in with profile 'sysadmin'...
Using AWS SAML endpoint https://signin.amazonaws.cn/saml
Assuming role arn:aws-cn:iam::123456789012:role/SysAdmin
```
<!-- tab Linux -->
1. 下载并安装 Docker Engine：
{% link https://docs.docker.com/engine/ %}
2. 拉取 Docker 镜像并挂载到 AWS CLI 配置文件目录：
```bash
docker run --rm -it -v ~/.aws:/root/.aws sportradar/aws-azure-login
```
3. 下载 docker-launch.sh 方便之后直接调用 aws-azure-login：
```bash
sudo curl -o /usr/local/bin/aws-azure-login https://raw.githubusercontent.com/sportradar/aws-azure-login/main/docker-launch.sh -L
sudo chmod o+x /usr/local/bin/aws-azure-login
```
4. 创建 AWS CLI 配置文件：
```bash
sudo nano .aws/config
```
5. 复制如下内容并根据实际情况进行修改，按 Ctrl+X 保存并退出，按 Y 确认，按 Enter 返回命令行：
```
[profile sysadmin]
region=cn-north-1
azure_tenant_id=252e61b2-75d3-4c0e-b7ee-0804af693f28
azure_app_id_uri=urn:amazon:webservices:cn-north-1
azure_default_username=parasol@waddledee.com
azure_default_role_arn=arn:aws-cn:iam::123456789012:role/SysAdmin
azure_default_duration_hours=12
azure_default_remember_me=true
```
{% box color:blue %}
- azure_tenant_id 和 azure_app_id_uri 的值可以在 App registrations > Overview 找到。
- 将 azure_default_remember_me 的值设为 true 即可使用浏览器 Cookies 作为登录凭据。
{% endbox %}
6. 首次登录时需要输入密码并批准 MFA 验证：
```
$ aws-azure-login --profile sysadmin --no-prompt
Logging in with profile 'sysadmin'...
Using AWS SAML endpoint https://signin.amazonaws.cn/saml
? Password: [hidden]
Open your Microsoft Authenticator app and approve the request to sign in​.​
Assuming role arn:aws-cn:iam::123456789012:role/SysAdmin
```
7. 再次登录时即可通过浏览器 Cookies 跳过密码和 MFA 验证：
```
$ aws-azure-login --profile sysadmin --no-prompt
Logging in with profile 'sysadmin'...
Using AWS SAML endpoint https://signin.amazonaws.cn/saml
Assuming role arn:aws-cn:iam::123456789012:role/SysAdmin
```
{% endtabs %}

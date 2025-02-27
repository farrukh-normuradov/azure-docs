---
title: Configure authentication session management - Azure Active Directory
description: Customize Azure AD authentication session configuration including user sign-in frequency and browser session persistence.

services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: how-to
ms.date: 04/21/2022

ms.author: joflore
author: MicrosoftGuyJFlo
manager: karenhoran
ms.reviewer: jlu, calebb, ripull

ms.collection: M365-identity-device-management
---
# Configure authentication session management with Conditional Access

In complex deployments, organizations might have a need to restrict authentication sessions. Some scenarios might include:

* Resource access from an unmanaged or shared device
* Access to sensitive information from an external network
* High impact users
* Critical business applications

Conditional Access controls allow you to create policies that target specific use cases within your organization without affecting all users.

Before diving into details on how to configure the policy, let’s examine the default configuration.

## User sign-in frequency

Sign-in frequency defines the time period before a user is asked to sign in again when attempting to access a resource.

The Azure Active Directory (Azure AD) default configuration for user sign-in frequency is a rolling window of 90 days. Asking users for credentials often seems like a sensible thing to do, but it can backfire: users that are trained to enter their credentials without thinking can unintentionally supply them to a malicious credential prompt.

It might sound alarming to not ask for a user to sign back in, in reality any violation of IT policies will revoke the session. Some examples include (but aren't limited to) a password change, an incompliant device, or account disable. You can also explicitly [revoke users’ sessions using PowerShell](/powershell/module/azuread/revoke-azureaduserallrefreshtoken). The Azure AD default configuration comes down to “don’t ask users to provide their credentials if security posture of their sessions hasn't changed”.

The sign-in frequency setting works with apps that have implemented OAuth2 or OIDC protocols according to the standards. Most Microsoft native apps for Windows, Mac, and Mobile including the following web applications comply with the setting.

- Word, Excel, PowerPoint Online
- OneNote Online
- Office.com
- Microsoft 365 Admin portal
- Exchange Online
- SharePoint and OneDrive
- Teams web client
- Dynamics CRM Online
- Azure portal

The sign-in frequency setting works with 3rd party SAML applications and apps that have implemented OAuth2 or OIDC protocols, as long as they don't drop their own cookies and are redirected back to Azure AD for authentication on regular basis.

### User sign-in frequency and multi-factor authentication

Sign-in frequency previously applied to only to the first factor authentication on devices that were Azure AD joined, Hybrid Azure AD joined, and Azure AD registered. There was no easy way for our customers to re-enforce multi factor authentication (MFA) on those devices. Based on customer feedback, sign-in frequency will apply for MFA as well.

[![Sign in frequency and MFA](media/howto-conditional-access-session-lifetime/conditional-access-flow-chart-small.png)](media/howto-conditional-access-session-lifetime/conditional-access-flow-chart.png#lightbox)

### User sign-in frequency and device identities

If you have Azure AD joined, hybrid Azure AD joined, or Azure AD registered devices, when a user unlocks their device or signs in interactively, this event will satisfy the sign-in frequency policy as well. In the following two examples user sign-in frequency is set to 1 hour:

Example 1:

- At 00:00, a user signs in to their Windows 10 Azure AD joined device and starts work on a document stored on SharePoint Online.
- The user continues working on the same document on their device for an hour.
- At 01:00, the user is prompted to sign in again based on the sign-in frequency requirement in the Conditional Access policy configured by their administrator.

Example 2:

- At 00:00, a user signs in to their Windows 10 Azure AD joined device and starts work on a document stored on SharePoint Online.
- At 00:30, the user gets up and takes a break locking their device.
- At 00:45, the user returns from their break and unlocks the device.
- At 01:45, the user is prompted to sign in again based on the sign-in frequency requirement in the Conditional Access policy configured by their administrator since the last sign-in happened at 00:45.

### Require reauthentication every time (preview)

There are scenarios where customers may want to require a fresh authentication, every time before a user performs specific actions. Sign-in frequency has a new option for **Every time** in addition to hours or days.

The public preview supports the following scenarios:

- Require user reauthentication during [Intune device enrollment](/mem/intune/fundamentals/deployment-guide-enrollment), regardless of their current MFA status.
- Require user reauthentication for risky users with the [require password change](concept-conditional-access-grant.md#require-password-change) grant control.
- Require user reauthentication for risky sign-ins with the [require multi-factor authentication](concept-conditional-access-grant.md#require-multi-factor-authentication) grant control.

When administrators select **Every time**, it will require full reauthentication when the session is evaluated.

> [!NOTE]
> An early preview version included the option to prompt for Secondary authentication methods only at reauthentication. This option is no longer supported and should not be used.

## Persistence of browsing sessions

A persistent browser session allows users to remain signed in after closing and reopening their browser window.

The Azure AD default for browser session persistence allows users on personal devices to choose whether to persist the session by showing a “Stay signed in?” prompt after successful authentication. If browser persistence is configured in AD FS using the guidance in the article [AD FS Single Sign-On Settings](/windows-server/identity/ad-fs/operations/ad-fs-single-sign-on-settings#enable-psso-for-office-365-users-to-access-sharepoint-online), we'll comply with that policy and persist the Azure AD session as well. You can also configure whether users in your tenant see the “Stay signed in?” prompt by changing the appropriate setting in the company branding pane in Azure portal using the guidance in the article [Customize your Azure AD sign-in page](../fundamentals/customize-branding.md).

## Configuring authentication session controls

Conditional Access is an Azure AD Premium capability and requires a premium license. If you would like to learn more about Conditional Access, see [What is Conditional Access in Azure Active Directory?](overview.md#license-requirements)

> [!WARNING]
> If you are using the [configurable token lifetime](../develop/active-directory-configurable-token-lifetimes.md) feature currently in public preview, please note that we don’t support creating two different policies for the same user or app combination: one with this feature and another one with configurable token lifetime feature. Microsoft retired the configurable token lifetime feature for refresh and session token lifetimes on January 30, 2021 and replaced it with the Conditional Access authentication session management feature.  
>
> Before enabling Sign-in Frequency, make sure other reauthentication settings are disabled in your tenant. If "Remember MFA on trusted devices" is enabled, be sure to disable it before using Sign-in frequency, as using these two settings together may lead to prompting users unexpectedly. To learn more about reauthentication prompts and session lifetime, see the article, [Optimize reauthentication prompts and understand session lifetime for Azure AD Multi-Factor Authentication](../authentication/concepts-azure-multi-factor-authentication-prompts-session-lifetime.md).

## Policy deployment

To make sure that your policy works as expected, the recommended best practice is to test it before rolling it out into production. Ideally, use a test tenant to verify whether your new policy works as intended. For more information, see the article [Plan a Conditional Access deployment](plan-conditional-access.md).

### Policy 1: Sign-in frequency control

1. Sign in to the **Azure portal** as a global administrator, security administrator, or Conditional Access administrator.
1. Browse to **Azure Active Directory** > **Security** > **Conditional Access**.
1. Select **New policy**.
1. Give your policy a name. We recommend that organizations create a meaningful standard for the names of their policies.
1. Choose all required conditions for customer’s environment, including the target cloud apps.

   > [!NOTE]
   > It is recommended to set equal authentication prompt frequency for key Microsoft Office apps such as Exchange Online and SharePoint Online for best user experience.

1. Under **Access controls** > **Session**.
   1. Select **Sign-in frequency**.
   1. Enter the required value of days or hours in the first text box.
   1. Select a value of **Hours** or **Days** from dropdown.
1. Save your policy.

![Conditional Access policy configured for sign-in frequency](media/howto-conditional-access-session-lifetime/conditional-access-policy-session-sign-in-frequency.png)

On Azure AD registered Windows devices, sign in to the device is considered a prompt. For example, if you've configured the sign-in frequency to 24 hours for Office apps, users on Azure AD registered Windows devices will satisfy the sign-in frequency policy by signing in to the device and will be not prompted again when opening Office apps.

### Policy 2: Persistent browser session

1. Sign in to the **Azure portal** as a global administrator, security administrator, or Conditional Access administrator.
1. Browse to **Azure Active Directory** > **Security** > **Conditional Access**.
1. Select **New policy**.
1. Give your policy a name. We recommend that organizations create a meaningful standard for the names of their policies.
1. Choose all required conditions.

   > [!NOTE]
   > Please note that this control requires to choose “All Cloud Apps” as a condition. Browser session persistence is controlled by authentication session token. All tabs in a browser session share a single session token and therefore they all must share persistence state.

1. Under **Access controls** > **Session**.
   1. Select **Persistent browser session**.
   1. Select a value from dropdown.
1. Save your policy.

![Conditional Access policy configured for persistent browser](media/howto-conditional-access-session-lifetime/conditional-access-policy-session-persistent-browser.png)

> [!NOTE]
> Persistent Browser Session configuration in Azure AD Conditional Access will overwrite the “Stay signed in?” setting in the company branding pane in the Azure portal for the same user if you have configured both policies.

### Policy 3: Sign-in frequency control every time risky user

1. Sign in to the **Azure portal** as a global administrator, security administrator, or Conditional Access administrator.
1. Browse to **Azure Active Directory** > **Security** > **Conditional Access**.
1. Select **New policy**.
1. Give your policy a name. We recommend that organizations create a meaningful standard for the names of their policies.
1. Under **Assignments**, select **Users and groups**.
   1. Under **Include**, select **All users**.
   1. Under **Exclude**, select **Users and groups** and choose your organization's [emergency access or break-glass accounts](../roles/security-emergency-access.md). 
   1. Select **Done**.
1. Under **Cloud apps or actions** > **Include**, select **All cloud apps**.
1. Under **Conditions** > **User risk**, set **Configure** to **Yes**. Under **Configure user risk levels needed for policy to be enforced** select **High**, then select **Done**.
1. Under **Access controls** > **Grant**, select **Grant access**, **Require password change**, and select **Select**.
1. Under **Session controls** > **Sign-in frequency**, select **Every time (preview)**.
1. Confirm your settings and set **Enable policy** to **Report-only**.
1. Select **Create** to create to enable your policy.

After administrators confirm your settings using [report-only mode](howto-conditional-access-insights-reporting.md), they can move the **Enable policy** toggle from **Report-only** to **On**.

### Validation

Use the What-If tool to simulate a login from the user to the target application and other conditions based on how you configured your policy. The authentication session management controls show up in the result of the tool.

![Conditional Access What If tool results](media/howto-conditional-access-session-lifetime/conditional-access-what-if-tool-result.png)

## Prompt tolerance

We factor for five minutes of clock skew, so that we don’t prompt users more often than once every five minutes. If the user has done MFA in the last 5 minutes, and they hit another Conditional Access policy that requires reauthentication, we won't prompt the user. Over-promoting users for reauthentication can impact their productivity and increase the risk of users approving MFA requests they didn’t initiate. We highly recommend using “Sign-in frequency – every time” only for specific business needs. 

## Known issues
- If you configure sign-in frequency for mobile devices, authentication after each sign-in frequency interval could be slow (it can take 30 seconds on average). Also, it could happen across various apps at the same time. 
- In iOS devices, if an app configures certificates as the first authentication factor and the app has both Sign-in frequency and [Intune mobile application management](/mem/intune/apps/app-lifecycle) policies applied, the end-users will be blocked from signing in to the app when the policy is triggered.

## Next steps

* If you're ready to configure Conditional Access policies for your environment, see the article [Plan a Conditional Access deployment](plan-conditional-access.md).

---
title: SharePoint Add-Ins and Azure ACS retirement FAQ
description: Answers frequently asked questions related to the retirements of SharePoint Add-In and Azure ACS in Microsoft 365.
ms.date: 01/02/2024
ms.localizationpriority: high
ms.service: sharepoint
---

# FAQ - SharePoint Add-In and Azure ACS retirement in Microsoft 365

[!INCLUDE [sp-add-in-deprecation](../../includes/snippets/sp-add-in-deprecation.md)]

[!INCLUDE [azure-acs-retirement](../../includes/snippets/azure-acs-deprecation.md)]

## Is SharePoint Framework (SPFx) impacted by these retirements?

No, SharePoint Framework (SPFx) is the recommended development model for SharePoint extensibility and is not impacted by these retirements. These retirements only affect SharePoint Add-Ins (a legacy development model) and Azure ACS (a legacy auth model).

## Will App Catalogs stay?

Absolutely, both tenant and site collection app catalogs are an essential part of the SharePoint developer ecosystem. They're used for both Add-Ins (retired) and SharePoint Framework (SPFx) based solutions (not retired). Using the app catalogs for SharePoint Framework solution deployment will continue to be fully supported; no changes! What will change is the support for SharePoint Add-Ins in the app catalog. When SharePoint Add-Ins in Microsoft 365 reach end-of-life in April 2026, you'll no longer be able to install and deploy SharePoint Add-Ins via the app catalog.

## Will the SharePoint Store/public marketplace stay?

Absolutely. The SharePoint Store and public marketplace are also used to offer SharePoint Framework (SPFx) based solutions, and that will not change. You can still acquire SPFx solutions from the store, now and after Add-Ins have retired. Acquiring SharePoint Add-Ins via the store will not be possible anymore from July 1st, 2024.

## My Add-In uses an app web; what about the data in that web?

If your SharePoint Hosted Add-In is storing data in its app web, then that's something you need to take into account when you're rewriting your Add-In using the SharePoint Framework (SPFx). A common use case is that Add-Ins store list items in a list in the app web. When using a SharePoint Framework (SPFx) solution you can still use a (hidden) list, but now the list will live in the site where you've installed the SharePoint Framework (SPFx) solution. If you want to automatically retain the data, you'll need to use SharePoint APIs to copy the needed data from the app web, and recreate it in the new location using a format that works for your updated application.

> [!Important]
> It's important you retrieve the data **before** you uninstall the SharePoint Add-In, because the app web is deleted when you uninstall the Add-In. In case you've accidentally deleted the Add-In, you can restore it from the recycle bin, which will also restore the Add-In's app web.

## Can I still use my Add-In on SharePoint on-premises?

SharePoint Add-Ins in SharePoint on-premises are not retired and will continue to work after April 2026 when deployed using a SharePoint on-premises app catalog. But acquiring them from the SharePoint Store/public marketplace will not be possible after April 2026.

## My Add-In is used for both SharePoint Online and SharePoint on-premises; what do you advise?

SharePoint Online is continuously being updated and getting new and updated features, whereas SharePoint on-premises follows a much slower release cadence. This means that on-premises SharePoint doesn't support the latest SharePoint Framework version, or the latest features. If you want to continue to maintain a single codebase, you'll need to use an SPFx version that's also supported for your target SharePoint on-premises environment, which will limit the options for your SharePoint Online solution. An alternative approach is to build and maintain two versions of your SPFx application, one for SharePoint Online and one for SharePoint on-premises, and use shared code where possible to maximize code reuse.

## Can I still perform user profile updates?

Yes, you can still read and update user profiles without Azure ACS. When you configure your Microsoft Entra (formerly Azure AD) application using the SharePoint `User.ReadWrite.All` permission, your app can update user profiles for all users. Note that if you want to update a user profile property that's using an underlying term set, you also need to grant the SharePoint `TermStore.ReadWrite.All` permission. See [Error when using app to update user profile properties in SharePoint Online](/sharepoint/troubleshoot/sharing-and-permissions/app-permission-error-update-user-profile-property) to learn more.

## Will remote event receivers also be retired?

Yes, remote event receivers are part of this retirement. Remote event receivers have a dependency on Azure ACS, and will stop working when Azure ACS is turned off. Although remote event receivers can still be programmatically added once Azure ACS has fully retired in April 2026, the events won't fire anymore. The recommended path forward is to use [SharePoint Online Webhooks](../apis/webhooks/overview-sharepoint-webhooks.md). More information on how to replace remote event receivers with webhooks can be found in the [Transform SharePoint Add-in model Remote Event Receivers to SharePoint Online Webhooks](../sp-add-ins-modernize/from-remote-event-receivers-to-webhooks.md) article. The article shows how to use the SharePoint REST API to add the webhooks, but you may prefer to use Microsoft Graph [change notifications](/graph/webhooks) to add a [subscription for a SharePoint List](/graph/api/resources/subscription).

As webhooks are asynchronous by definition, synchronous events that allow an app to block or cancel a SharePoint action are no longer possible. If event blocking is being used to prevent accidental data updates/deletes by unauthorized users, then possible workarounds are securing the protected files/folders so they cannot be updated/deleted anymore, or by moving this type of data to a hidden library. In general, moving from synchronous to asynchronous events will require updating your application logic.

> [!Note]
> It's possible to get remote event receivers to partly work without Azure ACS by using sites.selected. This approach is **not** recommended, and will stop working when Azure ACS fully retires in April 2026.

## Can I still perform taxonomy updates?

Yes, you can still update taxonomy (termsets, terms, etc) without Azure ACS. Configure your Microsoft Entra application with the SharePoint `TermStore.ReadWrite.All` permission (delegated or application) if you want to use the CSOM taxonomy APIs. Moving forward, the recommended approach is to use the Microsoft Graph taxonomy API. This will require the Microsoft Graph `TermStore.ReadWrite.All` permission (delegated or application). See the [Microsoft Graph documentation](/graph/api/resources/termstore-set) for more details.

## Can I still add OOB apps when I'm using the "classic" site contents page?

When you're using the classic "Site Contents" page (`_layouts/15/viewlsts.aspx`), and you click on "add an app", the list of apps you can add will be a mix of OOB (out-of-the-box) apps (e.g. Document Library, Custom List, Form Library) and any installed SharePoint Add-Ins. When SharePoint Add-Ins retire, this page will continue to list the OOB apps, which can still be added to the site.

## Are SharePoint CSOM or SharePoint JSOM part of this retirement?

No, both CSOM (client-side object model) and JSOM (JavaScript object model) will continue working after Add-Ins and Azure ACS have been fully retired. However, for new applications, and updates to existing applications, we recommend using the Microsoft Graph and SharePoint REST APIs for client-side code. For server-side code the recommendation is to use Microsoft Graph as a first choice. If the functionality you need is not yet available in the Microsoft Graph, then SharePoint REST or SharePoint CSOM can be used.

## Do I need to delete Azure ACS principals that are not needed anymore?

Azure ACS principals will automatically expire (default lifetime is 2 years) so no action is needed; you can just let the principals expire. If you want to delete the principals after remediation, then follow this guidance:

- For principals that have Site, Web or List permissions, navigate to the site collection holding the principal and append `_layouts/15/appprincipals.aspx` to the url to load the **Site Collection App Permissions** page. On this page, delete the unwanted principals by clicking on the **X** icon.
- For principals that have tenant permissions, copy the principal app id from the report and navigate to the [Microsoft Entra Enterprise Applications view](https://entra.microsoft.com/#view/Microsoft_AAD_IAM/EnterpriseApplicationListBladeV2), paste the app id in the search box and press **enter**. Click on the displayed enterprise app to load it, click on **Properties**, then click **Delete**.

For principals that have Site, Web or List permissions **and** Tenant permissions, you'll need to follow both procedures described above.

A good practice is for admins to turn off Azure ACS app-only access for the entire tenant, having first ensured there is no remaining business relevant Azure ACS usage, as mentioned in the [Azure ACS retirement announcement](https://aka.ms/retirement/acs/support). Doing this will automatically prevent any remaining un-expired principals from working.

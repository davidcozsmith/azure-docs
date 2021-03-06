---
title: Use and manage an App Service Environment
description: Learn how to create, publish, and scale apps in an App Service Environment. Find all the common tasks in this article.
author: ccompy

ms.assetid: a22450c4-9b8b-41d4-9568-c4646f4cf66b
ms.topic: article
ms.date: 01/01/2020
ms.author: ccompy
ms.custom: seodec18
---
# Use an App Service Environment

An App Service Environment (ASE) is a deployment of Azure App Service into a subnet in a customer's Azure Virtual Network instance. An ASE consists of:

- **Front ends**: Where HTTP or HTTPS terminates in an App Service Environment.
- **Workers**: The resources that host your apps.
- **Database**: Holds information that defines the environment.
- **Storage**: Used to host the customer-published apps.

You can deploy an ASE with an external or internal virtual IP (VIP) for app access. A deployment with an external VIP is commonly called an *External ASE*. A deployment with an internal VIP is called an *ILB ASE* because it uses an internal load balancer (ILB). To learn more about the ILB ASE, see [Create and use an ILB ASE][MakeILBASE].

## Create an app in an ASE

To create an app in an ASE, you use the same process as when you normally create an app, but with a few small differences. When you create a new App Service plan:

- Instead of choosing a geographic location in which to deploy your app, you choose an ASE as your location.
- All App Service plans created in an ASE can only be in an Isolated pricing tier.

If you don't have an ASE, you can create one by following the instructions in [Create an App Service Environment][MakeExternalASE].

To create an app in an ASE:

1. Select **Create a resource** > **Web + Mobile** > **Web App**.

1. Enter a name for the app. If you already selected an App Service plan in an ASE, the domain name for the app reflects the domain name of the ASE:

    ![App name selection][1]

1. Select a subscription.

1. Enter a name for a new resource group, or select **Use existing** and select one from the drop-down list.

1. Select your OS.

1. Select an existing App Service plan in your ASE, or create a new one by following these steps:

    a. From the Azure portal left-side menu, select **Create a resource > Web App**.

    b. Select the subscription.

    c. Select or create the resource group.

    d. Enter the name of your web app.

    e. Select **Code** or **DockerContainer**.

    f. Select a runtime stack.

    g. Select **Linux** or **Windows**. 

    h. Select your ASE in the **Region** drop-down list. 

    i. Select or create a new App Service plan. If creating a new App Service plan, select the appropriate **Isolated** SKU size.

    ![Isolated pricing tiers][2]

    > [!NOTE]
    > Linux apps and Windows apps cannot be in the same App Service plan, but they can be in the same App Service Environment.
    >

1. Select **Review + create**, make sure the information is correct, and then select **Create**.

## How scale works

Every App Service app runs in an App Service plan. App Service Environments hold App Service plans, and App Service plans hold apps. When you scale an app, you also scale the App Service plan and all the apps in that same plan.

When you scale an App Service plan, the needed infrastructure is added automatically. There's a time delay to scale operations while the infrastructure is being added. If you do several scale operations in sequence, the first infrastructure scale request is acted on and the others are queued. When the first scale operation finishes, the other infrastructure requests all operate together. And when the infrastructure is added, the App Service plans are assigned as appropriate. Creating a new App Service plan is itself a scale operation because it requests additional hardware.

In the multitenant App Service, scaling is immediate because a pool of resources is readily available to support it. In an ASE, there's no such buffer, and resources are allocated based on need.

In an ASE, you can scale an App Service plan up to 100 instances. An ASE can have up to 201 total instances across all the App Service plans in that ASE.

## IP addresses

App Service can allocate a dedicated IP address to an app. This capability is available after you configure IP-based SSL, as described in [Bind an existing custom TLS/SSL certificate to Azure App Service][ConfigureSSL]. In an ILB ASE, you can't add more IP addresses to be used for IP-based SSL.

With an External ASE, you can configure IP-based SSL for your app in the same way as in the multitenant App Service. There's always one spare address in the ASE, up to 30 IP addresses. Each time you use one, another is added so that an address is always readily available. A time delay is required to allocate another IP address. That delay prevents adding IP addresses in quick succession.

## Front-end scaling

When you scale out your App Service plans, workers are automatically added to support them. Every ASE is created with two front ends. The front ends automatically scale out at a rate of one front end for every set of 15 App Service plan instances. For example, if you have three App Service plans with five instances each, you'd have a total of 15 instances and three front ends. If you scale to a total of 30 instances, you have four front ends. This pattern continues as you scale out.

The number of front ends that are allocated by default is good for a moderate load. You can lower the ratio to as little as one front end for every five instances. You can also change the size of the front ends. By default, they're single core. In the Azure portal, you can change their size to two or four cores instead.

There's a charge for changing the ratio or the front-end sizes. For more information, see [Azure App Service pricing][Pricing]. If you want to improve the load capacity of your ASE, you'll get more improvement by first scaling to two-core front ends before you adjust the scale ratio. Changing the core size of your front ends will cause an upgrade of your ASE and should be done outside of regular business hours.

Front-end resources are the HTTP/HTTPS endpoint for the ASE. With the default front-end configuration, memory usage per front end is consistently around 60 percent. The primary reason to scale your front ends is CPU usage, which is primarily driven by HTTPS traffic.

## App access

In an External ASE, the domain suffix used for app creation is *.&lt;asename&gt;.p.azurewebsites.net*. If your ASE is named _external-ase_ and you host an app called _contoso_ in that ASE, you reach it at these URLs:

- contoso.external-ase.p.azurewebsites.net
- contoso.scm.external-ase.p.azurewebsites.net

For information about how to create an External ASE, see [Create an App Service Environment][MakeExternalASE].

In an ILB ASE, the domain suffix used for app creation is *.&lt;asename&gt;.appserviceenvironment.net*. If your ASE is named _ilb-ase_ and you host an app called _contoso_ in that ASE, you reach it at these URLs:

- contoso.ilb-ase.appserviceenvironment.net
- contoso.scm.ilb-ase.appserviceenvironment.net

For information about how to create an ILB ASE, see [Create and use an ILB ASE][MakeILBASE].

The SCM URL is used to access the Kudu console or for publishing your app by using Web Deploy. For information on the Kudu console, see [Kudu console for Azure App Service][Kudu]. The Kudu console gives you a web UI for debugging, uploading files, editing files, and much more.

## Publishing

In an ASE, as with the multitenant App Service, you can publish by these methods:

- Web deployment
- FTP
- Continuous integration (CI)
- Drag and drop in the Kudu console
- An IDE, such as Visual Studio, Eclipse, or IntelliJ IDEA

With an External ASE, these publishing options all work the same way. For more information, see [Deployment in Azure App Service][AppDeploy].

Publishing is significantly different with an ILB ASE, for which the publishing endpoints are all available only through the ILB. The ILB is on a private IP in the ASE subnet in the virtual network. If you don't have network access to the ILB, you can't publish any apps on that ASE. As noted in [Create and use an ILB ASE][MakeILBASE], you must configure DNS for the apps in the system. That requirement includes the SCM endpoint. If the endpoints aren't defined properly, you can't publish. Your IDEs must also have network access to the ILB to publish directly to it.

Without additional changes, internet-based CI systems like GitHub and Azure DevOps don't work with an ILB ASE because the publishing endpoint isn't internet accessible. You can enable publishing to an ILB ASE from Azure DevOps by installing a self-hosted release agent in the virtual network that contains the ILB ASE. Alternatively, you can also use a CI system that uses a pull model, such as Dropbox.

The publishing endpoints for apps in an ILB ASE use the domain that the ILB ASE was created with. You can see it in the app's publishing profile and in the app's portal pane (in **Overview** > **Essentials** and also in **Properties**).

## Storage

An ASE has 1 TB of storage for all the apps in the ASE. An App Service plan in the Isolated pricing SKU has a limit of 250 GB by default. If you have five or more App Service plans, be careful not to exceed the 1-TB limit of the ASE. If you need more than the 250-GB limit in one App Service plan, contact support to adjust the App Service plan limit to a maximum of 1 TB. When the plan limit is adjusted, there's still a limit of 1 TB across all the App Service plans in the ASE.

## Logging

You can integrate your ASE with Azure Monitor to send logs about the ASE to Azure Storage, Azure Event Hubs, or Log Analytics. These items are logged today:

| Situation | Message |
|---------|----------|
| ASE is unhealthy | The specified ASE is unhealthy due to an invalid virtual network configuration. The ASE will be suspended if the unhealthy state continues. Ensure the guidelines defined here are followed: https://docs.microsoft.com/azure/app-service/environment/network-info. |
| ASE subnet is almost out of space | The specified ASE is in a subnet that is almost out of space. There are {0} remaining addresses. Once these addresses are exhausted, the ASE will not be able to scale.  |
| ASE is approaching total instance limit | The specified ASE is approaching the total instance limit of the ASE. It currently contains {0} App Service Plan instances of a maximum 201 instances. |
| ASE is unable to reach a dependency | The specified ASE is not able to reach {0}.  Ensure the guidelines defined here are followed: https://docs.microsoft.com/azure/app-service/environment/network-info. |
| ASE is suspended | The specified ASE is suspended. The ASE suspension may be due to an account shortfall or an invalid virtual network configuration. Resolve the root cause and resume the ASE to continue serving traffic. |
| ASE upgrade has started | A platform upgrade to the specified ASE has begun. Expect delays in scaling operations. |
| ASE upgrade has completed | A platform upgrade to the specified ASE has finished. |
| Scale operations have started | An App Service plan ({0}) has begun scaling. Desired state: {1} I{2} workers.
| Scale operations have completed | An App Service plan ({0}) has finished scaling. Current state: {1} I{2} workers. |
| Scale operations have failed | An App Service plan ({0}) has failed to scale. Current state: {1} I{2} workers. |

To enable logging on your ASE:

1. In the portal, go to **Diagnostics settings**.
1. Select **Add diagnostic setting**.
1. Provide a name for the log integration.
1. Select and configure the log destinations that you want.
1. Select **AppServiceEnvironmentPlatformLogs**.

![ASE diagnostic log settings][4]

If you integrate with Log Analytics, you can see the logs by selecting **Logs** from the ASE portal and creating a query against **AppServiceEnvironmentPlatformLogs**.

## Upgrade preference

If you have multiple ASEs, you might want some ASEs to be upgraded before others. Within the ASE **HostingEnvironment Resource Manager** object, you can set a value for **upgradePreference**. The **upgradePreference** setting can be configured by using a template, ARMClient, or https://resources.azure.com. The three possible values are:

- **None**: Azure will upgrade your ASE in no particular batch. This value is the default.
- **Early**: Your ASE will be upgraded in the first half of the App Service upgrades.
- **Late**: Your ASE will be upgraded in the second half of the App Service upgrades.

If you're using https://resources.azure.com, follow these steps to set the **upgradePreferences** value:

1. Go to resources.azure.com and sign in with your Azure account.
1. Go through the resources to subscriptions\/\[subscription name\]\/resourceGroups\/\[resource group name\]\/providers\/Microsoft.Web\/hostingEnvironments\/\[ASE name\].
1. Select **Read/Write** at the top.
1. Select **Edit**.
1. Set **upgradePreference** to whichever one of the three values you want.
1. Select **Patch**.

![resources azure com display][5]

The **upgradePreferences** feature makes the most sense when you have multiple ASEs because your "Early" ASEs will be upgraded before your "Late" ASEs. When you have multiple ASEs, you should set your development and test ASEs to be "Early" and your production ASEs to be "Late".

## Pricing

The pricing SKU called *Isolated* is for use only with ASEs. All App Service plans that are hosted in the ASE are in the Isolated pricing SKU. Isolated rates for App Service plans can vary by region.

In addition to the price of your App Service plans, there's a flat rate for the ASE itself. The flat rate doesn't change with the size of your ASE. It pays for the ASE infrastructure at a default scale rate of one additional front end for every 15 App Service plan instances.

If the default scale rate of one front end for every 15 App Service plan instances is not fast enough, you can adjust the ratio at which front ends are added or the size of the front ends. When you adjust the ratio or size, you pay for the front-end cores that would not be added by default.

For example, if you adjust the scale ratio to 10, a front end is added for every 10 instances in your App Service plans. The flat fee covers a scale rate of one front end for every 15 instances. With a scale ratio of 10, you pay a fee for the third front end that's added for the 10 App Service plan instances. You don't need to pay for it when you reach 15 instances because it was added automatically.

If you adjust the size of the front ends to two cores but don't adjust the ratio, you pay for the extra cores. An ASE is created with two front ends, so even below the automatic scaling threshold you would pay for two extra cores if you increased the size to two-core front ends.

For more information, see [Azure App Service pricing][Pricing].

## Delete an ASE

To delete an ASE:

1. Select **Delete** at the top of the **App Service Environment** pane.

1. Enter the name of your ASE to confirm that you want to delete it. When you delete an ASE, you also delete all the content within it.

    ![ASE deletion][3]

1. Select **OK**.

<!--Image references-->
[1]: ./media/using_an_app_service_environment/usingase-appcreate.png
[2]: ./media/using_an_app_service_environment/usingase-pricingtiers.png
[3]: ./media/using_an_app_service_environment/usingase-delete.png
[4]: ./media/using_an_app_service_environment/usingase-logsetup.png
[4]: ./media/using_an_app_service_environment/usingase-logs.png
[5]: ./media/using_an_app_service_environment/usingase-upgradepref.png

<!--Links-->
[Intro]: ./intro.md
[MakeExternalASE]: ./create-external-ase.md
[MakeASEfromTemplate]: ./create-from-template.md
[MakeILBASE]: ./create-ilb-ase.md
[ASENetwork]: ./network-info.md
[UsingASE]: ./using-an-ase.md
[UDRs]: ../../virtual-network/virtual-networks-udr-overview.md
[NSGs]: ../../virtual-network/security-overview.md
[ConfigureASEv1]: app-service-web-configure-an-app-service-environment.md
[ASEv1Intro]: app-service-app-service-environment-intro.md
[Functions]: ../../azure-functions/index.yml
[Pricing]: https://azure.microsoft.com/pricing/details/app-service/
[ARMOverview]: ../../azure-resource-manager/management/overview.md
[ConfigureSSL]: ../configure-ssl-certificate.md
[Kudu]: https://azure.microsoft.com/resources/videos/super-secret-kudu-debug-console-for-azure-web-sites/
[AppDeploy]: ../deploy-local-git.md
[ASEWAF]: app-service-app-service-environment-web-application-firewall.md
[AppGW]: ../../application-gateway/application-gateway-web-application-firewall-overview.md

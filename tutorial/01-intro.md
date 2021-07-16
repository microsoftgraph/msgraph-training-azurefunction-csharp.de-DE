---
ms.openlocfilehash: d35318e05a5ebae2316afeb84b731da5c8342ddc
ms.sourcegitcommit: 7c550d2bcd30f505913e0cd441fbe12ae4a93b27
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 07/15/2021
ms.locfileid: "53445969"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="26844-101">In diesem Lernprogramm erfahren Sie, wie Sie eine Azure-Funktion erstellen, die die Microsoft Graph-API zum Abrufen von Kalenderinformationen für einen Benutzer verwendet.</span><span class="sxs-lookup"><span data-stu-id="26844-101">This tutorial teaches you how to build an Azure Function that uses the Microsoft Graph API to retrieve calendar information for a user.</span></span>

> [!TIP]
> <span data-ttu-id="26844-102">Wenn Sie es vorziehen, nur das abgeschlossene Lernprogramm herunterzuladen, können Sie das GitHub Repository herunterladen oder [klonen.](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp)</span><span class="sxs-lookup"><span data-stu-id="26844-102">If you prefer to just download the completed tutorial, you can download or clone the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp).</span></span> <span data-ttu-id="26844-103">Anweisungen zum Konfigurieren der App mit einer App-ID und einem geheimen Schlüssel finden Sie in der README-Datei im **Demoordner.**</span><span class="sxs-lookup"><span data-stu-id="26844-103">See the README file in the **demo** folder for instructions on configuring the app with an app ID and secret.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="26844-104">Voraussetzungen</span><span class="sxs-lookup"><span data-stu-id="26844-104">Prerequisites</span></span>

<span data-ttu-id="26844-105">Bevor Sie mit diesem Lernprogramm beginnen, sollten Sie die folgenden Tools auf Ihrem Entwicklungscomputer installiert haben.</span><span class="sxs-lookup"><span data-stu-id="26844-105">Before you start this tutorial, you should have the following tools installed on your development machine.</span></span>

- [<span data-ttu-id="26844-106">.NET Core SDK</span><span class="sxs-lookup"><span data-stu-id="26844-106">.NET Core SDK</span></span>](https://dotnet.microsoft.com/download)
- [<span data-ttu-id="26844-107">Azure Functions Core Tools</span><span class="sxs-lookup"><span data-stu-id="26844-107">Azure Functions Core Tools</span></span>](https://docs.microsoft.com/azure/azure-functions/functions-run-local)
- [<span data-ttu-id="26844-108">Azure CLI</span><span class="sxs-lookup"><span data-stu-id="26844-108">Azure CLI</span></span>](https://docs.microsoft.com/cli/azure/install-azure-cli)
- [<span data-ttu-id="26844-109">ngrok</span><span class="sxs-lookup"><span data-stu-id="26844-109">ngrok</span></span>](https://ngrok.com/)

<span data-ttu-id="26844-110">Sie sollten auch über ein Microsoft-Geschäfts-, Schul- oder Unikonto mit Zugriff auf ein globales Administratorkonto in derselben Organisation verfügen.</span><span class="sxs-lookup"><span data-stu-id="26844-110">You should also have a Microsoft work or school account, with access to a global administrator account in the same organization.</span></span> <span data-ttu-id="26844-111">Wenn Sie kein Microsoft-Konto haben, können Sie [sich für das Office 365-Entwicklerprogramm registrieren,](https://developer.microsoft.com/office/dev-program) um ein kostenloses Office 365 Abonnement zu erhalten.</span><span class="sxs-lookup"><span data-stu-id="26844-111">If you don't have a Microsoft account, you can [sign up for the Office 365 Developer Program](https://developer.microsoft.com/office/dev-program) to get a free Office 365 subscription.</span></span>

> [!NOTE]
> <span data-ttu-id="26844-112">Dieses Lernprogramm wurde mit den folgenden Versionen der oben genannten Tools geschrieben.</span><span class="sxs-lookup"><span data-stu-id="26844-112">This tutorial was written with the following versions of the above tools.</span></span> <span data-ttu-id="26844-113">Die Schritte in diesem Handbuch funktionieren möglicherweise mit anderen Versionen, die jedoch nicht getestet wurden.</span><span class="sxs-lookup"><span data-stu-id="26844-113">The steps in this guide may work with other versions, but that has not been tested.</span></span>
>
> - <span data-ttu-id="26844-114">.NET Core SDK 5.0.203</span><span class="sxs-lookup"><span data-stu-id="26844-114">.NET Core SDK 5.0.203</span></span>
> - <span data-ttu-id="26844-115">Azure Functions Core Tools 3.0.3442</span><span class="sxs-lookup"><span data-stu-id="26844-115">Azure Functions Core Tools 3.0.3442</span></span>
> - <span data-ttu-id="26844-116">Azure CLI 2.23.0</span><span class="sxs-lookup"><span data-stu-id="26844-116">Azure CLI 2.23.0</span></span>
> - <span data-ttu-id="26844-117">ngrok 2.3.40</span><span class="sxs-lookup"><span data-stu-id="26844-117">ngrok 2.3.40</span></span>

## <a name="feedback"></a><span data-ttu-id="26844-118">Feedback</span><span class="sxs-lookup"><span data-stu-id="26844-118">Feedback</span></span>

<span data-ttu-id="26844-119">Bitte geben Sie Feedback zu diesem Lernprogramm im [GitHub Repository.](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp)</span><span class="sxs-lookup"><span data-stu-id="26844-119">Please provide any feedback on this tutorial in the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp).</span></span>

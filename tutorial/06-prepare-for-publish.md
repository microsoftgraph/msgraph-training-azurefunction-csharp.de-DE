---
ms.openlocfilehash: f034aca537c878d988e8c4e7db1ff28d0817761a
ms.sourcegitcommit: 7c550d2bcd30f505913e0cd441fbe12ae4a93b27
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 07/15/2021
ms.locfileid: "53445948"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="f61f6-101">In dieser Übung erfahren Sie, welche Änderungen an der Azure-Beispielfunktion erforderlich sind, um sich auf die Veröffentlichung in [einer Azure Functions-App](https://docs.microsoft.com/azure/azure-functions/functions-run-local#publish)vorzubereiten.</span><span class="sxs-lookup"><span data-stu-id="f61f6-101">In this exercise you'll learn about what changes are needed to the sample Azure Function to prepare for [publishing to an Azure Functions app](https://docs.microsoft.com/azure/azure-functions/functions-run-local#publish).</span></span>

## <a name="update-code"></a><span data-ttu-id="f61f6-102">Aktualisieren des Codes</span><span class="sxs-lookup"><span data-stu-id="f61f6-102">Update code</span></span>

<span data-ttu-id="f61f6-103">Die Konfiguration wird aus dem geheimen Benutzerspeicher gelesen, was nur für Ihren Entwicklungscomputer gilt.</span><span class="sxs-lookup"><span data-stu-id="f61f6-103">Configuration is read from the user secret store, which only applies to your development machine.</span></span> <span data-ttu-id="f61f6-104">Vor der Veröffentlichung in Azure müssen Sie den Speicherort Ihrer Konfiguration ändern und den Code in **"Program.cs"** entsprechend aktualisieren.</span><span class="sxs-lookup"><span data-stu-id="f61f6-104">Before you publish to Azure, you'll need to change where you store your configuration, and update the code in **Program.cs** accordingly.</span></span>

<span data-ttu-id="f61f6-105">Anwendungsgeheimnisse sollten im sicheren Speicher gespeichert werden, z. B. [Azure Key Vault.](https://docs.microsoft.com/azure/key-vault/general/overview)</span><span class="sxs-lookup"><span data-stu-id="f61f6-105">Application secrets should be stored in secure storage, such as [Azure Key Vault](https://docs.microsoft.com/azure/key-vault/general/overview).</span></span>

## <a name="update-cors-setting-for-azure-function"></a><span data-ttu-id="f61f6-106">Aktualisieren der CORS-Einstellung für Azure-Funktion</span><span class="sxs-lookup"><span data-stu-id="f61f6-106">Update CORS setting for Azure Function</span></span>

<span data-ttu-id="f61f6-107">In diesem Beispiel haben wir CORS in **local.settings.jsaktiviert,** damit die Testanwendung die Funktion aufrufen kann.</span><span class="sxs-lookup"><span data-stu-id="f61f6-107">In this sample we configured CORS in **local.settings.json** to allow the test application to call the function.</span></span> <span data-ttu-id="f61f6-108">Sie müssen Ihre veröffentlichte Funktion so konfigurieren, dass alle SPA-Apps zugelassen werden, die sie aufrufen.</span><span class="sxs-lookup"><span data-stu-id="f61f6-108">You'll need to configure your published function to allow any SPA apps that will call it.</span></span>

## <a name="update-app-registrations"></a><span data-ttu-id="f61f6-109">Aktualisieren von App-Registrierungen</span><span class="sxs-lookup"><span data-stu-id="f61f6-109">Update app registrations</span></span>

<span data-ttu-id="f61f6-110">Die `knownClientApplications` Eigenschaft im Manifest für die Graph Azure **Function-App-Registrierung** muss mit den Anwendungs-IDs aller Apps aktualisiert werden, die die Azure-Funktion aufrufen.</span><span class="sxs-lookup"><span data-stu-id="f61f6-110">The `knownClientApplications` property in the manifest for the **Graph Azure Function** app registration will need to be updated with the application IDs of any apps that will be calling the Azure Function.</span></span>

## <a name="recreate-existing-subscriptions"></a><span data-ttu-id="f61f6-111">Neu erstellen vorhandener Abonnements</span><span class="sxs-lookup"><span data-stu-id="f61f6-111">Recreate existing subscriptions</span></span>

<span data-ttu-id="f61f6-112">Alle Abonnements, die mithilfe der Webhook-URL auf Ihrem lokalen Computer oder ngrok erstellt wurden, sollten mithilfe der Produktions-URL der Azure-Funktion neu erstellt `Notify` werden.</span><span class="sxs-lookup"><span data-stu-id="f61f6-112">Any subscriptions created using the webhook URL on your local machine or ngrok should be recreated using the production URL of the `Notify` Azure Function.</span></span>

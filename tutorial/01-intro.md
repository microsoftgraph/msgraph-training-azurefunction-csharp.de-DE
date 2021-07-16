---
ms.openlocfilehash: d35318e05a5ebae2316afeb84b731da5c8342ddc
ms.sourcegitcommit: 7c550d2bcd30f505913e0cd441fbe12ae4a93b27
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 07/15/2021
ms.locfileid: "53445969"
---
<!-- markdownlint-disable MD002 MD041 -->

In diesem Lernprogramm erfahren Sie, wie Sie eine Azure-Funktion erstellen, die die Microsoft Graph-API zum Abrufen von Kalenderinformationen für einen Benutzer verwendet.

> [!TIP]
> Wenn Sie es vorziehen, nur das abgeschlossene Lernprogramm herunterzuladen, können Sie das GitHub Repository herunterladen oder [klonen.](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp) Anweisungen zum Konfigurieren der App mit einer App-ID und einem geheimen Schlüssel finden Sie in der README-Datei im **Demoordner.**

## <a name="prerequisites"></a>Voraussetzungen

Bevor Sie mit diesem Lernprogramm beginnen, sollten Sie die folgenden Tools auf Ihrem Entwicklungscomputer installiert haben.

- [.NET Core SDK](https://dotnet.microsoft.com/download)
- [Azure Functions Core Tools](https://docs.microsoft.com/azure/azure-functions/functions-run-local)
- [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli)
- [ngrok](https://ngrok.com/)

Sie sollten auch über ein Microsoft-Geschäfts-, Schul- oder Unikonto mit Zugriff auf ein globales Administratorkonto in derselben Organisation verfügen. Wenn Sie kein Microsoft-Konto haben, können Sie [sich für das Office 365-Entwicklerprogramm registrieren,](https://developer.microsoft.com/office/dev-program) um ein kostenloses Office 365 Abonnement zu erhalten.

> [!NOTE]
> Dieses Lernprogramm wurde mit den folgenden Versionen der oben genannten Tools geschrieben. Die Schritte in diesem Handbuch funktionieren möglicherweise mit anderen Versionen, die jedoch nicht getestet wurden.
>
> - .NET Core SDK 5.0.203
> - Azure Functions Core Tools 3.0.3442
> - Azure CLI 2.23.0
> - ngrok 2.3.40

## <a name="feedback"></a>Feedback

Bitte geben Sie Feedback zu diesem Lernprogramm im [GitHub Repository.](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp)

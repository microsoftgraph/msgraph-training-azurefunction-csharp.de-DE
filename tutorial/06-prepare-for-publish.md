---
ms.openlocfilehash: f034aca537c878d988e8c4e7db1ff28d0817761a
ms.sourcegitcommit: 7c550d2bcd30f505913e0cd441fbe12ae4a93b27
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 07/15/2021
ms.locfileid: "53445948"
---
<!-- markdownlint-disable MD002 MD041 -->

In dieser Übung erfahren Sie, welche Änderungen an der Azure-Beispielfunktion erforderlich sind, um sich auf die Veröffentlichung in [einer Azure Functions-App](https://docs.microsoft.com/azure/azure-functions/functions-run-local#publish)vorzubereiten.

## <a name="update-code"></a>Aktualisieren des Codes

Die Konfiguration wird aus dem geheimen Benutzerspeicher gelesen, was nur für Ihren Entwicklungscomputer gilt. Vor der Veröffentlichung in Azure müssen Sie den Speicherort Ihrer Konfiguration ändern und den Code in **"Program.cs"** entsprechend aktualisieren.

Anwendungsgeheimnisse sollten im sicheren Speicher gespeichert werden, z. B. [Azure Key Vault.](https://docs.microsoft.com/azure/key-vault/general/overview)

## <a name="update-cors-setting-for-azure-function"></a>Aktualisieren der CORS-Einstellung für Azure-Funktion

In diesem Beispiel haben wir CORS in **local.settings.jsaktiviert,** damit die Testanwendung die Funktion aufrufen kann. Sie müssen Ihre veröffentlichte Funktion so konfigurieren, dass alle SPA-Apps zugelassen werden, die sie aufrufen.

## <a name="update-app-registrations"></a>Aktualisieren von App-Registrierungen

Die `knownClientApplications` Eigenschaft im Manifest für die Graph Azure **Function-App-Registrierung** muss mit den Anwendungs-IDs aller Apps aktualisiert werden, die die Azure-Funktion aufrufen.

## <a name="recreate-existing-subscriptions"></a>Neu erstellen vorhandener Abonnements

Alle Abonnements, die mithilfe der Webhook-URL auf Ihrem lokalen Computer oder ngrok erstellt wurden, sollten mithilfe der Produktions-URL der Azure-Funktion neu erstellt `Notify` werden.

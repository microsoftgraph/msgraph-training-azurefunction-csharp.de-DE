---
ms.openlocfilehash: 181afe9acfe45ff619a50cf874669228b421b475
ms.sourcegitcommit: 7c550d2bcd30f505913e0cd441fbe12ae4a93b27
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 07/15/2021
ms.locfileid: "53445976"
---
<!-- markdownlint-disable MD002 MD041 -->

In dieser Übung werden Sie die Implementierung der Azure-Funktionen abschließen `SetSubscription` `Notify` und die Testanwendung aktualisieren, um Änderungen im Posteingang eines Benutzers zu abonnieren und abzubestellen.

- Die `SetSubscription` Funktion fungiert als API, sodass die Test-App ein [Abonnement](https://docs.microsoft.com/graph/webhooks) für Änderungen im Posteingang eines Benutzers erstellen oder löschen kann.
- Die `Notify` Funktion fungiert als Webhook, der vom Abonnement generierte Änderungsbenachrichtigungen empfängt.

Beide Funktionen verwenden den Fluss zur [Gewährung von Clientanmeldeinformationen,](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow) um ein Nur-App-Token zum Aufrufen von Microsoft Graph abzurufen. Da ein Administrator die Administratorzustimmung zu den erforderlichen Berechtigungsbereichen erteilt hat, ist keine Benutzerinteraktion erforderlich, um das Token abzurufen.

## <a name="add-client-credentials-authentication-to-the-azure-functions-project"></a>Hinzufügen der Authentifizierung mit Clientanmeldeinformationen zum Azure Functions-Projekt

In diesem Abschnitt implementieren Sie den Clientanmeldeinformationsfluss im Azure Functions-Projekt, um ein Zugriffstoken zu erhalten, das mit Microsoft Graph kompatibel ist.

1. Öffnen Sie Ihre CLI in dem Verzeichnis, das **GraphTutorial.csproj** enthält.

1. Fügen Sie ihre Webhook-Anwendungs-ID und ihren geheimen Schlüssel mit den folgenden Befehlen zum geheimen Speicher hinzu. Ersetzen Sie `YOUR_WEBHOOK_APP_ID_HERE` dies durch die Anwendungs-ID für den **Graph Azure Function Webhook.** Ersetzen Sie `YOUR_WEBHOOK_APP_SECRET_HERE` dies durch den Anwendungsschlüssel, den Sie im Azure-Portal für den **Graph Azure Function Webhook** erstellt haben.

    ```Shell
    dotnet user-secrets set webHookId "YOUR_WEBHOOK_APP_ID_HERE"
    dotnet user-secrets set webHookSecret "YOUR_WEBHOOK_APP_SECRET_HERE"
    ```

### <a name="create-a-client-credentials-authentication-provider"></a>Erstellen eines Authentifizierungsanbieters für Clientanmeldeinformationen

1. Erstellen Sie eine neue Datei im **Verzeichnis "./GraphTutorial/Authentication"** mit dem Namen **"ClientCredentialsAuthProvider.cs",** und fügen Sie den folgenden Code hinzu.

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/ClientCredentialsAuthProvider.cs" id="AuthProviderSnippet":::

Nehmen Sie sich einen Moment Zeit, um zu überlegen, was der Code in **"ClientCredentialsAuthProvider.cs"** bewirkt.

- Im Konstruktor wird eine **ConfidentialClientApplication** aus dem `Microsoft.Identity.Client` Paket initialisiert. Es verwendet die `WithAuthority(AadAuthorityAudience.AzureAdMyOrg, true)` und `.WithTenantId(tenantId)` Funktionen, um die Anmeldegruppe auf die angegebene Microsoft 365 Organisation zu beschränken.
- In der `GetAccessToken` Funktion wird aufgerufen, `AcquireTokenForClient` um ein Token für die Anwendung abzurufen. Der Tokenfluss der Clientanmeldeinformationen ist immer nicht interaktiv.
- Sie implementiert die `Microsoft.Graph.IAuthenticationProvider` Schnittstelle, sodass diese Klasse im Konstruktor der zum Authentifizieren ausgehender Anforderungen übergeben werden `GraphServiceClient` kann.

## <a name="update-graphclientservice"></a>Aktualisieren von GraphClientService

1. Öffnen Sie **GraphClientService.cs,** und fügen Sie der Klasse die folgende Eigenschaft hinzu.

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="AppGraphClientMembers":::

1. Ersetzen Sie die vorhandene `GetAppGraphClient`-Funktion durch Folgendes.

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="AppGraphClientFunctions":::

## <a name="implement-notify-function"></a>Implementieren der Funktion "Benachrichtigen"

In diesem Abschnitt implementieren Sie die `Notify` Funktion, die als Benachrichtigungs-URL für Änderungsbenachrichtigungen verwendet wird.

1. Erstellen Sie ein neues Verzeichnis im **GraphTutorials-Verzeichnis** mit dem Namen **Models**.

1. Erstellen Sie eine neue Datei im **Verzeichnis "Models"** mit dem Namen **"ResourceData.cs",** und fügen Sie den folgenden Code hinzu.

    :::code language="csharp" source="../demo/GraphTutorial/Models/ResourceData.cs" id="ResourceDataSnippet":::

1. Erstellen Sie eine neue Datei im **Verzeichnis "Models"** mit dem Namen **"ChangeNotificationPayload.cs",** und fügen Sie den folgenden Code hinzu.

    :::code language="csharp" source="../demo/GraphTutorial/Models/ChangeNotificationPayload.cs" id="ChangeNotificationSnippet":::

1. Erstellen Sie eine neue Datei im **Verzeichnis "Models"** mit dem Namen **"NotificationList.cs",** und fügen Sie den folgenden Code hinzu.

    :::code language="csharp" source="../demo/GraphTutorial/Models/NotificationList.cs" id="NotificationListSnippet":::

1. Öffnen Sie **./GraphTutorial/Notify.cs,** und ersetzen Sie den gesamten Inhalt durch Folgendes.

    :::code language="csharp" source="../demo/GraphTutorial/Notify.cs" id="NotifySnippet":::

Nehmen Sie sich einen Moment Zeit, um zu überlegen, was der Code in **Notify.cs** bewirkt.

- Die `Run` Funktion überprüft, ob ein Abfrageparameter vorhanden `validationToken` ist. Wenn dieser Parameter vorhanden ist, verarbeitet er die Anforderung als [Validierungsanforderung](https://docs.microsoft.com/graph/webhooks#notification-endpoint-validation)und antwortet entsprechend.
- Wenn es sich bei der Anforderung nicht um eine Überprüfungsanforderung handelt, wird die JSON-Nutzlast in eine deserialisiert. `ChangeNotificationCollection`
- Jede Benachrichtigung in der Liste wird auf den erwarteten Clientstatuswert überprüft und verarbeitet.
- Die Nachricht, die die Benachrichtigung ausgelöst hat, wird mit Microsoft Graph abgerufen.

## <a name="implement-setsubscription-function"></a>Implementieren der SetSubscription-Funktion

In diesem Abschnitt implementieren Sie die SetSubscription-Funktion. Diese Funktion fungiert als API, die von der Testanwendung aufgerufen wird, um ein Abonnement im Posteingang eines Benutzers zu erstellen oder zu löschen.

1. Erstellen Sie eine neue Datei im **Verzeichnis "Models"** mit dem Namen **"SetSubscriptionPayload.cs",** und fügen Sie den folgenden Code hinzu.

    :::code language="csharp" source="../demo/GraphTutorial/Models/SetSubscriptionPayload.cs" id="SetSubscriptionPayloadSnippet":::

1. Öffnen Sie **./GraphTutorial/SetSubscription.cs,** und ersetzen Sie den gesamten Inhalt durch Folgendes.

    :::code language="csharp" source="../demo/GraphTutorial/SetSubscription.cs" id="SetSubscriptionSnippet":::

Nehmen Sie sich einen Moment Zeit, um zu überlegen, was der Code in **"SetSubscription.cs"** bewirkt.

- Die `Run` Funktion liest die JSON-Nutzlast, die in der POST-Anforderung gesendet wurde, um den Anforderungstyp (Abonnieren oder Kündigen des Abonnements), die benutzer-ID, die abonniert werden soll, und die Abonnement-ID zum Kündigen des Abonnements zu ermitteln.
- Wenn es sich bei der Anforderung um eine Abonnementanforderung handelt, verwendet sie das Microsoft Graph SDK, um ein neues Abonnement im Posteingang des angegebenen Benutzers zu erstellen. Das Abonnement benachrichtigt, wenn Nachrichten erstellt oder aktualisiert werden. Das neue Abonnement wird in der JSON-Nutzlast der Antwort zurückgegeben.
- Wenn es sich bei der Anforderung um eine Anforderung zum Kündigen des Abonnements handelt, verwendet sie das Microsoft Graph SDK, um das angegebene Abonnement zu löschen.

## <a name="call-setsubscription-from-the-test-app"></a>Aufrufen von "SetSubscription" aus der Test-App

In diesem Abschnitt implementieren Sie Funktionen zum Erstellen und Löschen von Abonnements in der Test-App.

1. Öffnen Sie **./TestClient/azurefunctions.js,** und fügen Sie die folgende Funktion hinzu.

    :::code language="javascript" source="../demo/TestClient/azurefunctions.js" id="createSubscriptionSnippet":::

    Dieser Code ruft die `SetSubscription` Azure-Funktion zum Abonnieren auf und fügt das neue Abonnement dem Array von Abonnements in der Sitzung hinzu.

1. Fügen Sie die folgende Funktion zu **azurefunctions.js** hinzu.

    :::code language="javascript" source="../demo/TestClient/azurefunctions.js" id="deleteSubscriptionSnippet":::

    Dieser Code ruft die `SetSubscription` Azure-Funktion auf, um sich vom Abonnement abzumelden, und entfernt das Abonnement aus dem Array der Abonnements in der Sitzung.

1. Wenn ngrok nicht ausgeführt wird, führen Sie ngrok ( `ngrok http 7071` ) aus, und kopieren Sie die HTTPS-Weiterleitungs-URL.

1. Fügen Sie die ngrok-URL zum geheimen Benutzerspeicher hinzu, indem Sie den folgenden Befehl ausführen.

    ```Shell
    dotnet user-secrets set ngrokUrl "YOUR_NGROK_URL_HERE"
    ```

    > [!IMPORTANT]
    > Wenn Sie ngrok neu starten, müssen Sie diesen Befehl wiederholen, um Ihre ngrok-URL zu aktualisieren.

1. Ändern Sie das aktuelle Verzeichnis in Ihrer CLI in das Verzeichnis **./GraphTutorial,** und führen Sie den folgenden Befehl aus, um die Azure-Funktion lokal zu starten.

    ```Shell
    func start
    ```

1. Aktualisieren Sie die  SPA, und wählen Sie das Abonnement-Navigationselement aus. Geben Sie eine Benutzer-ID für einen Benutzer in Ihrer Microsoft 365 Organisation ein, der über ein Exchange Online Postfach verfügt. Dies kann entweder die des Benutzers `id` (von Microsoft Graph) oder das des Benutzers `userPrincipalName` sein. Klicken Sie auf **"Abonnieren".**

1. Die Seite wird aktualisiert, auf der das neue Abonnement in der Tabelle angezeigt wird.

1. Senden Sie eine E-Mail an den Benutzer. Nach kurzer Zeit sollte die `Notify` Funktion aufgerufen werden. Sie können dies in der ngrok-Webschnittstelle ( `http://localhost:4040` ) oder in der Debugausgabe des Azure Function-Projekts überprüfen.

    ```Shell
    ...
    [7/8/2020 7:33:57 PM] The following message was created:
    [7/8/2020 7:33:57 PM] Subject: Hi Megan!, ID: AAMkAGUyN2I4N2RlLTEzMTAtNDBmYy1hODdlLTY2NTQwODE2MGEwZgBGAAAAAAA2J9QH-DvMRK3pBt_8rA6nBwCuPIFjbMEkToHcVnQirM5qAAAAAAEMAACuPIFjbMEkToHcVnQirM5qAACHmpAsAAA=
    [7/8/2020 7:33:57 PM] Executed 'Notify' (Succeeded, Id=9c40af0b-e082-4418-aa3a-aee624f30e7a)
    ...
    ```

1. Klicken Sie in der Test-App in der Tabellenzeile für das Abonnement auf **"Löschen".** Die Seite wird aktualisiert, und das Abonnement ist nicht mehr in der Tabelle enthalten.

---
ms.openlocfilehash: eb227079656e2a57550511c3abfacb49935fe46a
ms.sourcegitcommit: 141fe5c30dea84029ef61cf82558c35f2a744b65
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 12/11/2020
ms.locfileid: "49655240"
---
<!-- markdownlint-disable MD002 MD041 -->

In dieser Übung beenden Sie die Implementierung der Azure-Funktionen `SetSubscription` und `Notify` aktualisieren die Testanwendung so, dass Änderungen im Posteingang eines Benutzers abonniert und abgemeldet werden.

- Die `SetSubscription` Funktion fungiert als API, sodass die Test-App ein [Abonnement](https://docs.microsoft.com/graph/webhooks) für Änderungen im Posteingang eines Benutzers erstellen oder löschen kann.
- Die `Notify` Funktion fungiert als webhook, der vom Abonnement generierte Änderungsbenachrichtigungen empfängt.

Beide Funktionen verwenden den [Clientanmeldeinformationen Zuteilungs Fluss](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow) , um ein app-only-Token zum Aufrufen von Microsoft Graph zu erhalten. Da ein Administrator den erforderlichen Berechtigungs Bereichen Administrator Zustimmung erteilt hat, ist keine Benutzerinteraktion erforderlich, um das Token abzurufen.

## <a name="add-client-credentials-authentication-to-the-azure-functions-project"></a>Hinzufügen von Clientanmeldeinformationen Authentifizierung zum Azure-Funktionen-Projekt

In diesem Abschnitt implementieren Sie den Client-Anmelde Informationsfluss im Azure-Funktionen-Projekt, um ein mit Microsoft Graph kompatibles Zugriffstoken abzurufen.

1. Öffnen Sie die CLI in dem Verzeichnis, das **GraphTutorial. csproj** enthält.

1. Fügen Sie die webhook-Anwendungs-ID und das geheime Kennwort dem geheimen Speicher mithilfe der folgenden Befehle hinzu. Ersetzen Sie `YOUR_WEBHOOK_APP_ID_HERE` durch die Anwendungs-ID für die **Graph-Azure-Funktion webhook**. Ersetzen `YOUR_WEBHOOK_APP_SECRET_HERE` Sie durch den Anwendungsschlüssel, den Sie im Azure-Portal für den **Graph Azure Function webhook** erstellt haben.

    ```Shell
    dotnet user-secrets set webHookId "YOUR_WEBHOOK_APP_ID_HERE"
    dotnet user-secrets set webHookSecret "YOUR_WEBHOOK_APP_SECRET_HERE"
    ```

### <a name="create-a-client-credentials-authentication-provider"></a>Erstellen eines Authentifizierungsanbieters für Clientanmeldeinformationen

1. Erstellen Sie eine neue Datei im **./GraphTutorial/Authentication** -Verzeichnis mit dem Namen **ClientCredentialsAuthProvider.cs** , und fügen Sie den folgenden Code hinzu.

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/ClientCredentialsAuthProvider.cs" id="AuthProviderSnippet":::

Nehmen Sie sich einen Moment Zeit, um zu prüfen, was der Code in **ClientCredentialsAuthProvider.cs** macht.

- Im Konstruktor wird ein **ConfidentialClientApplication** aus dem `Microsoft.Identity.Client` Paket initialisiert. Es verwendet die `WithAuthority(AadAuthorityAudience.AzureAdMyOrg, true)` `.WithTenantId(tenantId)` Funktionen und, um die Anmeldegruppe auf die angegebene Microsoft 365-Organisation einzuschränken.
- In der `GetAccessToken` -Funktion wird aufgerufen, `AcquireTokenForClient` um ein Token für die Anwendung abzurufen. Der Token-Fluss des Clientanmeldeinformationen ist immer nicht interaktiv.
- Die `Microsoft.Graph.IAuthenticationProvider` Schnittstelle wird implementiert, sodass diese Klasse im Konstruktor von The übergeben werden kann, `GraphServiceClient` um ausgehende Anforderungen zu authentifizieren.

## <a name="update-graphclientservice"></a>GraphClientService aktualisieren

1. Öffnen Sie **GraphClientService.cs** , und fügen Sie der Klasse die folgende Eigenschaft hinzu.

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="AppGraphClientMembers":::

1. Ersetzen Sie die vorhandene `GetAppGraphClient`-Funktion durch Folgendes.

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="AppGraphClientFunctions":::

## <a name="implement-notify-function"></a>Implementieren der Notify-Funktion

In diesem Abschnitt implementieren Sie die `Notify` Funktion, die als Benachrichtigungs-URL für Änderungsbenachrichtigungen verwendet wird.

1. Erstellen Sie ein neues Verzeichnis im **GraphTutorials** -Verzeichnis mit dem Namen **Models**.

1. Erstellen Sie eine neue Datei im **Modell** Verzeichnis mit dem Namen **ResourceData.cs** , und fügen Sie den folgenden Code hinzu.

    :::code language="csharp" source="../demo/GraphTutorial/Models/ResourceData.cs" id="ResourceDataSnippet":::

1. Erstellen Sie eine neue Datei im **Modell** Verzeichnis mit dem Namen **ChangeNotification.cs** , und fügen Sie den folgenden Code hinzu.

    :::code language="csharp" source="../demo/GraphTutorial/Models/ChangeNotification.cs" id="ChangeNotificationSnippet":::

1. Erstellen Sie eine neue Datei im **Modell** Verzeichnis mit dem Namen **NotificationList.cs** , und fügen Sie den folgenden Code hinzu.

    :::code language="csharp" source="../demo/GraphTutorial/Models/NotificationList.cs" id="NotificationListSnippet":::

1. Öffnen Sie **/GraphTutorial/notify.cs** , und ersetzen Sie den gesamten Inhalt durch Folgendes.

    :::code language="csharp" source="../demo/GraphTutorial/Notify.cs" id="NotifySnippet":::

Nehmen Sie sich einen Moment Zeit, um zu prüfen, was der Code in **notify.cs** macht.

- Die `Run` Funktion überprüft das vorhanden sein eines `validationToken` Abfrageparameters. Wenn dieser Parameter vorhanden ist, wird die Anforderung als [Validierungs Anforderung](https://docs.microsoft.com/graph/webhooks#notification-endpoint-validation)verarbeitet und entsprechend reagiert.
- Wenn es sich bei der Anforderung nicht um eine Validierungs Anforderung handelt, wird die JSON-Nutzlast in eine deserialisiert `NotificationList` .
- Jede Benachrichtigung in der Liste wird auf den erwarteten Clientstatus Wert überprüft und verarbeitet.
- Die Nachricht, die die Benachrichtigung ausgelöst hat, wird mit Microsoft Graph abgerufen.

## <a name="implement-setsubscription-function"></a>Implementieren von setsubscription-Funktion

In diesem Abschnitt implementieren Sie die setsubscription-Funktion. Diese Funktion fungiert als API, die von der Testanwendung zum Erstellen oder Löschen eines Abonnements für den Posteingang eines Benutzers aufgerufen wird.

1. Erstellen Sie eine neue Datei im **Modell** Verzeichnis mit dem Namen **SetSubscriptionPayload.cs** , und fügen Sie den folgenden Code hinzu.

    :::code language="csharp" source="../demo/GraphTutorial/Models/SetSubscriptionPayload.cs" id="SetSubscriptionPayloadSnippet":::

1. Öffnen Sie **/GraphTutorial/SetSubscription.cs** , und ersetzen Sie den gesamten Inhalt durch Folgendes.

    :::code language="csharp" source="../demo/GraphTutorial/SetSubscription.cs" id="SetSubscriptionSnippet":::

Nehmen Sie sich einen Moment Zeit, um zu prüfen, was der Code in **SetSubscription.cs** macht.

- Die- `Run` Funktion liest die in der Post-Anforderung gesendete JSON-Nutzlast, um den Anforderungstyp (abonnieren oder kündigen), die zu abonnierende Benutzer-ID und die Abonnement-ID zum Kündigen des Abonnements zu ermitteln.
- Wenn es sich bei der Anforderung um eine subscribe-Anforderung handelt, wird mithilfe des Microsoft Graph-SDK ein neues Abonnement im Posteingang des angegebenen Benutzers erstellt. Das Abonnement wird benachrichtigt, wenn Nachrichten erstellt oder aktualisiert werden. Das neue Abonnement wird in der JSON-Nutzlast der Antwort zurückgegeben.
- Wenn es sich bei der Anforderung um eine unsubscribe-Anforderung handelt, wird das angegebene Abonnement mithilfe des Microsoft Graph-SDK gelöscht.

## <a name="call-setsubscription-from-the-test-app"></a>Aufrufen von setsubscription von der Test-App

In diesem Abschnitt implementieren Sie Funktionen zum Erstellen und Löschen von Abonnements in der Test-App.

1. Öffnen Sie **./Testclient/azurefunctions.js** , und fügen Sie die folgende Funktion hinzu.

    :::code language="javascript" source="../demo/TestClient/azurefunctions.js" id="createSubscriptionSnippet":::

    Dieser Code Ruft die `SetSubscription` Azure-Funktion zum abonnieren auf und fügt das neue Abonnement dem Array von Abonnements in der Sitzung hinzu.

1. Fügen Sie die folgende Funktion zu **azurefunctions.js** hinzu.

    :::code language="javascript" source="../demo/TestClient/azurefunctions.js" id="deleteSubscriptionSnippet":::

    Dieser Code Ruft die `SetSubscription` Azure-Funktion auf UN und entfernt das Abonnement aus dem Array von Abonnements in der Sitzung.

1. Wenn ngrok nicht ausgeführt wird, führen Sie ngrok () aus, `ngrok http 7071` und kopieren Sie die URL für die HTTPS-Weiterleitung.

1. Fügen Sie die ngrok-URL dem Benutzer geheimen Speicher hinzu, indem Sie den folgenden Befehl ausführen.

    ```Shell
    dotnet user-secrets set ngrokUrl "YOUR_NGROK_URL_HERE"
    ```

    > [!IMPORTANT]
    > Wenn Sie ngrok neu starten, müssen Sie diesen Befehl wiederholen, um die ngrok-URL zu aktualisieren.

1. Ändern Sie das aktuelle Verzeichnis in ihrer CLI in das Verzeichnis **./GraphTutorial** , und führen Sie den folgenden Befehl aus, um die Azure-Funktion lokal zu starten.

    ```Shell
    func start
    ```

1. Aktualisieren Sie das Spa, und wählen Sie das NAV-Element **Abonnements** aus. Geben Sie eine Benutzer-ID für einen Benutzer in Ihrer Microsoft 365-Organisation ein, der über ein Exchange Online Postfach verfügt. Dies kann entweder der Benutzer `id` (von Microsoft Graph) oder der des Benutzers sein `userPrincipalName` . Klicken Sie auf **abonnieren**.

1. Die Seite wird mit dem neuen Abonnement in der Tabelle aktualisiert.

1. Senden Sie eine e-Mail an den Benutzer. Nach kurzer Zeit `Notify` sollte die Funktion aufgerufen werden. Sie können dies in der ngrok-Weboberfläche ( `http://localhost:4040` ) oder in der Debug-Ausgabe des Azure-Funktions Projekts überprüfen.

    ```Shell
    ...
    [7/8/2020 7:33:57 PM] The following message was created:
    [7/8/2020 7:33:57 PM] Subject: Hi Megan!, ID: AAMkAGUyN2I4N2RlLTEzMTAtNDBmYy1hODdlLTY2NTQwODE2MGEwZgBGAAAAAAA2J9QH-DvMRK3pBt_8rA6nBwCuPIFjbMEkToHcVnQirM5qAAAAAAEMAACuPIFjbMEkToHcVnQirM5qAACHmpAsAAA=
    [7/8/2020 7:33:57 PM] Executed 'Notify' (Succeeded, Id=9c40af0b-e082-4418-aa3a-aee624f30e7a)
    ...
    ```

1. Klicken Sie in der Test-app in der Tabellenzeile für das Abonnement auf **Löschen** . Die Seite wird aktualisiert, und das Abonnement befindet sich nicht mehr in der Tabelle.

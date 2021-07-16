---
ms.openlocfilehash: 3e6a83c19de68b6047914a68d94e66dbab95c6c4
ms.sourcegitcommit: 7c550d2bcd30f505913e0cd441fbe12ae4a93b27
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 07/15/2021
ms.locfileid: "53445955"
---
<!-- markdownlint-disable MD002 MD041 -->

In dieser Übung werden Sie die Implementierung der Azure-Funktion abschließen `GetMyNewestMessage` und den Testclient aktualisieren, um die Funktion aufzurufen.

Die Azure-Funktion verwendet den [Im-Auftrag-von-Fluss.](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) Die grundlegende Reihenfolge der Ereignisse in diesem Fluss ist:

- Die Testanwendung verwendet einen interaktiven Authentifizierungsfluss, damit sich der Benutzer anmelden und seine Zustimmung erteilen kann. Es ruft ein Token zurück, das auf die Azure-Funktion beschränkt ist. Das Token enthält **KEINE** Microsoft Graph Bereiche.
- Die Testanwendung ruft die Azure-Funktion auf und sendet ihr Zugriffstoken im `Authorization` Header.
- Die Azure-Funktion überprüft das Token und tauscht es dann gegen ein zweites Zugriffstoken aus, das Microsoft Graph Bereiche enthält.
- Die Azure-Funktion ruft Microsoft Graph im Namen des Benutzers mithilfe des zweiten Zugriffstokens auf.

> [!IMPORTANT]
> Um das Speichern der Anwendungs-ID und des geheimen Schlüssels in der Quelle zu vermeiden, verwenden Sie den [.NET Secret Manager,](https://docs.microsoft.com/aspnet/core/security/app-secrets) um diese Werte zu speichern. Der Geheime Manager dient nur zu Entwicklungszwecken, Produktions-Apps sollten einen vertrauenswürdigen geheimen Manager zum Speichern geheimer Schlüssel verwenden.

## <a name="add-authentication-to-the-single-page-application"></a>Hinzufügen der Authentifizierung zur Einzelseitenanwendung

Beginnen Sie, indem Sie der SPA die Authentifizierung hinzufügen. Dadurch kann die Anwendung ein Zugriffstoken abrufen, das Zugriff zum Aufrufen der Azure-Funktion gewährt. Da es sich um eine SPA handelt, wird der [Autorisierungscodefluss mit PKCE](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-auth-code-flow)verwendet.

1. Erstellen Sie eine neue Datei im **TestClient-Verzeichnis** mit dem Namen **config.js,** und fügen Sie den folgenden Code hinzu.

    :::code language="javascript" source="../demo/TestClient/config.example.js" id="msalConfigSnippet":::

    Ersetzen Sie `YOUR_TEST_APP_APP_ID_HERE` dies durch die Anwendungs-ID, die Sie im Azure-Portal für die **Graph Azure Function Test App** erstellt haben. Ersetzen Sie `YOUR_TENANT_ID_HERE` dies durch den **Verzeichnis-ID-Wert (Mandant),** den Sie aus dem Azure-Portal kopiert haben. Ersetzen Sie `YOUR_AZURE_FUNCTION_APP_ID_HERE` dies durch die Anwendungs-ID für die **Graph Azure-Funktion.**

    > [!IMPORTANT]
    > Wenn Sie die Quellcodeverwaltung wie Git verwenden, wäre es jetzt ein guter Zeitpunkt, die **config.js** Datei aus der Quellcodeverwaltung auszuschließen, um zu vermeiden, dass Versehentlich Ihre App-IDs und Die Mandanten-ID offengelegt werden.

1. Erstellen Sie eine neue Datei im **TestClient-Verzeichnis** mit dem Namen **auth.js,** und fügen Sie den folgenden Code hinzu.

    :::code language="javascript" source="../demo/TestClient/auth.js" id="signInSignOutSnippet":::

    Überlegen Sie, was dieser Code bewirkt.

    - Es initialisiert eine `PublicClientApplication` Verwendung der in **config.js** gespeicherten Werte.
    - Es wird `loginPopup` verwendet, um den Benutzer mithilfe des Berechtigungsbereichs für die Azure-Funktion anzumelden.
    - Der Benutzername des Benutzers wird in der Sitzung gespeichert.

    > [!IMPORTANT]
    > Da die App verwendet `loginPopup` wird, müssen Sie möglicherweise den Popupblocker Ihres Browsers ändern, um Popups von `http://localhost:8080` zuzulassen.

1. Aktualisieren Sie die Seite, und melden Sie sich an. Die Seite sollte mit dem Benutzernamen aktualisiert werden, der angibt, dass die Anmeldung erfolgreich war.

## <a name="add-authentication-to-the-azure-function"></a>Hinzufügen der Authentifizierung zur Azure-Funktion

In diesem Abschnitt implementieren Sie den Im-Auftrag-von-Fluss in der `GetMyNewestMessage` Azure-Funktion, um ein Zugriffstoken zu erhalten, das mit Microsoft Graph kompatibel ist.

1. Initialisieren Sie den geheimen .NET-Entwicklungsspeicher, indem Sie Ihre CLI in dem Verzeichnis öffnen, das **GraphTutorial.csproj** enthält, und führen Sie den folgenden Befehl aus.

    ```Shell
    dotnet user-secrets init
    ```

1. Fügen Sie die Anwendungs-ID, den geheimen Schlüssel und die Mandanten-ID mit den folgenden Befehlen zum geheimen Speicher hinzu. Ersetzen Sie `YOUR_API_FUNCTION_APP_ID_HERE` dies durch die Anwendungs-ID für die **Graph Azure-Funktion.** Ersetzen Sie `YOUR_API_FUNCTION_APP_SECRET_HERE` dies durch den Anwendungsschlüssel, den Sie im Azure-Portal für die **Graph Azure-Funktion** erstellt haben. Ersetzen Sie `YOUR_TENANT_ID_HERE` dies durch den **Verzeichnis-ID-Wert (Mandant),** den Sie aus dem Azure-Portal kopiert haben.

    ```Shell
    dotnet user-secrets set apiFunctionId "YOUR_API_FUNCTION_APP_ID_HERE"
    dotnet user-secrets set apiFunctionSecret "YOUR_API_FUNCTION_APP_SECRET_HERE"
    dotnet user-secrets set tenantId "YOUR_TENANT_ID_HERE"
    ```

### <a name="process-the-incoming-bearer-token"></a>Verarbeiten des eingehenden Bearertokens

In diesem Abschnitt implementieren Sie eine Klasse zum Überprüfen und Verarbeiten des Bearertokens, das von der SPA an die Azure-Funktion gesendet wurde.

1. Erstellen Sie ein neues Verzeichnis im **GraphTutorial-Verzeichnis** mit dem Namen **"Authentifizierung".**

1. Erstellen Sie eine neue Datei namens **TokenValidationResult.cs** im Ordner **./GraphTutorial/Authentication,** und fügen Sie den folgenden Code hinzu.

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/TokenValidationResult.cs" id="TokenValidationResultSnippet":::

1. Erstellen Sie eine neue Datei namens **"TokenValidation.cs"** im Ordner **"./GraphTutorial/Authentication",** und fügen Sie den folgenden Code hinzu.

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/TokenValidation.cs" id="TokenValidationSnippet":::

Überlegen Sie, was dieser Code bewirkt.

- Es wird sichergestellt, dass ein Bearertoken im Header vorhanden `Authorization` ist.
- Es überprüft die Signatur und den Aussteller aus der veröffentlichten OpenID-Konfiguration von Azure.
- Es wird überprüft, ob die Zielgruppe ( `aud` Anspruch) mit der Anwendungs-ID der Azure-Funktion übereinstimmt.
- Es analysiert das Token und generiert eine MSAL-Konto-ID, die benötigt wird, um die Vorteile der Tokenzwischenspeicherung zu nutzen.

### <a name="create-an-on-behalf-of-authentication-provider"></a>Erstellen eines Authentifizierungsanbieters im Auftrag von

1. Erstellen Sie eine neue Datei im **Authentifizierungsverzeichnis** **"OnBehalfOfAuthProvider.cs",** und fügen Sie der Datei den folgenden Code hinzu.

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/OnBehalfOfAuthProvider.cs" id="AuthProviderSnippet":::

Nehmen Sie sich einen Moment Zeit, um zu überlegen, was der Code in **"OnBehalfOfAuthProvider.cs"** bewirkt.

- In der `GetAccessToken` Funktion wird zunächst versucht, ein Benutzertoken mithilfe von `AcquireTokenSilent` . Wenn dies fehlschlägt, wird das von der Test-App an die Azure-Funktion gesendete Bearertoken verwendet, um eine Benutzer assertion zu generieren. Anschließend wird diese Benutzerassertion verwendet, um ein Graph-kompatibles Token mithilfe von `AcquireTokenOnBehalfOf` abzurufen.
- Sie implementiert die `Microsoft.Graph.IAuthenticationProvider` Schnittstelle, sodass diese Klasse im Konstruktor der zum Authentifizieren ausgehender Anforderungen übergeben werden `GraphServiceClient` kann.

### <a name="implement-a-graph-client-service"></a>Implementieren eines Graph-Clientdiensts

In diesem Abschnitt implementieren Sie einen Dienst, der für die [Abhängigkeitsinjektion](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection)registriert werden kann. Der Dienst wird verwendet, um einen authentifizierten Graph-Client abzurufen.

1. Erstellen Sie ein neues Verzeichnis im **GraphTutorial-Verzeichnis** namens **"Dienste".**

1. Erstellen Sie eine neue Datei im **Verzeichnis "Services"** mit dem Namen **"IGraphClientService.cs",** und fügen Sie der Datei den folgenden Code hinzu.

    :::code language="csharp" source="../demo/GraphTutorial/Services/IGraphClientService.cs" id="IGraphClientServiceSnippet":::

1. Erstellen Sie eine neue Datei im **Verzeichnis "Services"** mit dem Namen **"GraphClientService.cs",** und fügen Sie der Datei den folgenden Code hinzu.

    ```csharp
    using GraphTutorial.Authentication;
    using Microsoft.Extensions.Configuration;
    using Microsoft.Extensions.Logging;
    using Microsoft.Identity.Client;
    using Microsoft.Graph;

    namespace GraphTutorial.Services
    {
        // Service added via dependency injection
        // Used to get an authenticated Graph client
        public class GraphClientService : IGraphClientService
        {
        }
    }
    ```

1. Fügen Sie der Klasse die folgenden Eigenschaften `GraphClientService` hinzu.

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="UserGraphClientMembers":::

1. Fügen Sie der Klasse die folgenden Funktionen `GraphClientService` hinzu.

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="UserGraphClientFunctions":::

1. Fügen Sie eine Platzhalterimplementierung für die `GetAppGraphClient` Funktion hinzu. Sie werden dies in späteren Abschnitten implementieren.

    ```csharp
    public GraphServiceClient GetAppGraphClient()
    {
        throw new System.NotImplementedException();
    }
    ```

    Die `GetUserGraphClient` Funktion übernimmt die Ergebnisse der Tokenüberprüfung und erstellt eine `GraphServiceClient` authentifizierte für den Benutzer.

1. Öffnen Sie **./GraphTutorial/Program.cs,** und ersetzen Sie den Inhalt durch Folgendes.

    :::code language="csharp" source="../demo/GraphTutorial/Program.cs" id="ProgramSnippet" highlight="15-23":::

    Dieser Code fügt der Konfiguration geheime Benutzerschlüssel hinzu und aktiviert die [Abhängigkeitsinjektion](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection) in Ihren Azure-Funktionen, wodurch der Dienst verfügbar `GraphClientService` wird.

### <a name="implement-getmynewestmessage-function"></a>Implementieren der GetMyNewestMessage-Funktion

1. Öffnen Sie **./GraphTutorial/GetMyNewestMessage.cs,** und ersetzen Sie den gesamten Inhalt durch Folgendes.

    :::code language="csharp" source="../demo/GraphTutorial/GetMyNewestMessage.cs" id="GetMyNewestMessageSnippet":::

#### <a name="review-the-code-in-getmynewestmessagecs"></a>Überprüfen des Codes in "GetMyNewestMessage.cs"

Nehmen Sie sich einen Moment Zeit, um zu überlegen, was der Code in **"GetMyNewestMessage.cs"** bewirkt.

- Im Konstruktor werden die objekte gespeichert, die über die `IConfiguration` `IGraphClientService` Abhängigkeitsinjektion übergeben werden.
- In der `Run` Funktion wird Folgendes ausgeführt:
  - Überprüft, ob die erforderlichen Konfigurationswerte im Objekt vorhanden `IConfiguration` sind.
  - Überprüft das Bearertoken und gibt einen `401` Statuscode zurück, wenn das Token ungültig ist.
  - Ruft einen Graph Client für `GraphClientService` den Benutzer ab, der diese Anforderung ausgeführt hat.
  - Verwendet das Microsoft Graph SDK, um die neueste Nachricht aus dem Posteingang des Benutzers abzurufen, und gibt sie als JSON-Text in der Antwort zurück.

## <a name="call-the-azure-function-from-the-test-app"></a>Aufrufen der Azure-Funktion aus der Test-App

1. Öffnen **Sieauth.js,** und fügen Sie die folgende Funktion hinzu, um ein Zugriffstoken abzurufen.

    :::code language="javascript" source="../demo/TestClient/auth.js" id="getTokenSnippet":::

    Überlegen Sie, was dieser Code bewirkt.

    - Zunächst wird versucht, ein Zugriffstoken im Hintergrund ohne Benutzerinteraktion abzurufen. Da der Benutzer bereits angemeldet sein sollte, sollte MSAL Token für den Benutzer im Cache haben.
    - Wenn dies mit einem Fehler fehlschlägt, der angibt, dass der Benutzer interagieren muss, wird versucht, ein Token interaktiv abzurufen.

    > [!TIP]
    > Sie können das Zugriffstoken analysieren [https://jwt.ms](https://jwt.ms) und bestätigen, dass der `aud` Anspruch die App-ID für die Azure-Funktion ist und dass der `scp` Anspruch den Berechtigungsbereich der Azure-Funktion enthält, nicht Microsoft Graph.

1. Erstellen Sie eine neue Datei im **TestClient-Verzeichnis** mit dem Namen **azurefunctions.js,** und fügen Sie den folgenden Code hinzu.

    :::code language="javascript" source="../demo/TestClient/azurefunctions.js" id="getLatestMessageSnippet":::

1. Ändern Sie das aktuelle Verzeichnis in Ihrer CLI in das Verzeichnis **./GraphTutorial,** und führen Sie den folgenden Befehl aus, um die Azure-Funktion lokal zu starten.

    ```Shell
    func start
    ```

1. Wenn die SPA nicht bereits bedient wird, öffnen Sie ein zweites CLI-Fenster, und ändern Sie das aktuelle Verzeichnis in das Verzeichnis **./TestClient.** Führen Sie den folgenden Befehl aus, um die Testanwendung auszuführen.

    ```Shell
    dotnet serve -h "Cache-Control: no-cache, no-store, must-revalidate"
    ```

1. Öffnen Sie Ihren Browser und navigieren Sie zu `http://localhost:8080`. Melden Sie sich an, und wählen Sie das Navigationselement **"Neueste Nachricht"** aus. Die App zeigt Informationen zur neuesten Nachricht im Posteingang des Benutzers an.

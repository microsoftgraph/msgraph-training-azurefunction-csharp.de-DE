---
ms.openlocfilehash: 914957809f268ad29f8cfc44c21dd9fb63699322
ms.sourcegitcommit: 7c550d2bcd30f505913e0cd441fbe12ae4a93b27
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 07/15/2021
ms.locfileid: "53445962"
---
<!-- markdownlint-disable MD002 MD041 -->

In diesem Lernprogramm erstellen Sie eine einfache Azure-Funktion, die HTTP-Triggerfunktionen implementiert, die Microsoft Graph aufrufen. Diese Funktionen umfassen die folgenden Szenarien:

- Implementiert eine API, um mithilfe der Flussauthentifizierung [im Auftrag von](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) Benutzern auf den Posteingang eines Benutzers zuzugreifen.
- Implementiert eine API zum Abonnieren und Kündigen von Benachrichtigungen im Posteingang eines Benutzers mithilfe der Flussauthentifizierung mit [Clientanmeldeinformationen.](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow)
- Implementiert einen Webhook, um [Änderungsbenachrichtigungen](https://docs.microsoft.com/graph/webhooks) von Microsoft Graph zu empfangen und mithilfe des Flusses zur Gewährung von Clientanmeldeinformationen auf Daten zuzugreifen.

Sie erstellen auch eine einfache JavaScript-Einzelseitenanwendung (Single-Page Application, SPA), um die in der Azure-Funktion implementierten APIs aufzurufen.

## <a name="create-azure-functions-project"></a>Erstellen eines Azure Functions-Projekts

1. Öffnen Sie die Befehlszeilenschnittstelle (CLI) in einem Verzeichnis, in dem Sie das Projekt erstellen möchten. Führen Sie den folgenden Befehl aus.

    ```Shell
    func init GraphTutorial --worker-runtime dotnetisolated
    ```

1. Ändern Sie das aktuelle Verzeichnis in Ihrer CLI in das **GraphTutorial-Verzeichnis,** und führen Sie die folgenden Befehle aus, um drei Funktionen im Projekt zu erstellen.

    ```Shell
    func new --name GetMyNewestMessage --template "HTTP trigger"
    func new --name SetSubscription --template "HTTP trigger"
    func new --name Notify --template "HTTP trigger"
    ```

1. Öffnen **Sielocal.settings.js,** und fügen Sie der Datei Folgendes hinzu, um CORS von `http://localhost:8080` der URL für die Testanwendung zuzulassen.

    ```json
    "Host": {
      "CORS": "http://localhost:8080"
    }
    ```

1. Führen Sie den folgenden Befehl aus, um das Projekt lokal auszuführen.

    ```Shell
    func start
    ```

1. Wenn alles funktioniert, wird die folgende Ausgabe angezeigt:

    ```Shell
    Functions:

        GetMyNewestMessage: [GET,POST] http://localhost:7071/api/GetMyNewestMessage

        Notify: [GET,POST] http://localhost:7071/api/Notify

        SetSubscription: [GET,POST] http://localhost:7071/api/SetSubscription
    ```

1. Stellen Sie sicher, dass die Funktionen ordnungsgemäß funktionieren, indem Sie Ihren Browser öffnen und zu den in der Ausgabe angezeigten Funktions-URLs navigieren. Die folgende Meldung sollte in Ihrem Browser angezeigt werden: `Welcome to Azure Functions!` .

## <a name="create-single-page-application"></a>Erstellen einer einzelseitigen Anwendung

1. Öffnen Sie Ihre CLI in einem Verzeichnis, in dem Sie das Projekt erstellen möchten. Erstellen Sie ein Verzeichnis mit dem Namen **"TestClient",** um Ihre HTML- und JavaScript-Dateien zu speichern.

1. Erstellen Sie eine neue Datei mit dem Namen **index.html** im **Verzeichnis "TestClient",** und fügen Sie den folgenden Code hinzu.

    :::code language="html" source="../demo/TestClient/index.html" id="indexSnippet":::

    Dies definiert das grundlegende Layout der App, einschließlich einer Navigationsleiste. Außerdem wird Folgendes hinzugefügt:

    - [Bootstrap](https://getbootstrap.com/) und das unterstützende JavaScript
    - [FontAwesome](https://fontawesome.com/)
    - [Microsoft-Authentifizierungsbibliothek für JavaScript (MSAL.js) 2.0](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-browser)

    > [!TIP]
    > Die Seite enthält ein Favicon, ( `<link rel="shortcut icon" href="g-raph.png">` ). Sie können diese Zeile entfernen oder die **g-raph.png** Datei aus [GitHub](https://github.com/microsoftgraph/g-raph)herunterladen.

1. Erstellen Sie eine neue Datei namens **"style.css"** im **Verzeichnis "TestClient",** und fügen Sie den folgenden Code hinzu.

    :::code language="css" source="../demo/TestClient/style.css":::

1. Erstellen Sie eine neue Datei mit dem Namen **ui.js** im **Verzeichnis "TestClient",** und fügen Sie den folgenden Code hinzu.

    :::code language="javascript" source="../demo/TestClient/ui.js" id="uiJsSnippet":::

    Dieser Code verwendet JavaScript, um die aktuelle Seite basierend auf der ausgewählten Ansicht zu rendern.

### <a name="test-the-single-page-application"></a>Testen der einzelseitigen Anwendung

> [!NOTE]
> Dieser Abschnitt enthält Anweisungen für die Verwendung von [dotnet-serve](https://github.com/natemcmaster/dotnet-serve) zum Ausführen eines einfachen HTTP-Testservers auf Ihrem Entwicklungscomputer. Die Verwendung dieses bestimmten Tools ist nicht erforderlich. Sie können einen beliebigen Testserver verwenden, den Sie für das **TestClient-Verzeichnis** bevorzugen.

1. Führen Sie den folgenden Befehl in Der CLI aus, um **dotnet-serve** zu installieren.

    ```Shell
    dotnet tool install --global dotnet-serve
    ```

1. Ändern Sie das aktuelle Verzeichnis in Ihrer CLI in das **TestClient-Verzeichnis,** und führen Sie den folgenden Befehl aus, um einen HTTP-Server zu starten.

    ```Shell
    dotnet serve -h "Cache-Control: no-cache, no-store, must-revalidate" -p 8080
    ```

1. Öffnen Sie Ihren Browser und navigieren Sie zu `http://localhost:8080`. Die Seite sollte gerendert werden, aber keine der Schaltflächen funktioniert derzeit.

## <a name="add-nuget-packages"></a>Hinzufügen von NuGet-Paketen

Bevor Sie fortfahren, installieren Sie einige zusätzliche NuGet Pakete, die Sie später verwenden werden.

- [Microsoft.Azure.Functions.Extensions](https://www.nuget.org/packages/Microsoft.Azure.Functions.Extensions) zum Aktivieren der Abhängigkeitsinjektion im Azure Functions-Projekt.
- [Microsoft.Extensions.ConfigUration. UserSecrets](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.UserSecrets) zum Lesen der Anwendungskonfiguration aus dem [geheimen .NET-Entwicklungsgeheimnisspeicher.](https://docs.microsoft.com/aspnet/core/security/app-secrets)
- [Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph/) zum Aufrufen von Microsoft Graph
- [Microsoft.Identity.Client](https://www.nuget.org/packages/Microsoft.Identity.Client/) zum Authentifizieren und Verwalten von Token.
- [Microsoft.IdentityModel.Protocols.OpenIdConnect](https://www.nuget.org/packages/Microsoft.IdentityModel.Protocols.OpenIdConnect) zum Abrufen der OpenID-Konfiguration für die Tokenüberprüfung.
- [System.IdentityModel.Tokens.Jwt](https://www.nuget.org/packages/System.IdentityModel.Tokens.Jwt) zum Überprüfen von Token, die an die Web-API gesendet werden.

1. Ändern Sie das aktuelle Verzeichnis in Ihrer CLI in das **GraphTutorial-Verzeichnis,** und führen Sie die folgenden Befehle aus.

    ```Shell
    dotnet add package Microsoft.Azure.Functions.Extensions --version 1.1.0
    dotnet add package Microsoft.Extensions.Configuration.UserSecrets --version 5.0.0
    dotnet add package Microsoft.Graph --version 4.0.0
    dotnet add package Microsoft.Identity.Client --version 4.34.0
    dotnet add package Microsoft.IdentityModel.Protocols.OpenIdConnect --version 6.11.1
    dotnet add package System.IdentityModel.Tokens.Jwt --version 6.11.1
    ```

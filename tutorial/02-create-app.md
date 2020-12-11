---
ms.openlocfilehash: 17c93f353c84ea2db28cd2e0203d30c5f320e36e
ms.sourcegitcommit: 141fe5c30dea84029ef61cf82558c35f2a744b65
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 12/11/2020
ms.locfileid: "49655233"
---
<!-- markdownlint-disable MD002 MD041 -->

In diesem Lernprogramm erstellen Sie eine einfache Azure-Funktion, mit der http-Triggerfunktionen implementiert werden, die Microsoft Graph aufrufen. Diese Funktionen beziehen sich auf die folgenden Szenarien:

- Implementiert eine API für den Zugriff auf den Posteingang eines Benutzers mithilfe [der im Auftrag von Ablauf](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) Authentifizierung.
- Implementiert eine API zum abonnieren und Aufheben des Abonnements für Benachrichtigungen im Posteingang eines Benutzers mithilfe der Verwendung von [Clientanmeldeinformationen zur Gewährung der Fluss](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow) Authentifizierung.
- Implementiert einen webhook zum Empfangen von [Änderungsbenachrichtigungen](https://docs.microsoft.com/graph/webhooks) von Microsoft Graph und zum Zugreifen auf Daten mithilfe von Clientanmeldeinformationen Grant Flow.

Außerdem erstellen Sie eine einfache JavaScript-Anwendung für einzelne Seiten (Spa), um die in der Azure-Funktion implementierten APIs aufzurufen.

## <a name="create-azure-functions-project"></a>Azure-Funktionen-Projekt erstellen

1. Öffnen Sie die Befehlszeilenschnittstelle (CLI) in einem Verzeichnis, in dem Sie das Projekt erstellen möchten. Führen Sie den folgenden Befehl aus.

    ```Shell
    func init GraphTutorial --dotnet
    ```

1. Ändern Sie das aktuelle Verzeichnis in ihrer CLI in das **GraphTutorial** -Verzeichnis, und führen Sie die folgenden Befehle aus, um drei Funktionen im Projekt zu erstellen.

    ```Shell
    func new --name GetMyNewestMessage --template "HTTP trigger" --language C#
    func new --name SetSubscription --template "HTTP trigger" --language C#
    func new --name Notify --template "HTTP trigger" --language C#
    ```

1. Öffnen Sie **local.settings.jsauf** , und fügen Sie der Datei, von der die `http://localhost:8080` URL für die Testanwendung zugelassen werden soll, die folgenden CORS hinzu.

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
    Http Functions:

        GetMyNewestMessage: [GET,POST] http://localhost:7071/api/GetMyNewestMessage

        Notify: [GET,POST] http://localhost:7071/api/Notify

        SetSubscription: [GET,POST] http://localhost:7071/api/SetSubscription
    ```

1. Stellen Sie sicher, dass die Funktionen ordnungsgemäß funktionieren, indem Sie Ihren Browser öffnen und zu den Funktions-URLs navigieren, die in der Ausgabe angezeigt werden. In Ihrem Browser sollte die folgende Meldung angezeigt werden: `This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response.` .

## <a name="create-single-page-application"></a>Erstellen einer einzelnen Seite-Anwendung

1. Öffnen Sie die CLI in einem Verzeichnis, in dem Sie das Projekt erstellen möchten. Erstellen Sie ein Verzeichnis mit dem Namen **Testclient** , um Ihre HTML-und JavaScript-Dateien aufzubewahren.

1. Erstellen Sie eine neue Datei mit dem Namen **index.html** im Verzeichnis **Testclient** , und fügen Sie den folgenden Code hinzu.

    :::code language="html" source="../demo/TestClient/index.html" id="indexSnippet":::

    Dadurch wird das grundlegende Layout der APP definiert, einschließlich einer Navigationsleiste. Außerdem wird Folgendes hinzugefügt:

    - [Bootstrap](https://getbootstrap.com/) und sein unterstützendes JavaScript
    - [FontAwesome](https://fontawesome.com/)
    - [Microsoft-Authentifizierungsbibliothek für JavaScript (MSAL.js) 2,0](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-browser)

    > [!TIP]
    > Die Seite enthält ein Favicon, ( `<link rel="shortcut icon" href="g-raph.png">` ). Sie können diese Verbindung entfernen oder die **g-raph.png** Datei von [GitHub](https://github.com/microsoftgraph/g-raph)herunterladen.

1. Erstellen Sie eine neue Datei mit dem Namen **Style. CSS** im **Testclient** -Verzeichnis, und fügen Sie den folgenden Code hinzu.

    :::code language="css" source="../demo/TestClient/style.css":::

1. Erstellen Sie eine neue Datei namens **ui.js** im Verzeichnis **Testclient** , und fügen Sie den folgenden Code hinzu.

    :::code language="javascript" source="../demo/TestClient/ui.js" id="uiJsSnippet":::

    In diesem Code wird JavaScript verwendet, um die aktuelle Seite basierend auf der ausgewählten Ansicht darzustellen.

### <a name="test-the-single-page-application"></a>Testen der Anwendung für eine einzelne Seite

> [!NOTE]
> Dieser Abschnitt enthält Anweisungen zur Verwendung von [dotnet-Serve](https://github.com/natemcmaster/dotnet-serve) zum Ausführen eines einfachen Test-HTTP-Servers auf dem Entwicklungscomputer. Die Verwendung dieses spezifischen Tools ist nicht erforderlich. Sie können einen beliebigen Testserver verwenden, den Sie bevorzugen, um das **Testclient** -Verzeichnis zu bedienen.

1. Führen Sie den folgenden Befehl in der CLI aus, um **dotnet-Serve** zu installieren.

    ```Shell
    dotnet tool install --global dotnet-serve
    ```

1. Ändern Sie das aktuelle Verzeichnis in ihrer CLI in das **Testclient** -Verzeichnis, und führen Sie den folgenden Befehl aus, um einen HTTP-Server zu starten.

    ```Shell
    dotnet serve -h "Cache-Control: no-cache, no-store, must-revalidate" -p 8080
    ```

1. Öffnen Sie Ihren Browser und navigieren Sie zu `http://localhost:8080`. Die Seite sollte gerendert werden, aber keine der Schaltflächen funktioniert derzeit.

## <a name="add-nuget-packages"></a>Hinzufügen von NuGet-Paketen

Bevor Sie fortfahren, installieren Sie einige zusätzliche NuGet-Pakete, die Sie später verwenden werden.

- [Microsoft. Azure. Functions. Extensions](https://www.nuget.org/packages/Microsoft.Azure.Functions.Extensions) zum Aktivieren der Abhängigkeitsinjektion im Azure-Funktionen-Projekt.
- [Microsoft.Extensions.Configuration. UserSecrets](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.UserSecrets) zum Lesen der Anwendungskonfiguration aus dem [geheimen .net-entwicklungsspeicher](https://docs.microsoft.com/aspnet/core/security/app-secrets).
- [Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph/) zum Aufrufen von Microsoft Graph
- [Microsoft. Identity. Client](https://www.nuget.org/packages/Microsoft.Identity.Client/) zum Authentifizieren und Verwalten von Token.
- [Microsoft. IdentityModel. Protocols. OpenIdConnect](https://www.nuget.org/packages/Microsoft.IdentityModel.Protocols.OpenIdConnect) zum Abrufen der OpenID-Konfiguration für die Token-Validierung.
- [System. IdentityModel. Tokens. JWT](https://www.nuget.org/packages/System.IdentityModel.Tokens.Jwt) zum Validieren von an die WebAPI gesendeten Token.

1. Ändern Sie das aktuelle Verzeichnis in ihrer CLI in das **GraphTutorial** -Verzeichnis, und führen Sie die folgenden Befehle aus.

    ```Shell
    dotnet add package Microsoft.Azure.Functions.Extensions --version 1.0.0
    dotnet add package Microsoft.Extensions.Configuration.UserSecrets --version 3.1.5
    dotnet add package Microsoft.Graph --version 3.8.0
    dotnet add package Microsoft.Identity.Client --version 4.15.0
    dotnet add package Microsoft.IdentityModel.Protocols.OpenIdConnect --version 6.7.1
    dotnet add package System.IdentityModel.Tokens.Jwt --version 6.7.1
    ```

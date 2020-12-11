---
ms.openlocfilehash: 17c93f353c84ea2db28cd2e0203d30c5f320e36e
ms.sourcegitcommit: 141fe5c30dea84029ef61cf82558c35f2a744b65
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 12/11/2020
ms.locfileid: "49655233"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="94aa7-101">In diesem Lernprogramm erstellen Sie eine einfache Azure-Funktion, mit der http-Triggerfunktionen implementiert werden, die Microsoft Graph aufrufen.</span><span class="sxs-lookup"><span data-stu-id="94aa7-101">In this tutorial, you will create a simple Azure Function that implements HTTP trigger functions that call Microsoft Graph.</span></span> <span data-ttu-id="94aa7-102">Diese Funktionen beziehen sich auf die folgenden Szenarien:</span><span class="sxs-lookup"><span data-stu-id="94aa7-102">These functions will cover the following scenarios:</span></span>

- <span data-ttu-id="94aa7-103">Implementiert eine API für den Zugriff auf den Posteingang eines Benutzers mithilfe [der im Auftrag von Ablauf](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) Authentifizierung.</span><span class="sxs-lookup"><span data-stu-id="94aa7-103">Implements an API to access a user's inbox using [on-behalf-of flow](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) authentication.</span></span>
- <span data-ttu-id="94aa7-104">Implementiert eine API zum abonnieren und Aufheben des Abonnements für Benachrichtigungen im Posteingang eines Benutzers mithilfe der Verwendung von [Clientanmeldeinformationen zur Gewährung der Fluss](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow) Authentifizierung.</span><span class="sxs-lookup"><span data-stu-id="94aa7-104">Implements an API to subscribe and unsubscribe for notifications on a user's inbox, using using [client credentials grant flow](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow) authentication.</span></span>
- <span data-ttu-id="94aa7-105">Implementiert einen webhook zum Empfangen von [Änderungsbenachrichtigungen](https://docs.microsoft.com/graph/webhooks) von Microsoft Graph und zum Zugreifen auf Daten mithilfe von Clientanmeldeinformationen Grant Flow.</span><span class="sxs-lookup"><span data-stu-id="94aa7-105">Implements a webhook to receive [change notifications](https://docs.microsoft.com/graph/webhooks) from Microsoft Graph and access data using client credentials grant flow.</span></span>

<span data-ttu-id="94aa7-106">Außerdem erstellen Sie eine einfache JavaScript-Anwendung für einzelne Seiten (Spa), um die in der Azure-Funktion implementierten APIs aufzurufen.</span><span class="sxs-lookup"><span data-stu-id="94aa7-106">You will also create a simple JavaScript single-page application (SPA) to call the APIs implemented in the Azure Function.</span></span>

## <a name="create-azure-functions-project"></a><span data-ttu-id="94aa7-107">Azure-Funktionen-Projekt erstellen</span><span class="sxs-lookup"><span data-stu-id="94aa7-107">Create Azure Functions project</span></span>

1. <span data-ttu-id="94aa7-108">Öffnen Sie die Befehlszeilenschnittstelle (CLI) in einem Verzeichnis, in dem Sie das Projekt erstellen möchten.</span><span class="sxs-lookup"><span data-stu-id="94aa7-108">Open your command-line interface (CLI) in a directory where you want to create the project.</span></span> <span data-ttu-id="94aa7-109">Führen Sie den folgenden Befehl aus.</span><span class="sxs-lookup"><span data-stu-id="94aa7-109">Run the following command.</span></span>

    ```Shell
    func init GraphTutorial --dotnet
    ```

1. <span data-ttu-id="94aa7-110">Ändern Sie das aktuelle Verzeichnis in ihrer CLI in das **GraphTutorial** -Verzeichnis, und führen Sie die folgenden Befehle aus, um drei Funktionen im Projekt zu erstellen.</span><span class="sxs-lookup"><span data-stu-id="94aa7-110">Change the current directory in your CLI to the **GraphTutorial** directory and run the following commands to create three functions in the project.</span></span>

    ```Shell
    func new --name GetMyNewestMessage --template "HTTP trigger" --language C#
    func new --name SetSubscription --template "HTTP trigger" --language C#
    func new --name Notify --template "HTTP trigger" --language C#
    ```

1. <span data-ttu-id="94aa7-111">Öffnen Sie **local.settings.jsauf** , und fügen Sie der Datei, von der die `http://localhost:8080` URL für die Testanwendung zugelassen werden soll, die folgenden CORS hinzu.</span><span class="sxs-lookup"><span data-stu-id="94aa7-111">Open **local.settings.json** and add the following to the file to allow CORS from `http://localhost:8080`, the URL for the test application.</span></span>

    ```json
    "Host": {
      "CORS": "http://localhost:8080"
    }
    ```

1. <span data-ttu-id="94aa7-112">Führen Sie den folgenden Befehl aus, um das Projekt lokal auszuführen.</span><span class="sxs-lookup"><span data-stu-id="94aa7-112">Run the following command to run the project locally.</span></span>

    ```Shell
    func start
    ```

1. <span data-ttu-id="94aa7-113">Wenn alles funktioniert, wird die folgende Ausgabe angezeigt:</span><span class="sxs-lookup"><span data-stu-id="94aa7-113">If everything is working, you will see the following output:</span></span>

    ```Shell
    Http Functions:

        GetMyNewestMessage: [GET,POST] http://localhost:7071/api/GetMyNewestMessage

        Notify: [GET,POST] http://localhost:7071/api/Notify

        SetSubscription: [GET,POST] http://localhost:7071/api/SetSubscription
    ```

1. <span data-ttu-id="94aa7-114">Stellen Sie sicher, dass die Funktionen ordnungsgemäß funktionieren, indem Sie Ihren Browser öffnen und zu den Funktions-URLs navigieren, die in der Ausgabe angezeigt werden.</span><span class="sxs-lookup"><span data-stu-id="94aa7-114">Verify that the functions are working correctly by opening your browser and browsing to the function URLs shown in the output.</span></span> <span data-ttu-id="94aa7-115">In Ihrem Browser sollte die folgende Meldung angezeigt werden: `This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response.` .</span><span class="sxs-lookup"><span data-stu-id="94aa7-115">You should see the following message in your browser: `This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response.`.</span></span>

## <a name="create-single-page-application"></a><span data-ttu-id="94aa7-116">Erstellen einer einzelnen Seite-Anwendung</span><span class="sxs-lookup"><span data-stu-id="94aa7-116">Create single-page application</span></span>

1. <span data-ttu-id="94aa7-117">Öffnen Sie die CLI in einem Verzeichnis, in dem Sie das Projekt erstellen möchten.</span><span class="sxs-lookup"><span data-stu-id="94aa7-117">Open your CLI in a directory where you want to create the project.</span></span> <span data-ttu-id="94aa7-118">Erstellen Sie ein Verzeichnis mit dem Namen **Testclient** , um Ihre HTML-und JavaScript-Dateien aufzubewahren.</span><span class="sxs-lookup"><span data-stu-id="94aa7-118">Create a directory named **TestClient** to hold your HTML and JavaScript files.</span></span>

1. <span data-ttu-id="94aa7-119">Erstellen Sie eine neue Datei mit dem Namen **index.html** im Verzeichnis **Testclient** , und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="94aa7-119">Create a new file named **index.html** in the **TestClient** directory and add the following code.</span></span>

    :::code language="html" source="../demo/TestClient/index.html" id="indexSnippet":::

    <span data-ttu-id="94aa7-120">Dadurch wird das grundlegende Layout der APP definiert, einschließlich einer Navigationsleiste.</span><span class="sxs-lookup"><span data-stu-id="94aa7-120">This defines the basic layout of the app, including a navigation bar.</span></span> <span data-ttu-id="94aa7-121">Außerdem wird Folgendes hinzugefügt:</span><span class="sxs-lookup"><span data-stu-id="94aa7-121">It also adds the following:</span></span>

    - <span data-ttu-id="94aa7-122">[Bootstrap](https://getbootstrap.com/) und sein unterstützendes JavaScript</span><span class="sxs-lookup"><span data-stu-id="94aa7-122">[Bootstrap](https://getbootstrap.com/) and its supporting JavaScript</span></span>
    - [<span data-ttu-id="94aa7-123">FontAwesome</span><span class="sxs-lookup"><span data-stu-id="94aa7-123">FontAwesome</span></span>](https://fontawesome.com/)
    - [<span data-ttu-id="94aa7-124">Microsoft-Authentifizierungsbibliothek für JavaScript (MSAL.js) 2,0</span><span class="sxs-lookup"><span data-stu-id="94aa7-124">Microsoft Authentication Library for JavaScript (MSAL.js) 2.0</span></span>](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-browser)

    > [!TIP]
    > <span data-ttu-id="94aa7-125">Die Seite enthält ein Favicon, ( `<link rel="shortcut icon" href="g-raph.png">` ).</span><span class="sxs-lookup"><span data-stu-id="94aa7-125">The page includes a favicon, (`<link rel="shortcut icon" href="g-raph.png">`).</span></span> <span data-ttu-id="94aa7-126">Sie können diese Verbindung entfernen oder die **g-raph.png** Datei von [GitHub](https://github.com/microsoftgraph/g-raph)herunterladen.</span><span class="sxs-lookup"><span data-stu-id="94aa7-126">You can remove this line, or you can download the **g-raph.png** file from [GitHub](https://github.com/microsoftgraph/g-raph).</span></span>

1. <span data-ttu-id="94aa7-127">Erstellen Sie eine neue Datei mit dem Namen **Style. CSS** im **Testclient** -Verzeichnis, und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="94aa7-127">Create a new file named **style.css** in the **TestClient** directory and add the following code.</span></span>

    :::code language="css" source="../demo/TestClient/style.css":::

1. <span data-ttu-id="94aa7-128">Erstellen Sie eine neue Datei namens **ui.js** im Verzeichnis **Testclient** , und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="94aa7-128">Create a new file named **ui.js** in the **TestClient** directory and add the following code.</span></span>

    :::code language="javascript" source="../demo/TestClient/ui.js" id="uiJsSnippet":::

    <span data-ttu-id="94aa7-129">In diesem Code wird JavaScript verwendet, um die aktuelle Seite basierend auf der ausgewählten Ansicht darzustellen.</span><span class="sxs-lookup"><span data-stu-id="94aa7-129">This code uses JavaScript to render the current page based on the selected view.</span></span>

### <a name="test-the-single-page-application"></a><span data-ttu-id="94aa7-130">Testen der Anwendung für eine einzelne Seite</span><span class="sxs-lookup"><span data-stu-id="94aa7-130">Test the single-page application</span></span>

> [!NOTE]
> <span data-ttu-id="94aa7-131">Dieser Abschnitt enthält Anweisungen zur Verwendung von [dotnet-Serve](https://github.com/natemcmaster/dotnet-serve) zum Ausführen eines einfachen Test-HTTP-Servers auf dem Entwicklungscomputer.</span><span class="sxs-lookup"><span data-stu-id="94aa7-131">This section includes instructions for using [dotnet-serve](https://github.com/natemcmaster/dotnet-serve) to run a simple testing HTTP server on your development machine.</span></span> <span data-ttu-id="94aa7-132">Die Verwendung dieses spezifischen Tools ist nicht erforderlich.</span><span class="sxs-lookup"><span data-stu-id="94aa7-132">Using this specific tool is not required.</span></span> <span data-ttu-id="94aa7-133">Sie können einen beliebigen Testserver verwenden, den Sie bevorzugen, um das **Testclient** -Verzeichnis zu bedienen.</span><span class="sxs-lookup"><span data-stu-id="94aa7-133">You can use any testing server you prefer to serve the **TestClient** directory.</span></span>

1. <span data-ttu-id="94aa7-134">Führen Sie den folgenden Befehl in der CLI aus, um **dotnet-Serve** zu installieren.</span><span class="sxs-lookup"><span data-stu-id="94aa7-134">Run the following command in your CLI to install **dotnet-serve**.</span></span>

    ```Shell
    dotnet tool install --global dotnet-serve
    ```

1. <span data-ttu-id="94aa7-135">Ändern Sie das aktuelle Verzeichnis in ihrer CLI in das **Testclient** -Verzeichnis, und führen Sie den folgenden Befehl aus, um einen HTTP-Server zu starten.</span><span class="sxs-lookup"><span data-stu-id="94aa7-135">Change the current directory in your CLI to the **TestClient** directory and run the following command to start an HTTP server.</span></span>

    ```Shell
    dotnet serve -h "Cache-Control: no-cache, no-store, must-revalidate" -p 8080
    ```

1. <span data-ttu-id="94aa7-136">Öffnen Sie Ihren Browser und navigieren Sie zu `http://localhost:8080`.</span><span class="sxs-lookup"><span data-stu-id="94aa7-136">Open your browser and navigate to `http://localhost:8080`.</span></span> <span data-ttu-id="94aa7-137">Die Seite sollte gerendert werden, aber keine der Schaltflächen funktioniert derzeit.</span><span class="sxs-lookup"><span data-stu-id="94aa7-137">The page should render, but none of the buttons currently work.</span></span>

## <a name="add-nuget-packages"></a><span data-ttu-id="94aa7-138">Hinzufügen von NuGet-Paketen</span><span class="sxs-lookup"><span data-stu-id="94aa7-138">Add NuGet packages</span></span>

<span data-ttu-id="94aa7-139">Bevor Sie fortfahren, installieren Sie einige zusätzliche NuGet-Pakete, die Sie später verwenden werden.</span><span class="sxs-lookup"><span data-stu-id="94aa7-139">Before moving on, install some additional NuGet packages that you will use later.</span></span>

- <span data-ttu-id="94aa7-140">[Microsoft. Azure. Functions. Extensions](https://www.nuget.org/packages/Microsoft.Azure.Functions.Extensions) zum Aktivieren der Abhängigkeitsinjektion im Azure-Funktionen-Projekt.</span><span class="sxs-lookup"><span data-stu-id="94aa7-140">[Microsoft.Azure.Functions.Extensions](https://www.nuget.org/packages/Microsoft.Azure.Functions.Extensions) to enable dependency injection in the Azure Functions project.</span></span>
- <span data-ttu-id="94aa7-141">[Microsoft.Extensions.Configuration. UserSecrets](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.UserSecrets) zum Lesen der Anwendungskonfiguration aus dem [geheimen .net-entwicklungsspeicher](https://docs.microsoft.com/aspnet/core/security/app-secrets).</span><span class="sxs-lookup"><span data-stu-id="94aa7-141">[Microsoft.Extensions.Configuration.UserSecrets](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.UserSecrets) to read application configuration from the [.NET development secret store](https://docs.microsoft.com/aspnet/core/security/app-secrets).</span></span>
- <span data-ttu-id="94aa7-142">[Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph/) zum Aufrufen von Microsoft Graph</span><span class="sxs-lookup"><span data-stu-id="94aa7-142">[Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph/) for making calls to Microsoft Graph.</span></span>
- <span data-ttu-id="94aa7-143">[Microsoft. Identity. Client](https://www.nuget.org/packages/Microsoft.Identity.Client/) zum Authentifizieren und Verwalten von Token.</span><span class="sxs-lookup"><span data-stu-id="94aa7-143">[Microsoft.Identity.Client](https://www.nuget.org/packages/Microsoft.Identity.Client/) for authenticating and managing tokens.</span></span>
- <span data-ttu-id="94aa7-144">[Microsoft. IdentityModel. Protocols. OpenIdConnect](https://www.nuget.org/packages/Microsoft.IdentityModel.Protocols.OpenIdConnect) zum Abrufen der OpenID-Konfiguration für die Token-Validierung.</span><span class="sxs-lookup"><span data-stu-id="94aa7-144">[Microsoft.IdentityModel.Protocols.OpenIdConnect](https://www.nuget.org/packages/Microsoft.IdentityModel.Protocols.OpenIdConnect) for retrieving OpenID configuration for token validation.</span></span>
- <span data-ttu-id="94aa7-145">[System. IdentityModel. Tokens. JWT](https://www.nuget.org/packages/System.IdentityModel.Tokens.Jwt) zum Validieren von an die WebAPI gesendeten Token.</span><span class="sxs-lookup"><span data-stu-id="94aa7-145">[System.IdentityModel.Tokens.Jwt](https://www.nuget.org/packages/System.IdentityModel.Tokens.Jwt) for validating tokens sent to the web API.</span></span>

1. <span data-ttu-id="94aa7-146">Ändern Sie das aktuelle Verzeichnis in ihrer CLI in das **GraphTutorial** -Verzeichnis, und führen Sie die folgenden Befehle aus.</span><span class="sxs-lookup"><span data-stu-id="94aa7-146">Change the current directory in your CLI to the **GraphTutorial** directory and run the following commands.</span></span>

    ```Shell
    dotnet add package Microsoft.Azure.Functions.Extensions --version 1.0.0
    dotnet add package Microsoft.Extensions.Configuration.UserSecrets --version 3.1.5
    dotnet add package Microsoft.Graph --version 3.8.0
    dotnet add package Microsoft.Identity.Client --version 4.15.0
    dotnet add package Microsoft.IdentityModel.Protocols.OpenIdConnect --version 6.7.1
    dotnet add package System.IdentityModel.Tokens.Jwt --version 6.7.1
    ```

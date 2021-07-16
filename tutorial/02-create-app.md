---
ms.openlocfilehash: 914957809f268ad29f8cfc44c21dd9fb63699322
ms.sourcegitcommit: 7c550d2bcd30f505913e0cd441fbe12ae4a93b27
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 07/15/2021
ms.locfileid: "53445962"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="04584-101">In diesem Lernprogramm erstellen Sie eine einfache Azure-Funktion, die HTTP-Triggerfunktionen implementiert, die Microsoft Graph aufrufen.</span><span class="sxs-lookup"><span data-stu-id="04584-101">In this tutorial, you will create a simple Azure Function that implements HTTP trigger functions that call Microsoft Graph.</span></span> <span data-ttu-id="04584-102">Diese Funktionen umfassen die folgenden Szenarien:</span><span class="sxs-lookup"><span data-stu-id="04584-102">These functions will cover the following scenarios:</span></span>

- <span data-ttu-id="04584-103">Implementiert eine API, um mithilfe der Flussauthentifizierung [im Auftrag von](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) Benutzern auf den Posteingang eines Benutzers zuzugreifen.</span><span class="sxs-lookup"><span data-stu-id="04584-103">Implements an API to access a user's inbox using [on-behalf-of flow](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) authentication.</span></span>
- <span data-ttu-id="04584-104">Implementiert eine API zum Abonnieren und Kündigen von Benachrichtigungen im Posteingang eines Benutzers mithilfe der Flussauthentifizierung mit [Clientanmeldeinformationen.](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow)</span><span class="sxs-lookup"><span data-stu-id="04584-104">Implements an API to subscribe and unsubscribe for notifications on a user's inbox, using using [client credentials grant flow](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow) authentication.</span></span>
- <span data-ttu-id="04584-105">Implementiert einen Webhook, um [Änderungsbenachrichtigungen](https://docs.microsoft.com/graph/webhooks) von Microsoft Graph zu empfangen und mithilfe des Flusses zur Gewährung von Clientanmeldeinformationen auf Daten zuzugreifen.</span><span class="sxs-lookup"><span data-stu-id="04584-105">Implements a webhook to receive [change notifications](https://docs.microsoft.com/graph/webhooks) from Microsoft Graph and access data using client credentials grant flow.</span></span>

<span data-ttu-id="04584-106">Sie erstellen auch eine einfache JavaScript-Einzelseitenanwendung (Single-Page Application, SPA), um die in der Azure-Funktion implementierten APIs aufzurufen.</span><span class="sxs-lookup"><span data-stu-id="04584-106">You will also create a simple JavaScript single-page application (SPA) to call the APIs implemented in the Azure Function.</span></span>

## <a name="create-azure-functions-project"></a><span data-ttu-id="04584-107">Erstellen eines Azure Functions-Projekts</span><span class="sxs-lookup"><span data-stu-id="04584-107">Create Azure Functions project</span></span>

1. <span data-ttu-id="04584-108">Öffnen Sie die Befehlszeilenschnittstelle (CLI) in einem Verzeichnis, in dem Sie das Projekt erstellen möchten.</span><span class="sxs-lookup"><span data-stu-id="04584-108">Open your command-line interface (CLI) in a directory where you want to create the project.</span></span> <span data-ttu-id="04584-109">Führen Sie den folgenden Befehl aus.</span><span class="sxs-lookup"><span data-stu-id="04584-109">Run the following command.</span></span>

    ```Shell
    func init GraphTutorial --worker-runtime dotnetisolated
    ```

1. <span data-ttu-id="04584-110">Ändern Sie das aktuelle Verzeichnis in Ihrer CLI in das **GraphTutorial-Verzeichnis,** und führen Sie die folgenden Befehle aus, um drei Funktionen im Projekt zu erstellen.</span><span class="sxs-lookup"><span data-stu-id="04584-110">Change the current directory in your CLI to the **GraphTutorial** directory and run the following commands to create three functions in the project.</span></span>

    ```Shell
    func new --name GetMyNewestMessage --template "HTTP trigger"
    func new --name SetSubscription --template "HTTP trigger"
    func new --name Notify --template "HTTP trigger"
    ```

1. <span data-ttu-id="04584-111">Öffnen **Sielocal.settings.js,** und fügen Sie der Datei Folgendes hinzu, um CORS von `http://localhost:8080` der URL für die Testanwendung zuzulassen.</span><span class="sxs-lookup"><span data-stu-id="04584-111">Open **local.settings.json** and add the following to the file to allow CORS from `http://localhost:8080`, the URL for the test application.</span></span>

    ```json
    "Host": {
      "CORS": "http://localhost:8080"
    }
    ```

1. <span data-ttu-id="04584-112">Führen Sie den folgenden Befehl aus, um das Projekt lokal auszuführen.</span><span class="sxs-lookup"><span data-stu-id="04584-112">Run the following command to run the project locally.</span></span>

    ```Shell
    func start
    ```

1. <span data-ttu-id="04584-113">Wenn alles funktioniert, wird die folgende Ausgabe angezeigt:</span><span class="sxs-lookup"><span data-stu-id="04584-113">If everything is working, you will see the following output:</span></span>

    ```Shell
    Functions:

        GetMyNewestMessage: [GET,POST] http://localhost:7071/api/GetMyNewestMessage

        Notify: [GET,POST] http://localhost:7071/api/Notify

        SetSubscription: [GET,POST] http://localhost:7071/api/SetSubscription
    ```

1. <span data-ttu-id="04584-114">Stellen Sie sicher, dass die Funktionen ordnungsgemäß funktionieren, indem Sie Ihren Browser öffnen und zu den in der Ausgabe angezeigten Funktions-URLs navigieren.</span><span class="sxs-lookup"><span data-stu-id="04584-114">Verify that the functions are working correctly by opening your browser and browsing to the function URLs shown in the output.</span></span> <span data-ttu-id="04584-115">Die folgende Meldung sollte in Ihrem Browser angezeigt werden: `Welcome to Azure Functions!` .</span><span class="sxs-lookup"><span data-stu-id="04584-115">You should see the following message in your browser: `Welcome to Azure Functions!`.</span></span>

## <a name="create-single-page-application"></a><span data-ttu-id="04584-116">Erstellen einer einzelseitigen Anwendung</span><span class="sxs-lookup"><span data-stu-id="04584-116">Create single-page application</span></span>

1. <span data-ttu-id="04584-117">Öffnen Sie Ihre CLI in einem Verzeichnis, in dem Sie das Projekt erstellen möchten.</span><span class="sxs-lookup"><span data-stu-id="04584-117">Open your CLI in a directory where you want to create the project.</span></span> <span data-ttu-id="04584-118">Erstellen Sie ein Verzeichnis mit dem Namen **"TestClient",** um Ihre HTML- und JavaScript-Dateien zu speichern.</span><span class="sxs-lookup"><span data-stu-id="04584-118">Create a directory named **TestClient** to hold your HTML and JavaScript files.</span></span>

1. <span data-ttu-id="04584-119">Erstellen Sie eine neue Datei mit dem Namen **index.html** im **Verzeichnis "TestClient",** und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="04584-119">Create a new file named **index.html** in the **TestClient** directory and add the following code.</span></span>

    :::code language="html" source="../demo/TestClient/index.html" id="indexSnippet":::

    <span data-ttu-id="04584-120">Dies definiert das grundlegende Layout der App, einschließlich einer Navigationsleiste.</span><span class="sxs-lookup"><span data-stu-id="04584-120">This defines the basic layout of the app, including a navigation bar.</span></span> <span data-ttu-id="04584-121">Außerdem wird Folgendes hinzugefügt:</span><span class="sxs-lookup"><span data-stu-id="04584-121">It also adds the following:</span></span>

    - <span data-ttu-id="04584-122">[Bootstrap](https://getbootstrap.com/) und das unterstützende JavaScript</span><span class="sxs-lookup"><span data-stu-id="04584-122">[Bootstrap](https://getbootstrap.com/) and its supporting JavaScript</span></span>
    - [<span data-ttu-id="04584-123">FontAwesome</span><span class="sxs-lookup"><span data-stu-id="04584-123">FontAwesome</span></span>](https://fontawesome.com/)
    - [<span data-ttu-id="04584-124">Microsoft-Authentifizierungsbibliothek für JavaScript (MSAL.js) 2.0</span><span class="sxs-lookup"><span data-stu-id="04584-124">Microsoft Authentication Library for JavaScript (MSAL.js) 2.0</span></span>](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-browser)

    > [!TIP]
    > <span data-ttu-id="04584-125">Die Seite enthält ein Favicon, ( `<link rel="shortcut icon" href="g-raph.png">` ).</span><span class="sxs-lookup"><span data-stu-id="04584-125">The page includes a favicon, (`<link rel="shortcut icon" href="g-raph.png">`).</span></span> <span data-ttu-id="04584-126">Sie können diese Zeile entfernen oder die **g-raph.png** Datei aus [GitHub](https://github.com/microsoftgraph/g-raph)herunterladen.</span><span class="sxs-lookup"><span data-stu-id="04584-126">You can remove this line, or you can download the **g-raph.png** file from [GitHub](https://github.com/microsoftgraph/g-raph).</span></span>

1. <span data-ttu-id="04584-127">Erstellen Sie eine neue Datei namens **"style.css"** im **Verzeichnis "TestClient",** und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="04584-127">Create a new file named **style.css** in the **TestClient** directory and add the following code.</span></span>

    :::code language="css" source="../demo/TestClient/style.css":::

1. <span data-ttu-id="04584-128">Erstellen Sie eine neue Datei mit dem Namen **ui.js** im **Verzeichnis "TestClient",** und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="04584-128">Create a new file named **ui.js** in the **TestClient** directory and add the following code.</span></span>

    :::code language="javascript" source="../demo/TestClient/ui.js" id="uiJsSnippet":::

    <span data-ttu-id="04584-129">Dieser Code verwendet JavaScript, um die aktuelle Seite basierend auf der ausgewählten Ansicht zu rendern.</span><span class="sxs-lookup"><span data-stu-id="04584-129">This code uses JavaScript to render the current page based on the selected view.</span></span>

### <a name="test-the-single-page-application"></a><span data-ttu-id="04584-130">Testen der einzelseitigen Anwendung</span><span class="sxs-lookup"><span data-stu-id="04584-130">Test the single-page application</span></span>

> [!NOTE]
> <span data-ttu-id="04584-131">Dieser Abschnitt enthält Anweisungen für die Verwendung von [dotnet-serve](https://github.com/natemcmaster/dotnet-serve) zum Ausführen eines einfachen HTTP-Testservers auf Ihrem Entwicklungscomputer.</span><span class="sxs-lookup"><span data-stu-id="04584-131">This section includes instructions for using [dotnet-serve](https://github.com/natemcmaster/dotnet-serve) to run a simple testing HTTP server on your development machine.</span></span> <span data-ttu-id="04584-132">Die Verwendung dieses bestimmten Tools ist nicht erforderlich.</span><span class="sxs-lookup"><span data-stu-id="04584-132">Using this specific tool is not required.</span></span> <span data-ttu-id="04584-133">Sie können einen beliebigen Testserver verwenden, den Sie für das **TestClient-Verzeichnis** bevorzugen.</span><span class="sxs-lookup"><span data-stu-id="04584-133">You can use any testing server you prefer to serve the **TestClient** directory.</span></span>

1. <span data-ttu-id="04584-134">Führen Sie den folgenden Befehl in Der CLI aus, um **dotnet-serve** zu installieren.</span><span class="sxs-lookup"><span data-stu-id="04584-134">Run the following command in your CLI to install **dotnet-serve**.</span></span>

    ```Shell
    dotnet tool install --global dotnet-serve
    ```

1. <span data-ttu-id="04584-135">Ändern Sie das aktuelle Verzeichnis in Ihrer CLI in das **TestClient-Verzeichnis,** und führen Sie den folgenden Befehl aus, um einen HTTP-Server zu starten.</span><span class="sxs-lookup"><span data-stu-id="04584-135">Change the current directory in your CLI to the **TestClient** directory and run the following command to start an HTTP server.</span></span>

    ```Shell
    dotnet serve -h "Cache-Control: no-cache, no-store, must-revalidate" -p 8080
    ```

1. <span data-ttu-id="04584-136">Öffnen Sie Ihren Browser und navigieren Sie zu `http://localhost:8080`.</span><span class="sxs-lookup"><span data-stu-id="04584-136">Open your browser and navigate to `http://localhost:8080`.</span></span> <span data-ttu-id="04584-137">Die Seite sollte gerendert werden, aber keine der Schaltflächen funktioniert derzeit.</span><span class="sxs-lookup"><span data-stu-id="04584-137">The page should render, but none of the buttons currently work.</span></span>

## <a name="add-nuget-packages"></a><span data-ttu-id="04584-138">Hinzufügen von NuGet-Paketen</span><span class="sxs-lookup"><span data-stu-id="04584-138">Add NuGet packages</span></span>

<span data-ttu-id="04584-139">Bevor Sie fortfahren, installieren Sie einige zusätzliche NuGet Pakete, die Sie später verwenden werden.</span><span class="sxs-lookup"><span data-stu-id="04584-139">Before moving on, install some additional NuGet packages that you will use later.</span></span>

- <span data-ttu-id="04584-140">[Microsoft.Azure.Functions.Extensions](https://www.nuget.org/packages/Microsoft.Azure.Functions.Extensions) zum Aktivieren der Abhängigkeitsinjektion im Azure Functions-Projekt.</span><span class="sxs-lookup"><span data-stu-id="04584-140">[Microsoft.Azure.Functions.Extensions](https://www.nuget.org/packages/Microsoft.Azure.Functions.Extensions) to enable dependency injection in the Azure Functions project.</span></span>
- <span data-ttu-id="04584-141">[Microsoft.Extensions.ConfigUration. UserSecrets](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.UserSecrets) zum Lesen der Anwendungskonfiguration aus dem [geheimen .NET-Entwicklungsgeheimnisspeicher.](https://docs.microsoft.com/aspnet/core/security/app-secrets)</span><span class="sxs-lookup"><span data-stu-id="04584-141">[Microsoft.Extensions.Configuration.UserSecrets](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.UserSecrets) to read application configuration from the [.NET development secret store](https://docs.microsoft.com/aspnet/core/security/app-secrets).</span></span>
- <span data-ttu-id="04584-142">[Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph/) zum Aufrufen von Microsoft Graph</span><span class="sxs-lookup"><span data-stu-id="04584-142">[Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph/) for making calls to Microsoft Graph.</span></span>
- <span data-ttu-id="04584-143">[Microsoft.Identity.Client](https://www.nuget.org/packages/Microsoft.Identity.Client/) zum Authentifizieren und Verwalten von Token.</span><span class="sxs-lookup"><span data-stu-id="04584-143">[Microsoft.Identity.Client](https://www.nuget.org/packages/Microsoft.Identity.Client/) for authenticating and managing tokens.</span></span>
- <span data-ttu-id="04584-144">[Microsoft.IdentityModel.Protocols.OpenIdConnect](https://www.nuget.org/packages/Microsoft.IdentityModel.Protocols.OpenIdConnect) zum Abrufen der OpenID-Konfiguration für die Tokenüberprüfung.</span><span class="sxs-lookup"><span data-stu-id="04584-144">[Microsoft.IdentityModel.Protocols.OpenIdConnect](https://www.nuget.org/packages/Microsoft.IdentityModel.Protocols.OpenIdConnect) for retrieving OpenID configuration for token validation.</span></span>
- <span data-ttu-id="04584-145">[System.IdentityModel.Tokens.Jwt](https://www.nuget.org/packages/System.IdentityModel.Tokens.Jwt) zum Überprüfen von Token, die an die Web-API gesendet werden.</span><span class="sxs-lookup"><span data-stu-id="04584-145">[System.IdentityModel.Tokens.Jwt](https://www.nuget.org/packages/System.IdentityModel.Tokens.Jwt) for validating tokens sent to the web API.</span></span>

1. <span data-ttu-id="04584-146">Ändern Sie das aktuelle Verzeichnis in Ihrer CLI in das **GraphTutorial-Verzeichnis,** und führen Sie die folgenden Befehle aus.</span><span class="sxs-lookup"><span data-stu-id="04584-146">Change the current directory in your CLI to the **GraphTutorial** directory and run the following commands.</span></span>

    ```Shell
    dotnet add package Microsoft.Azure.Functions.Extensions --version 1.1.0
    dotnet add package Microsoft.Extensions.Configuration.UserSecrets --version 5.0.0
    dotnet add package Microsoft.Graph --version 4.0.0
    dotnet add package Microsoft.Identity.Client --version 4.34.0
    dotnet add package Microsoft.IdentityModel.Protocols.OpenIdConnect --version 6.11.1
    dotnet add package System.IdentityModel.Tokens.Jwt --version 6.11.1
    ```

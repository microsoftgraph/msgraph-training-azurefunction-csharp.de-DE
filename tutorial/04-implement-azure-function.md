---
ms.openlocfilehash: 3e6a83c19de68b6047914a68d94e66dbab95c6c4
ms.sourcegitcommit: 7c550d2bcd30f505913e0cd441fbe12ae4a93b27
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 07/15/2021
ms.locfileid: "53445955"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="8201e-101">In dieser Übung werden Sie die Implementierung der Azure-Funktion abschließen `GetMyNewestMessage` und den Testclient aktualisieren, um die Funktion aufzurufen.</span><span class="sxs-lookup"><span data-stu-id="8201e-101">In this exercise you will finish implementing the Azure Function `GetMyNewestMessage` and update the test client to call the function.</span></span>

<span data-ttu-id="8201e-102">Die Azure-Funktion verwendet den [Im-Auftrag-von-Fluss.](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow)</span><span class="sxs-lookup"><span data-stu-id="8201e-102">The Azure Function uses the [on-behalf-of flow](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow).</span></span> <span data-ttu-id="8201e-103">Die grundlegende Reihenfolge der Ereignisse in diesem Fluss ist:</span><span class="sxs-lookup"><span data-stu-id="8201e-103">The basic order of events in this flow are:</span></span>

- <span data-ttu-id="8201e-104">Die Testanwendung verwendet einen interaktiven Authentifizierungsfluss, damit sich der Benutzer anmelden und seine Zustimmung erteilen kann.</span><span class="sxs-lookup"><span data-stu-id="8201e-104">The test application uses an interactive auth flow to allow the user to sign in and grant consent.</span></span> <span data-ttu-id="8201e-105">Es ruft ein Token zurück, das auf die Azure-Funktion beschränkt ist.</span><span class="sxs-lookup"><span data-stu-id="8201e-105">It gets back a token that is scoped to the Azure Function.</span></span> <span data-ttu-id="8201e-106">Das Token enthält **KEINE** Microsoft Graph Bereiche.</span><span class="sxs-lookup"><span data-stu-id="8201e-106">The token does **NOT** contain any Microsoft Graph scopes.</span></span>
- <span data-ttu-id="8201e-107">Die Testanwendung ruft die Azure-Funktion auf und sendet ihr Zugriffstoken im `Authorization` Header.</span><span class="sxs-lookup"><span data-stu-id="8201e-107">The test application invokes the Azure Function, sending its access token in the `Authorization` header.</span></span>
- <span data-ttu-id="8201e-108">Die Azure-Funktion überprüft das Token und tauscht es dann gegen ein zweites Zugriffstoken aus, das Microsoft Graph Bereiche enthält.</span><span class="sxs-lookup"><span data-stu-id="8201e-108">The Azure Function validates the token, then exchanges that token for a second access token that contains Microsoft Graph scopes.</span></span>
- <span data-ttu-id="8201e-109">Die Azure-Funktion ruft Microsoft Graph im Namen des Benutzers mithilfe des zweiten Zugriffstokens auf.</span><span class="sxs-lookup"><span data-stu-id="8201e-109">The Azure Function calls Microsoft Graph on the user's behalf using the second access token.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="8201e-110">Um das Speichern der Anwendungs-ID und des geheimen Schlüssels in der Quelle zu vermeiden, verwenden Sie den [.NET Secret Manager,](https://docs.microsoft.com/aspnet/core/security/app-secrets) um diese Werte zu speichern.</span><span class="sxs-lookup"><span data-stu-id="8201e-110">To avoid storing the application ID and secret in source, you will use the [.NET Secret Manager](https://docs.microsoft.com/aspnet/core/security/app-secrets) to store these values.</span></span> <span data-ttu-id="8201e-111">Der Geheime Manager dient nur zu Entwicklungszwecken, Produktions-Apps sollten einen vertrauenswürdigen geheimen Manager zum Speichern geheimer Schlüssel verwenden.</span><span class="sxs-lookup"><span data-stu-id="8201e-111">The Secret Manager is for development purposes only, production apps should use a trusted secret manager for storing secrets.</span></span>

## <a name="add-authentication-to-the-single-page-application"></a><span data-ttu-id="8201e-112">Hinzufügen der Authentifizierung zur Einzelseitenanwendung</span><span class="sxs-lookup"><span data-stu-id="8201e-112">Add authentication to the single page application</span></span>

<span data-ttu-id="8201e-113">Beginnen Sie, indem Sie der SPA die Authentifizierung hinzufügen.</span><span class="sxs-lookup"><span data-stu-id="8201e-113">Start by adding authentication to the SPA.</span></span> <span data-ttu-id="8201e-114">Dadurch kann die Anwendung ein Zugriffstoken abrufen, das Zugriff zum Aufrufen der Azure-Funktion gewährt.</span><span class="sxs-lookup"><span data-stu-id="8201e-114">This will allow the application to get an access token granting access to call the Azure Function.</span></span> <span data-ttu-id="8201e-115">Da es sich um eine SPA handelt, wird der [Autorisierungscodefluss mit PKCE](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-auth-code-flow)verwendet.</span><span class="sxs-lookup"><span data-stu-id="8201e-115">Because this is a SPA, it will use the [authorization code flow with PKCE](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-auth-code-flow).</span></span>

1. <span data-ttu-id="8201e-116">Erstellen Sie eine neue Datei im **TestClient-Verzeichnis** mit dem Namen **config.js,** und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="8201e-116">Create a new file in the **TestClient** directory named **config.js** and add the following code.</span></span>

    :::code language="javascript" source="../demo/TestClient/config.example.js" id="msalConfigSnippet":::

    <span data-ttu-id="8201e-117">Ersetzen Sie `YOUR_TEST_APP_APP_ID_HERE` dies durch die Anwendungs-ID, die Sie im Azure-Portal für die **Graph Azure Function Test App** erstellt haben.</span><span class="sxs-lookup"><span data-stu-id="8201e-117">Replace `YOUR_TEST_APP_APP_ID_HERE` with the application ID you created in the Azure portal for the **Graph Azure Function Test App**.</span></span> <span data-ttu-id="8201e-118">Ersetzen Sie `YOUR_TENANT_ID_HERE` dies durch den **Verzeichnis-ID-Wert (Mandant),** den Sie aus dem Azure-Portal kopiert haben.</span><span class="sxs-lookup"><span data-stu-id="8201e-118">Replace `YOUR_TENANT_ID_HERE` with the **Directory (tenant) ID** value you copied from the Azure portal.</span></span> <span data-ttu-id="8201e-119">Ersetzen Sie `YOUR_AZURE_FUNCTION_APP_ID_HERE` dies durch die Anwendungs-ID für die **Graph Azure-Funktion.**</span><span class="sxs-lookup"><span data-stu-id="8201e-119">Replace `YOUR_AZURE_FUNCTION_APP_ID_HERE` with the application ID for the **Graph Azure Function**.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="8201e-120">Wenn Sie die Quellcodeverwaltung wie Git verwenden, wäre es jetzt ein guter Zeitpunkt, die **config.js** Datei aus der Quellcodeverwaltung auszuschließen, um zu vermeiden, dass Versehentlich Ihre App-IDs und Die Mandanten-ID offengelegt werden.</span><span class="sxs-lookup"><span data-stu-id="8201e-120">If you're using source control such as git, now would be a good time to exclude the **config.js** file from source control to avoid inadvertently leaking your app IDs and tenant ID.</span></span>

1. <span data-ttu-id="8201e-121">Erstellen Sie eine neue Datei im **TestClient-Verzeichnis** mit dem Namen **auth.js,** und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="8201e-121">Create a new file in the **TestClient** directory named **auth.js** and add the following code.</span></span>

    :::code language="javascript" source="../demo/TestClient/auth.js" id="signInSignOutSnippet":::

    <span data-ttu-id="8201e-122">Überlegen Sie, was dieser Code bewirkt.</span><span class="sxs-lookup"><span data-stu-id="8201e-122">Consider what this code does.</span></span>

    - <span data-ttu-id="8201e-123">Es initialisiert eine `PublicClientApplication` Verwendung der in **config.js** gespeicherten Werte.</span><span class="sxs-lookup"><span data-stu-id="8201e-123">It initializes a `PublicClientApplication` using the values stored in **config.js**.</span></span>
    - <span data-ttu-id="8201e-124">Es wird `loginPopup` verwendet, um den Benutzer mithilfe des Berechtigungsbereichs für die Azure-Funktion anzumelden.</span><span class="sxs-lookup"><span data-stu-id="8201e-124">It uses `loginPopup` to sign the user in, using the permission scope for the Azure Function.</span></span>
    - <span data-ttu-id="8201e-125">Der Benutzername des Benutzers wird in der Sitzung gespeichert.</span><span class="sxs-lookup"><span data-stu-id="8201e-125">It stores the user's username in the session.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="8201e-126">Da die App verwendet `loginPopup` wird, müssen Sie möglicherweise den Popupblocker Ihres Browsers ändern, um Popups von `http://localhost:8080` zuzulassen.</span><span class="sxs-lookup"><span data-stu-id="8201e-126">Since the app uses `loginPopup`, you may need to change your browser's pop-up blocker to allow pop-ups from `http://localhost:8080`.</span></span>

1. <span data-ttu-id="8201e-127">Aktualisieren Sie die Seite, und melden Sie sich an.</span><span class="sxs-lookup"><span data-stu-id="8201e-127">Refresh the page and sign in.</span></span> <span data-ttu-id="8201e-128">Die Seite sollte mit dem Benutzernamen aktualisiert werden, der angibt, dass die Anmeldung erfolgreich war.</span><span class="sxs-lookup"><span data-stu-id="8201e-128">The page should update with the user name, indicating that the sign in was successful.</span></span>

## <a name="add-authentication-to-the-azure-function"></a><span data-ttu-id="8201e-129">Hinzufügen der Authentifizierung zur Azure-Funktion</span><span class="sxs-lookup"><span data-stu-id="8201e-129">Add authentication to the Azure Function</span></span>

<span data-ttu-id="8201e-130">In diesem Abschnitt implementieren Sie den Im-Auftrag-von-Fluss in der `GetMyNewestMessage` Azure-Funktion, um ein Zugriffstoken zu erhalten, das mit Microsoft Graph kompatibel ist.</span><span class="sxs-lookup"><span data-stu-id="8201e-130">In this section you'll implement the on-behalf-of flow in the `GetMyNewestMessage` Azure Function to get an access token compatible with Microsoft Graph.</span></span>

1. <span data-ttu-id="8201e-131">Initialisieren Sie den geheimen .NET-Entwicklungsspeicher, indem Sie Ihre CLI in dem Verzeichnis öffnen, das **GraphTutorial.csproj** enthält, und führen Sie den folgenden Befehl aus.</span><span class="sxs-lookup"><span data-stu-id="8201e-131">Initialize the .NET development secret store by opening your CLI in the directory that contains **GraphTutorial.csproj** and running the following command.</span></span>

    ```Shell
    dotnet user-secrets init
    ```

1. <span data-ttu-id="8201e-132">Fügen Sie die Anwendungs-ID, den geheimen Schlüssel und die Mandanten-ID mit den folgenden Befehlen zum geheimen Speicher hinzu.</span><span class="sxs-lookup"><span data-stu-id="8201e-132">Add your application ID, secret, and tenant ID to the secret store using the following commands.</span></span> <span data-ttu-id="8201e-133">Ersetzen Sie `YOUR_API_FUNCTION_APP_ID_HERE` dies durch die Anwendungs-ID für die **Graph Azure-Funktion.**</span><span class="sxs-lookup"><span data-stu-id="8201e-133">Replace `YOUR_API_FUNCTION_APP_ID_HERE` with the application ID for the **Graph Azure Function**.</span></span> <span data-ttu-id="8201e-134">Ersetzen Sie `YOUR_API_FUNCTION_APP_SECRET_HERE` dies durch den Anwendungsschlüssel, den Sie im Azure-Portal für die **Graph Azure-Funktion** erstellt haben.</span><span class="sxs-lookup"><span data-stu-id="8201e-134">Replace `YOUR_API_FUNCTION_APP_SECRET_HERE` with the application secret you created in the Azure portal for the **Graph Azure Function**.</span></span> <span data-ttu-id="8201e-135">Ersetzen Sie `YOUR_TENANT_ID_HERE` dies durch den **Verzeichnis-ID-Wert (Mandant),** den Sie aus dem Azure-Portal kopiert haben.</span><span class="sxs-lookup"><span data-stu-id="8201e-135">Replace `YOUR_TENANT_ID_HERE` with the **Directory (tenant) ID** value you copied from the Azure portal.</span></span>

    ```Shell
    dotnet user-secrets set apiFunctionId "YOUR_API_FUNCTION_APP_ID_HERE"
    dotnet user-secrets set apiFunctionSecret "YOUR_API_FUNCTION_APP_SECRET_HERE"
    dotnet user-secrets set tenantId "YOUR_TENANT_ID_HERE"
    ```

### <a name="process-the-incoming-bearer-token"></a><span data-ttu-id="8201e-136">Verarbeiten des eingehenden Bearertokens</span><span class="sxs-lookup"><span data-stu-id="8201e-136">Process the incoming bearer token</span></span>

<span data-ttu-id="8201e-137">In diesem Abschnitt implementieren Sie eine Klasse zum Überprüfen und Verarbeiten des Bearertokens, das von der SPA an die Azure-Funktion gesendet wurde.</span><span class="sxs-lookup"><span data-stu-id="8201e-137">In this section you'll implement a class to validate and process the bearer token sent from the SPA to the Azure Function.</span></span>

1. <span data-ttu-id="8201e-138">Erstellen Sie ein neues Verzeichnis im **GraphTutorial-Verzeichnis** mit dem Namen **"Authentifizierung".**</span><span class="sxs-lookup"><span data-stu-id="8201e-138">Create a new directory in the **GraphTutorial** directory named **Authentication**.</span></span>

1. <span data-ttu-id="8201e-139">Erstellen Sie eine neue Datei namens **TokenValidationResult.cs** im Ordner **./GraphTutorial/Authentication,** und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="8201e-139">Create a new file named **TokenValidationResult.cs** in the **./GraphTutorial/Authentication** folder, and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/TokenValidationResult.cs" id="TokenValidationResultSnippet":::

1. <span data-ttu-id="8201e-140">Erstellen Sie eine neue Datei namens **"TokenValidation.cs"** im Ordner **"./GraphTutorial/Authentication",** und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="8201e-140">Create a new file named **TokenValidation.cs** in the **./GraphTutorial/Authentication** folder, and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/TokenValidation.cs" id="TokenValidationSnippet":::

<span data-ttu-id="8201e-141">Überlegen Sie, was dieser Code bewirkt.</span><span class="sxs-lookup"><span data-stu-id="8201e-141">Consider what this code does.</span></span>

- <span data-ttu-id="8201e-142">Es wird sichergestellt, dass ein Bearertoken im Header vorhanden `Authorization` ist.</span><span class="sxs-lookup"><span data-stu-id="8201e-142">It ensure there is a bearer token in the `Authorization` header.</span></span>
- <span data-ttu-id="8201e-143">Es überprüft die Signatur und den Aussteller aus der veröffentlichten OpenID-Konfiguration von Azure.</span><span class="sxs-lookup"><span data-stu-id="8201e-143">It verifies the signature and issuer from Azure's published OpenID configuration.</span></span>
- <span data-ttu-id="8201e-144">Es wird überprüft, ob die Zielgruppe ( `aud` Anspruch) mit der Anwendungs-ID der Azure-Funktion übereinstimmt.</span><span class="sxs-lookup"><span data-stu-id="8201e-144">It verifies that the audience (`aud` claim) matches the Azure Function's application ID.</span></span>
- <span data-ttu-id="8201e-145">Es analysiert das Token und generiert eine MSAL-Konto-ID, die benötigt wird, um die Vorteile der Tokenzwischenspeicherung zu nutzen.</span><span class="sxs-lookup"><span data-stu-id="8201e-145">It parses the token and generates an MSAL account ID, which will be needed to take advantage of token caching.</span></span>

### <a name="create-an-on-behalf-of-authentication-provider"></a><span data-ttu-id="8201e-146">Erstellen eines Authentifizierungsanbieters im Auftrag von</span><span class="sxs-lookup"><span data-stu-id="8201e-146">Create an on-behalf-of authentication provider</span></span>

1. <span data-ttu-id="8201e-147">Erstellen Sie eine neue Datei im **Authentifizierungsverzeichnis** **"OnBehalfOfAuthProvider.cs",** und fügen Sie der Datei den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="8201e-147">Create a new file in the **Authentication** directory named **OnBehalfOfAuthProvider.cs** and add the following code to that file.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/OnBehalfOfAuthProvider.cs" id="AuthProviderSnippet":::

<span data-ttu-id="8201e-148">Nehmen Sie sich einen Moment Zeit, um zu überlegen, was der Code in **"OnBehalfOfAuthProvider.cs"** bewirkt.</span><span class="sxs-lookup"><span data-stu-id="8201e-148">Take a moment to consider what the code in **OnBehalfOfAuthProvider.cs** does.</span></span>

- <span data-ttu-id="8201e-149">In der `GetAccessToken` Funktion wird zunächst versucht, ein Benutzertoken mithilfe von `AcquireTokenSilent` .</span><span class="sxs-lookup"><span data-stu-id="8201e-149">In the `GetAccessToken` function, it first attempts to get a user token from the token cache using `AcquireTokenSilent`.</span></span> <span data-ttu-id="8201e-150">Wenn dies fehlschlägt, wird das von der Test-App an die Azure-Funktion gesendete Bearertoken verwendet, um eine Benutzer assertion zu generieren.</span><span class="sxs-lookup"><span data-stu-id="8201e-150">If this fails, it uses the bearer token sent by the test app to the Azure Function to generate a user assertion.</span></span> <span data-ttu-id="8201e-151">Anschließend wird diese Benutzerassertion verwendet, um ein Graph-kompatibles Token mithilfe von `AcquireTokenOnBehalfOf` abzurufen.</span><span class="sxs-lookup"><span data-stu-id="8201e-151">It then uses that user assertion to get a Graph-compatible token using `AcquireTokenOnBehalfOf`.</span></span>
- <span data-ttu-id="8201e-152">Sie implementiert die `Microsoft.Graph.IAuthenticationProvider` Schnittstelle, sodass diese Klasse im Konstruktor der zum Authentifizieren ausgehender Anforderungen übergeben werden `GraphServiceClient` kann.</span><span class="sxs-lookup"><span data-stu-id="8201e-152">It implements the `Microsoft.Graph.IAuthenticationProvider` interface, allowing this class to be passed in the constructor of the `GraphServiceClient` to authenticate outgoing requests.</span></span>

### <a name="implement-a-graph-client-service"></a><span data-ttu-id="8201e-153">Implementieren eines Graph-Clientdiensts</span><span class="sxs-lookup"><span data-stu-id="8201e-153">Implement a Graph client service</span></span>

<span data-ttu-id="8201e-154">In diesem Abschnitt implementieren Sie einen Dienst, der für die [Abhängigkeitsinjektion](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection)registriert werden kann.</span><span class="sxs-lookup"><span data-stu-id="8201e-154">In this section you'll implement a service that can be registered for [dependency injection](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection).</span></span> <span data-ttu-id="8201e-155">Der Dienst wird verwendet, um einen authentifizierten Graph-Client abzurufen.</span><span class="sxs-lookup"><span data-stu-id="8201e-155">The service will be used to get an authenticated Graph client.</span></span>

1. <span data-ttu-id="8201e-156">Erstellen Sie ein neues Verzeichnis im **GraphTutorial-Verzeichnis** namens **"Dienste".**</span><span class="sxs-lookup"><span data-stu-id="8201e-156">Create a new directory in the **GraphTutorial** directory named **Services**.</span></span>

1. <span data-ttu-id="8201e-157">Erstellen Sie eine neue Datei im **Verzeichnis "Services"** mit dem Namen **"IGraphClientService.cs",** und fügen Sie der Datei den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="8201e-157">Create a new file in the **Services** directory named **IGraphClientService.cs** and add the following code to that file.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Services/IGraphClientService.cs" id="IGraphClientServiceSnippet":::

1. <span data-ttu-id="8201e-158">Erstellen Sie eine neue Datei im **Verzeichnis "Services"** mit dem Namen **"GraphClientService.cs",** und fügen Sie der Datei den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="8201e-158">Create a new file in the **Services** directory named **GraphClientService.cs** and add the following code to that file.</span></span>

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

1. <span data-ttu-id="8201e-159">Fügen Sie der Klasse die folgenden Eigenschaften `GraphClientService` hinzu.</span><span class="sxs-lookup"><span data-stu-id="8201e-159">Add the following properties to the `GraphClientService` class.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="UserGraphClientMembers":::

1. <span data-ttu-id="8201e-160">Fügen Sie der Klasse die folgenden Funktionen `GraphClientService` hinzu.</span><span class="sxs-lookup"><span data-stu-id="8201e-160">Add the following functions to the `GraphClientService` class.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="UserGraphClientFunctions":::

1. <span data-ttu-id="8201e-161">Fügen Sie eine Platzhalterimplementierung für die `GetAppGraphClient` Funktion hinzu.</span><span class="sxs-lookup"><span data-stu-id="8201e-161">Add a placeholder implementation for the `GetAppGraphClient` function.</span></span> <span data-ttu-id="8201e-162">Sie werden dies in späteren Abschnitten implementieren.</span><span class="sxs-lookup"><span data-stu-id="8201e-162">You will implement that in later sections.</span></span>

    ```csharp
    public GraphServiceClient GetAppGraphClient()
    {
        throw new System.NotImplementedException();
    }
    ```

    <span data-ttu-id="8201e-163">Die `GetUserGraphClient` Funktion übernimmt die Ergebnisse der Tokenüberprüfung und erstellt eine `GraphServiceClient` authentifizierte für den Benutzer.</span><span class="sxs-lookup"><span data-stu-id="8201e-163">The `GetUserGraphClient` function takes the results of token validation and builds an authenticated `GraphServiceClient` for the user.</span></span>

1. <span data-ttu-id="8201e-164">Öffnen Sie **./GraphTutorial/Program.cs,** und ersetzen Sie den Inhalt durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="8201e-164">Open **./GraphTutorial/Program.cs** and replace its contents with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Program.cs" id="ProgramSnippet" highlight="15-23":::

    <span data-ttu-id="8201e-165">Dieser Code fügt der Konfiguration geheime Benutzerschlüssel hinzu und aktiviert die [Abhängigkeitsinjektion](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection) in Ihren Azure-Funktionen, wodurch der Dienst verfügbar `GraphClientService` wird.</span><span class="sxs-lookup"><span data-stu-id="8201e-165">This code will add user secrets to the configuration, and enable [dependency injection](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection) in your Azure Functions, exposing the `GraphClientService` service.</span></span>

### <a name="implement-getmynewestmessage-function"></a><span data-ttu-id="8201e-166">Implementieren der GetMyNewestMessage-Funktion</span><span class="sxs-lookup"><span data-stu-id="8201e-166">Implement GetMyNewestMessage function</span></span>

1. <span data-ttu-id="8201e-167">Öffnen Sie **./GraphTutorial/GetMyNewestMessage.cs,** und ersetzen Sie den gesamten Inhalt durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="8201e-167">Open **./GraphTutorial/GetMyNewestMessage.cs** and replace its entire contents with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GetMyNewestMessage.cs" id="GetMyNewestMessageSnippet":::

#### <a name="review-the-code-in-getmynewestmessagecs"></a><span data-ttu-id="8201e-168">Überprüfen des Codes in "GetMyNewestMessage.cs"</span><span class="sxs-lookup"><span data-stu-id="8201e-168">Review the code in GetMyNewestMessage.cs</span></span>

<span data-ttu-id="8201e-169">Nehmen Sie sich einen Moment Zeit, um zu überlegen, was der Code in **"GetMyNewestMessage.cs"** bewirkt.</span><span class="sxs-lookup"><span data-stu-id="8201e-169">Take a moment to consider what the code in **GetMyNewestMessage.cs** does.</span></span>

- <span data-ttu-id="8201e-170">Im Konstruktor werden die objekte gespeichert, die über die `IConfiguration` `IGraphClientService` Abhängigkeitsinjektion übergeben werden.</span><span class="sxs-lookup"><span data-stu-id="8201e-170">In the constructor, it saves the `IConfiguration` and `IGraphClientService` objects passed in via dependency injection.</span></span>
- <span data-ttu-id="8201e-171">In der `Run` Funktion wird Folgendes ausgeführt:</span><span class="sxs-lookup"><span data-stu-id="8201e-171">In the `Run` function, it does the following:</span></span>
  - <span data-ttu-id="8201e-172">Überprüft, ob die erforderlichen Konfigurationswerte im Objekt vorhanden `IConfiguration` sind.</span><span class="sxs-lookup"><span data-stu-id="8201e-172">Validates the required configuration values are present in the `IConfiguration` object.</span></span>
  - <span data-ttu-id="8201e-173">Überprüft das Bearertoken und gibt einen `401` Statuscode zurück, wenn das Token ungültig ist.</span><span class="sxs-lookup"><span data-stu-id="8201e-173">Validates the bearer token and returns a `401` status code if the token is invalid.</span></span>
  - <span data-ttu-id="8201e-174">Ruft einen Graph Client für `GraphClientService` den Benutzer ab, der diese Anforderung ausgeführt hat.</span><span class="sxs-lookup"><span data-stu-id="8201e-174">Gets a Graph client from the `GraphClientService` for the user that made this request.</span></span>
  - <span data-ttu-id="8201e-175">Verwendet das Microsoft Graph SDK, um die neueste Nachricht aus dem Posteingang des Benutzers abzurufen, und gibt sie als JSON-Text in der Antwort zurück.</span><span class="sxs-lookup"><span data-stu-id="8201e-175">Uses the Microsoft Graph SDK to get the newest message from the user's inbox and returns it as a JSON body in the response.</span></span>

## <a name="call-the-azure-function-from-the-test-app"></a><span data-ttu-id="8201e-176">Aufrufen der Azure-Funktion aus der Test-App</span><span class="sxs-lookup"><span data-stu-id="8201e-176">Call the Azure Function from the test app</span></span>

1. <span data-ttu-id="8201e-177">Öffnen **Sieauth.js,** und fügen Sie die folgende Funktion hinzu, um ein Zugriffstoken abzurufen.</span><span class="sxs-lookup"><span data-stu-id="8201e-177">Open **auth.js** and add the following function to get an access token.</span></span>

    :::code language="javascript" source="../demo/TestClient/auth.js" id="getTokenSnippet":::

    <span data-ttu-id="8201e-178">Überlegen Sie, was dieser Code bewirkt.</span><span class="sxs-lookup"><span data-stu-id="8201e-178">Consider what this code does.</span></span>

    - <span data-ttu-id="8201e-179">Zunächst wird versucht, ein Zugriffstoken im Hintergrund ohne Benutzerinteraktion abzurufen.</span><span class="sxs-lookup"><span data-stu-id="8201e-179">It first attempts to get an access token silently, without user interaction.</span></span> <span data-ttu-id="8201e-180">Da der Benutzer bereits angemeldet sein sollte, sollte MSAL Token für den Benutzer im Cache haben.</span><span class="sxs-lookup"><span data-stu-id="8201e-180">Since the user should already be signed in, MSAL should have tokens for the user in its cache.</span></span>
    - <span data-ttu-id="8201e-181">Wenn dies mit einem Fehler fehlschlägt, der angibt, dass der Benutzer interagieren muss, wird versucht, ein Token interaktiv abzurufen.</span><span class="sxs-lookup"><span data-stu-id="8201e-181">If that fails with an error that indicates the user needs to interact, it attempts to get a token interactively.</span></span>

    > [!TIP]
    > <span data-ttu-id="8201e-182">Sie können das Zugriffstoken analysieren [https://jwt.ms](https://jwt.ms) und bestätigen, dass der `aud` Anspruch die App-ID für die Azure-Funktion ist und dass der `scp` Anspruch den Berechtigungsbereich der Azure-Funktion enthält, nicht Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="8201e-182">You can parse the access token at [https://jwt.ms](https://jwt.ms) and confirm that the `aud` claim is the app ID for the Azure Function, and that the `scp` claim contains the Azure Function's permission scope, not Microsoft Graph.</span></span>

1. <span data-ttu-id="8201e-183">Erstellen Sie eine neue Datei im **TestClient-Verzeichnis** mit dem Namen **azurefunctions.js,** und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="8201e-183">Create a new file in the **TestClient** directory named **azurefunctions.js** and add the following code.</span></span>

    :::code language="javascript" source="../demo/TestClient/azurefunctions.js" id="getLatestMessageSnippet":::

1. <span data-ttu-id="8201e-184">Ändern Sie das aktuelle Verzeichnis in Ihrer CLI in das Verzeichnis **./GraphTutorial,** und führen Sie den folgenden Befehl aus, um die Azure-Funktion lokal zu starten.</span><span class="sxs-lookup"><span data-stu-id="8201e-184">Change the current directory in your CLI to the **./GraphTutorial** directory and run the following command to start the Azure Function locally.</span></span>

    ```Shell
    func start
    ```

1. <span data-ttu-id="8201e-185">Wenn die SPA nicht bereits bedient wird, öffnen Sie ein zweites CLI-Fenster, und ändern Sie das aktuelle Verzeichnis in das Verzeichnis **./TestClient.**</span><span class="sxs-lookup"><span data-stu-id="8201e-185">If not already serving the SPA, open a second CLI window and change the current directory to the **./TestClient** directory.</span></span> <span data-ttu-id="8201e-186">Führen Sie den folgenden Befehl aus, um die Testanwendung auszuführen.</span><span class="sxs-lookup"><span data-stu-id="8201e-186">Run the following command to run the test application.</span></span>

    ```Shell
    dotnet serve -h "Cache-Control: no-cache, no-store, must-revalidate"
    ```

1. <span data-ttu-id="8201e-187">Öffnen Sie Ihren Browser und navigieren Sie zu `http://localhost:8080`.</span><span class="sxs-lookup"><span data-stu-id="8201e-187">Open your browser and navigate to `http://localhost:8080`.</span></span> <span data-ttu-id="8201e-188">Melden Sie sich an, und wählen Sie das Navigationselement **"Neueste Nachricht"** aus.</span><span class="sxs-lookup"><span data-stu-id="8201e-188">Sign in and select the **Latest Message** navigation item.</span></span> <span data-ttu-id="8201e-189">Die App zeigt Informationen zur neuesten Nachricht im Posteingang des Benutzers an.</span><span class="sxs-lookup"><span data-stu-id="8201e-189">The app displays information about the newest message in the user's inbox.</span></span>

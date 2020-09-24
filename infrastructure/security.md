# Security

The application is secured with the open standard for API authorization OAuth 2.0.
In the enterprise environment, Microsoft's cloud-based identity and access management service AzureAD can be used to provide this security.
The following describes how to set up a single API service and a client in AzureAD so that both are authorized to communicate with each other.

![](assets/azure_ad_config.svg)

## Azure AD: API Service configuration
- Open the [Azure AD portal](https://portal.azure.com/)
- Login with your Microsoft account
- Open Azure Active Directory in the portal menu on the left handside
1. Open the submenu App registration
- Search for the service to configure, e.g. "COLID AppData Service"
- If no service is shown in the list, register a new one
    * Select a name, e.g. "COLID AppData Service"

### Configuration Steps
Step by step open the left configuration pages under the "Manage" section

#### Expose an API
Define to expose the API to all services available in Azure AD. Other services (and the one you are just about to configure) can access this API later.

Steps:
1. Click on "Add a scope"
1. A Panel opens
    * Name the scope to something meaningful, e.g. "UserData.Read"
    * Who can consent? Select "Admins and users"
    * Enter Admin consent display name, e.g. "Read and write user data"
    * Enter Admin consent description, e.g. "Allows the app to read and write the signed-in user data"
    * Click on "Add scope"

The following steps are relevant to allow access to other client applications:

1. Click on "Add a client application"
1. Copy the GUID of Application ID URI at the top of the screen (or any other GUID of an application  to allow access)
1. Enter the GUID on the right opening panel to "Client ID"
1. Select the chechbox of the created scope
1. Click on "Add application"

#### Authentication
Add an URI as the accepted destination after successfully authenticating user

Steps:
1. In section "Web" add a Redirect URI
    * Click on "Add URI"
    * Enter the redirect URI of Swagger, e.g. "https://localhost:51810/swagger/oauth2-redirect.html"

#### Certificates & secrets
Add a secret password that the application uses to prove its identity when requesting a token as a Client

Steps:
1. Add a new Client secret
    * Click on "+ New client secret"
    * Enter a valid description and and expiration date (mostly "Never")
    * Click "Add"
1. Copy the generated Client secret from the Value column of the table
1. Save it in the password manager
1. The Value will be hidden after refreshing the page
1. This secret password should be used as the Swagger Client Secret

#### API permissions
Steps:
1. Click on "Add a permission"
1. In the panel on the right side, click on "APIs my organization uses"
1. Search for the service you are currently visiting, e.g. "COLID AppData Service"
1. Click on the name of the service
1. Click on "Delegated permissions"
1. Select the permissions in the table created in step "Expose and API"
1. Click on "Add permissions" at the bottom
1. The service has access to its own API now
1. Repeat this two last steps for other clients

#### Manifest
Allow Swagger to get an oauth2 token for the service API

Steps:
1. Open the Manifest, search for the following fields:
    * oauth2AllowIdTokenImplicitFlow
    * oauth2AllowImplicitFlow
1. Change the values to "true"
1. Click on "Save" at the top

<!--## CASA Configuration: Creating a new application

To create a new application in general, the first step is to create it in CASA.

1. Open [CASA](https://bayergroup.sharepoint.com/sites/002392/Lists/Azure%20AD%20Application%20Register/AllItems.aspx)
2. Click on "+ New" in the header
1. A new dialog opens
    1. Enter a title of the client, e.g. "COLID Wikipedia Crawler Client"
        * *Hint: For testing applications add a suffix like "QA" to the title*
    1. Select the Portfolio System, in general "BEAT"
    1. Enter the portfolio ID of your application ecosystem your client references to
        * COLID has the Portfolio ID 3001413
    1. Enter your secondary CWID
        * This will be the application owner in Azure AD
        * The owner can add more application owners in Azure AD later
    1. Save the application
    1. After a moment the Azure AD Application ID will be generated
1. In the table select the new application. The AAD Application ID should be visible on the right handside.
1. Since the application has been created with the secondary CWID in CASA, login with your secondary CWID into Azure AD to see the application with owner privileges-->

## Azure AD: Client application configuration

1. Open the [Azure Portal](https://portal.azure.com/) as the user with owner privileges of the application
1. Go to the Azure service "Azure Active Directory" in the portal menu on the left handside
1. Open the submenu App registration
1. Search for the client to configure, e.g. "COLID Wikipedia Crawler Client"
    * *Hint:* If you log in as the application owner, the client should be visible in the "Owned applications" tab
- If no service is shown in the list, register a new one
    * Select a name, e.g. "COLID Wikipedia Crawler Client"
1. Click on the client you want to configure

### Application (client) ID
1. Open the Overview section in the left navigation menu
1. In the center the Application (client) ID and the global Directory (tenant) ID is visible
1. Copy the Application (client) ID and the global Directory (tenant) ID
1. We needs this GUIDs later to retrieve an oauth2 token from Azure.

### Configuration Steps

The following configuration steps are visible in the left navigation menu.

#### Certificates & secrets
Add a secret password that the application uses to prove its identity when requesting a token as a Client

Steps:
1. Add a new Client secret
    * Click on "+ New client secret"
    * Enter a valid description and and expiration date (mostly "Never")
    * Click "Add"
1. Copy the generated Client secret from the Value column of the table
1. Save it in the password manager
1. The value will be hidden after refreshing the page
1. This secret password should be used as the client secret

#### API permissions
Steps:
1. Click on "Add a permission"
1. In the panel on the right side, click on "APIs my organization uses"
1. Search for the service you want to have access to, e.g. "COLID Registration Service"
1. Click on the name of the service
1. In the top of the panel, the resource ID of the API service is visible beneath the name, e.g. `api://89841b71-b3c2-4ee6-b893-778b54ede85e`. Copy the resource ID (here `89841b71-b3c2-4ee6-b893-778b54ede85e`) for later usage to retrieve the oauth2 token from Azure
1. Click on "Application permissions"
1. Select the permissions of the API service that are required for the client, e.g. for API 2 API usage "PID.Apps.ReadWrite"
1. Click on "Add permissions" at the bottom
1. Inform the owner of the API service to grant the newly created permission
1. Wait for the API service owner to giving the grant

## Azure AD: Granting access via Microsoft Graph API as an API service Application Owner

The clients that need access to the API service must receive a grant from the owner of the API service. This can be done through Microsoft's Graph API by the API service owner. The following data is required for this:
1. Object ID Enterprise App API service
    1. Open the [Azure Portal](https://portal.azure.com/) as the user with owner privileges of the application
    1. Go to the Azure service "Azure Active Directory" in the navigation menu on the left handside
    1. Open the submenu App registration
    1. Search for the API service, e.g. "COLID Registration Service"
        * *Hint:* If you log in as the application owner, the client should be visible in the "Owned applications" tab
    1. Open the Overview section in the left navigation menu
    1. On the right side under "Managed appliaction in local directory" click on the application name
    1. In the new window, copy  the "Object ID" in the "Properties" section
1. Object ID of Enterprise App Client Application
    1. Open the [Azure Portal](https://portal.azure.com/) as the user with owner privileges of the application
    1. Go to the Azure service "Azure Active Directory" in the navigation menu on the left handside
    1. Open the submenu App registration
    1. Search for the client, e.g. "COLID Wikipedia Crawler Client"
        * *Hint:* If you log in as the application owner, the client should be visible in the "Owned applications" tab
    1. Open the Overview section in the left navigation menu
    1. On the right side under "Managed appliaction in local directory" click on the application name
    1. In the new window, copy  the "Object ID" in the "Properties" section
1. App Role ID from API service manifest
    1. Open the [Azure Portal](https://portal.azure.com/) as the user with owner privileges of the application
    1. Go to the Azure service "Azure Active Directory" in the navigation menu on the left handside
    1. Open the submenu App registration
    1. Search for the API service, e.g. "COLID Registration Service"
        * *Hint:* If you log in as the application owner, the client should be visible in the "Owned applications" tab
    1. Open the Manifest section in the left navigation menu
    1. Search in the JSON file in the "appRoles" field for the API service permission requested from the client, e.g. "PID.Apps.ReadWrite"
    1. Copy the "id" field of the application role with the searched value (e.g. "PID.Apps.ReadWrite")
1. Open Microsoft's [Graph Explorer](https://developer.microsoft.com/en-us/graph/graph-explorer/preview) as the Preview version
1. Login as the API service owner
1. Select to send a "POST request to the "beta" API and enter the following request with the value retrieved above
    ```bash
    POST https://graph.microsoft.com/beta/servicePrincipals/<objectid_enterprise_app_api_service>/appRoleAssignedTo
    {
      "principalId":"<objectid_enterprise_app_client_application>",     // 02559dc6-1919-4c74-b00c-55fb87a89988
      "appRoleId":"<app_role_id_from_api_service_manifest>", // 93bb06fd-f923-423e-9d80-800c7a9ea3e3
      "resourceId":"<objectid_enterprise_app_api_service>",  // 3927dd86-1909-4f06-a8ac-a1a367c26f76
      "principalType":"ServicePrincipal"
    }
    ```

## Testing the permissions by retrieving an oauth2 token from Azure AD

After following all the steps above, the client application should now have permission to the selected API service. In this section you need the following data:
1. Global Directory (tenant) ID
2. Application (client) ID of your newly created client
3. Private client secret created in chapter "Azure AD: Client application configuration"
4. Application (client) ID of API service you want to access (a.k.a. resource ID),
   <br>copied in step Azure AD: Client application configuration > API permissions > Step 5
5. Open your beloved cURL client and send the following command. Fill the values in the placeholders.
    ```bash
    curl --location --request POST 'https://login.microsoftonline.com/<global_directory_tenant_id>/oauth2/token' \
    --header 'Content-Type: application/x-www-form-urlencoded' \
    --data-urlencode 'grant_type=client_credentials' \
    --data-urlencode 'client_id=<client_id>' \
    --data-urlencode 'resource=<resource_id>' \
    --data-urlencode 'client_secret=<client_secret>'
    ```

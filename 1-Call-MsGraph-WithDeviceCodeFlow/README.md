---
topic: sample
languages:
  - python
  - azurepowershell
products:
  - azure-active-directory
  - office-ms-graph
description: "Python device code flow using MSAL Python to get an access token and call Microsoft Graph."
---

# A simple Python device code flow application calling Microsoft Graph

## About this sample

### Overview

This sample application shows how to use the [Microsoft identity platform endpoint](http://aka.ms/aadv2) to access the data of Microsoft customers.  The device code flow can be used to authenticate a user and then call to a web api, in this case, the [Microsoft Graph](https://graph.microsoft.io).

The app can run as a Python Console Application. It gets the list of users in an Azure AD tenant by using `Microsoft Authentication Library (MSAL) for Python` to acquire a token.

## Scenario

The application obtains tokens through a two steps process especially designed for devices and operating systems that cannot display any UX. Examples of such applications are applications running on iOT, or Command-Line tools (CLI). The idea is that:

![Topology](./ReadmeFiles/Topology.png)

1. Whenever a user authentication is required, the command-line app provides a code and asks the user to use another device (such as an internet-connected smartphone) to navigate to https://microsoft.com/devicelogin, where the user will be prompted to enter the code. That done, the web page will lead the user through a normal authentication experience, including consent prompts and multi factor authentication if necessary.

![Enter code in browser](./ReadmeFiles/deviceCodeFlow.png)

1. Upon successful authentication, the command-line app will receive the required tokens through a back channel and will use it to perform the web API calls it needs. In this case, the sample displays information about the user who signed-in and their manager.

> ### Daemon applications can use two forms of secrets to authenticate themselves with Azure AD:
>
> - **application secrets** (this Readme.md).
> - **certificates**, in the [../2-Call-MsGraph-WithCertificate](../2-Call-MsGraph-WithCertificate) folder

## How to run this sample

To run this sample, you'll need:

> - [Python 2.7+](https://www.python.org/downloads/release/python-2713/) or [Python 3+](https://www.python.org/downloads/release/python-364/)
> - An Azure Active Directory (Azure AD) tenant. For more information on how to get an Azure AD tenant, see [how to get an Azure AD tenant.](https://docs.microsoft.com/azure/active-directory/develop/quickstart-create-new-tenant)

### Step 1:  Clone or download this repository

From your shell or command line:

```Shell
git clone https://github.com/Azure-Samples/ms-identity-python-devicecodeflow.git
```

Go to the `"1-Call-MsGraph-WithDeviceCodeFlow"` folder

```Shell
cd "1-Call-MsGraph-WithDeviceCodeFlow"
```

or download and exact the repository .zip file.

> Given that the name of the sample is pretty long, you might want to clone it in a folder close to the root of your hard drive, to avoid file size limitations on Windows.

### Step 2:  (optional) Register the sample with your Azure Active Directory tenant

Without registration, your app is multitenant.  Anybody can run the sample against that app entry.  There is one project in this sample. To register it, you can:

- either follow the steps [Step 2: Register the sample with your Azure Active Directory tenant](#step-2-register-the-sample-with-your-azure-active-directory-tenant) and [Step 3:  Configure the sample to use your Azure AD tenant](#choose-the-azure-ad-tenant-where-you-want-to-create-your-applications)
- or use PowerShell scripts that:
  - **automatically** creates the Azure AD applications and related objects (passwords, permissions, dependencies) for you

If you want to use this automation:
1. On Windows run PowerShell and navigate to the root of the cloned directory
1. In PowerShell run:
   ```PowerShell
   Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope Process -Force
   ```
1. Run the script to create your Azure AD application and configure the code of the sample application accordingly. 
   ```PowerShell
   .\AppCreationScripts\Configure.ps1
   ```
   > Other ways of running the scripts are described in [App Creation Scripts](./AppCreationScripts/AppCreationScripts.md)

1. Run the sample

   You'll need to install the dependencies using pip as follows:
  
   ```Shell
    pip install -r requirements.txt
    ```

    Run `device_flow_sample.py` with the parameters for the app:

   ```Shell
   python device_flow_sample.py parameters.json
   ```

If you don't want to use this automation, follow the steps below

#### Choose the Azure AD tenant where you want to create your applications

As a first step you'll need to:

1. Sign in to the [Azure portal](https://portal.azure.com) using either a work or school account or a personal Microsoft account.
1. If your account is present in more than one Azure AD tenant, select `Directory + Subscription` at the top right corner in the menu on top of the page, and switch your portal session to the desired Azure AD tenant.
1. In the left-hand navigation pane, select the **Azure Active Directory** service, and then select **App registrations**.

#### Register the client app (active-directory-dotnet-deviceprofile)

1. In **App registrations** page, select **New registration**.
1. When the **Register an application page** appears, enter your application's registration information:
   - In the **Name** section, enter a meaningful application name that will be displayed to users of the app, for example `device-code-sample`.
   - In the **Supported account types** section, select **Accounts in any organizational directory**.
1. Select **Register** to create the application.
1. On the app **Overview** page, find the **Application (client) ID** value and record it for later. You'll need it to configure your app.
1. In the list of pages for the app, select **Manifest**, and:
   - In the manifest editor, set the ``allowPublicClient`` property to **true** 
   - Select **Save** in the bar above the manifest editor.
1. In the list of pages for the app, select **API permissions**
   - Click the **Add a permission** button and then,
   - Ensure that the **Microsoft APIs** tab is selected
   - In the *Commonly used Microsoft APIs* section, click on **Microsoft Graph**
   - In the **Delegated permissions** section, ensure that the right permissions are checked: **User.Read**, **User.ReadBasic.All**. Use the search box if necessary.
   - Select the **Add permissions** button

1. At this stage permissions are assigned correctly but the client app does not allow interaction. 
   Therefore no consent can be presented via a UI and accepted to use the service app. 
   Click the **Grant/revoke admin consent for {tenant}** button, and then select **Yes** when you are asked if you want to grant consent for the
   requested permissions for all account in the tenant.
   You need to be an Azure AD tenant admin to do this.

### Step 3:  Configure the sample to use your Azure AD tenant

In the steps below, "ClientID" is the same as "Application ID" or "AppId".

Open the parameters.json file

#### Configure the client project

> Note: if you used the setup scripts, the changes below will have been applied for you

1. Open the `parameters.json` file
1. Find the string key `organizations` in the `authority` variable and replace the existing value with your Azure AD tenant name.
1. Find the string key `your_client_id` and replace the existing value with the application ID (clientId) of the `device-code-sample` application copied from the Azure portal.
1. (Optional) Find the line where `Tenant` is set and replace the existing value with your tenant ID.

### Step 4: Run the sample

You'll need to install the dependencies using pip as follows:
  
```Shell
pip install -r requirements.txt
```

Start the application, it will display some Json string containing the users in the tenant.

```Shell
python confidential_client_secret_sample.py parameters.json
```

## About the code

The relevant code for this sample is in the `device_code_sample.py` file. The steps are:

1. Create the MSAL confidential client application.

    Important note: even if we are building a console application, it is a daemon, and therefore a confidential client application, as it does not
    access Web APIs on behalf of a user, but on its own application behalf.

    ```Python
    app = msal.PublicClientApplication(
      config["client_id"], authority=config["authority"],
    )
    ```

2. The scopes are defined in the parameters.json file.

   In the default parameters.json file you have:

    ```JSon
    "scope": ["User.Read"]
    ```

3. Acquire the token

    ```Python
    result = None
    # Firstly, looks up a token from cache
    # If that fails, attempt the device code flow
    accounts = app.get_accounts()
    # Skipping account iteration and cache lookup
    flow = app.initiate_device_flow(scopes=config["scope"])
    # Skipping error condition
    result = app.acquire_token_by_device_flow(flow)
    ```

4. Call the API

    In that case calling "https://graph.microsoft.com/v1.0/users" with the access token as a bearer token.

    ```Python
    if "access_token" in result:
        # Calling graph using the access token
        graph_data = requests.get(  # Use token to call downstream service
        config["endpoint"],
        headers={'Authorization': 'Bearer ' + result['access_token']}, ).json()
    print("Users from graph: " + str(graph_data))
    else:
        print(result.get("error"))
        print(result.get("error_description"))
        print(result.get("correlation_id"))  # You may need this when reporting a bug
    ```

## Troubleshooting


## Community Help and Support

Use [Stack Overflow](http://stackoverflow.com/questions/tagged/msal) to get support from the community.
Ask your questions on Stack Overflow first and browse existing issues to see if someone has asked your question before.
Make sure that your questions or comments are tagged with [`msal` `python`].

If you find a bug in the sample, please raise the issue on [GitHub Issues](../../issues).

If you find a bug in Msal Python, please raise the issue on [MSAL Python GitHub Issues](https://github.com/AzureAD/microsoft-authentication-library-for-python/issues).

To provide a recommendation, visit the following [User Voice page](https://feedback.azure.com/forums/169401-azure-active-directory).

## Contributing

If you'd like to contribute to this sample, see [CONTRIBUTING.MD](/CONTRIBUTING.md).

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/). For more information, see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

## More information

For more information, see MSAL Python's conceptual documentation:

- [Quickstart: Register an application with the Microsoft identity platform](https://docs.microsoft.com/azure/active-directory/develop/quickstart-register-app)
- [Quickstart: Configure a client application to access web APIs](https://docs.microsoft.com/azure/active-directory/develop/quickstart-configure-app-access-web-apis)
- [Device Code Flow app scenario](https://docs.microsoft.com/en-us/azure/active-directory/develop/msal-authentication-flows#device-code)

For more information about the underlying protocol:

- [Microsoft identity platform and the OAuth 2.0 device code flow](https://docs.microsoft.com/en-us/azure/active-directory/develop/v2-oauth2-device-code)
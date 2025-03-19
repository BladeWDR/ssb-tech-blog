+++
title = 'Connecting rclone to SharePoint Online'
date = 2025-03-19T15:18:48-04:00
draft = false
+++

# Connecting rclone to SharePoint Online

I found the [official instructions](https://rclone.org/onedrive/) for this extremely difficult to follow, so here's what worked for me, with an Office 365 Business tenant.

First you need to create a custom client id. The default client ID will likely end up getting throttled, and the token will expire after an hour or so, causing your operations to stall.

## App Registration

- Open [this link](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade) and log in with an administrator account that has privileges to create a new app registration for your tenant.
- Click "New registration"
- Give your app a name, I just called mine "rclone".
- For Supported account types, you want the "Accounts in this organizational directory only" option.
- create a redirect URI of type Web. Type (don't copy and paste) this into the URI field. `http://localhost:53682/`. Don't miss the trailing slash.
- Copy and keep the Application (client) ID under the app name for later use.
- Under manage select Certificates & secrets, click New client secret. Enter a description (can be anything) and set Expires to however long you'd like. Copy and keep that secret Value for later use (you won't be able to see this value afterwards).
- Under Manage select API Permissions. Click "add a permission" and select Microsoft Graph. You want "Application Permissions", not "Delegated permissions".
- Give the API key the following permissions:
    - Files.ReadWrite
    - Files.Read.All
    - Files.ReadWrite.All
    - User.Read
    - Sites.Read.All
- Click the button for "Grant Admin consent for <org name>"

## Create rclone remote

- `rclone config`
- Give the remote a name
- Select `35` for OneDrive.
- Enter your client ID and client secret when prompted.
- **When you get to the step where it asks you to authenticate in a browser, it will likely fail no matter what you do. This is okay.**
- Open your `rclone.conf` in a text editor. On Linux it's by default in `$HOME/.config/rclone`

    ```vim $HOME/rclone/rclone.conf```

- In the app registration you created earlier you can find the tenant ID in the overview. You can also find it in the Entra ID admin panel. Save that value.
- Add the following lines to your `rclone.conf`

    ```bash
    auth_url = https://login.microsoftonline.com/YOUR_TENANT_ID/oauth2/v2.0/authorize
    token_url = https://login.microsoftonline.com/YOUR_TENANT_ID/oauth2/v2.0/token
    tenant = YOUR_TENANT_ID
    client_credentials = true
    ``` 

## Getting the DriveId

Next, get the DriveID of the SharePoint site that you're trying to create a remote for.

We'll be using the Microsoft Graph PowerShell module for this. You'll want to connect to Graph with a user that has admin privileges.

The "site-name" mentioned here is the same one at the end of the SharePoint link, like  

https://contoso.sharepoint.com/sites/**Accounting**

So in this case, Accounting would be the site name.

```ps1
Connect-MgGraph -Scopes "Sites.Read.All"
$site = Get-MgSite -Search "Site-name"
$drive = Get-MgSiteDrive -SiteId $site.Id
$drive | Format-List
```

The last command here will print quite a bit of information - there may be more than one document library associated with a single SharePoint site, especially if you've enabled Teams for that site.

Add this as a line to your configured Sharepoint remote.

```bash
drive_id = DRIVE_ID_HERE
drive_type = documentLibrary
```

## Attempt reconnection

Run the following command:

```bash
rclone config reconnect <remotename>:
```

At this point you should be able to follow the steps and get rclone connected.

You can verify by running:

```bash
rclone ls <remotename>:
```

Don't get me wrong, I'm sure there are some unnecessary steps here, but this is what I did, and it works.

It will allow you to connect rclone to SharePoint with a custom client ID and avoid it timing out after an hour or so.

I used this to migrate 3TB of data from a SharePoint site to a local NAS.

## Additional tips

If you still run into issues with rate limiting, you can look into the `--tpslimit`, `--transfers` or `--checkers` options to limit the number of API calls rclone is making to SharePoint. I wound up using `--tpslimit 10`. It copied at a reasonable speed without making Microsoft throttle me down to nothing.

[rclone official website](https://rclone.org/)

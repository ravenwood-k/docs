---
description: The Delegated Administration extension allows you to expose the Users dashboard to a group of users, without allowing them access to the dashboard.
url: /extensions/delegated-admin
toc: true
---

# Delegated Administration

The **Delegated Administration** extension allows you to grant a select group of people administrative permissions to the [Users page](${manage_url}/#/users) without providing access to any other area of the Management Dashboard by exposing the [Users Dashboard](${manage_url}/#/users) as an Auth0 client.

:::panel Appliance Availability
The Delegated Administration extension is available for [Appliance](/appliance) customers who:

* Are running build **10755** or later;
* Have User Search enabled.
:::

Prior to configuring the extension, you will need to:

* [Create and configure an Auth0 Client](#create-a-client);
* [Enable a Connection on the Client](#enable-a-connection-on-the-client);
* [Add a user to the Connection](#add-a-user-to-the-new-connection).

## Create a Client

The first step is to create the Client that the extension exposes to those who should have adminsitrative priviledges to the Users page.

After you've logged into the [Management Dashboard](${manage_url}), navigate to [Clients](${manage_url}/#/clients) and click on **+Create Client**. Provide a name for your Client (such as `Users Dashboard`) and set the Client type to `Single Page Web Applications`. Click **Create** to proceed.

![Create a Client](/media/articles/extensions/delegated-admin/create-client.png)

### Configure Client Settings

Once you've created your Client, you'll need to make the following Client configuration changes.

Click on the **Settings** tab and set the **Allowed Callback URLs**. This varies based on your location:

| Location | Allowed Callback URL |
| --- | --- |
| USA | `https://${account.tenant}.us.webtask.io/auth0-delegated-admin/login` |
| Europe | `https://${account.tenant}.eu.webtask.io/auth0-delegated-admin/login` |
| Australia | `https://${account.tenant}.au.webtask.io/auth0-delegated-admin/login` |

You will also need to configure the **Allowed Logout URLs**:
 
| Location | Allowed Logout URL |
| --- | --- |
| USA | `https://${account.tenant}.us.webtask.io/auth0-delegated-admin` |
| Europe | `https://${account.tenant}.eu.webtask.io/auth0-delegated-admin` |
| Australia | `https://${account.tenant}.au.webtask.io/auth0-delegated-admin` |

Copy the **Client ID** value.

Navigate to **Settings > Show Advanced Settings > OAuth** and paste the **Client ID** value to the **Allowed APPs / APIs** field.

Set the **JsonWebToken Signature Algorithm** to `RS256`.

![Change JsonWebToken Signature Algorithm](/media/articles/extensions/delegated-admin/set-rs256.png)

Click **Save Changes** to proceed.

### Enable a Connection on the Client

When you create a new Client, Auth0 enables all [Connections](/identityproviders) associated with your account by default. For the purposes of this tutorial, we will disable all Connections (this helps keep our Client secure, since no one can add themselves using one of our existing Connections), create a new Database Connection, and enable only the newly-created Database Connection. However, you can choose to use any type of Connection.

#### Disable All Existing Connections

Switch over to the Client's **Connections** tab and disable all the Connections using the associated switches.

#### Create a New Connection

In the navigation pane of the Management Dashboard, click on **Connections** > [Database Connections](${manage_url}/#/connections/database).

On the Database Connections page, click on **+Create DB Connection**. Provide a name for your Connection, such as `Helpdesk`. 

Click **Save** to proceed.

![Create DB Connection](/media/articles/extensions/delegated-admin/create-connection.png)

Navigate to the **Settings** tab of your new Connection and enable the **Disable Sign Ups** option. For security reasons, this ensures that even users who have the link to our Connection cannot sign themselves up.

![Disable Sign Ups](/media/articles/extensions/delegated-admin/disable-signup.png)

Under the **Clients Using This Connection** section, enable this Connection for your `Users Dashboard` Client.

### Add a User to the New Connection

You will need to add at least one user to your Connection. You can do this via the [Users page](${manage_url}/#/users), where you can specify the Connection for the user during the configuration process.

### Assign Roles to Users

Auth0 grants the user(s) in your Connection access to the Delegated Administration extension based on their roles:

- **Delegated Admin - User**: Grants permission to search for users, create users, open users and execute actions on these users (such as `delete`, `block`, and so on);

- **Delegated Admin - Administrator**: In addition to all of the rights a user has, administrators can see all logs in the account and configure Hooks.

To use the extension, users must have either of these roles defined in one of the following fields of their user profiles:

* `user.roles`
* `user.app_metadata.roles`
* `user.app_metadata.authorization.roles`

You can set these fields manually or via [rules](/rules).

#### Set User Roles via Rules

This rule gives users from the `IT Department` the `Delegated Admin - Administrator` role and users from `Department Managers` are the `Delegated Admin - User` role.

```js
function (user, context, callback) {
 if (context.clientID === 'CLIENT_ID') {
   if (user.groups && user.groups.indexOf('IT Department') > -1) {
     user.roles = user.roles || [ ];
     user.roles.push('Delegated Admin - Administrator');
     return callback(null, user, context);
   } else if (user.app_metadata && user.app_metadata.isDepartmentManager && user.app_metadata.department && user.app_metadata.department.length) {
     user.roles = user.roles || [ ];
     user.roles.push('Delegated Admin - User');
     return callback(null, user, context);
   }

   return callback(new UnauthorizedError('You are not allowed to use this application.'));
 }

 callback(null, user, context);
}
```

## Install the Extension

Now that we've created and configured a Client, a Connection, and our users, we can install and configure the extension itself.

On the Management Dashboard, navigate to the [Extensions](${manage_url}/#/extensions) page. Click on the **Delegated Administration** box in the list of provided extensions. The **Install Extension** window will open.

![Install Extension](/media/articles/extensions/delegated-admin/install-extension.png)

Set the following configuration variables:

- **EXTENSION_CLIENT_ID**: The **Client ID** value of the Client you will use. You can find this value on the **Settings** page of your Client.

- **TITLE** (optional): Set a title for your Client. It will be displayed at the header of the page.

- **CUSTOM_CSS** (optional): Procide a CSS script to customize the look and feel of your Client.

Once done, click **Install**. Your extension is now ready to use!

If you navigate back to the [Clients](${manage_url}/#/clients) view, you will see that the extension automatically created an additional client called `auth0-delegated-admin`.

![](/media/articles/extensions/delegated-admin/two-clients.png)

Because the client is authorized to access the [Management API](/api/management/v2), you shouldn't modify it.

## Use the Extension

To access your newly created users dashboard, navigate to [**Extensions**](${manage_url}/#/extensions) > **Installed Extensions** > **Delegated Administration Dashboard**.

A new tab will open to display the login prompt.

![](/media/articles/extensions/delegated-admin/login-prompt.png)

Because we disabled signups for this Connection during the configuration period, the login screen doesn't display a Sign Up option.

Once you provide valid credentials, you'll be redirected to the *Delegated Administration Dashboard*.

![](/media/articles/extensions/delegated-admin/standard-dashboard.png)

## Keep Reading

* [Customizing the Delegated Administration Extension Using Hooks](/extensions/delegated-admin/hooks)

* [Managing Users in the Delegated Administration Extension Dashboard](/extensions/delegated-admin/manage-users)

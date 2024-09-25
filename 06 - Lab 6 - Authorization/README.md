# How to Configure Users in Keycloak

## How to enable Identity in your Helm Chart

Identity needs to be enabled. It is also required to define the redirect URLs, that are used by Keycloak to access the applications.

```
global:
  identity:
    auth:
      enabled: true
      publicIssuerUrl: "http://keycloak.localhost/auth/realms/camunda-platform"
      operate:
        redirectUrl: "http://operate.localhost"
      tasklist:
        redirectUrl: "http://tasklist.localhost"
      optimize:
        redirectUrl: "http://optimize.localhost"
```

All Configuration Options can be found in the official [Helm Chart Documentation](https://github.com/camunda/camunda-platform-helm/tree/main/charts/camunda-platform).

Now you need to install the Camunda Platform chart:

```
helm install camunda-platform camunda/camunda-platform -f values.yaml
```

When observing your Kubernetes Cluster (e.g. `kubectl get pods`), you will see that next to Identity, also a Keycloak Instance and a Postgres will be deployed. Keycloak is a dependency for Camunda Identity, and PostgreSQL is a dependency for Keycloak.

```
camunda-platform
  |_ identity
    |_ keycloak
      |_ postgresql
```

Once all your pods are ready, you are able to login into Identity using the default credentials. Username: demo and Password: demo

![Identity Login](./screenshots/manual/01-identity-login.png)


When navigating to the Users Tab, you will see that the default user is the only user that currently exists.

![Identity User](./screenshots/manual/02-identity-user.png)

To add new users you have to access Keycloak and use one of the explained approaches.

## Add Users

For adding users, we rely on the configuration options of Keycloak.
Therfore we first of all need to access Keycloak.

You can access the Adminstration Console via the tile with the same name.

keycloak.localhost/auth

![Keycloak Menu](./screenshots/manual/03-keycloak-menu.png)

Now you need to login with the Keycloak Admin. The default admin user can be set via the Helm Chart:

```
identity:
  keycloak:
    auth:
      adminUser: admin
      adminPassword: admin
```

![Keycloak Login](./screenshots/manual/04-keycloak-login.png)

Alternatively you can retrieve the Admin Password with following command:

```
kubectl get secrets/ccsm-keycloak -o jsonpath='{.data.admin-password}' | base64 -d
```

The default Admin Username is `user`.

### Add Users Manually

To add users manually, you need to navigate to `Users` in the `Manage` section.

![User Overview](./screenshots/manual/05-keycloak-user-overview.png)

Now press on `Add user`.

![Add User](./screenshots/manual/06-keycloak-add-user.png)

Once saved, you can also set the password of the user in the Credentials Tab.

![Set Password](./screenshots/manual/07-keycloak-set-password.png)

The created User is now also visible in Identity. However, currently no roles are assigned, besides the default user role.

![New Identity User](./screenshots/manual/08-new-user-identity.png)

Press `Assign role`. Now you can select one or multiple roles. The example below shows how to grant access only for Operate and Tasklist.

![Assign Role](./screenshots/manual/09-assign-roles%20to%20user.png)

To verify the configuration, you can now login to one of the applications with your newly created user.

![Validate User](./screenshots/manual/10-validate-user-allowed.png)

This approach works fine for a small set of users. However, Keycloak also allows you to connect to existing IAM systems, to minimize the manual configuration effort.

### User Federation (LDAP / AD / Kerberos)

To connect to your LDAP, Active Directory, or Kerberos server, select User Federation in the main menu, click `Add provider...`, and fill in all required configuration settings.
The below example walks you through the configuration of a ldap connection. 

#### Configure User Synchronization

![Add LDAP](./screenshots/user-federation/01-add-ldap.png)

Below you can find an example configuration that uses the LDAP Interface of Okta.
Press On `Test authentication` to make sure that everything is configured correctly.

![Configuration Example](./screenshots/user-federation/02-configuration-example.png)

Afterwards press on `Save` and `Synchronize all users`. A notification message will show you how many users were imported.

![Sync Users](./screenshots/user-federation/03-sync-users.png)

You can verify the user import by navigating to the Users Section and click on `View all users`.

![Verify User](./screenshots/user-federation/04-verify-user.png)

As in the manual approach, you can now assign roles to the individual users.
Alternatively you can also map LDAP Groups to Roles within Keycloak. However, by default the groups will not be synchronized.

#### Configure Group Synchronization

Next to the Settings for your User Federation you can also define Mappings in the Mappers Tab. Press on `Create`.

![Add Mapper](./screenshots/user-federation/05-add-mapper.png)

Now select the Mapper Type `group-ldap-mapper`.

![Group LDAP Mapper](./screenshots/user-federation/06-group-ldap-mapper.png)

Below you can see an example configuration that uses the LDAP Interface of Okta. Once you provided your configuration parameters, click on `Sync LDAP Groups To Keycloak`. 
In the Notification you can see how many groups were imported.

![Sync Groups](./screenshots/user-federation/07-sync-ldap-groups.png)

Verify the Group Synchronization in the Groups section. 

![Improted Groups](./screenshots/user-federation/08-imported-groups.png)

Instead of now adding each user individual to a role in Identity, you can create a role mapping for a group.

To do this, navigate to Groups and select one of the groups that you would like to map to a role. Select the Tab Role Mappings.

![Role Mappings](./screenshots/user-federation/09-role-mappings.png)

You can now select the roles that you want to assign to the group. E.g. if you want to give access to Operate to every developer, click on the Role `Operate` and click `Add selected`.

Now everyone in the LDAP Group "developer" has access to Operate.

![Assign Roles](./screenshots/user-federation/10-assign-roles.png)

### Identity Provider (OIDC, SAML, ..)

To add an OpenID Connect or SAML provider, select Identity Providers in the main menu, click `Add provider...`, and fill in all required configuration settings.

![Add Provider](./screenshots/identity-provider/01-add-provider.png)

#### OIDC Example

![Okta](./screenshots/identity-provider/oidc/02-okta.png)

If you want to map profile information automatically, make sure to configure the default scope to `openid profile`.

When you now want to login to one of the web applications, you can see that there is now an alternative login option that says `Login with Okta`.

![Login with Okta](./screenshots/identity-provider/oidc/03-login-with-okta.png)

When now pressing on that button, you will be redirect to your Identity Provider

![Okta Login page](./screenshots/identity-provider/oidc/04-okta-form.png)

For the first login, you will be asked to update the account information.

![Update Profile](./screenshots/identity-provider/oidc/05-update-profile.png)

#### SAML Example

Below, Okta will be used as an Example.

First you need to create an App Integration

![Create App Integration](./screenshots/identity-provider/saml/01-create-application-okta.png)

Select SAML 2.0

![SAML 2.0](./screenshots/identity-provider/saml/02-saml-integration-okta.png)

Provide a name for your App

![App Name](./screenshots/identity-provider/saml/03-app-name-okta.png)

When configuring a SAML-based Identity Provider in Keycloak, you will also receive the required configuration parameters.

For configuring the App Integration you need:
- The Redirect URI (here: http://keycloak.localhost/auth/realms/camunda-platform/broker/okta/endpoint)
- Service Provider Entity ID (here: http://keycloak.localhost/auth/realms/camunda-platform )

![Keycloak Parameters](./screenshots/identity-provider/saml/04-identity-provider-keycloak.png)

Those parameters now need to be used when configuring your App Integration.

![General Settings](./screenshots/identity-provider/saml/05-general-saml-settings-okta.png).

If you want to pass on the first name, last name and email and groups, you need to include the following attribute statements.

![Attribute Statements](./screenshots/identity-provider/saml/06-attribute-statements-okta.png)

After completing the app integration setup, you need to import the resulting metadata into Keycloak.

![Import Metadata](./screenshots/identity-provider/saml/07-import-metadata.png)

Now you can login via Okta!

![Okta Login](./screenshots/identity-provider/saml/08-login-with-okta.png)

Without providing the right mappers, you will be asked to update the account information manually.

![Update Account Information](./screenshots/identity-provider/saml/09-update-account.png)

With Identity Provider Mappers, the account information can retrieved from the attributes.

Below example shows an Mapper for the lastName. You would need to perform the same for firstName and email.

![lastName Mapper](./screenshots/identity-provider/saml/10-attribute-importer.png)

You can also map groups to Roles. Below an example of mapping the group "Everyone" to the Role "Tasklist"

![Group Mapper](./screenshots/identity-provider/saml/11-everyone-to-tasklist-mapper.png)

Now everyone who is part of the Group "Everyone" is able to login to Tasklist

#### Login only via Identity Provider

By default, you will still have the option to login via username and password, eventhough an Identity Provider is configured. To avoid this, it is possible to adjust the authentication flow to make sure the user is redirect to the Identity Provider.

Press on Actions > Configure in the Authentication Flows (Browser)

![Group Mapper](./screenshots/identity-provider/idp-login-only/01-authentication-flows.png)

Provide the alias of your Identity Provider as a default identity provider. 





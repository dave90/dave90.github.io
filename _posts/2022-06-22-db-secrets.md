---
title: The secrets of Databricks
categories: [Big Data, Databricks]
tags: [Big Data, Databricks, Secrets]
---

# Fantastic Best Practices: The secrets of Databricks
<img src="/assets/img/posts/db-secrets/secrets_of_db.jpg" width="75%" height="75%">

The use of hard-coded passwords or credentials is a really bad practice in the world of software develpment.  

> If hard-coded passwords are used, it is almost certain that malicious users will gain access through the account in question.

<div align="center">
<figure>
<img src="/assets/img/posts/db-secrets/hackerman.jpeg" class="center" width="75%" height="75%">
<figcaption align="center"><b>Typical malicious users</b></figcaption>
</figure>
</div>

So in this blog post, we will see how to securely manage secrets and passwords in our favorites big data platform: Databricks. 

Managing secrets begins with creating a secret scope. 

>A secret scope is collection of secrets identified by a name.

There are two types of secret scope: **Azure Key Vault-backed** and **Databricks-backed**.
Obviously, Azure Key Vault-backed is only avaliable using Azure.

<img src="/assets/img/posts/db-secrets/cp_obs.jpeg" class="center" width="75%" height="75%">



## <span style="color: var(--link-color);">Databricks-backed secret scope</span>

For the creation of the  secret scope we can use the [databricks CLI](https://learn.microsoft.com/en-us/azure/databricks/dev-tools/cli/):

```shell
databricks secrets create-scope --scope my-db-scope --initial-manage-principal "users"
```

The manage permission (--initial-manage-principal) define who can manage the secrets but
if your account does not have the Premium Plan (like me), you must override that default and explicitly grant the MANAGE permission to “users” (all users) when you create the scope. 

After the creation of the scope you can list the secret scope:

```shell
databricks secrets list-scopes
```

<img src="/assets/img/posts/db-secrets/secret_list.png" class="center" width="100%" height="100%">

And now we can insert secrets with the command:

```shell
databricks secrets put --scope my-db-scope --key my-key-1 --string-value "HELLO. I am a key :)"
```
If we want to get the secrets in databricks we can use *dbutils*:

<img src="/assets/img/posts/db-secrets/res_1.png" class="center" width="100%" height="100%">

**NOTE**: secrets are redacted in the logs of databricks ;)

## <span style="color: var(--link-color);">Azure Key Vault-backed</span>

>Verify that you have the Directory Readers role in your Azure Active Directory tenant.

>Verify that you have Contributor or Owner role on the Azure key vault instance that you want to use to back the secret scope.

You can create AKV secret scope using web UI or CLI. We are lazy so let's do with UI ;).

>NOTE: If you want to use databricks CLI or API you need an Azure AD user token to create an Azure Key Vault-backed secret scope with the Databricks CLI. You cannot use an Azure Databricks personal access token or an Azure AD application token that belongs to a service principal.

Go to *https://\<databricks-instance\>#secrets/createScope* and insert the name of the secret scope and the informations of the AKV.

<img src="/assets/img/posts/db-secrets/create_scope_akv.png" class="center" width="100%" height="100%">

These properties of the AKV are available from the Properties tab of an Azure Key Vault in your Azure portal where DNS name is the vault URI.

<img src="/assets/img/posts/db-secrets/akv_info.png" class="center" width="100%" height="100%">

Then click the Create button.


Now let's create one secret in our AKV and test on databricks.

<img src="/assets/img/posts/db-secrets/secret_add.png" class="center" width="100%" height="100%">
<img src="/assets/img/posts/db-secrets/akv_test1.png" class="center" width="100%" height="100%">
<img src="/assets/img/posts/db-secrets/akv_test2.png" class="center" width="100%" height="100%">



## <span style="color: var(--link-color);">Use a secret in a Spark configuration</span>
You can use a secret in a Spark configuration property or environment variable. 

The syntax must be:

```
{ { secrets/<scope-name>/<secret-name> } }
```

 The value must start with **{ { secrets/** and end with **}}**. 

<img src="/assets/img/posts/db-secrets/env.png" class="center" width="100%" height="100%">

## <span style="color: var(--link-color);">Reference link</span>
[Azure databricks secret](https://learn.microsoft.com/en-us/azure/databricks/security/secrets/secret-scopes
)

[OWASP hard code password](https://owasp.org/www-community/vulnerabilities/Use_of_hard-coded_password)

[More azure databricks secrets](https://learn.microsoft.com/en-us/azure/databricks/security/secrets/secrets#--use-a-secret-in-a-spark-configuration-property-or-environment-variable)

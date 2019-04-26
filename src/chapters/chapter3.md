## Increasing the Security of Deployed Apps 

In this chapter, you'll learn:
- How to secure secrets (like SQL databse connection strings) with Key Vaults
- How to add authentication to your web apps instantly

### Securing Secrets in Web Apps

Since the early days of .NET, developers have been putting SQL database connection strings in configuration files. This is not only a bad practice, but a serious security risk for several reasons:
- login credentials are saved in plain-text 
- it's easy to accidentally check in the credentials

Because of the risks of plain-text credentials, developers started encrypting the vaules, but this brought challenges with how to encrypt and decrypt, and made a developer's job more difficult.

<img src="images/chapter3/better-way.gif" />

There's got to be a better way!

#### Key Vault to the Rescue!

Introducing Key Vault! Azure Key Vault helps solve the following problems:

- **Secrets Management**: Azure Key Vault can be used to Securely store and tightly control access to tokens, passwords, certificates, API keys, and other secrets
- **Key Management**: Azure Key Vault can also be used as a Key Management solution. Azure Key Vault makes it easy to create and control the encryption keys used to encrypt your data.
- **Certificate Management**: Azure Key Vault is also a service that lets you easily provision, manage, and deploy public and private Secure Sockets Layer/Transport Layer Security (SSL/TLS) certificates for use with Azure and your internal connected resources.
- **Store secrets backed by Hardware Security Modules**: The secrets and keys can be protected either by software or FIPS 140-2 Level 2 validates HSMs

So, with Key Vault, you store your connection strings inside of a secret store instead of in your app configuration file. Then, you simply store the location or URL of the Key Vault in your config file, so when your app starts up, you access the Key Vault, extract your secrets, then continue on as normal.

> **But wait...don't I need a secret/password to access a Key Vault**
>
> Yes you do, but it doesn't make much sense to store that secret inside of your app configuration file. Afterall, that's exactly what we're trying to avoid!

#### Accessing KeyVault without a Password

That's right. You can actually access the Key Vault without a password - at least a password that you're aware of. 

The second critical component to securing your app secrets is something called a Managed Service Identity (or MSI for short). You can think of an MSI like a service account, but a service account that doesn't have a password. 

Within Azure, you can assign an MSI to a Web App, so when the app runs it "acts" or "takes on" the role of the MSI. So, once it's been assigned to a Web App, you can give the Web App MSI permissions to access your Key Vault. 

To the developer, this seems like magic - when you app runs, it becomes an MSI, then has permissions to access the Key Vault without a password. But behind the scenes Azure is managing the entire process, ensuring your Web App is the only resource using the assigned MSI.

### Assigning an MSI to your Web App

Let's get started by assigning an MSI to your web app!

<h4 class="exercise-start">
    <b>Exercise</b>: Assigning an MSI through the Azure Portal
</h4>

Navigate to the web app you deployed in an earlier chapter. Mine was called *contoso-web-meb*.

Scroll down on the left to find a section named *Identity*. Click on *Identity*:

<img src="images/chapter3/identity.png" class="img-medium" />

On the *System assigned* tab, change the status to *On* and press *Save*.

<img src="images/chapter3/system-assigned.png" class="img-medium" />

Click *Yes* when prompted.

<img src="images/chapter3/accept.png" class="img-override" />

...and that's it! Your MSI (Managed Service Identity) is assigned. 

<img src="images/chapter3/assigned.png" class="img-override" />

This concludes the exercise. 

<div class="exercise-end"></div> 

### Creating a Key Vault

Now that you have an MSI assigned to your web app, it's time to create a Key Vault and give the MSI access to read secrets.

<h4 class="exercise-start">
    <b>Exercise</b>: Creating a Key Vault and Assigning Access
</h4>

In the Azure portal, search for the Key Vault by searching for it, like you did the SQL Database in an earlier chapter.

<img src="images/chapter3/search-kv.png" class="img-medium" />

Place the Key Vault in the resource group you created for the workshop:

<img src="images/chapter3/create-kv.png" />

Click the *Access policies* section, then the *+ Add new* link to add the web app MSI to the Key Vault's authorized users:

<img src="images/chapter3/access-policies-kv.png" class="img-medium" />

Do the following:
1. Select *Secret management* from the *Configure from template* option
2. Click *Select principal*
3. In the box to the right, type the name of your web app (mine was contoso-web-meb)
4. Click your web app's name when it appears below
5. Click the *Select* button
6. Click *Ok*

You should now see your web app added:

<img src="images/chapter3/added.png" class="img-medium" />

Click *Ok* to return to the Key Vault creation screen, and *Create* to provision the Key Vault.

It will take a few minutes to deploy the Key Vault. When it's finished provisioning, pin it to your dashboard (we'll be using it a lot today).

This concludes the exercise. 

<div class="exercise-end"></div> 


### Adding Secrets to the Key Vault

Now that you have created a Key Vault and assigned access to it, let's add a secret!

<h4 class="exercise-start">
    <b>Exercise</b>: Adding a secret to a Key Vault
</h4>

Navigate to your Key Vault and select the *Secrets* area on the left:

<img src="images/chapter3/secrets.png" class="img-medium" />

Click *Generate/Import*. 

For the secret name, enter *ConnectionStrings--DefaultConnection*. Copy and paste your conneciton string from the app configuration file for the value.

> **Secret Names Matter**
>
> Go back and look at the app configuration file for Contoso University. You'll notice the JSON structure has a ConnectionStrings key following by a sub key of DefaultConnection. This is important. 
>
> In Key Vault, we'll mimick this struture by using the same key names and a double dash for the key name. Later, when we tell our web app to read the Key Vault secrets, it will be able to parse the secret name and dynamically replace the correct key/value pair in our configuration file.

<img src="images/chapter3/create-secret.png" />

Click *Create*.

<img src="images/chapter3/created.png" />

This concludes the exercise. 

<div class="exercise-end"></div> 

### Incorporating Key Vault Access into Web Apps

Now that we have secrets in our Key Vault, let's configure the web app to access the Key Vault. 

<h4 class="exercise-start">
    <b>Exercise</b>: Configuring a Web App to read configuration from a Key Vault
</h4>

To configure the Contoso University app to read configuration data from the Key Vault, we need to modify the app's startup process to load it's current ocnfiguraiton file, then reach out to the Key Vault, read the secrets, and dynamically load the secret values into our configuration.

Let's start by adding a few NuGet packages to the ContosoUniversity project.

Right-click the Solution, and add the following packages. **Be midful to add the specific version(s) we've specified below!**

- Microsoft.Azure.Services.AppAuthentication, v1.2.0-preview2
- Microsoft.Extensions.Configuration.AzureKeyVault, v2.2.0

Next, create a new class in the root of the ContosoUniversity project named `PrefixKeyVaultSecretManager.cs`. Place the following code inside.

```c#
using Microsoft.Azure.KeyVault.Models;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.Configuration.AzureKeyVault;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

namespace ContosoUniversity
{
    public class PrefixKeyVaultSecretManager : IKeyVaultSecretManager
    {
        public string GetKey(SecretBundle secret)
        {
            // Remove the prefix from the secret name and replace two 
            // dashes in any name with the KeyDelimiter, which is the 
            // delimiter used in configuration (usually a colon). Azure 
            // Key Vault doesn't allow a colon in secret names.
            return secret.SecretIdentifier.Name
                .Replace("--", ConfigurationPath.KeyDelimiter);
        }

        public bool Load(SecretItem secret)
        {
            return true;
        }
    }
}
```

You can think of htis class as a *conversion* class. The *GetKey()* function is called each time a key/value pair is restrived from the Key Vault. The function then translates any double dashes into colons. This is an important step because .NET Core apps (like this one) natively translate colons into a heirarchical configuraiton file manner when parsing config values. 

So, a value of *ConnectionStrings--DefaultConnection* is translated by the *GetKey()* function into *ConnectionStrings:DefaultConnection*. Then, .NET Core reads the string and replaces the configuration key of ConnectionStrings with sub key DefaultConnection with the value pulled from the Key Vault.

> **Why not use colons in the Key Vault secret?**
>
> You may be wondering why we had to convert double dashes into colons. Unfortunately, key vault key names cannot contain colons, so some type of replacement delimiter must be used. I chose a double dash. 

Next, update the *CreateWebHostBuilder()* function in the `Program.cs` file to tie everything together. After updaitng the function, don't forget to add the missing references at top. 

```c#
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .ConfigureAppConfiguration((context, builder) =>
        {
            // only run in "production" mode, so you don't override 
            // config values when running locally for development
            if (context.HostingEnvironment.IsProduction())
            {
                var config = builder.Build();

                // get MSI token from running web app
                var tokenProvider = new AzureServiceTokenProvider(); 

                // create kv client, passing MSI token for authorization
                var keyvaultClient = new KeyVaultClient((authority, resource, scope)
                    => tokenProvider.KeyVaultTokenCallback(authority, resource, scope));

                // add the Key Vault "provider" that scans the KV, loading secrets into configuration
                builder.AddAzureKeyVault(config["KeyVault:BaseUrl"], keyvaultClient, new PrefixKeyVaultSecretManager());
            }
        })
        .UseStartup<Startup>();
```

These additional lines of code essentially does the following:
1. Only runs the Key Vault loading code when runningin production (so you don't override your development settings)
2. Gets the MSI token from the running web app
3. Creates a client to talk to the Key Vault through, using the token for an authorization context
4. Adds the Key Vault "provider" into the pipeline that builds the configuration file. It also assumes a configuration file value of "KeyVault:BaseUrl" is present

Finally, let's head on over to the configuration file and make a few changes:

Change the database connection string back to the MS SQL Local DB value (this way, we'll dev in our local environment):

```
Server=(localdb)\\mssqllocaldb;Database=ContosoUniversity3;Trusted_Connection=True;MultipleActiveResultSets=true
```

Add a key/value pair for the *KeyVault:BaseUrl*. Be sure to substitute the URL of your Key Vault (you can find it in the Azure Portal).

```json
  "KeyVault": {
    "BaseUrl": "{your_keyvault_url_goes_here}"
  }
```

This is what my configuration file looks like when I'm finished:

<img src="images/chapter3/config.png" class="img-medium" />

To test your changes, run your app locally and add a new student. Then, publish your app to Azure. Navigate to the app's Azure URL and verify that the student you previously added isn't present.

This concludes the exercise. 

<div class="exercise-end"></div> 

### Adding Authentication to your App

Now that we've secured our app's secrets in the Key Vault, it's time to worry about the app itself! So far, anybody can access Contoso University's site and make changes. Let's lock it down.

<h4 class="exercise-start">
    <b>Exercise</b>: Adding Authentication to Contoso University's app
</h4>

This is a lot easier than you may think ;-)

Navigate to your web app in the Azure portal and select the "Authentication / Authorization" option on the left.

<img src="images/chapter3/auth.png" class="img-override" />

Change the *App Service Authentication* option to *On*, select *Log in with Azure Active Directory* form the drop down. Then, click *Azure Active Directory* authentication provider.

On the next screen, select *Express* management mode, and press *Ok*. Accept all other defaults.

<img src="images/chapter3/aad-auth.png" class="img-override" />

Press *Save* at top.

That's it. Your app is now secured by Azure Active Directory. Anyone trying to access the site willbe automatically prompted to login. Try it. The first time you access it, it will prompt you to login (or detect that you're already logged into the Azure portal with the same credentials) and ask you for consent to read your login profile. Accept it.

<img src="images/chapter3/prompt.png" class="img-medium" />

This concludes the exercise. 

<div class="exercise-end"></div> 

Phew. This was a long chapter, but you learned some important cloud-native ways of securing secrets with the Key Vault.

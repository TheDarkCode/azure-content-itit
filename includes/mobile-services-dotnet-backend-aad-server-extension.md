### (Facoltativo) Configurare il servizio mobile .NET per l'accesso ad Azure Active Directory

>[AZURE.NOTE]Questa procedura è facoltativa perché si applica solo al provider di accesso di Azure Active Directory.

1. Installare il pacchetto NuGet [WindowsAzure.MobileServices.Backend.Security](https://www.nuget.org/packages/WindowsAzure.MobileServices.Backend.Security).

2. In Visual Studio espandere App\_Start e aprire il file WebApiConfig.cs. Aggiungere l'istruzione `using` seguente nella parte superiore:

        using Microsoft.WindowsAzure.Mobile.Service.Security.Providers;

3. Inoltre in WebApiConfig.cs aggiungere il codice seguente al metodo `Register`, immediatamente dopo la creazione di un'istanza di `options`:

        options.LoginProviders.Remove(typeof(AzureActiveDirectoryLoginProvider));
        options.LoginProviders.Add(typeof(AzureActiveDirectoryExtendedLoginProvider));

<!---HONumber=Oct15_HO3-->
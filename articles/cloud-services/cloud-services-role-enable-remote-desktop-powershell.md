<properties
pageTitle="Abilitare una connessione Desktop remoto per un ruolo in Servizi cloud di Azure con PowerShell"
description="Come configurare l'applicazione del servizio cloud di Azure utilizzando PowerShell per consentire le connessioni Desktop remoto"
services="cloud-services"
documentationCenter=""
authors="thraka"
manager="timlt"
editor=""/>  
<tags
ms.service="cloud-services"
ms.workload="tbd"
ms.tgt_pltfrm="na"
ms.devlang="na"
ms.topic="article"
ms.date="08/05/2016"
ms.author="adegeo"/>

# Abilitare una connessione Desktop remoto per un ruolo in Servizi cloud di Azure con PowerShell

>[AZURE.SELECTOR]
- [Portale di Azure classico](cloud-services-role-enable-remote-desktop.md)
- [PowerShell](cloud-services-role-enable-remote-desktop-powershell.md)
- [Visual Studio](../vs-azure-tools-remote-desktop-roles.md)


Desktop remoto consente di accedere al desktop di un ruolo in esecuzione in Azure. È possibile usare una connessione Desktop remoto per risolvere e diagnosticare i problemi dell'applicazione mentre è in esecuzione.

Questo articolo descrive come abilitare Desktop remoto nei ruoli dei servizi cloud con PowerShell. Per i prerequisiti necessari per questo articolo, vedere [Come installare e configurare Azure PowerShell](../powershell-install-configure.md). PowerShell usa l'estensione di Desktop remoto per poter abilitare Desktop remoto dopo la distribuzione dell'applicazione.


## Configurare Remote Desktop da PowerShell

Il cmdlet [Set AzureServiceRemoteDesktopExtension](https://msdn.microsoft.com/library/azure/dn495117.aspx) consente di abilitare Desktop remoto su tutti o specifici ruoli di distribuzione del servizio cloud. Il cmdlet consente di specificare il nome utente e la password per l'utente Desktop remoto tramite il parametro *Credential* che accetta un oggetto PSCredential.

Se si usa PowerShell in modo interattivo è possibile impostare facilmente l'oggetto PSCredential chiamando il cmdlet [Get-Credentials](https://technet.microsoft.com/library/hh849815.aspx).

```
$remoteusercredentials = Get-Credential
```

Il comando visualizza una finestra di dialogo in cui immettere in modo sicuro il nome utente e la password per l'utente remoto.

Poiché PowerShell è utile negli scenari di automazione, è anche possibile configurare l'oggetto **PSCredential** in modo che non richieda l'interazione dell'utente. In questo caso, è necessario configurare una password di protezione. Per iniziare, specificare una password non crittografata e convertirla in una stringa sicura con [ConvertTo-SecureString](https://technet.microsoft.com/library/hh849818.aspx). Quindi è necessario convertire questa stringa sicura in una stringa standard crittografata con [ConvertFrom-SecureString](https://technet.microsoft.com/library/hh849814.aspx). Ora è possibile salvare questa stringa crittografata standard in un file con [Set-Content](https://technet.microsoft.com/library/ee176959.aspx).

È anche possibile creare un file della password di protezione in modo che non sia necessario digitarla ogni volta. Inoltre, un file della password di protezione è preferibile a un file di testo normale. Usare il codice di PowerShell seguente per creare un file della password sicura:

```
ConvertTo-SecureString -String "Password123" -AsPlainText -Force | ConvertFrom-SecureString | Set-Content "password.txt"
```

>[AZURE.IMPORTANT] Quando si imposta la password, assicurarsi che soddisfi i [requisiti di complessità](https://technet.microsoft.com/library/cc786468.aspx).

Per creare l'oggetto credential dal file della password di protezione, è necessario leggere il contenuto del file e riconvertirlo in una stringa sicura con [ConvertTo-SecureString](https://technet.microsoft.com/library/hh849818.aspx).

Il cmdlet [Set AzureServiceRemoteDesktopExtension](https://msdn.microsoft.com/library/azure/dn495117.aspx) accetta anche un parametro *Expiration* che specifica un valore **DateTime** che specifica la scadenza dell'account utente. Ad esempio, è possibile impostare la scadenza dell'account alcuni giorni dopo la data e l'ora correnti.

Questo esempio di PowerShell illustra come impostare l'estensione di Desktop remoto in un servizio cloud:

```
$servicename = "cloudservice"
$username = "RemoteDesktopUser"
$securepassword = Get-Content -Path "password.txt" | ConvertTo-SecureString
$expiry = $(Get-Date).AddDays(1)
$credential = New-Object System.Management.Automation.PSCredential $username,$securepassword
Set-AzureServiceRemoteDesktopExtension -ServiceName $servicename -Credential $credential -Expiration $expiry
```
È anche possibile specificare lo slot di distribuzione e i ruoli per i quali si vuole abilitare Desktop remoto. Se questi parametri non vengono specificati, il cmdlet abilita Desktop remoto su tutti i ruoli nello slot di distribuzione **Produzione**.

L'estensione per il Desktop remoto è associata a una distribuzione. Se si crea una nuova distribuzione per il servizio, si dovrà abilitare Desktop remoto in quella distribuzione. Se si vuole che Desktop remoto sia sempre abilitato, considerare la possibilità di integrare gli script di PowerShell nel flusso di lavoro di distribuzione.


## Desktop remoto in un'istanza del ruolo
Il cmdlet [Get-AzureRemoteDesktopFile](https://msdn.microsoft.com/library/azure/dn495261.aspx) viene usato per integrare Desktop remoto in una specifica istanza del ruolo del servizio cloud. È possibile usare il parametro *LocalPath* per scaricare il file RDP in locale. In alternativa si può usare il parametro *Launch* per avviare direttamente la finestra di dialogo Connessione Desktop remoto per accedere all'istanza del ruolo del servizio cloud.

```
Get-AzureRemoteDesktopFile -ServiceName $servicename -Name "WorkerRole1_IN_0" -Launch
```


## Verificare se l'estensione di Desktop remoto è attivata in un servizio
Il cmdlet [Get AzureServiceRemoteDesktopExtension](https://msdn.microsoft.com/library/azure/dn495261.aspx) indica che Desktop remoto è abilitato o disabilitato in una distribuzione del servizio. Il cmdlet restituisce il nome utente per l'utente Desktop remoto e i ruoli per quali è abilitata l'estensione di Desktop remoto. Per impostazione predefinita, ciò avviene nello slot di distribuzione ed è possibile scegliere di usare in alternativa lo slot di staging.

```
Get-AzureServiceRemoteDesktopExtension -ServiceName $servicename
```

## Rimuovere l'estensione di Desktop remoto da un servizio
Se già stata abilitata l'estensione di Desktop remoto in una distribuzione ed è necessario aggiornare le impostazioni di Desktop remoto, occorre prima di tutto rimuovere l'estensione e quindi riabilitarla con le nuove impostazioni. Ad esempio, se si vuole impostare una nuova password per l'account utente remoto o l'account scaduto. Questa operazione è necessaria nelle distribuzioni esistenti in cui è abilitata l'estensione di Desktop remoto. Per le nuove distribuzioni è possibile applicare semplicemente l'estensione direttamente.

Per rimuovere l'estensione di Desktop remoto dalla distribuzione, è possibile usare il cmdlet [Remove AzureServiceRemoteDesktopExtension](https://msdn.microsoft.com/library/azure/dn495280.aspx). È anche possibile specificare lo slot di distribuzione e il ruolo dal quale si vuole rimuovere l'estensione di Desktop remoto.

```
Remove-AzureServiceRemoteDesktopExtension -ServiceName $servicename -UninstallConfiguration
```

>[AZURE.NOTE] Per rimuovere completamente la configurazione dell'estensione, è necessario chiamare il cmdlet *remove* con il parametro **UninstallConfiguration**.
>
>Il parametro **UninstallConfiguration** disinstalla qualsiasi configurazione dell'estensione applicata al servizio. Ogni configurazione dell'estensione è associata alla configurazione del servizio. Se si chiama il cmdlet *remove* senza **UninstallConfiguration** la <mark>distribuzione</mark> verrà dissociata dalla configurazione dell'estensione, rimuovendo in effetti l'estensione. La configurazione dell'estensione rimane comunque associata al servizio.



## Risorse aggiuntive

[Come configurare i servizi cloud](cloud-services-how-to-configure.md)

<!---HONumber=AcomDC_0810_2016-->
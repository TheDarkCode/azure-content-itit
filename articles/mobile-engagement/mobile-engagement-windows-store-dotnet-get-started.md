<properties
	pageTitle="Introduzione ad Azure Mobile Engagement per app universali di Windows"
	description="Informazioni sull'uso di Azure Mobile Engagement con funzionalità di analisi e notifiche push per le app di Windows universali."
	services="mobile-engagement"
	documentationCenter="windows"
	authors="piyushjo"
	manager="dwrede"
	editor="" />

<tags
	ms.service="mobile-engagement"
	ms.workload="mobile"
	ms.tgt_pltfrm="mobile-windows-store"
	ms.devlang="dotnet"
	ms.topic="hero-article"
	ms.date="08/12/2016"
	ms.author="piyushjo;ricksal" />

# Introduzione a Azure Mobile Engagement per app di Windows universali

[AZURE.INCLUDE [Banner per la selezione del tipo di esercitazione](../../includes/mobile-engagement-hero-tutorial-switcher.md)]

Questo argomento descrive come usare Azure Mobile Engagement per ottenere informazioni sull'utilizzo dell'app e inviare notifiche push a utenti segmentati di un'applicazione universale di Windows. Questa esercitazione illustra uno scenario di trasmissione semplice tramite Mobile Engagement. Si crea un'app universale di Windows vuota che raccoglie dati di base relativi all'utilizzo dell'app e riceve notifiche push tramite Servizi notifica Push Windows (WNS).

## Prerequisiti

[AZURE.INCLUDE [Prerequisiti](../../includes/mobile-engagement-windows-store-prereqs.md)]


## Configurare Mobile Engagement per l'app universale di Windows

[AZURE.INCLUDE [Creare l'app Mobile Engagement nel portale](../../includes/mobile-engagement-create-app-in-portal.md)]

##<a id="connecting-app"></a>Connettere l'app al back-end di Mobile Engagement

Questa esercitazione presenta una "integrazione di base", che è la configurazione minima necessaria per raccogliere i dati e inviare una notifica push. La documentazione sull'integrazione completa è reperibile nella [integrazione di Mobile Engagement SDK per app di Windows universali](mobile-engagement-windows-store-sdk-overview.md).

Si crea un'app di base con Visual Studio per illustrare l'integrazione.

###Creare un progetto di app universale di Windows

I passaggi seguenti presuppongono l'utilizzo di Visual Studio 2015 anche se i passaggi sono simili nelle versioni precedenti di Visual Studio.

1. Avviare Visual Studio e selezionare **Nuovo progetto** nella schermata **Home**.

2. Nel popup selezionare **Windows** -> **Universale** -> **App vuota (Windows universale)**. Inserire **Nome** e **Nome soluzione** dell’applicazione, quindi fare clic su **OK**.

    ![][1]

A questo punto è stata creata un'app universale di Windows in cui si integra quindi Azure Mobile Engagement SDK.

###Connettere l'app al back-end di Mobile Engagement

1. Installare il pacchetto NuGet [MicrosoftAzure.MobileEngagement] nel progetto. Se l'app è destinata a entrambe le piattaforme Windows e Windows Phone, è necessario eseguire questa operazione per entrambi i progetti. Per Windows 8.x e Windows Phone 8.1, lo stesso pacchetto NuGet inserisce i file binari corretti specifici della piattaforma in ogni progetto.

2. Aprire **Package.appxmanifest** e assicurarsi che venga aggiunta la funzionalità seguente:

		Internet (Client)

	![][2]

3. Copiare la stringa di connessione copiata in precedenza per l'app Mobile Engagement e incollarla nel file `Resources\EngagementConfiguration.xml`, tra i tag `<connectionString>` e `</connectionString>`:

	![][3]

	>[AZURE.TIP] Se l'app è destinata a entrambe le piattaforme Windows e Windows Phone, è comunque preferibile creare due applicazioni Mobile Engagement, una per ogni piattaforma supportata. Avere due app garantisce di poter definire la segmentazione corretta dei destinatari e di poter inviare notifiche opportunamente mirate per ogni piattaforma.

4. Nel file `App.xaml.cs`:

	a. Aggiungere l'istruzione `using`:

			using Microsoft.Azure.Engagement;

	b. Aggiungere un metodo che inizializza la soluzione di engagement:

           private void InitEngagement(IActivatedEventArgs e)
           {
             EngagementAgent.Instance.Init(e);

			 //... rest of the code
           }

    c. Inizializzare l'SDK nel metodo **OnLaunched**:

			protected override void OnLaunched(LaunchActivatedEventArgs e)
			{
			  InitEngagement(e);

			  //... rest of the code
			}

	c. Inserire il codice seguente nel metodo **OnActivated** e aggiungere il metodo, se non è già presente:

			protected override void OnActivated(IActivatedEventArgs e)
			{
			  InitEngagement(e);

			  //... rest of the code
			}

##<a id="monitor"></a>Abilitare il monitoraggio in tempo reale

Per iniziare a inviare dati e assicurarsi che gli utenti siano attivi, è necessario inviare almeno una schermata (Activity) al back-end di Mobile Engagement.

1. 	Nel file **MainPage.xaml.cs** aggiungere la istruzione `using` seguente:

		using Microsoft.Azure.Engagement.Overlay;

2. Modificare la classe base **MainPage** da **Page** a **EngagementPageOverlay**:

		class MainPage : EngagementPageOverlay

3. Nel file `MainPage.xaml`:

	a. Aggiungere le dichiarazioni di spazi dei nomi:

		xmlns:engagement="using:Microsoft.Azure.Engagement.Overlay"

	b. Sostituire **Page** nel nome del tag XML con **engagement:EngagementPageOverlay**.

> [AZURE.IMPORTANT] Se la pagina esegue l'override del metodo `OnNavigatedTo`, accertarsi di chiamare `base.OnNavigatedTo(e)`. In caso contrario, l'attività non viene segnalata. `EngagementPage` chiama `StartActivity` nel metodo `OnNavigatedTo`. Questo aspetto è particolarmente importante in un progetto Windows Phone in cui il modello predefinito ha un metodo `OnNavigatedTo`.

##<a id="monitor"></a>Connettere l'app con monitoraggio in tempo reale

[AZURE.INCLUDE [Connettere l'app con monitoraggio in tempo reale](../../includes/mobile-engagement-connect-app-with-monitor.md)]

##<a id="integrate-push"></a>Abilitare le notifiche push e la messaggistica in-app

Mobile Engagement consente di interagire con gli utenti e coinvolgerli tramite notifiche push e messaggistica in-app nel contesto di campagne. Questo modulo è denominato REACH nel portale di Mobile Engagement. Le sezioni seguenti consentono di configurare l'app per la ricezione.

###Abilitare l'app alla ricezione di notifiche push WNS

1. Nel file `Package.appxmanifest` scegliere la scheda **Applicazione** e impostare **Popup supportati** su **Sì** in **Notifiche**.

	![][5]

###Inizializzare REACH SDK

In `App.xaml.cs` chiamare **EngagementReach.Instance.Init(e);** nella funzione **InitEngagement** subito dopo l'inizializzazione dell'agente:

        private void InitEngagement(IActivatedEventArgs e)
		{
		   EngagementAgent.Instance.Init(e);
		   EngagementReach.Instance.Init(e);
		}

Ora è possibile inviare un avviso popup. Viene quindi verificato che l'integrazione di base sia stata eseguita correttamente.

###Concedere l'accesso a Mobile Engagement per inviare notifiche

1. Aprire [Dev Center per Windows Store] nel Web browser, accedere e creare un account, se necessario.
2. Fare clic su **Dashboard** nell'angolo superiore destro, quindi scegliere **Crea nuova applicazione** dal menu nel pannello sinistro.

	![][9]

2. Creare l'app riservandone il nome.

	![][10]

3. Dopo la creazione dell'app, passare a **Servizi -> Notifiche push** dal menu a sinistra.

	![][11]

4. Nella sezione Notifiche push fare clic sul collegamento del **sito dei servizi Live**.

	![][12]

5. Viene visualizzata la sezione relativa alle credenziali push. Assicurarsi di trovarsi nella sezione **Impostazioni app** e quindi copiare i valori di **SID pacchetto** e **Segreto client**.

	![][13]

6. Passare a **Impostazioni** del portale Mobile Engagement e fare clic sulla sezione **Push nativo** a sinistra. Fare quindi clic sul pulsante **Modifica** per immettere l'**ID di sicurezza (SID) del pacchetto** e la **Chiave privata**, come illustrato:

	![][6]

8. Assicurarsi infine di avere associato l'app di Visual Studio all'app creata nell'App Store. Fare clic su **Associa applicazione a Store** in Visual Studio. ![][7]

##<a id="send"></a>Inviare una notifica all'app

[AZURE.INCLUDE [Creare una campagna Push Windows](../../includes/mobile-engagement-windows-push-campaign.md)]

Se l'app è in esecuzione, viene visualizzata una notifica in-app. In caso contrario, se l'app è chiusa, viene visualizzata una notifica di tipo avviso popup. Se viene visualizzata una notifica in-app, ma non una notifica di tipo avviso popup e si esegue l'app in modalità debug in Visual Studio, provare a selezionare **Eventi ciclo di vita -> Sospendi** sulla barra degli strumenti per assicurarsi che l'app venga sospesa. Se è stato fatto clic sul pulsante Home durante il debug dell'applicazione in Visual Studio, non sempre viene sospesa e, anche se la notifica viene visualizzata come in-app, non compare come notifica di tipo avviso popup.

![][8]

<!-- URLs. -->
[Mobile Engagement Windows Universal SDK documentation]: ../mobile-engagement-windows-store-integrate-engagement/
[MicrosoftAzure.MobileEngagement]: http://go.microsoft.com/?linkid=9864592
[Dev Center per Windows Store]: https://dev.windows.com
[Windows Universal Apps - Overlay integration]: ../mobile-engagement-windows-store-integrate-engagement-reach/#overlay-integration

<!-- Images. -->
[1]: ./media/mobile-engagement-windows-store-dotnet-get-started/universal-app-creation.png
[2]: ./media/mobile-engagement-windows-store-dotnet-get-started/manifest-capabilities.png
[3]: ./media/mobile-engagement-windows-store-dotnet-get-started/add-connection-info.png
[5]: ./media/mobile-engagement-windows-store-dotnet-get-started/manifest-toast.png
[6]: ./media/mobile-engagement-windows-store-dotnet-get-started/enter-credentials.png
[7]: ./media/mobile-engagement-windows-store-dotnet-get-started/associate-app-store.png
[8]: ./media/mobile-engagement-windows-store-dotnet-get-started/vs-suspend.png
[9]: ./media/mobile-engagement-windows-store-dotnet-get-started/dashboard_create_app.png
[10]: ./media/mobile-engagement-windows-store-dotnet-get-started/dashboard_app_name.png
[11]: ./media/mobile-engagement-windows-store-dotnet-get-started/dashboard_services_push.png
[12]: ./media/mobile-engagement-windows-store-dotnet-get-started/dashboard_services_push_1.png
[13]: ./media/mobile-engagement-windows-store-dotnet-get-started/dashboard_services_push_creds.png

<!---HONumber=AcomDC_0817_2016-->
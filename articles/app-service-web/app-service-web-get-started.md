<properties 
	pageTitle="Distribuire la prima app Web in Azure in cinque minuti | Microsoft Azure" 
	description="Informazioni sulla facilità di esecuzione delle app Web nel servizio app mediante la distribuzione di un'app di esempio. È possibile eseguire rapidamente vere e proprie attività di sviluppo con risultati immediati." 
	services="app-service\web"
	documentationCenter=""
	authors="cephalin"
	manager="wpickett"
	editor=""
/>

<tags
	ms.service="app-service-web"
	ms.workload="web"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="hero-article"
	ms.date="09/09/2016" 
	ms.author="cephalin"
/>
	
# Distribuire la prima app Web in Azure in cinque minuti

Questa esercitazione illustra come distribuire la prima app Web nel [Servizio app di Azure](../app-service/app-service-value-prop-what-is.md). Il servizio app consente di creare app Web, [back-end di app per dispositivi mobili](/documentation/learning-paths/appservice-mobileapps/) e [app per le API](../app-service-api/app-service-api-apps-why-best-platform.md).

Si apprenderà come:

- Creare un'app Web nel servizio app di Azure.
- Distribuire codice di esempio scegliendo tra ASP.NET, PHP, Node.js, Java o Python.
- Vedere il codice in esecuzione nell'ambiente di produzione.
- Aggiornare l'app Web nello stesso modo in cui si effettua il [push dei commit Git](https://git-scm.com/docs/git-push).

## Prerequisiti

- [Installare Git](http://www.git-scm.com/downloads). Verificare l'esito positivo dell'installazione eseguendo `git --version` da un nuovo prompt dei comandi di Windows, una finestra di PowerShell, una shell di Linux o un terminale OS X.
- Ottenere un account Microsoft Azure. Se non è disponibile un account, è possibile [iscriversi per ottenere una versione di valutazione gratuita](/pricing/free-trial/?WT.mc_id=A261C142F) oppure [attivare i benefici della sottoscrizione di Visual Studio](/pricing/member-offers/msdn-benefits-details/?WT.mc_id=A261C142F).

>[AZURE.NOTE] È possibile [provare il servizio app](http://go.microsoft.com/fwlink/?LinkId=523751) senza avere un account Azure. Creare un'app iniziale e provarla per un'ora, senza impegno e senza dover usare la carta di credito.

<a name="create"></a>
## Creare un'app Web

1. Accedere al [portale di Azure](https://portal.azure.com) con il proprio account Azure.

2. Nel menu a sinistra fare clic su **Nuovo** > **Web e dispositivi mobili** > **App Web**.

    ![Iniziare a creare la prima app Web in Azure](./media/app-service-web-get-started/create-web-app-portal.png)

3. Nel pannello di creazione dell'app usare le impostazioni seguenti per la nuova app:

    - **Nome dell'app**: digitare un nome univoco.
    - **Gruppo di risorse**: selezionare **Crea nuovo** e assegnare un nome al gruppo di risorse.
    - **Piano di servizio app/Località**: fare clic per configurare e quindi scegliere **Crea nuovo** per impostare il nome, la località e il piano tariffario del piano di servizio app. È possibile usare il piano tariffario **gratuito**.

    Al termine, il pannello di creazione dell'app dovrebbe avere un aspetto simile al seguente:

    ![Configurare la prima app Web in Azure](./media/app-service-web-get-started/create-web-app-settings.png)

3. Fare clic su **Crea** nella parte inferiore della schermata. È possibile fare clic sull'icona delle **notifiche** in alto per visualizzare lo stato di avanzamento.

    ![Notifica sulla creazione della prima app Web in Azure](./media/app-service-web-get-started/create-web-app-started.png)

4. Al termine della distribuzione dovrebbe essere visualizzato il messaggio di notifica seguente. Fare clic sul messaggio per aprire il pannello della distribuzione.

    ![Messaggio di distribuzione terminata per la prima app Web in Azure](./media/app-service-web-get-started/create-web-app-finished.png)

5. Nel pannello **La distribuzione è riuscita** fare clic sul collegamento **Risorsa** per aprire il pannello della nuova app Web.

    ![Collegamento Risorsa per la prima app Web in Azure](./media/app-service-web-get-started/create-web-app-resource.png)

## Distribuire codice nell'app Web

L'esercitazione passa ora a illustrare come distribuire codice in Azure tramite Git.

5. Nel pannello dell'app Web scorrere verso il basso fino alla voce **Opzioni di distribuzione** o cercarla e fare clic su di essa.

    ![Opzioni di distribuzione per la prima app Web in Azure](./media/app-service-web-get-started/deploy-web-app-deployment-options.png)

6. Fare clic su **Scegliere l'origine** > **Repository Git locale** > **OK**.

7. Tornare al pannello dell'app Web e fare clic su **Credenziali di distribuzione**.

8. Impostare le credenziali di distribuzione e fare clic su **Salva**.

7. Tornare al pannello dell'app Web e scorrere verso il basso fino alla voce **Proprietà** o cercarla e fare clic su di essa. Fare clic sul pulsante **Copia** accanto a **URL GIT**.

    ![Pannello Proprietà per la prima app Web in Azure](./media/app-service-web-get-started/deploy-web-app-properties.png)

    A questo punto è possibile distribuire il codice con Git.

1. Nel terminale della riga di comando passare a una directory di lavoro, `CD`, e clonare l'app di esempio come segue:

        git clone <github_sample_url>

    ![Clonare il codice dell'app di esempio per la prima app Web in Azure](./media/app-service-web-get-started/html-git-clone.png)

    Per *&lt;github\_sample\_url>* usare uno degli URL seguenti, a seconda del framework preferito:

    - HTML+CSS+JS: [https://github.com/Azure-Samples/app-service-web-html-get-started.git](https://github.com/Azure-Samples/app-service-web-html-get-started.git)
    - ASP.NET: [https://github.com/Azure-Samples/app-service-web-dotnet-get-started.git](https://github.com/Azure-Samples/app-service-web-dotnet-get-started.git)
    - PHP (CodeIgniter): [https://github.com/Azure-Samples/app-service-web-php-get-started.git](https://github.com/Azure-Samples/app-service-web-php-get-started.git)
    - Node.js (Express): [https://github.com/Azure-Samples/app-service-web-nodejs-get-started.git](https://github.com/Azure-Samples/app-service-web-nodejs-get-started.git)
    - Java: [https://github.com/Azure-Samples/app-service-web-java-get-started.git](https://github.com/Azure-Samples/app-service-web-java-get-started.git)
    - Python (Django): [https://github.com/Azure-Samples/app-service-web-python-get-started.git](https://github.com/Azure-Samples/app-service-web-python-get-started.git)

2. Passare al repository dell'app di esempio. Ad esempio,

        cd app-service-web-html-get-started

3. Configurare il Git remoto per l'app Azure con l'URL GIT copiato dal portale in uno dei passaggi precedenti.

        git remote add azure <giturlfromportal>

4. Distribuire il codice di esempio nell'app Azure nello stesso modo in cui si effettua il push del codice con Git:

        git push azure master

    ![Eseguire il push di codice nella prima app Web in Azure](./media/app-service-web-get-started/html-git-push.png)

    Se è stato usato uno dei framework di linguaggio, verrà visualizzato un output diverso. Ciò dipende dal fatto che `git push` non solo inserisce il codice in Azure, ma attiva anche le attività di distribuzione nel motore di distribuzione. Se nella radice del progetto (repository) sono presenti file package.json (Node.js) o requirements.txt (Python) o se nel progetto ASP.NET è presente un file packages.config, lo script di distribuzione ripristina automaticamente i pacchetti necessari. È anche possibile [abilitare l'estensione Composer](web-sites-php-mysql-deploy-use-git.md#composer) per elaborare automaticamente i file composer.json nell'app PHP.

L'operazione è terminata. A questo punto il codice è in esecuzione in Azure. Nel browser passare a http://*&lt;appname>*.azurewebsites.net per vederlo in azione.

## Eseguire aggiornamenti dell'app

Ora è possibile usare Git per effettuare il push dalla radice del progetto (repository) in qualsiasi momento per poter eseguire un aggiornamento del sito attivo. La procedura è simile a quella usata per la prima distribuzione del codice. Ad esempio, quando si vuole effettuare il push di una nuova modifica testata in locale, è sufficiente eseguire i comandi seguenti dalla radice del progetto (repository):

    git add .
    git commit -m "<your_message>"
    git push azure master

## Passaggi successivi

Trovare la procedura di sviluppo e distribuzione più adatta al framework del linguaggio:

> [AZURE.SELECTOR]
- [.NET](web-sites-dotnet-get-started.md)
- [PHP](app-service-web-php-get-started.md)
- [Node.JS](app-service-web-nodejs-get-started.md)
- [Python](web-sites-python-ptvs-django-mysql.md)
- [Java](web-sites-java-get-started.md)

In alternativa è possibile fare ulteriori prove con la prima app Web, ad esempio:

- Provare [altri modi per distribuire il codice in Azure](../app-service-web/web-sites-deploy.md). Per eseguire la distribuzione da un repository GitHub, ad esempio, è sufficiente selezionare **GitHub** anziché **Repository Git locale** in **Opzioni di distribuzione**.
- Ottimizzare l'app di Azure. Autenticare gli utenti. Ridimensionarla in base alla richiesta. Configurare alcuni avvisi sulle prestazioni. Tutto con pochi clic. Vedere [Aggiungere funzionalità alla prima app Web](app-service-web-get-started-2.md).

<!---HONumber=AcomDC_0914_2016-->
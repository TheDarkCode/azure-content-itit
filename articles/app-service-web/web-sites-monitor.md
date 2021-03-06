<properties
	pageTitle="Eseguire il monitoraggio delle app nel servizio app di Azure"
	description="Informazioni su come monitorare le app nel servizio app di Azure tramite il portale di Azure."
	services="app-service"
	documentationCenter=""
	authors="btardif"
	manager="wpickett"
	editor="mollybos"/>

<tags
	ms.service="app-service"
	ms.workload="na"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="07/31/2016"
	ms.author="byvinyal"/>

# Procedura: Eseguire il monitoraggio delle app nel servizio app di Azure

Il [servizio app](http://go.microsoft.com/fwlink/?LinkId=529714) offre funzionalità di monitoraggio incorporate nel [portale di Azure](https://portal.azure.com). Consente, ad esempio, di esaminare **quote** e **metriche** di un'app e il relativo piano di servizio app, configurare **avvisi** e impostare la **scalabilità** automatica in base a tali metriche.

[AZURE.INCLUDE [app-service-web-to-api-and-mobile](../../includes/app-service-web-to-api-and-mobile.md)]

## Informazioni su quote e metriche

### Quote

Le applicazioni ospitate nel servizio app di Azure sono soggette a determinati *limiti* in merito alle risorse che possono usare. I limiti sono definiti dal **piano di servizio app** associato all'app.

Se l'applicazione è ospitata in un piano **Gratuito** o **Condiviso**, i limiti sulle risorse che l'app può usare sono definiti dalle **quote**.

Se l'applicazione è ospitata in un piano **Basic**, **Standard** o **Premium**, i limiti sulle risorse che può usare sono definiti dalle **dimensioni** (Small, Medium, Large) e dal **numero di istanze** (1, 2, 3...) del **piano di servizio app**.

Le **quote** per le app ospitate nel piano **Gratuito** o **Condiviso** sono:

* **CPU (breve)**
   * Quantità di CPU consentita per l'applicazione in un intervallo di 3 minuti. Questa quota viene reimpostata automaticamente ogni 3 minuti.
* **CPU (giorno)**
   * Quantità totale di CPU consentita per l'applicazione in un giorno. Questa quota viene reimpostata automaticamente ogni 24 ore a mezzanotte (ora UTC).
* **Memoria**
   * Quantità totale di memoria consentita per l'applicazione.
* **Larghezza di banda**
   * Quantità totale di larghezza di banda in uscita consentita per l'applicazione in un giorno. Questa quota viene reimpostata automaticamente ogni 24 ore a mezzanotte (ora UTC).
* **File system**
   * Quantità totale di spazio di archiviazione consentita.
   
L'unica quota applicabile alle app ospitate nei piani **Basic**, **Standard** e **Premium** è **File system**.

Per altre informazioni su quote, funzionalità e limiti specifici per i diversi SKU del piano di servizio app, vedere i [limiti del servizio della sottoscrizione di Azure](../azure-subscription-service-limits.md#app-service-limits)

#### Imposizione della quota

Se l'utilizzo di un'applicazione supera le quote per **CPU (breve)**, **CPU (giorno)** o **larghezza di banda**, l'applicazione verrà arrestata fino al momento in cui viene reimpostata la quota. Durante questo periodo, per tutte le richieste in ingresso verrà restituito un errore di tipo **HTTP 403**. ![][http403]

Se viene superata la quota di **memoria**, l'applicazione viene riavviata.

Se viene superata la quota per il **file system**, tutte le operazioni di scrittura avranno esito negativo, incluse le operazioni di scrittura nei log.

È possibile aumentare o rimuovere le quote dall'app aggiornando il piano di servizio app.

### Metrica

Le **Metriche** forniscono informazioni sull'app o sul comportamento del piano di servizio app.

Per un'**applicazione**, le metriche disponibili sono:

* **Tempo medio di risposta**
   * Tempo medio impiegato dall'app per gestire le richieste, in millisecondi.
* **Working set della memoria medio**
   * Quantità media di memoria usata dall'app, in MiB.
* **Tempo CPU**
   * Quantità di CPU utilizzata dall'app, in secondi. Per altre informazioni su questa metrica, vedere [Tempo CPU e percentuale CPU](#cpu-time-vs-cpu-percentage)
* **Dati in entrata**
   * Quantità di larghezza di banda in entrata utilizzata dall'app, in MiB.
* **Dati in uscita**
   * Quantità di larghezza di banda in uscita utilizzata dall'app, in MiB.
* **Http 2xx**
   * Numero di richieste che hanno restituito un codice di stato HTTP >= 200 e < 300.
* **Http 3xx**
   * Numero di richieste che hanno restituito un codice di stato HTTP >= 300 e < 400.
* **Http 401**
   * Numero di richieste che hanno restituito un codice di stato HTTP 401.
* **Http 403**
   * Numero di richieste che hanno restituito un codice di stato HTTP 403.
* **Http 404**
   * Numero di richieste che hanno restituito un codice di stato HTTP 404.
* **Http 406**
   * Numero di richieste che hanno restituito un codice di stato HTTP 406.
* **Http 4xx**
   * Numero di richieste che hanno restituito un codice di stato HTTP >= 400 e < 500.
* **Errori server HTTP**
   * Numero di richieste che hanno restituito un codice di stato HTTP >= 500 e < 600.
* **Working set della memoria**
   * Quantità di memoria corrente usata dall'app, in MiB.
* **Richieste**
   * Numero totale di richieste, indipendentemente dal codice di stato HTTP restituito.

Per un **piano di servizio app**, le metriche disponibili sono:

>[AZURE.NOTE] Le metriche del piano di servizio app sono disponibili solo per i piani nello SKU **Basic**, **Standard** e **Premium**.

* **Percentuale di CPU**
   * CPU media usata tra tutte le istanze del piano.
* **Percentuale memoria**
   * Memoria media usata tra tutte le istanze del piano.
* **Dati in entrata**
   * Larghezza di banda in ingresso media usata tra tutte le istanze del piano.
* **Dati in uscita**
   * Larghezza di banda in uscita media usata tra tutte le istanze del piano.
* **Lunghezza coda disco**
   * Numero medio di richieste di lettura e scrittura accodate nella risorsa di archiviazione. Una lunghezza coda disco elevata può indicare il rallentamento di un'applicazione a causa di operazioni di I/O su disco eccessive.
* **Lunghezza coda HTTP**
   * Numero medio di richieste HTTP che hanno dovuto attendere in coda prima di essere completate. Una lunghezza coda HTTP elevata o in aumento indica che il piano si trova in condizioni di carico eccessivo.

### Tempo CPU e percentuale CPU
<!-- To do: Fix Anchor (#CPU-time-vs.-CPU-percentage) -->

Le metriche che riflettono l'utilizzo della CPU sono due. **Tempo CPU** e **Percentuale CPU**

**Tempo CPU** è utile per le app ospitate nei piani **Gratuito** o **Condiviso**, dal momento che una delle relative quote è definita in minuti di CPU usati dall'app.

**Percentuale CPU** è invece utile per le app ospitate nei piani **Basic**, **Standard** e **Premium** dal momento che è possibile aumentarne il numero di istanze. Questa metrica è un ottimo indicatore dell'utilizzo complessivo tra tutte le istanze.

##Granularità delle metriche e criteri di conservazione

Le metriche per un'applicazione e un piano di servizio app vengono registrate e aggregate dal servizio con le granularità e i criteri di conservazione seguenti:

 * Le metriche di granularità **minuto** vengono mantenute per **24 ore**
 * Le metriche di granularità **ora** vengono mantenute per **7 giorni**
 * Le metriche di granularità **giorno** vengono mantenute per **30 giorni**

## Monitoraggio di quote e metriche nel portale di Azure

È possibile esaminare lo stato delle diverse **quote** e **metriche** che interessano un'applicazione nel [portale di Azure](https://portal.azure.com).

![][quotas] Le **quote** sono disponibili in Impostazioni>**Quote**. L'interfaccia utente consente di esaminare: (1) nomi delle quote, (2) intervallo di reimpostazione, (3) limiti correnti e (4) valore corrente.

![][metrics] È invece possibile accedere alle **metriche** direttamente dal pannello della risorsa. Per personalizzare il grafico: (1) fare **clic** su di esso e selezionare (2) **Modifica grafico**. Da qui è possibile modificare le opzioni relative a (3) **Intervallo di tempo**, (4) **Tipo di grafico** e le (5) **metriche** da visualizzare.

Per altre informazioni sulle metriche, vedere [Monitorare le metriche del servizio](../azure-portal/insights-how-to-customize-monitoring.md).

## Avvisi e scalabilità automatica
È possibile collegare le metriche per un'app o un piano di servizio app agli avvisi. Per altre informazioni, vedere [Ricevere notifiche di avviso](../azure-portal/insights-receive-alert-notifications.md)

Le app del servizio app ospitate nei piani di servizio app Basic, Standard e Premium supportano la **scalabilità automatica**. È quindi possibile configurare regole per il monitoraggio delle metriche del piano di servizio app e aumentare o ridurre automaticamente il numero di istanze per fornire risorse aggiuntive quando necessario o per risparmiare denaro quando il provisioning dell'applicazione è eccessivo rispetto all'utilizzo effettivo. Per altre informazioni sulla scalabilità automatica, vedere l'articolo che illustra [come ridimensionare il numero di istanze manualmente o automaticamente](../azure-portal/insights-how-to-scale.md) e [Procedure consigliate per la scalabilità automatica in Azure Insights](../azure-portal/insights-autoscale-best-practices.md)

>[AZURE.NOTE] Per iniziare a usare il servizio app di Azure prima di registrarsi per ottenere un account Azure, andare a [Prova il servizio app](http://go.microsoft.com/fwlink/?LinkId=523751), dove è possibile creare un'app Web iniziale temporanea nel servizio app. Non è necessario fornire una carta di credito né impegnarsi in alcun modo.

## Modifiche apportate
* Per una guida relativa al passaggio da Siti Web al servizio app, vedere [Servizio app di Azure e impatto sui servizi di Azure esistenti](http://go.microsoft.com/fwlink/?LinkId=529714)

[fzilla]: http://go.microsoft.com/fwlink/?LinkId=247914
[vmsizes]: http://go.microsoft.com/fwlink/?LinkID=309169



<!-- Images. -->
[http403]: ./media/web-sites-monitor/http403.png
[quotas]: ./media/web-sites-monitor/quotas.png
[metrics]: ./media/web-sites-monitor/metrics.png

<!---HONumber=AcomDC_0803_2016-->
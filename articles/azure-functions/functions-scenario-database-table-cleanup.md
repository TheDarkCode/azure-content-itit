<properties
   pageTitle="Usare Funzioni di Azure per eseguire un'attività di pulizia pianificata | Microsoft Azure"
   description="Usare Funzioni di Azure per creare una funzione C# che viene eseguita in base a un timer di eventi."
   services="azure-functions"
   documentationCenter="na"
   authors="ggailey777"
   manager="erikre"
   editor=""
   tags=""
   />

<tags
   ms.service="functions"
   ms.devlang="multiple"
   ms.topic="article"
   ms.tgt_pltfrm="multiple"
   ms.workload="na"
   ms.date="06/05/2016"
   ms.author="glenga"/>
   
# Usare Funzioni di Azure per eseguire un'attività di pulizia pianificata

Questo argomento mostra come usare le funzioni di Azure per creare una nuova funzione in C# eseguita in base a un timer eventi per la pulizia delle righe in una tabella di database. La nuova funzione viene creata in base a un modello predefinito nel portale di Funzioni di Azure. Per supportare questo scenario, è necessario anche impostare una stringa di connessione di database come impostazione di servizio app nell'app per le funzioni.

## Prerequisiti 

Prima di creare una funzione, è necessario avere un account Azure attivo. Se non si possiede un account di Azure, [sono disponibili account gratuiti](https://azure.microsoft.com/free/).

Questo argomento illustra un comando Transact-SQL che esegue un'operazione di pulizia in blocco nella tabella denominata *TodoItems* di un database SQL. Questa stessa tabella TodoItems viene creata eseguendo l'[esercitazione rapida sull'app per dispositivi mobili del servizio app di Azure](../app-service-mobile/app-service-mobile-ios-get-started.md). È possibile anche usare un database di esempio. Se si sceglie di usare una tabella diversa, sarà necessario modificare il comando.

È possibile ottenere la stringa di connessione utilizzata da un back-end per app mobili dal portale, in **Tutte le impostazioni** > **Impostazioni applicazione** > **Stringhe di connessione** > **Mostra i valori delle stringhe di connessione** > **MS\_TableConnectionString**. È possibile anche ottenere la stringa di connessione direttamente da un database SQL nel portale, in **Tutte le impostazioni** > **Proprietà** > **Show database connection strings **(Mostra stringhe di connessione del database) > **ADO.NET (autenticazione di SQL)**.

Questo scenario esegue un'operazione in blocco sul database. Affinché la funzione elabori le singole operazioni CRUD in una tabella di un'app per dispositivi mobili, è necessario usare invece l'associazione mobile table.

## Impostare una stringa di connessione di database SQL nell'app per le funzioni

Un'app per le funzioni ospita l'esecuzione delle funzioni in Azure. È consigliabile archiviare le stringhe di connessione e altre informazioni riservate nelle impostazioni dell'app per le funzioni. Ciò ne impedisce la diffusione accidentale quando il codice della funzione finisce in qualche repository.

1. Passare al [portale di Funzioni di Azure](https://functions.azure.com/signin) e accedere con l'account Azure.

2. Se si dispone di un'app per le funzioni esistente da usare, selezionarla da **App per le funzioni personali** e fare clic su **Apri**. Per creare una nuova app per le funzioni, digitare un **nome** univoco da assegnare o accettare quello generato automaticamente, selezionare l'**area** desiderata, quindi fare clic su **Crea + Attività iniziali**.

3. Nell'app per le funzioni fare clic su **Function app settings** (Impostazioni dell'app per le funzioni) > **Go to App Service settings** (Vai a Impostazioni del servizio app).

	![Pannello Impostazioni dell'app per le funzioni](./media/functions-create-an-event-processing-function/functions-app-service-settings.png)

4. Nell'app per le funzioni fare clic su **All settings** (Tutte le impostazioni), scorrere verso il basso fino a **Application settings** (Impostazioni applicazione), quindi in **Connection strings** (Stringhe di connessione) digitare `sqldb_connection` per **Name** (Nome), incollare la stringa di connessione nel campo **Value** (Valore), quindi fare clic su **Salva** e chiudere il pannello dell'app per le funzioni per tornare al portale Funzioni.

    ![Stringa di connessione per l'impostazione del servizio app](./media/functions-create-an-event-processing-function/functions-app-service-settings-connection-strings.png)

A questo punto, è possibile aggiungere il codice della funzione C# che si connette al database SQL.

## Creare una funzione attivata tramite timer a partire dal modello

1. Nell'app per le funzioni fare clic su **+ Nuova funzione** > **TimerTrigger - Nodo** > **Crea**. Verrà creata una funzione con un nome predefinito, che viene eseguita in base alla pianificazione predefinita, ovvero una volta al minuto. 

	![Creare una nuova funzione attivata tramite timer](./media/functions-create-an-event-processing-function/functions-create-new-timer-trigger.png)

2. Nel riquadro **Codice** della scheda **Sviluppo** aggiungere i riferimenti di assembly seguenti all'inizio del codice della funzione esistente:

		#r "System.Configuration"
		#r "System.Data"

3. Aggiungere le istruzioni `using` seguenti alla funzione:

		using System.Configuration;
		using System.Data.SqlClient;
		using System.Threading.Tasks; 

4. Sostituire la funzione **Run** esistente con il codice seguente:

		public static async Task Run(TimerInfo myTimer, TraceWriter log)
		{
		    var str = ConfigurationManager.ConnectionStrings["sqldb_connection"].ConnectionString;
		    using (SqlConnection conn = new SqlConnection(str))
		    {
		        conn.Open();
		        var text = "DELETE from dbo.TodoItems WHERE Complete='True'";
		        using (SqlCommand cmd = new SqlCommand(text, conn))
		        {
					// Execute the command and log the # rows deleted.
		            var rows = await cmd.ExecuteNonQueryAsync();
		            log.Info($"{rows} rows were deleted");
		        }
		    }
		}

5. Fare clic su **Salva**, osservare le finestre **Log** per la successiva esecuzione di funzione, quindi prendere nota del numero di righe eliminate dalla tabella TodoItems.

6. (Facoltativo) Utilizzando l'[app di guida introduttiva dell'App per dispositivi mobili](../app-service-mobile/app-service-mobile-ios-get-started.md), contrassegnare elementi aggiuntivi, come "completati", quindi tornare alle finestre **Log** e osservare che lo stesso numero di righe vengono eliminate dalla funzione durante l'esecuzione successiva.

##Passaggi successivi

Vedere gli argomenti seguenti per altre informazioni su Funzioni di Azure.

+ [Guida di riferimento per gli sviluppatori di Funzioni di Azure](functions-reference.md) Informazioni di riferimento per programmatori in merito alla codifica delle funzioni e alla definizione di trigger e associazioni.
+ [Test di Funzioni di Azure](functions-test-a-function.md) Descrive diversi strumenti e tecniche per il test delle funzioni.
+ [Come aumentare le prestazioni di Funzioni di Azure](functions-scale.md) Presenta i piani di servizio disponibili con Funzioni di Azure, tra cui il piano di servizio dinamico, e come scegliere quello più appropriato.  

[AZURE.INCLUDE [Nota introduttiva](../../includes/functions-get-help.md)]

<!---HONumber=AcomDC_0615_2016-->
<properties
	pageTitle="Creare report in database cloud con numero maggiore di istanze (partizionamento orizzontale) | Microsoft Azure"
	description="come utilizzare tra le query di database tra database"
	services="sql-database"
	documentationCenter=""  
	manager="jhubbard"
	authors="SilviaDoomra"/>

<tags
	ms.service="sql-database"
	ms.workload="sql-database"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="05/23/2016"
	ms.author="SilviaDoomra" />

# Creare report in database cloud con numero maggiore di istanze (anteprima)

È possibile creare report da più database SQL di Azure da un unico punto di connessione usando una [query elastica](sql-database-elastic-query-overview.md). Il database deve essere con partizionamento orizzontale.

Se si ha un database esistente, vedere [Migrazione dei database esistenti in database con un numero maggiore di istanze](sql-database-elastic-convert-to-use-elastic-tools.md).

Per comprendere quali sono gli oggetti SQL necessari per eseguire una query, vedere [Eseguire query in database con partizionamento orizzontale](sql-database-elastic-query-horizontal-partitioning.md).

## Prerequisiti

Scaricare ed eseguire [Introduzione allo strumento di esempio del Database elastico](sql-database-elastic-scale-get-started.md).

## Creare un gestore mappe partizione utilizzando l'applicazione di esempio

Di seguito si creerà un gestore mappe partizione con diverse partizioni, seguita dall'inserimento di dati nelle partizioni. Se si dispone già di programma di installazione di partizioni con dati partizionati in essi, è possibile ignorare i passaggi seguenti e passare alla sezione successiva.

1. Compilare ed eseguire l’applicazione di esempio **Introduzione agli strumenti del Database elastico**. Seguire i passaggi fino al passaggio 7 nella sezione [Scaricare ed eseguire l'app di esempio](sql-database-elastic-scale-get-started.md#Getting-started-with-elastic-database-tools). Alla fine del passaggio 7, verrà visualizzato il seguente prompt dei comandi:

	![Aprire il prompt dei comandi.][1]

2.  Nella finestra di comando, digitare "1" e premere **Invio**. Viene creato il gestore delle mappe partizioni e aggiunge due partizioni al server. Digitare "3" e premere **Invio**; ripetere l'azione quattro volte. Consente di inserire righe di dati di esempio nelle partizioni.
3.  Nel [portale di Azure](https://portal.azure.com) dovrebbero essere visualizzati tre nuovi database nel server v12:

	![Conferma di Visual Studio][2]

	A questo punto, le query tra database sono supportate tramite le risorse del client del Database elastico. Nella finestra di comando, ad esempio, utilizzare l'opzione 4. I risultati di una query con più partizioni sono sempre un **UNION ALL** dei risultati di tutte le partizioni.

	Nella sezione successiva, verrà creato un endpoint del database di esempio che supporta l'esecuzione più completa delle query dei dati tra partizioni.

## Creare un database di query elastico

1. Aprire il [portale di Azure](https://portal.azure.com) ed effettuare l’accesso.
2. Creare un nuovo database SQL Azure nello stesso server del programma di installazione del partizionamento. Denominare il database "ElasticDBQuery".

	![Portale di Azure e il livello di prezzo][3]

	Nota: è possibile usare un database esistente. Se è possibile farlo, non deve essere una delle partizioni su cui si desidera eseguire la query. Questo database verrà utilizzato per la creazione di oggetti di metadati per una query di database elastico.


## Creare oggetti di database

### Chiave master con ambito database e credenziali

Questi vengono utilizzati per la connessione per la gestione di gestore di mappe di partizioni e partizioni:

1. Apri SQL Server Management Studio e SQL Server Data Tools in Visual Studio
2. Connettersi al database ElasticDBQuery ed eseguire i comandi T-SQL seguenti:

		CREATE MASTER KEY ENCRYPTION BY PASSWORD = '<password>';

		CREATE DATABASE SCOPED CREDENTIAL ElasticDBQueryCred
		WITH IDENTITY = '<username>',
		SECRET = '<password>';

	"username" e "password" devono essere uguali alle informazioni di accesso utilizzate nel passaggio 6 della [Scaricare ed eseguire l'applicazione di esempio](sql-database-elastic-scale-get-started.md#Getting-started-with-elastic-database-tools) in [Introduzione agli strumenti del database elastico](sql-database-elastic-scale-get-started.md).

### DROP EXTERNAL DATA SOURCE

Per creare un'origine dati esterna, eseguire il comando seguente sul database ElasticDBQuery:

	CREATE EXTERNAL DATA SOURCE MyElasticDBQueryDataSrc WITH
      (TYPE = SHARD_MAP_MANAGER,
      LOCATION = '<server_name>.database.windows.net',
      DATABASE_NAME = 'ElasticScaleStarterKit_ShardMapManagerDb',
	  CREDENTIAL = ElasticDBQueryCred,
 	  SHARD_MAP_NAME = 'CustomerIDShardMap'
    ) ;

 "CustomerIDShardMap" è il nome della mappa del partizionamento, se è stato creato il mapping della partizione e la gestione di mapping di partizione utilizzando l'esempio di strumenti di database flessibile. Tuttavia, se si utilizza il programma di installazione personalizzato per questo esempio, deve essere il nome di mappa partizionamento che scelto nell'applicazione.

### Tabelle esterne

Creare una tabella esterna corrispondente della tabella Customers nelle partizioni eseguendo il comando seguente sul database ElasticDBQuery:

	CREATE EXTERNAL TABLE [dbo].[Customers]
	( [CustomerId] [int] NOT NULL,
	  [Name] [nvarchar](256) NOT NULL,
	  [RegionId] [int] NOT NULL)
	WITH
	( DATA_SOURCE = MyElasticDBQueryDataSrc,
      DISTRIBUTION = SHARDED([CustomerId])
	) ;

## Eseguire una query di esempio elastica database T-SQL

Dopo aver definito l'origine dati esterna e le tabelle esterne è ora possibile utilizzare T-SQL completa tramite le tabelle esterne.

Eseguire questa query sul database ElasticDBQuery:

	select count(CustomerId) from [dbo].[Customers]

Si noterà che la query di aggregare i risultati di tutte le partizioni e produce il seguente output:

![Dettagli dell'output][4]

## Importare i risultati della query database elastica in Excel

 È possibile importare i risultati da di una query a un file di Excel.

1. Avviare Excel 2013.
2. 	Individuare il **dati** della barra multifunzione.
3. 	Fare clic su**Da altre fonti**quindi**Da SQL Server**.

	![Importazione di Excel da altre origini][5]
4. 	In **Connessione guidata dati** digitare le credenziali di accesso e il nome del server. Quindi fare clic su **Avanti**.
5. 	Nella finestra di dialogo **Selezionare il database che contiene i dati che si desidera**, selezionare il database **ElasticDBQuery**.
6. 	Selezionare la tabella **Clienti** nella visualizzazione elenco e fare clic su **Avanti**. Fare clic su **Fine**.
7. 	Nel modulo **Importazione dati** sotto **Selezionare come si desidera visualizzare i dati nella cartella di lavoro**, selezionare **Tabella** e fare clic su **OK**.

Tutte le righe dalla tabella **Clienti**, archiviate in diverse partizioni sono riportate nel foglio Excel.

## Passaggi successivi
È ora possibile utilizzare funzioni di visualizzazione avanzata dei dati di Excel. È possibile utilizzare la stringa di connessione con il nome del server, nome del database e credenziali per gli strumenti di integrazione di Business Intelligence e i dati di connettersi al database query elastica. Assicurarsi che SQL Server sia supportato come origine dati per lo strumento. È possibile fare riferimento al database elastica query e tabelle esterne come qualsiasi altro database di SQL Server e tabelle di SQL Server è necessario connettersi allo strumento.

### Costi
Non esiste senza alcun costo aggiuntivo per utilizzare la funzione elastica Query del Database.

Per informazioni sui prezzi, vedere [Dettagli prezzi del database SQL](https://azure.microsoft.com/pricing/details/sql-database/).


[AZURE.INCLUDE [elastic-scale-include](../../includes/elastic-scale-include.md)]

<!--Image references-->
[1]: ./media/sql-database-elastic-query-getting-started/cmd-prompt.png
[2]: ./media/sql-database-elastic-query-getting-started/portal.png
[3]: ./media/sql-database-elastic-query-getting-started/tiers.png
[4]: ./media/sql-database-elastic-query-getting-started/details.png
[5]: ./media/sql-database-elastic-query-getting-started/exel-sources.png
<!--anchors-->

<!---HONumber=AcomDC_0601_2016-->
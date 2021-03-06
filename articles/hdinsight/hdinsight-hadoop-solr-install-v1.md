<properties 
	pageTitle="Usare l'azione script per installare Solr in cluster Hadoop| Microsoft Azure" 
	description="Informazioni su come personalizzare un cluster HDInsight con Solr. Verrà usata un'opzione di configurazione Azione script per installare Solr tramite uno script." 
	services="hdinsight" 
	documentationCenter="" 
	authors="nitinme" 
	manager="paulettm" 
	editor="cgronlun"/>

<tags 
	ms.service="hdinsight" 
	ms.workload="big-data" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="05/17/2016" 
	ms.author="nitinme"/>

# Installare e usare Solr nei cluster Hadoop di HDInsight


Informazioni su come personalizzare i cluster HDInsight basati su Windows con Solr usando Script azione e su come utilizzare Solr per cercare i dati. Per informazioni sull'uso di Solr con un cluster basato su Linux, vedere [Installare e usare Solr nei cluster Hadoop HDInsight (Linux)](hdinsight-hadoop-solr-install-linux.md).
 
È possibile installare Solr in qualsiasi tipo di cluster (Hadoop, Storm, HBase, Spark) su Azure HDInsight usando *Script azione*. Uno script di esempio per l'installazione di Solr in un cluster HDInsight è disponibile in un BLOB di archiviazione di Azure di sola lettura all'indirizzo [https://hdiconfigactions.blob.core.windows.net/solrconfigactionv01/solr-installer-v01.ps1](https://hdiconfigactions.blob.core.windows.net/solrconfigactionv01/solr-installer-v01.ps1).

Lo script di esempio funziona solo con cluster HDInsight versione 3.1. Per altre informazioni sulle versioni dei cluster HDInsight, vedere [Versioni cluster HDInsight](hdinsight-component-versioning.md).

Lo script di esempio usato in questo argomento crea un cluster Solr basato su Windows con una configurazione specifica. Per configurare il cluster Solr con raccolte, partizioni, schemi, repliche diverse e così via, sarà necessario modificare di conseguenza lo script e i file binari di Solr.

**Articoli correlati**

- [Installare Solr nei cluster HDInsight](hdinsight-hadoop-solr-install.md): installare Solr nel cluster HDInsight tramite il portale di Azure
- [Installare e usare Solr nei cluster Hadoop di HDInsight (Linux)](hdinsight-hadoop-solr-install-linux.md)
- [Creare cluster Hadoop in HDInsight](hdinsight-provision-clusters.md): informazioni generali sulla creazione di cluster HDInsight
- [Personalizzare cluster HDInsight mediante Script azione][hdinsight-cluster-customize]\: informazioni generali sulla personalizzazione di cluster HDInsight tramite Script azione
- [Sviluppare script di Azione script per HDInsight](hdinsight-hadoop-script-actions.md)

## Che cos'è Solr?

[Apache Solr](http://lucene.apache.org/solr/features.html) è una piattaforma di ricerca aziendale che permette di eseguire ricerche full-text avanzate sui dati. Mentre Hadoop consente di archiviare e gestire quantità elevate di dati, Apache Solr offre le funzionalità di ricerca necessarie per recuperare rapidamente i dati.

## Installare Solr utilizzando il portale

[AZURE.INCLUDE [hdinsight-azure-portal](../../includes/hdinsight-azure-portal.md)]

* [Installare Solr in cluster HDInsight](hdinsight-hadoop-solr-install.md)

1. Avviare la creazione di un cluster tramite l'opzione **CREAZIONE PERSONALIZZATA**, come descritto in [Creare cluster Hadoop in HDInsight](hdinsight-provision-clusters.md#portal). 
2. Nella pagina **Azioni script** della procedura guidata fare clic su **aggiungi azione script** per specificare i dettagli relativi all'azione script, come descritto di seguito:

	![Usare l'azione script per personalizzare un cluster](./media/hdinsight-hadoop-solr-install-v1/hdi-script-action-solr.png "Usare l'azione script per personalizzare un cluster")
	
	<table border='1'>
		<tr><th>Proprietà</th><th>Valore</th></tr>
		<tr><td>Nome</td>
			<td>Specificare un nome per l'azione script. Ad esempio, <b>Install Solr</b>.</td></tr>
		<tr><td>URI script</td>
			<td>Specificare l'URI (Uniform Resource Identifier) dello script da richiamare per personalizzare il cluster. Ad esempio, <i>https://hdiconfigactions.blob.core.windows.net/solrconfigactionv01/solr-installer-v01.ps1</i></td></tr>
		<tr><td>Tipo di nodo</td>
			<td>Specificare i nodi in cui viene eseguito lo script di personalizzazione. È possibile scegliere <b>Tutti i nodi</b>, <b>Solo nodi head</b> o <b>Solo nodi di lavoro</b>.
		<tr><td>Parametri</td>
			<td>Specificare i parametri, se richiesti dallo script. Lo script per installare Solr non richiede alcun parametro, di conseguenza è possibile lasciare vuoto questo campo.</td></tr>
	</table>	

	È possibile aggiungere altre azioni script per installare più componenti nel cluster. Dopo aver aggiunto gli script, fare clic sul segno di spunta per avviare la creazione del cluster.

È inoltre possibile usare lo script per installare Solr in HDInsight usando Azure PowerShell o HDInsight .NET SDK. Le istruzioni relative a queste procedure vengono fornite più avanti in questo argomento.

## Utilizzare Solr

È prima di tutto necessario indicizzare Solr con alcuni file di dati. Sarà quindi possibile usare Solr per eseguire query di ricerca sui dati indicizzati. Eseguire la procedura seguente per usare Solr in un cluster HDInsight:

1. **Usare RDP (Remote Desktop Protocol) per accedere in remoto al cluster HDInsight con Solr installato**. Dal portale di Azure abilitare il desktop remoto per il cluster creato con Solr installato, quindi accedere in remoto al cluster. Per istruzioni, vedere [Connettersi a cluster HDInsight tramite RDP](hdinsight-administer-use-management-portal.md#rdp).

2. **Indicizzare Solr mediante il caricamento di file di dati**. Quando si indicizza Solr, si inseriscono documenti su cui potrebbe essere necessario eseguire ricerche. Per indicizzare Solr, usare RDP per accedere in remoto al cluster, passare al desktop, aprire la riga di comando di Hadoop e quindi passare a **C:\\apps\\dist\\solr-4.7.2\\example\\exampledocs**. Eseguire il comando seguente:
	
		java -jar post.jar solr.xml monitor.xml

	Nella console viene visualizzato il seguente risultato:

		POSTing file solr.xml
		POSTing file monitor.xml
		2 files indexed.
		COMMITting Solr index changes to http://localhost:8983/solr/update..
		Time spent: 0:00:01.624

	L'utilità post.jar indicizza Solr con due documenti di esempio, **solr.xml** e **monitor.xml**. L'utilità post.jar e i documenti di esempio sono disponibili con l'installazione di Solr.

3. **Usare il dashboard di Solr per eseguire ricerche nei documenti indicizzati**. Nella sessione RDP per il cluster HDInsight, aprire Internet Explorer, quindi avviare il dashboard di Solr all'indirizzo **http://headnodehost:8983/solr/#/**. Nel riquadro a sinistra selezionare **collection1** dal menu a discesa **Core Selector** e quindi fare clic su **Query**. Ad esempio, per selezionare e restituire tutti i documenti in Solr, specificare i valori seguenti:
	1. Nella casella di testo **q** immettere ***:***. Verranno restituiti tutti i documenti indicizzati in Solr. Per cercare una stringa specifica nei documenti, è possibile immettere qui la stringa.
	2. Selezionare il formato di output nella casella di testo **wt**. Il valore predefinito è **json**. Fare clic su **Esegui query**.

		![Usare l'azione script per personalizzare un cluster](./media/hdinsight-hadoop-solr-install-v1/hdi-solr-dashboard-query.png "Eseguire una query sul dashboard Solr")
	
	L'output restituisce i due documenti usati per l'indicizzazione di Solr. L'output sarà simile al seguente:

			"response": {
			    "numFound": 2,
			    "start": 0,
			    "maxScore": 1,
			    "docs": [
			      {
			        "id": "SOLR1000",
			        "name": "Solr, the Enterprise Search Server",
			        "manu": "Apache Software Foundation",
			        "cat": [
			          "software",
			          "search"
			        ],
			        "features": [
			          "Advanced Full-Text Search Capabilities using Lucene",
			          "Optimized for High Volume Web Traffic",
			          "Standards Based Open Interfaces - XML and HTTP",
			          "Comprehensive HTML Administration Interfaces",
			          "Scalability - Efficient Replication to other Solr Search Servers",
			          "Flexible and Adaptable with XML configuration and Schema",
			          "Good unicode support: héllo (hello with an accent over the e)"
			        ],
			        "price": 0,
			        "price_c": "0,USD",
			        "popularity": 10,
			        "inStock": true,
			        "incubationdate_dt": "2006-01-17T00:00:00Z",
			        "_version_": 1486960636996878300
			      },
			      {
			        "id": "3007WFP",
			        "name": "Dell Widescreen UltraSharp 3007WFP",
			        "manu": "Dell, Inc.",
			        "manu_id_s": "dell",
			        "cat": [
			          "electronics and computer1"
			        ],
			        "features": [
			          "30" TFT active matrix LCD, 2560 x 1600, .25mm dot pitch, 700:1 contrast"
			        ],
			        "includes": "USB cable",
			        "weight": 401.6,
			        "price": 2199,
			        "price_c": "2199,USD",
			        "popularity": 6,
			        "inStock": true,
			        "store": "43.17614,-90.57341",
			        "_version_": 1486960637584081000
			      }
			    ]
			  }
   

4. **Consigliato: backup dei dati indicizzati da Solr all'archivio BLOB di Azure associato al cluster HDInsight**. È consigliabile eseguire il backup dei dati indicizzati dai nodi del cluster Solr nell'archivio BLOB di Azure. Eseguire quindi la procedura seguente:

	1. Aprire Internet Explorer dalla sessione RDP, quindi selezionare l'URL seguente:

			http://localhost:8983/solr/replication?command=backup

		Viene visualizzata una risposta simile alla seguente:

			<?xml version="1.0" encoding="UTF-8"?>
			<response>
			  <lst name="responseHeader">
			    <int name="status">0</int>
			    <int name="QTime">9</int>
			  </lst>
			  <str name="status">OK</str>
			</response>

	2. Nella sessione remota passare a {SOLR\_HOME}{Collection}\\data. Per il cluster creato tramite lo script di esempio, sarà uguale a **C:\\apps\\dist\\solr-4.7.2\\example\\solr\\collection1\\data**. In questo percorso dovrebbe essere creata una cartella snapshot con nome simile a **snapshot.*timestamp***.
	
	3. Comprimere la cartella snapshot e caricarla nell'archivio BLOB di Azure. Dalla riga di comando di Hadoop passare al percorso della cartella snapshot usando il comando seguente:

			  hadoop fs -CopyFromLocal snapshot._timestamp_.zip /example/data

		Questo comando copia lo snapshot in /example/data/ nel contenitore disponibile nell'account di archiviazione predefinito associato al cluster.

## Installare Solr tramite Aure PowerShell

Vedere [Personalizzare cluster HDInsight mediante Script azione](hdinsight-hadoop-customize-cluster.md#call_scripts_using_powershell). L'esempio illustra come installare Spark tramite Azure PowerShell. È necessario personalizzare lo script da usare [https://hdiconfigactions.blob.core.windows.net/solrconfigactionv01/giraph-installer-v01.ps1](https://hdiconfigactions.blob.core.windows.net/solrconfigactionv01/solr-installer-v01.ps1).

## Installare Sole tramite .NET SDK

Vedere [Personalizzare cluster HDInsight mediante Script azione](hdinsight-hadoop-customize-cluster.md#call_scripts_using_azure_powershell). L'esempio illustra come installare Spark tramite .NET SDK. È necessario personalizzare lo script da usare [https://hdiconfigactions.blob.core.windows.net/solrconfigactionv01/giraph-installer-v01.ps1](https://hdiconfigactions.blob.core.windows.net/solrconfigactionv01/solr-installer-v01.ps1).

## Vedere anche

- [Installare Solr nei cluster HDInsight](hdinsight-hadoop-solr-install.md): installare Solr nel cluster HDInsight tramite il portale di Azure
- [Installare e usare Solr nei cluster Hadoop di HDInsight (Linux)](hdinsight-hadoop-solr-install-linux.md)
- [Creare cluster Hadoop in HDInsight](hdinsight-provision-clusters.md): informazioni generali sulla creazione di cluster HDInsight
- [Personalizzare cluster HDInsight mediante Script azione][hdinsight-cluster-customize]\: informazioni generali sulla personalizzazione di cluster HDInsight tramite Script azione
- [Sviluppare script di Azione script per HDInsight](hdinsight-hadoop-script-actions.md)
- [Installare e usare Spark nei cluster HDInsight][hdinsight-install-spark]\: esempio di Script azione sull'installazione di Spark
- [Installare R nei cluster HDInsight][hdinsight-install-r]\: esempio di Script azione sull'installazione di R.
- [Installare e usare Spark nei cluster HDInsight](hdinsight-hadoop-giraph-install.md): esempio di Script azione sull'installazione di Giraph

[powershell-install-configure]: powershell-install-configure.md
[hdinsight-provision]: hdinsight-provision-clusters.md
[hdinsight-install-r]: hdinsight-hadoop-r-scripts.md
[hdinsight-install-spark]: hdinsight-hadoop-spark-install.md
[hdinsight-cluster-customize]: hdinsight-hadoop-customize-cluster.md
 

<!---HONumber=AcomDC_0518_2016-->
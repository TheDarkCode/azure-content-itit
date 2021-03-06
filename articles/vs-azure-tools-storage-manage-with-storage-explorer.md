<properties
	pageTitle="Guida introduttiva a Storage Explorer (anteprima) | Microsoft Azure"
	description="Gestire le risorsa di archiviazione di Azure con Storage Explorer (anteprima)"
	services="storage"
	documentationCenter="na"
	authors="TomArcher"
	manager="douge"
	editor="" />

 <tags
	ms.service="storage"
	ms.devlang="multiple"
	ms.topic="get-started-article"
	ms.tgt_pltfrm="na"
	ms.workload="na"
	ms.date="08/17/2016"
	ms.author="tarcher" />

# Guida introduttiva a Storage Explorer (anteprima)

## Overview 

Microsoft Azure Storage Explorer (anteprima), è un'app autonoma che consente di usare facilmente dati di Archiviazione di Azure in Windows, OS X e Linux. Questo articolo illustra diversi modi per connettersi agli account di archiviazione di Azure e per gestirli.

![Microsoft Azure Storage Explorer (anteprima)][15]

## Prerequisiti

- [Scaricare e installare Storage Explorer (anteprima)](http://www.storageexplorer.com)

## Connettersi a un account o a un servizio di archiviazione

Storage Explorer (anteprima) offre numerosi modi per connettersi agli account di archiviazione, tra cui la connessione agli account di archiviazione associati alle sottoscrizioni di Azure, la connessione agli account e ai servizi di archiviazione condivisi da altre sottoscrizioni di Azure e anche la connessione e la gestione della risorsa di archiviazione locale con Emulatore di archiviazione di Azure:

- [Connettersi a una sottoscrizione di Azure.](#connect-to-an-azure-subscription): gestire le risorse di archiviazione appartenenti alla sottoscrizione di Azure.
- [Utilizzare la risorsa di archiviazione locale](#work-with-local-development-storage): gestire la risorsa di archiviazione locale usando l'Emulatore di archiviazione di Azure.
- [Collegarsi a una risorsa di archiviazione esterna](#attach-or-detach-an-external-storage-account): gestire le risorse di archiviazione appartenenti a un'altra sottoscrizione di Azure usando il nome e la chiave dell'account di archiviazione.
- [Collegare l'account di archiviazione usando la firma di accesso condiviso](#attach-storage-account-using-sas): gestire le risorse di archiviazione appartenenti a un'altra sottoscrizione di Azure usando una firma di accesso condiviso.
- [Collegare il servizio usando la firma di accesso condiviso](#attach-service-using-sas): gestire un servizio di archiviazione specifico (contenitore BLOB, coda o tabella) appartenente a un'altra sottoscrizione di Azure usando una firma di accesso condiviso.

## Connettersi a una sottoscrizione di Azure

> [AZURE.NOTE] Se non si ha un account Azure, è possibile [iscriversi per ottenere una versione di valutazione gratuita](https://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A261C142F) oppure [attivare i benefici della sottoscrizione di Visual Studio](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/?WT.mc_id=A261C142F).

1. In Storage Explorer (anteprima), selezionare **Azure Account settings** (Impostazioni account Azure).

	![Impostazioni dell'account Azure][0]

1. Nel riquadro sinistro verranno ora visualizzati tutti gli account Microsoft a cui si è connessi. Per connettersi a un altro account, selezionare **Aggiungi un account** e seguire le istruzioni nelle finestre di dialogo per accedere con un account Microsoft associato ad almeno una sottoscrizione di Azure attiva.

	![Aggiungi un account][1]

1. Dopo avere effettuato l'acceso con un account Microsoft, il riquadro sinistro verrà popolato con le sottoscrizioni di Azure associate a quell'account. Selezionare le sottoscrizioni di Azure da utilizzare e quindi selezionare **Applica**. Selezionando **Tutte le sottoscrizioni**, viene alternata la selezione di tutte o di nessuna delle sottoscrizioni di Azure elencate.

	![Selezionare le sottoscrizioni di Azure][3]

1. Il riquadro sinistro visualizzerà ora gli account di archiviazione associati alle sottoscrizioni di Azure selezionate.

	![Sottoscrizioni di Azure selezionate][4]

## Utilizzare l'archivio di sviluppo locale

Storage Explorer (anteprima) consente di lavorare con la risorsa di archiviazione locale usando Emulatore di archiviazione di Azure che consente di scrivere codice per la risorsa di archiviazione e di testarla senza necessariamente avere un account di archiviazione distribuito in Azure, perché l'account di archiviazione viene emulato da Emulatore di archiviazione di Azure.

>[AZURE.NOTE] Emulatore di archiviazione di Azure è attualmente supportato solo per Windows.

1. Nel riquadro sinistro di Storage Explorer (anteprima) espandere il nodo **(Local and Attached)** (Locale e collegato) > **Account di archiviazione** > **(Sviluppo)**.

	![Nodo di sviluppo locale][21]

1. Se Emulatore di archiviazione di Azure non è ancora stato installato, verrà chiesto di farlo tramite una barra informazioni. Se la barra informazioni è visualizzata, selezionare **Scarica la versione più recente** e installare l'emulatore.

	![Richiesta di download dell'Emulatore di archiviazione di Azure][22]

1. Una volta installato l'emulatore, sarà possibile creare e usare BLOB, code e tabelle locali. Per informazioni su come usare ogni tipo di account di archiviazione, selezionare sotto il collegamento appropriato:

	- [Manage Azure blob storage resources (Gestire risorse di archiviazione BLOB di Azure)](./vs-azure-tools-storage-explorer-blobs.md)
	- Manage Azure file share storage resources (Gestire risorse di archiviazione tabelle di Azure): *presto disponibile*
	- Manage Azure queue storage resources (Gestire risorse di archiviazione code di Azure): *presto disponibile*
	- Manage Azure table storage resources (Gestire risorse di archiviazione tabelle di Azure): *presto disponibile*

## Collegare o scollegare un account di archiviazione esterno

Storage Explorer (anteprima) consente di collegarsi agli account di archiviazione esterni per poter condividere facilmente gli account di archiviazione. Questa sezione illustra come collegarsi (e scollegarsi) agli account di archiviazione esterni.

### Ottenere le credenziali dell'account di archiviazione

Per condividere un account di archiviazione esterno, il proprietario di tale account deve ottenere prima le credenziali (nome e chiave dell'account) per l'account e quindi condividere tali informazioni con la persona che vuole collegarsi a tale account (esterno). Per ottenere le credenziali dell'account di archiviazione, è possibile seguire questa procedura nel portale di Azure:

1.	Accedere al [portale di Azure](https://portal.azure.com).
1.	Selezionare **Esplora**.
1.	Selezionare **Account di archiviazione**.
1.	Nel pannello **Account di archiviazione** selezionare l'account di archiviazione desiderato.
1.	Nel pannello **Impostazioni** per l'account di archiviazione selezionato selezionare **Chiavi di accesso**.

	![Opzione Chiavi di accesso][5]
	
1.	Nel pannello **Chiavi di accesso** copiare i valori di **NOME ACCOUNT DI ARCHIVIAZIONE** e **CHIAVE 1** da usare quando ci si collega all'account di archiviazione.

	![Chiavi di accesso][6]

### Collegarsi a un account di archiviazione esterno
Per connettersi a un account di archiviazione esterno, sono necessari la chiave e il nome dell'account. La sezione *Ottenere le credenziali dell'account di archiviazione* spiega come ottenere questi valori dal portale di Azure. Tuttavia, si noti che nel portale la chiave dell'account viene chiamata "chiave 1", quindi nel punto in cui Storage Explorer (anteprima) richiede una chiave dell'account, si dovrà immettere o incollare il valore della "chiave 1".
 
1.	In Storage Explorer (anteprima), selezionare **Connect to Azure storage** (Connetti ad Archiviazione di Azure).

	![Opzione Connect to Azure storage (Connetti ad Archiviazione di Azure)][23]

1.	Nella finestra di dialogo **Connect to Azure Storage** (Connetti ad archiviazione di Azure) specificare la chiave dell'account (valore di "chiave 1" dal portale di Azure) e quindi selezionare **Avanti**.

	![Finestra di dialogo Connect to Azure storage (Connetti ad Archiviazione di Azure)][24]

1.	Nella finestra di dialogo **Associa archiviazione esterna** immettere il nome dell'account di archiviazione nella casella **Nome account**, specificare eventuali altre impostazioni e selezionare **Avanti** al termine.

	![Finestra di dialogo Associa archiviazione esterna][8]

1.	Nella finestra di dialogo **Riepilogo connessione** verificare le informazioni. Se si vogliono apportare modifiche, selezionare **Indietro** e immettere di nuovo le impostazioni desiderate. Al termine, selezionare **Connetti**.

1.	Una volta connessi, l'account di archiviazione esterno verrà visualizzato con il testo **(Esterno)** aggiunto al nome dell'account di archiviazione.

	![Risultato della connessione a un account di archiviazione esterno][9]

### Scollegarsi da un account di archiviazione esterno

1. 	Fare clic con il pulsante destro del mouse sull'account di archiviazione esterno che si vuole scollegare e scegliere **Scollega** dal menu di scelta rapida.

	![Opzione per scollegarsi da un account di archiviazione][10]

1.	Quando viene visualizzata la finestra di dialogo del messaggio di conferma, selezionare **Sì** per confermare che si vuole scollegare l'account di archiviazione esterno.

## Collegare l'account di archiviazione usando la firma di accesso condiviso

Una [Firma di accesso condiviso](storage/storage-dotnet-shared-access-signature-part-1.md) consente all'amministratore di una sottoscrizione di Azure di concedere temporaneamente l'accesso a un account di archiviazione senza dover fornire le credenziali della sottoscrizione di Azure.

Per illustrare questo concetto, si supponga che l'utente A sia l'amministratore di una sottoscrizione di Azure e che l'utente A voglia consentire all'utente B di accedere a un account di archiviazione per un periodo limitato con determinate autorizzazioni:

1. L'utente A genera una firma di accesso condiviso (costituita dalla stringa di connessione per l'account di archiviazione) per un periodo di tempo specifico e con le autorizzazioni desiderate.
1. L'utente A condivide la firma di accesso condiviso con la persona che vuole accedere all'account di archiviazione, nel nostro esempio l'utente B.
1. L'utente B usa Storage Explorer (anteprima) per collegarsi all'account appartenente all'utente A usando la firma di accesso condiviso fornita.

### Ottenere una firma di accesso condiviso per l'account che si vuole condividere

1.	In Storage Explorer (anteprima) fare clic con il pulsante destro del mouse sull'account di archiviazione che si vuole condividere e scegliere **Get Shared Access Signature** (Ottieni firma di accesso condiviso) dal menu di scelta rapida.

	![Menu di scelta rapida Get SAS (Ottieni firma di accesso condiviso)][13]

1. Nella finestra di dialogo **Shared Access Signature** (Firma di accesso condiviso) specificare l'intervallo di tempo e le autorizzazioni per l'account e selezionare **Crea**.

	![Finestra di dialogo Get SAS (Ottieni firma di accesso condiviso)][14]
 
1. Verrà visualizzata una seconda finestra di dialogo **Shared Access Signature** (Firma di accesso condiviso) contenente la firma di accesso condiviso. Selezionare **Copia** accanto a **Stringa di connessione** per copiarla negli Appunti. Selezionare **Chiudi** per chiudere la finestra di dialogo.

### Collegarsi all'account condiviso usando la firma di accesso condiviso

1.	In Storage Explorer (anteprima), selezionare **Connect to Azure storage** (Connetti ad Archiviazione di Azure).

	![Opzione Connect to Azure storage (Connetti ad Archiviazione di Azure)][23]

1.	Nella finestra di dialogo **Connect to Azure Storage** (Connetti ad Archiviazione di Azure) specificare la stringa di connessione e quindi selezionare **Avanti**.

	![Finestra di dialogo Connect to Azure storage (Connetti ad Archiviazione di Azure)][24]

1.	Nella finestra di dialogo **Riepilogo connessione** verificare le informazioni. Se si vogliono apportare modifiche, selezionare **Indietro** e immettere di nuovo le impostazioni desiderate. Al termine, selezionare **Connetti**.

1.	Una volta collegato, l'account di archiviazione verrà visualizzato con il testo (SAS) aggiunto al nome dell'account fornito.

	![Risultato del collegamento a un account con firma di accesso condiviso][17]

## Collegare il servizio usando la firma di accesso condiviso

La sezione [Collegare l'account di archiviazione usando la firma di accesso condiviso](#attach-storage-account-using-sas) illustra come l'amministratore di una sottoscrizione di Azure possa concedere l'accesso temporaneo a un account di archiviazione generando (e condividendo) una firma di accesso condiviso per l'account di archiviazione. Analogamente, una firma di accesso condiviso può essere generata per un servizio specifico (contenitore BLOB, coda o tabella) in un account di archiviazione.

### Generare una firma di accesso condiviso per il servizio che si vuole condividere

In questo contesto, un servizio può essere un contenitore BLOB, una coda o una tabella. Le sezioni seguenti illustrano come generare la firma di accesso condiviso per il servizio elencato:

- [Get the SAS for a blob container (Ottenere la firma di accesso condiviso per un contenitore BLOB)](./vs-azure-tools-storage-explorer-blobs.md#get-the-sas-for-a-blob-container)
- Get the SAS for a file share (Ottenere la firma di accesso condiviso per una condivisione file): *presto disponibile*
- Get the SAS for a queue (Ottenere la firma di accesso condiviso per una coda): *presto disponibile*
- Get the SAS for a table (Ottenere la firma di accesso condiviso per una tabella): *presto disponibile*

### Collegarsi al servizio account condiviso usando la firma di accesso condiviso

1.	In Storage Explorer (anteprima), selezionare **Connect to Azure storage** (Connetti ad Archiviazione di Azure).

	![Opzione Connect to Azure storage (Connetti ad Archiviazione di Azure)][23]

1.	Nella finestra di dialogo **Connect to Azure Storage** (Connetti ad Archiviazione di Azure) specificare l'URI della firma di accesso condiviso e quindi selezionare **Avanti**.

	![Finestra di dialogo Connect to Azure storage (Connetti ad Archiviazione di Azure)][24]

1.	Nella finestra di dialogo **Riepilogo connessione** verificare le informazioni. Se si vogliono apportare modifiche, selezionare **Indietro** e immettere di nuovo le impostazioni desiderate. Al termine, selezionare **Connetti**.

1.	Una volta collegato, il nuovo servizio collegato verrà visualizzato nel nodo **(Service SAS)** (Firma di accesso condiviso servizio).

	![Risultato della connessione a un servizio condiviso usando una firma di accesso condiviso][20]

## Cercare gli account di archiviazione

Se l'elenco di account di archiviazione è lungo, è possibile trovare rapidamente un determinato account di archiviazione usando la casella di ricerca nella parte superiore del riquadro sinistro.

Mentre si digita nella casella di ricerca, il riquadro sinistro visualizzerà solo gli account di archiviazione che corrispondono al valore di ricerca immesso fino a quel momento. Lo screenshot seguente illustra un esempio in cui sono stati cercati tutti gli account di archiviazione il cui nome contiene il testo "tarcher".

![Ricerca dell'account di archiviazione][11]
	
Per cancellare i criteri di ricerca, selezionare **x** nella casella di ricerca.

## Passaggi successivi
- [Gestire le risorse di archiviazione BLOB di Azure con Storage Explorer (anteprima)](./vs-azure-tools-storage-explorer-blobs.md)

[0]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/settings-icon.png
[1]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/add-account-link.png
[3]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/subscriptions-list.png
[4]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/storage-accounts-list.png
[5]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/access-keys.png
[6]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/access-keys-copy.png
[8]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/attach-external-storage-dlg.png
[9]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/external-storage-account.png
[10]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/detach-external-storage.png
[11]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/storage-account-search.png
[12]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/detach-external-storage-confirmation.png
[13]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/get-sas-context-menu.png
[14]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/get-sas-dlg1.png
[15]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/mase.png
[17]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/attach-account-using-sas-finished.png
[20]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/attach-service-using-sas-finished.png
[21]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/local-storage-drop-down.png
[22]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/download-storage-emulator.png
[23]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/connect-to-azure-storage-icon.png
[24]: ./media/vs-azure-tools-storage-manage-with-storage-explorer/connect-to-azure-storage-next.png

<!---HONumber=AcomDC_0817_2016-->
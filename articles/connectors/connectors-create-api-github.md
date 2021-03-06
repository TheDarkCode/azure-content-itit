<properties
pageTitle="GitHub | Microsoft Azure"
description="Creare app per la logica in Servizio app di Azure. GitHub è un servizio di hosting di repository Git basato sul Web. Offre tutte le funzionalità di Git per il controllo delle revisioni distribuite e la gestione del codice sorgente , oltre a diverse funzionalità aggiuntive."
services="logic-apps"	
documentationCenter=".net,nodejs,java" 	
authors="msftman"	
manager="erikre"	
editor=""
tags="connectors" />

<tags
ms.service="logic-apps"
ms.devlang="multiple"
ms.topic="article"
ms.tgt_pltfrm="na"
ms.workload="integration"
ms.date="08/18/2016"
ms.author="deonhe"/>

# Introduzione al connettore GitHub

GitHub è un servizio di hosting di repository Git basato sul Web. Offre tutte le funzionalità di Git per il controllo delle revisioni distribuite e la gestione del codice sorgente , oltre a diverse funzionalità aggiuntive.

>[AZURE.NOTE] Questa versione dell'articolo si applica alla versione dello schema 2015-08-01-preview delle app per la logica.

Per iniziare subito a creare un'app per la logica, vedere [Creare una nuova app per la logica che connette servizi SaaS](../app-service-logic/app-service-logic-create-a-logic-app.md).

## Trigger e azioni

Il connettore GitHub può essere usato come azione e include trigger. Tutti i connettori supportano dati nei formati JSON e XML.

 Nel connettore GitHub sono disponibili le azioni e/o i trigger seguenti:

### Azioni di GitHub
È possibile eseguire queste azioni:

|Azione|Descrizione|
|--- | ---|
|[CreateIssue](connectors-create-api-github.md#createissue)|Crea un problema|
### Trigger di GitHub
È possibile ascoltare questi eventi:

|Trigger | Descrizione|
|--- | ---|
|Quando viene aperto un problema|Viene aperto un problema|
|Quando viene chiuso un problema|Viene chiuso un problema|
|Quando viene assegnato un problema|Viene assegnato un problema|


## Creare una connessione a GitHub
Per creare app per la logica con GitHub, è prima necessario creare una **connessione** e quindi fornire i dettagli per le proprietà seguenti:

|Proprietà| Obbligatorio|Descrizione|
| ---|---|---|
|Token|Sì|Fornisce le credenziali per GitHub|
Dopo aver creato la connessione, è possibile usarla per eseguire le azioni e restare in ascolto dei trigger descritti in questo articolo.

>[AZURE.INCLUDE [Passaggi per creare una connessione a GitHub](../../includes/connectors-create-api-github.md)]

>[AZURE.TIP] È possibile usare questa connessione in altre app per la logica.

## Informazioni di riferimento per GitHub
Si applica alla versione: 1.0

## CreateIssue
Creare un problema: crea un problema

```POST: /repos/{repositoryOwner}/{repositoryName}/issues```

| Nome| Tipo di dati|Obbligatorio|Posizione|Valore predefinito|Descrizione|
| ---|---|---|---|---|---|
|repositoryOwner|string|yes|path|nessuno|Proprietario del repository|
|repositoryName|stringa|yes|path|nessuno|Nome del repository|
|issueBasicDetails| |sì|body|nessuno|Dettagli del problema|

#### Response

|Name|Descrizione|
|---|---|
|200|OK|
|400|Bad Request|
|401|Non autorizzata|
|403|Accesso negato|
|404|Non trovato|
|500|Errore interno del server. Si è verificato un errore sconosciuto|
|default|Operazione non riuscita.|


## IssueOpened
Quando viene aperto un problema: viene aperto un problema

```GET: /trigger/issueOpened```

Non sono disponibili parametri per questa chiamata
#### Response

|Nome|Descrizione|
|---|---|
|200|OK|
|400|Bad Request|
|401|Non autorizzata|
|403|Accesso negato|
|404|Non trovato|
|500|Errore interno del server. Si è verificato un errore sconosciuto|
|default|Operazione non riuscita.|


## IssueClosed
Quando viene chiuso un problema: viene chiuso un problema

```GET: /trigger/issueClosed```

Non sono disponibili parametri per questa chiamata
#### Response

|Nome|Descrizione|
|---|---|
|200|OK|
|400|Bad Request|
|401|Non autorizzata|
|403|Accesso negato|
|404|Non trovato|
|500|Errore interno del server. Si è verificato un errore sconosciuto|
|default|Operazione non riuscita.|


## IssueAssigned
Quando viene assegnato un problema: viene assegnato un problema

```GET: /trigger/issueAssigned```

Non sono disponibili parametri per questa chiamata
#### Response

|Nome|Descrizione|
|---|---|
|200|OK|
|400|Bad Request|
|401|Non autorizzata|
|403|Accesso negato|
|404|Non trovato|
|500|Errore interno del server. Si è verificato un errore sconosciuto|
|default|Operazione non riuscita.|


## Definizioni oggetti 

### IssueBasicDetailsModel


| Nome proprietà | Tipo di dati | Obbligatorio |
|---|---|---|
|title|string|Sì |
|body|stringa|Sì |
|assignee|stringa|Sì |



### IssueDetailsModel


| Nome proprietà | Tipo di dati | Obbligatorio |
|---|---|---|
|title|string|Sì |
|body|stringa|Sì |
|assignee|string|Sì |
|number|string|No |
|state|stringa|No |
|created\_at|string|No |
|repository\_url|stringa|No |


## Passaggi successivi
[Creare un'app per la logica](../app-service-logic/app-service-logic-create-a-logic-app.md)

<!---HONumber=AcomDC_0824_2016-->
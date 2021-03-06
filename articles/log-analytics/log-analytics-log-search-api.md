<properties
	pageTitle="API REST di ricerca nei log di Log Analytics | Microsoft Azure"
	description="Questa guida contiene un'esercitazione di base che descrive come usare l'API REST di ricerca di Log Analytics in Operations Management Suite (OMS) e offre esempi che indicano come usare i comandi."
	services="log-analytics"
	documentationCenter=""
	authors="bandersmsft"
	manager="jwhit"
	editor=""/>

<tags
	ms.service="log-analytics"
	ms.workload="na"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="04/28/2016"
	ms.author="banders"/>


# API REST di Log Analytics per la ricerca nei log

Questa guida contiene un'esercitazione di base che descrive come usare l'API REST di ricerca di Log Analytics in Operations Management Suite (OMS) e offre esempi che indicano come usare i comandi. Alcuni degli esempi in questo articolo si riferiscono a Operational Insights, ovvero il nome della versione precedente di Log Analytics.

## Panoramica dell'API REST di ricerca di log

L'API REST di ricerca di Log Analytics è RESTful e accessibile tramite l'API Azure Resource Manager. In questo documento sono disponibili esempi in cui viene usata l'API tramite [ARMClient](https://github.com/projectkudu/ARMClient), uno strumento di riga di comando open source che semplifica la chiamata dell'API di Gestione risorse di Azure. L'uso di ARMClient e PowerShell è una delle numerose opzioni di accesso all'API di ricerca di Log Analytics. Un'altra possibilità è quella di usare il modulo Azure PowerShell per OperationalInsights che include cmdlet per l'accesso alla ricerca. Con questi strumenti è possibile usare l'API RESTful Azure Resource Manager per effettuare chiamate alle aree di lavoro di OMS ed eseguire i comandi di ricerca al loro interno. L'API fornirà risultati della ricerca per l'utente in formato JSON, consentendo di usare i risultati della ricerca in molti modi diversi a livello di codice.

È possibile usare Gestione risorse di Azure tramite una [libreria per .NET](https://msdn.microsoft.com/library/azure/dn910477.aspx) nonché una [API REST](https://msdn.microsoft.com/library/azure/mt163658.aspx). Esaminare le pagine Web per altre informazioni.

## Esercitazione di base sull'API REST di ricerca di Log Analytics

### Usare il client ARM

1. Installare [Chocolatey](https://chocolatey.org/), un gestore di pacchetti open source per Windows. Aprire una finestra di prompt di comandi come amministratore ed eseguire il comando seguente:

    ```
    @powershell -NoProfile -ExecutionPolicy unrestricted -Command "iex ((new-object net.webclient).DownloadString('https://chocolatey.org/install.ps1'))" && SET PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin
    ```

2. Installare l'ARMClient eseguendo il comando seguente:

    ```
    choco install armclient
    ```

### Per eseguire una ricerca semplice usando ARMClient

1. Accedere al proprio account Microsoft o OrgID:

    ```
    armclient login
    ```

    Se l’accesso viene effettuato correttamente, vengono elencate tutte le sottoscrizioni associate all'account specificato.

    ```
    PS C:\Users\SampleUserName> armclient login
    Welcome YourEmail@ORG.com (Tenant: zzzzzzzz-zzzz-zzzz-zzzz-zzzzzzzzzzzz)
    User: YourEmail@ORG.com, Tenant: zzzzzzzz-zzzz-zzzz-zzzz-zzzzzzzzzzzz (Example org)
    There are 3 subscriptions
    Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx (Example Name 1)
    Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx (Example Name 2)
    Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx (Example Name 3)
    ```

2. Ottenere le aree di lavoro di Operations Management Suite.

    ```
    armclient get /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces?api-version=2015-03-20
    ```

    Una chiamata Get riuscita genera tutte le aree di lavoro associate alla sottoscrizione.

    ```
    {
    "value": [
    {
      "properties": {
    "source": "External",
    "customerId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "portalUrl": "https://eus.login.mms.microsoft.com/SignIn.aspx?returnUrl=https%3a%2f%2feus.mms.microsoft.com%2fMain.aspx%3fcid%xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
      },
      "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/oi-default-east-us/providers/microsoft.operationalinsights/workspaces/{WORKSPACE NAME}",
      "name": "{WORKSPACE NAME}",
      "type": "Microsoft.OperationalInsights/workspaces",
      "location": "East US"
       ]
    }
    ```
3. Creare la variabile di ricerca.

    ```
    $mySearch = "{ 'top':150, 'query':'Error'}";
    ```
4. Eseguire la ricerca usando la nuova variabile di ricerca.

    ```
    armclient post /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/{WORKSPACE NAME}/search?api-version=2015-03-20 $mySearch
    ```

## Esempi di riferimento per l'API REST di ricerca di Log Analytics
I codici di esempio seguenti mostrano come usare l’API di ricerca.

### Ricerca - Azione/Lettura

**Url di esempio:**

```
	/subscriptions/{SubscriptionId}/resourceGroups/{ResourceGroupId}/providers/Microsoft.OperationalInsights/workspaces/{WorkspaceName}/search?api-version=2015-03-20
```

**Richiesta:**

```
	$savedSearchParametersJson =
	{
	  "top":150,
	  "highlight":{
	    "pre":"{[hl]}",
	    "post":"{[/hl]}"
	  },
	  "query":"*",
	  "start":"2015-02-04T21:03:29.231Z",
	  "end":"2015-02-11T21:03:29.231Z"
	}
	armclient post /subscriptions/{Subscription ID}/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/{Workspace ID}/search?api-version=2015-03-20 $searchParametersJson
```
La tabella seguente descrive le proprietà disponibili.

|**Proprietà**|**Descrizione**|
|---|---|
|top|Il numero massimo di risultati da restituire.|
|highlight|Contiene i parametri pre e post, in genere usati per evidenziare i campi corrispondenti|
|pre|Antepone la stringa specificata ai campi corrispondenti.|
|post|Appone la stringa specificata ai campi corrispondenti.|
|query|La query di ricerca usata per raccogliere e restituire i risultati.|
|start|L’inizio dell'intervallo di tempo da cui si desidera trovare risultati.|
|end|La fine dell'intervallo di tempo da cui si desidera trovare risultati.|


**Risposta:**

```
	{
	  "id" : "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
	  "__metadata" : {
	    "resultType" : "raw",
	    "total" : 1455,
	    "top" : 150,
	    "StartTime" : "2015-02-11T21:09:07.0345815Z",
	    "Status" : "Successful",
	    "LastUpdated" : "2015-02-11T21:09:07.331463Z",
	    "CoreResponses" : [],
	    "sort" : [{
	      "name" : "TimeGenerated",
	      "order" : "desc"
	    }],
	    "requestTime" : 450
	  },
	  "value": [{
	    "SourceSystem" : "OpsManager",
	    "TimeGenerated" : "2015-02-07T14:07:33Z",
	    "Source" : "SideBySide",
	    "EventLog" : "Application",
	    "Computer" : "BAMBAM",
	    "EventCategory" : 0,
	    "EventLevel" : 1,
	    "EventLevelName" : "Error",
	    "UserName" : "N/A",
	    "EventID" : 78,
	    "MG" : "00000000-0000-0000-0000-000000000001",
	    "TimeCollected" : "2015-02-07T14:10:04.69Z",
	    "ManagementGroupName" : "AOI-5bf9a37f-e841-462b-80d2-1d19cd97dc40",
	    "id" : "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
	    "Type" : "Event",
	    "__metadata" : {
	      "Type" : "Event",
	      "TimeGenerated" : "2015-02-07T14:07:33Z",
	      "highlighting" : {
	      "EventLevelName" : ["{[hl]}Error{[/hl]}"]
	    }
	  }]
	],
	        "start" : "2015-02-04T21:03:29.231Z",
	        "end" : "2015-02-11T21:03:29.231Z"
	      }
	    }
	  }]
	}
```

### Ricerca/{ID} - Azione/Lettura

**Richiedere il contenuto di una ricerca salvata:**

```
	armclient post /subscriptions/{SubId}/resourceGroups/{ResourceGroupId}/providers/Microsoft.OperationalInsights/workspaces/{WorkspaceName}/search/{SearchId}?api-version=2015-03-20
```

>[AZURE.NOTE] Se la ricerca restituisce lo stato 'Sospeso', il polling dei risultati aggiornati può essere effettuato tramite questa API. Dopo 6 min il risultato della ricerca verrà rimosso dalla cache e sarà restituito Http Gone. Se la richiesta di ricerca iniziale ha restituito immediatamente lo stato "Riuscito", non verrà aggiunto alla cache, determinando la restituzione di Http Gone da parte dell’API in caso di query. Il contenuto di un risultato Http 200 sarà nello stesso formato della richiesta di ricerca iniziale, ma con i valori aggiornati.

### Ricerche salvate - solo REST

**Elenco di richieste delle ricerche salvate:**

```
	armclient post /subscriptions/{SubId}/resourceGroups/{ResourceGroupId}/providers/Microsoft.OperationalInsights/workspaces/{WorkspaceName}/savedSearches?api-version=2015-03-20
```

Metodi supportati: GET PUT DELETE

Metodi di raccolta supportati: GET

La tabella seguente descrive le proprietà disponibili.

|Proprietà|Descrizione|
|---|---|
|ID|Identificatore univoco.|
|ETag|**Obbligatoria per l'applicazione di patch**. Aggiornata dal server a ogni scrittura. Il valore deve essere uguale al valore archiviato attuale o a "*" per effettuare l'aggiornamento. Viene restituito il codice 409 per i valori obsoleti/non validi.|
|properties.query|**Obbligatoria**. Query di ricerca.|
|properties.displayName|**Obbligatoria**. Nome visualizzato della query, definito dall'utente. Se modellata come risorsa di Azure, corrisponderà a un tag.|
|properties.category|**Obbligatoria**. Categoria della query, definita dall'utente. Se modellata come risorsa di Azure, corrisponderà a un tag.|

>[AZURE.NOTE] L'API di ricerca di Log Analytics restituisce attualmente le ricerche salvate create dall'utente quando viene eseguito il polling di ricerche salvate in un'area di lavoro. L'API non restituirà attualmente le ricerche salvate fornite dalle soluzioni. Questa funzionalità verrà aggiunta in un secondo momento.

### Eliminare le ricerche salvate

**Richiesta:**

```
	armclient delete /subscriptions/{Subscription ID}/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/{Workspace ID}/savedSearches/thisIsMyId?api-version=2015-03-20
```

### Aggiornare le ricerche salvate

 **Richiesta:**

```
	$savedSearchParametersJson = "{'etag': 'W/`"datetime\'2015-04-16T23%3A35%3A35.3182423Z\'`"', 'properties': { 'Category': 'myCategory', 'DisplayName':'myDisplayName', 'Query':'* | measure Count() by Source', 'Version':'1'  }"
	armclient put /subscriptions/{Subscription ID}/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/{Workspace ID}/savedSearches/thisIsMyId?api-version=2015-03-20 $savedSearchParametersJson
```

### Metadati - Solo JSON

Ecco come visualizzare i campi per tutti i tipi di log per i dati raccolti nell'area di lavoro. Ad esempio, se si vuole sapere se il tipo di evento include un campo denominato Computer, è possibile procedere in questo modo per la verifica e la conferma.

**Richiesta di campi:**

```
	armclient get /subscriptions/{SubId}/resourceGroups/{ResourceGroupId}/providers/Microsoft.OperationalInsights/workspaces/{WorkspaceName}/schema?api-version=2015-03-20
```

**Risposta:**

```
	{
	  "__metadata" : {
	    "schema" : {
	      "name" : "Example Name",
	      "version" : 2
	    },
	    "resultType" : "schema",
	    "requestTime" : 35
	  },
	  "value" : [{
	      "name" : "MG",
	      "displayName" : "MG",
	      "type" : "Guid",
	      "facetable" : true,
	      "display" : false,
	      "ownerType" : ["PerfHourly", "ProtectionStatus", "Capacity_SMBUtilizationByHost", "Capacity_ArrayUtilization", "Capacity_SMBShareUtilization", "SQLAssessmentRecommendation", "Event", "ConfigurationChange", "ConfigurationAlert", "ADAssessmentRecommendation", "ConfigurationObject", "ConfigurationObjectProperty"]
	    }, {
	      "name" : "ManagementGroupName",
	      "displayName" : "ManagementGroupName",
	      "type" : "String",
	      "facetable" : true,
	      "display" : true,
	      "ownerType" : ["PerfHourly", "ProtectionStatus", "Event", "ConfigurationChange", "ConfigurationAlert", "W3CIISLog", "AlertHistory", "Recommendation", "Alert", "SecurityEvent", "UpdateAgent", "RequiredUpdate", "ConfigurationObject", "ConfigurationObjectProperty"]
	    }
	  ]
	}
```

La tabella seguente descrive le proprietà disponibili.

|**Proprietà**|**Descrizione**|
|---|---|
|name|Nome del campo.|
|displayName|Nome visualizzato del campo.|
|type|Tipo del valore del campo.|
|facetable|Combinazione delle proprietà "indexed", "stored" e "facet" attuali.|
|display|Proprietà "display" attuale. True se il campo è visibile nella ricerca.|
|ownerType|Ridotta solo ai tipi appartenenti agli indirizzi IP caricati.|


## Parametri facoltativi
Le informazioni seguenti illustrano i parametri facoltativi disponibili.

### Evidenziazione

Il parametro "Highlight" è un parametro facoltativo che può essere usato per richiedere che il sottosistema di ricerca includa un set di marcatori nella risposta.

Questi marcatori indicano l'inizio e la fine del testo evidenziato che corrisponde ai termini forniti nella query di ricerca. È possibile specificare i marcatori di inizio e di fine che verranno usati dalla ricerca per eseguire il wrapping del termine evidenziato.

**Query di ricerca di esempio**

```
	$savedSearchParametersJson =
	{
	  "top":150,
	  "highlight":{
	    "pre":"{[hl]}",
	    "post":"{[/hl]}"
	  },
	  "query":"*",
	  "start":"2015-02-04T21:03:29.231Z",
	  "end":"2015-02-11T21:03:29.231Z"
	}
	armclient post /subscriptions/{Subscription ID}/resourceGroups/OI-Default-East-US/providers/Microsoft.OperationalInsights/workspaces/{Workspace ID}/search?api-version=2015-03-20 $searchParametersJson
```

**Risultati di esempio:**

```
	{
	    "TimeGenerated":
	    "2015-05-18T23:55:59Z", "Source":
	    "EventLog": "Application",
	    "Computer": "smokedturkey.net",
	    "EventCategory": 0,
	    "EventLevel":1,
	    "EventLevelName":
	    "Error"
	    "Manager ", "__metadata":
	    {
	        "Type": "Event",
	        "TimeGenerated": "2015-05-18T23:55:59Z",
	        "highlighting": {
	            "EventLevelName":
	            ["{[hl]}Error{[/hl]}"]
	        }
	    }
	}
```

Si noti che il risultato precedente contiene un messaggio di errore con prefisso e suffisso.

## Gruppi di computer

I gruppi di computer sono ricerche salvate speciali che restituiscono un set di computer. È possibile usare un gruppo di computer in altre query per limitare i risultati ai computer nel gruppo. Un gruppo di computer viene implementato come una ricerca salvata con un tag Gruppo con il valore Computer.

Di seguito viene riportata una risposta di esempio per un gruppo di computer.

	"etag": "W/"datetime'2016-04-01T13%3A38%3A04.7763203Z'"",
	"properties": {
		"Category": "My Computer Groups",
		"DisplayName": "My Computer Group",
		"Query": "srv* | Distinct Computer",
		"Tags": [{
        	"Name": "Group",
        	"Value": "Computer"
      	}],
    "Version": 1
	}

### Recupero dei gruppi di computer

Usare il metodo Get con l'ID del gruppo per recuperare un gruppo di computer.

```
armclient get /subscriptions/{Subscription ID}/resourceGroups/{Resource Group Name}/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Group ID}`?api-version=2015-03-20
```

### Creazione o aggiornamento di un gruppo di computer

Usare il metodo Put con un ID di ricerca salvato univoco per creare un nuovo gruppo di computer. Se si usa un ID gruppo di computer esistente, verrà modificato. Quando si crea un gruppo di computer nella console di OMS, l'ID viene creato dal gruppo e dal nome.

La query usata per la definizione del gruppo deve restituire un insieme di computer per il gruppo per funzionare correttamente. È consigliabile concludere la query con il *| Computer distinto* per assicurarsi che vengano restituiti i dati corretti.

La definizione di una ricerca salvata deve includere un tag denominato Gruppo con un valore Computer affinché la ricerca venga classificata come un gruppo di computer.

	$etag=Get-Date -Format yyyy-MM-ddThh:mm:ss.msZ
	$groupName="My Computer Group"
	$groupQuery = "Computer=srv* | Distinct Computer"
	$groupCategory="My Computer Groups"
	$groupID = "My Computer Groups | My Computer Group"

	$groupJson = "{'etag': 'W/`"datetime\'" + $etag + "\'`"', 'properties': { 'Category': '" + $groupCategory + "', 'DisplayName':'"  + $groupName + "', 'Query':'" + $groupQuery + "', 'Tags': [{'Name': 'Group', 'Value': 'Computer'}], 'Version':'1'  }"

	armclient put /subscriptions/{Subscription ID}/resourceGroups/{Resource Group Name}/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/$groupId`?api-version=2015-03-20 $groupJson

### Eliminazione di gruppi di computer

Usare il metodo Delete con l'ID del gruppo per eliminare un gruppo di computer.

```
armclient delete /subscriptions/{Subscription ID}/resourceGroups/{Resource Group Name}/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/$groupId`?api-version=2015-03-20
```


## Passaggi successivi

- Informazioni sulle [ricerche nei log](log-analytics-log-searches.md) per creare query con i campi personalizzati per i criteri.

<!---HONumber=AcomDC_0518_2016-->
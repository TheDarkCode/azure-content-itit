<properties
 pageTitle="Autenticazione in uscita dell'Utilità di pianificazione"
 description="Autenticazione in uscita dell'Utilità di pianificazione"
 services="scheduler"
 documentationCenter=".NET"
 authors="krisragh"
 manager="dwrede"
 editor=""/>
<tags
 ms.service="scheduler"
 ms.workload="infrastructure-services"
 ms.tgt_pltfrm="na"
 ms.devlang="dotnet"
 ms.topic="article"
 ms.date="08/15/2016"
 ms.author="krisragh"/>

# Autenticazione in uscita dell'Utilità di pianificazione

I processi dell'Utilità di pianificazione potrebbero richiedere servizi che necessitano di autenticazione. In questo modo, un servizio richiamato può determinare se l'utilità di pianificazione può accedere alle sue risorse. Alcuni di questi servizi includono altri servizi di Azure, Salesforce.com, Facebook e siti Web custom protetti.

## Aggiunta e rimozione di autenticazione

Aggiungere un'autenticazione a un'utilità di pianificazione è semplice – è sufficiente aggiungere un elemento figlio JSON `authentication` all'`request` elemento durante la creazione o l’aggiornamento di un processo. I segreti passati al servizio dell’Utilità di pianificazione in una richiesta PUT, PATCH o POST – come parte dell’oggetto `authentication` – non vengono mai restituiti nelle risposte. Nelle risposte, le informazioni segrete sono impostati su null o potrebbero avere un token pubblico che rappresenti l'entità autenticata.

Per rimuovere l'autenticazione, effettuare un’azione PUT o PATCH in modo esplicito, impostando l’oggetto `authentication` su null. Nessuna proprietà di autenticazione verrà visualizzata nella risposta.

Attualmente, gli unici tipi di autenticazione supportati sono il modello `ClientCertificate` (per l'utilizzo di certificati client SSL/TLS), il modello `Basic` (per l'autenticazione di Base) e il modello `ActiveDirectoryOAuth` (per l'autenticazione OAuth di Active Directory).

## Corpo della richiesta per l'autenticazione ClientCertificate

Quando si aggiunge un'autenticazione utilizzando il modello `ClientCertificate`, specificare i seguenti elementi aggiuntivi nel corpo della richiesta.

|Elemento|Descrizione|
|:---|:---|
|_authentication (parent element)_|Oggetto di autenticazione per l'utilizzo di un certificato client SSL.|
|_type_|Obbligatorio. Tipo di autenticazione. Per i certificati client SSL, il valore deve essere `ClientCertificate`.|
|_pfx_|Obbligatorio. I contenuti del file PFX in codifica Base64.|
|_password_|Obbligatorio. La password per accedere al file PFX.|


## Corpo della risposta per l'autenticazione ClientCertificate

Quando viene inviata una richiesta con le informazioni di autenticazione, la risposta contiene i seguenti elementi relativi all'autenticazione.

|Elemento |Descrizione |
|:--|:--|
|_authentication (parent element)_ |Oggetto di autenticazione per l'utilizzo di un certificato client SSL.|
|_type_ |Tipo di autenticazione. Per i certificati client SSL, il valore è `ClientCertificate`.|
|_certificateThumbprint_ |L'identificazione personale del certificato.|
|_certificateSubjectName_ |Il nome distintivo del soggetto del certificato.|
|_certificateExpiration_ |La data di scadenza del certificato.|

## Richiesta REST di esempio per l'autenticazione ClientCertificate

```
PUT https://management.azure.com/subscriptions/1fe0abdf-581e-4dfe-9ec7-e5cb8e7b205e/resourceGroups/CS-SoutheastAsia-scheduler/providers/Microsoft.Scheduler/jobcollections/southeastasiajc/jobs/httpjob?api-version=2016-01-01 HTTP/1.1
User-Agent: Fiddler
Host: management.azure.com
Authorization: Bearer sometoken
Content-Type: application/json; charset=utf-8

{
  "properties": {
    "startTime": "2015-05-14T14:10:00Z",
    "action": {
      "request": {
        "uri": "https://mywebserviceendpoint.com",
        "method": "GET",
		"headers": {
          "x-ms-version": "2013-03-01"
        },
		"authentication": {
          "type": "clientcertificate",
          "password": "password",
          "pfx": "pfx key"
        }
      },
      "type": "http"
    },
    "recurrence": {
      "frequency": "minute",
      "endTime": "2016-04-10T08:00:00Z",
      "interval": 1
    },
    "state": "enabled",
  }
}
```

## Risposta REST di esempio per l'autenticazione ClientCertificate

```
HTTP/1.1 200 OK
Cache-Control: no-cache
Pragma: no-cache
Content-Length: 858
Content-Type: application/json; charset=utf-8
Expires: -1
x-ms-request-id: 56c7b40e-721a-437e-88e6-f68562a73aa8
Server: Microsoft-IIS/8.5
X-AspNet-Version: 4.0.30319
X-Powered-By: ASP.NET
x-ms-ratelimit-remaining-subscription-resource-requests: 599
x-ms-correlation-request-id: 1075219e-e879-4030-bc81-094e54fbabce
x-ms-routing-request-id: WESTUS:20160316T190424Z:1075219e-e879-4030-bc81-094e54fbabce
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Wed, 16 Mar 2016 19:04:23 GMT

{
  "id": "/subscriptions/1fe0abdf-581e-4dfe-9ec7-e5cb8e7b205e/resourceGroups/CS-SoutheastAsia-scheduler/providers/Microsoft.Scheduler/jobCollections/southeastasiajc/jobs/httpjob",
  "type": "Microsoft.Scheduler/jobCollections/jobs",
  "name": "southeastasiajc/httpjob",
  "properties": {
    "startTime": "2015-05-14T14:10:00Z",
    "action": {
      "request": {
        "uri": "https://mywebserviceendpoint.com",
        "method": "GET",
        "headers": {
          "x-ms-version": "2013-03-01"
        },
        "authentication": {
          "certificateThumbprint": "88105CG9DF9ADE75B835711D899296CB217D7055",
          "certificateExpiration": "2021-01-01T07:00:00Z",
          "certificateSubjectName": "CN=Scheduler Mgmt",
          "type": "ClientCertificate"
        }
      },
      "type": "http"
    },
    "recurrence": {
      "frequency": "minute",
      "endTime": "2016-04-10T08:00:00Z",
      "interval": 1
    },
    "state": "enabled",
    "status": {
      "nextExecutionTime": "2016-03-16T19:05:00Z",
      "executionCount": 0,
      "failureCount": 0,
      "faultedCount": 0
    }
  }
}
```

## Corpo della richiesta per l'autenticazione di Base

Quando si aggiunge un'autenticazione utilizzando il modello `Basic`, specificare i seguenti elementi aggiuntivi nel corpo della richiesta.

|Elemento|Descrizione|
|:--|:--|
|_authentication (parent element)_ |Oggetto di autenticazione per l’utilizzo dell'autenticazione di Base.|
|_type_ |Obbligatorio. Tipo di autenticazione. Per l'autenticazione di Base, il valore deve essere `Basic`.|
|_username_ |Obbligatorio. Nome utente da autenticare.|
|_password_ |Obbligatorio. Password da autenticare.|

## Corpo della risposta per l'autenticazione di Base

Quando viene inviata una richiesta con le informazioni di autenticazione, la risposta contiene i seguenti elementi relativi all'autenticazione.

|Elemento|Descrizione|
|:--|:--|
|_authentication (parent element)_ |Oggetto di autenticazione per l’utilizzo dell'autenticazione di Base.|
|_type_ |Tipo di autenticazione. Per l'autenticazione di Base, il valore è `Basic`.|
|_username_ |Il nome utente autenticato.|

## Richiesta REST di esempio per l'autenticazione di base

```
PUT https://management.azure.com/subscriptions/1d908808-e491-4fe5-b97e-29886e18efd4/resourceGroups/CS-SoutheastAsia-scheduler/providers/Microsoft.Scheduler/jobcollections/southeastasiajc/jobs/httpjob?api-version=2016-01-01 HTTP/1.1
User-Agent: Fiddler
Host: management.azure.com
Authorization: Bearer sometoken
Content-Length: 562
Content-Type: application/json; charset=utf-8

{
  "properties": {
    "startTime": "2015-05-14T14:10:00Z",
    "action": {
      "request": {
        "uri": "https://mywebserviceendpoint.com",
        "method": "GET",
		"headers": {
          "x-ms-version": "2013-03-01"
        },
		"authentication": {
          "type": "basic",
		  "username": "user",
          "password": "password"
        }
      },
      "type": "http"
    },
    "recurrence": {
      "frequency": "minute",
      "endTime": "2016-04-10T08:00:00Z",
      "interval": 1
    },
    "state": "enabled",
  }
}
```

## Risposta REST di esempio per l'autenticazione di base

```
HTTP/1.1 200 OK
Cache-Control: no-cache
Pragma: no-cache
Content-Length: 701
Content-Type: application/json; charset=utf-8
Expires: -1
x-ms-request-id: a2dcb9cd-1aea-4887-8893-d81273a8cf04
Server: Microsoft-IIS/8.5
X-AspNet-Version: 4.0.30319
X-Powered-By: ASP.NET
x-ms-ratelimit-remaining-subscription-resource-requests: 599
x-ms-correlation-request-id: 7816f222-6ea7-468d-b919-e6ddebbd7e95
x-ms-routing-request-id: WESTUS:20160316T190506Z:7816f222-6ea7-468d-b919-e6ddebbd7e95
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Wed, 16 Mar 2016 19:05:06 GMT

{  
   "id":"/subscriptions/1d908808-e491-4fe5-b97e-29886e18efd4/resourceGroups/CS-SoutheastAsia-scheduler/providers/Microsoft.Scheduler/jobCollections/southeastasiajc/jobs/httpjob",
   "type":"Microsoft.Scheduler/jobCollections/jobs",
   "name":"southeastasiajc/httpjob",
   "properties":{  
      "startTime":"2015-05-14T14:10:00Z",
      "action":{  
         "request":{  
            "uri":"https://mywebserviceendpoint.com",
            "method":"GET",
            "headers":{  
               "x-ms-version":"2013-03-01"
            },
            "authentication":{  
               "username":"user1",
               "type":"Basic"
            }
         },
         "type":"http"
      },
      "recurrence":{  
         "frequency":"minute",
         "endTime":"2016-04-10T08:00:00Z",
         "interval":1
      },
      "state":"enabled",
      "status":{  
         "nextExecutionTime":"2016-03-16T19:06:00Z",
         "executionCount":0,
         "failureCount":0,
         "faultedCount":0
      }
   }
}
```

## Corpo della richiesta per l'autenticazione ActiveDirectoryOAuth

Quando si aggiunge un'autenticazione utilizzando il modello `ActiveDirectoryOAuth`, specificare i seguenti elementi aggiuntivi nel corpo della richiesta.

|Elemento |Descrizione |
|:--|:--|
|_authentication (parent element)_ |Oggetto di autenticazione per l'autenticazione basata su ActiveDirectoryOAuth.|
|_type_ |Obbligatorio. Tipo di autenticazione. Per l'autenticazione ActiveDirectoryOAuth, il valore deve essere `ActiveDirectoryOAuth`.|
|_tenant_ |Obbligatorio. L'identificatore del tenant di Azure AD.|
|_audience_ |Obbligatorio. Viene impostato su https://management.core.windows.net/.|
|_clientId_ |Obbligatorio. Fornisce l'identificativo del client per l'applicazione Active Directory di Azure.|
|_secret_ |Obbligatorio. Segreto del client che richiede il token.|

### Determinazione del l'identificatore del tenant

È possibile trovare l'identificatore del tenant di Azure AD eseguendo `Get-AzureAccount` in Azure PowerShell.

## Corpo della risposta per l'autenticazione ActiveDirectoryOAuth

Quando viene inviata una richiesta con le informazioni di autenticazione, la risposta contiene i seguenti elementi relativi all'autenticazione.

|Elemento |Description |
|:--|:--|
|_authentication (parent element)_ |Oggetto di autenticazione per l'autenticazione basata su ActiveDirectoryOAuth.|
|_type_ |Tipo di autenticazione. Per l'autenticazione ActiveDirectoryOAuth, il valore è `ActiveDirectoryOAuth`.|
|_tenant_ |L'identificatore del tenant di Azure AD. |
|_audience_ |Viene impostato su https://management.core.windows.net/.|
|_clientId_ |Identificativo del client per l'applicazione Azure AD.|

## Richiesta REST di esempio per l'autenticazione ActiveDirectoryOAuth

```
PUT https://management.azure.com/subscriptions/1d908808-e491-4fe5-b97e-29886e18efd4/resourceGroups/CS-SoutheastAsia-scheduler/providers/Microsoft.Scheduler/jobcollections/southeastasiajc/jobs/httpjob?api-version=2016-01-01 HTTP/1.1
User-Agent: Fiddler
Host: management.azure.com
Authorization: Bearer sometoken
Content-Length: 757
Content-Type: application/json; charset=utf-8

{
  "properties": {
    "startTime": "2015-05-14T14:10:00Z",
    "action": {
      "request": {
        "uri": "https://mywebserviceendpoint.com",
        "method": "GET",
		"headers": {
          "x-ms-version": "2013-03-01"
        },
		"authentication": {
          "tenant":"microsoft.onmicrosoft.com",
          "audience":"https://management.core.windows.net/",
          "clientId":"dc23e764-9be6-4a33-9b9a-c46e36f0c137",
          "secret": "G6u071r8Gjw4V4KSibnb+VK4+tX399hkHaj7LOyHuj5=",
          "type":"ActiveDirectoryOAuth"
        }
      },
      "type": "http"
    },
    "recurrence": {
      "frequency": "minute",
      "endTime": "2016-04-10T08:00:00Z",
      "interval": 1
    },
    "state": "enabled",
  }
}
```

## Risposta REST di esempio per l'autenticazione ActiveDirectoryOAuth

```
HTTP/1.1 200 OK
Cache-Control: no-cache
Pragma: no-cache
Content-Length: 885
Content-Type: application/json; charset=utf-8
Expires: -1
x-ms-request-id: 86d8e9fd-ac0d-4bed-9420-9baba1af3251
Server: Microsoft-IIS/8.5
X-AspNet-Version: 4.0.30319
X-Powered-By: ASP.NET
x-ms-ratelimit-remaining-subscription-resource-requests: 599
x-ms-correlation-request-id: 5183bbf4-9fa1-44bb-98c6-6872e3f2e7ce
x-ms-routing-request-id: WESTUS:20160316T191003Z:5183bbf4-9fa1-44bb-98c6-6872e3f2e7ce
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Wed, 16 Mar 2016 19:10:02 GMT

{  
   "id":"/subscriptions/1d908808-e491-4fe5-b97e-29886e18efd4/resourceGroups/CS-SoutheastAsia-scheduler/providers/Microsoft.Scheduler/jobCollections/southeastasiajc/jobs/httpjob",
   "type":"Microsoft.Scheduler/jobCollections/jobs",
   "name":"southeastasiajc/httpjob",
   "properties":{  
      "startTime":"2015-05-14T14:10:00Z",
      "action":{  
         "request":{  
            "uri":"https://mywebserviceendpoint.com",
            "method":"GET",
            "headers":{  
               "x-ms-version":"2013-03-01"
            },
            "authentication":{  
               "tenant":"microsoft.onmicrosoft.com",
               "audience":"https://management.core.windows.net/",
               "clientId":"dc23e764-9be6-4a33-9b9a-c46e36f0c137",
               "type":"ActiveDirectoryOAuth"
            }
         },
         "type":"http"
      },
      "recurrence":{  
         "frequency":"minute",
         "endTime":"2016-04-10T08:00:00Z",
         "interval":1
      },
      "state":"enabled",
      "status":{  
         "lastExecutionTime":"2016-03-16T19:10:00.3762123Z",
         "nextExecutionTime":"2016-03-16T19:11:00Z",
         "executionCount":5,
         "failureCount":5,
         "faultedCount":1
      }
   }
}
```

## Vedere anche


 [Che cos'è l'Utilità di pianificazione?](scheduler-intro.md)

 [Concetti, terminologia e gerarchia di entità dell'Utilità di pianificazione di Azure](scheduler-concepts-terms.md)

 [Introduzione all'uso dell'Utilità di pianificazione di Azure nel portale di Azure](scheduler-get-started-portal.md)

 [Piani e fatturazione nell'utilità di pianificazione di Azure](scheduler-plans-billing.md)

 [Informazioni di riferimento sull'API REST dell'Utilità di pianificazione di Azure](https://msdn.microsoft.com/library/mt629143)

 [Informazioni di riferimento sui cmdlet PowerShell dell'Utilità di pianificazione di Azure](scheduler-powershell-reference.md)

 [Livelli elevati di disponibilità e affidabilità dell'Utilità di pianificazione di Azure](scheduler-high-availability-reliability.md)

 [Limiti, valori predefiniti e codici di errore dell'Utilità di pianificazione di Azure](scheduler-limits-defaults-errors.md)

<!---HONumber=AcomDC_0817_2016-->
<properties
   pageTitle="Bilanciare il carico di un cluster del servizio contenitore di Azure | Microsoft Azure"
   description="Bilanciare il carico di un cluster del servizio contenitore di Azure."
   services="container-service"
   documentationCenter=""
   authors="rgardler"
   manager="timlt"
   editor=""
   tags="acs, azure-container-service"
   keywords="Contenitori, Micro-Service, controller di dominio/sistema operativo, Azure"/>

<tags
   ms.service="container-service"
   ms.devlang="na"
   ms.topic="get-started-article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="07/11/2016"
   ms.author="rogardle"/>

# Bilanciare il carico di un cluster del servizio contenitore di Azure

In questo articolo verrà configurato un front-end Web in un servizio contenitore di Azure gestito di tipo DC/OS. Verrà anche configurato un servizio di bilanciamento del carico Marathon-LB per consentire la scalabilità dell'applicazione.

## Prerequisiti

[Distribuire un cluster del servizio contenitore di Azure](container-service-deployment.md) con un agente di orchestrazione di tipo DC/OS e [assicurarsi che il client possa connettersi al cluster](container-service-connect.md).

## Bilanciamento del carico.

Il cluster del servizio contenitore che verrà creato include due livelli di bilanciamento del carico:

  1. Azure Load Balancer offre punti di ingresso pubblici, ovvero i punti che verranno usati dagli utenti finali. Questa funzionalità viene fornita automaticamente dal servizio contenitore di Azure ed è configurata per impostazione predefinita in modo da esporre le porte 80, 443 e 8080.
  2. Il servizio di bilanciamento del carico Marathon (marathon-lb) indirizza le richieste in ingresso a istanze del contenitore che gestiscono tali richieste. Quando i contenitori che mettono a disposizione il servizio Web vengono ridimensionati, marathon-lb si adatterà in modo dinamico. Questo servizio di bilanciamento del carico non viene fornito per impostazione predefinita nel servizio contenitore, ma è molto semplice da installare.

## Servizio di bilanciamento del carico Marathon

Il servizio di bilanciamento del carico Marathon si riconfigura in base ai contenitori distribuiti. È anche resiliente in caso di perdita di un contenitore o di un agente. In questi casi, Apache Mesos riavvierà semplicemente il contenitore in un'altra posizione e marathon-lb si adatterà.

Per installare il servizio di bilanciamento del carico Marathon, è possibile usare l'interfaccia utente Web di DC/OS o la riga di comando.

### Installare Marathon-LB usando l'interfaccia utente Web di DC/OS

  1. Fare clic su 'Universe' (Universo)
  2. Cercare 'Marathon-LB'
  3. Fare clic su 'Install' (Installa)

![Installazione di marathon-lb mediante l'interfaccia Web di DC/OS](./media/dcos/marathon-lb-install.png)

### Installare Marathon-LB usando l'interfaccia della riga di comando di DC/OS

Dopo l'installazione dell'interfaccia della riga di comando di DC/OS e dopo avere verificato che è possibile connettersi al cluster, eseguire il comando seguente dal computer client:

```bash
dcos package install marathon-lb
```

## Distribuire un'applicazione Web con bilanciamento del carico

Ora che il pacchetto marathon-lb è disponibile, è possibile distribuire un semplice server Web usando la configurazione seguente:

```json
{
  "id": "web",
  "container": {
    "type": "DOCKER",
    "docker": {
      "image": "yeasy/simple-web",
      "network": "BRIDGE",
      "portMappings": [
        { "hostPort": 0, "containerPort": 80, "servicePort": 10000 }
      ],
      "forcePullImage":true
    }
  },
  "instances": 3,
  "cpus": 0.1,
  "mem": 65,
  "healthChecks": [{
      "protocol": "HTTP",
      "path": "/",
      "portIndex": 0,
      "timeoutSeconds": 10,
      "gracePeriodSeconds": 10,
      "intervalSeconds": 2,
      "maxConsecutiveFailures": 10
  }],
  "labels":{
    "HAPROXY_GROUP":"external",
    "HAPROXY_0_VHOST":"YOUR FQDN",
    "HAPROXY_0_MODE":"http"
  }
}

```

  * Impostare il valore di `HAProxy_0_VHOST` sul nome di dominio completo (FQDN) del servizio di bilanciamento del carico per gli agenti, con il formato `<acsName>agents.<region>.cloudapp.azure.com`. Se ad esempio è stato creato un cluster del servizio contenitore con nome `myacs` nell'area `West US`, il nome di dominio completo sarà `myacsagents.westus.cloudapp.azure.com`. Per trovarlo è anche possibile cercare il servizio di bilanciamento del carico con "agent" nel nome quando si esaminano le risorse del gruppo di risorse creato per il servizio contenitore nel [portale di Azure](https://portal.azure.com).
  * Impostare servicePort su una porta >= 10.000. Viene così identificato il servizio in esecuzione nel contenitore; marathon-lb può così identificare i servizi tra cui eseguire il bilanciamento.
  * Impostare l'etichetta `HAPROXY_GROUP` su "external".
  * Impostare `hostPort` su 0. Ciò significa che Marathon allocherà arbitrariamente una porta disponibile.
  * Impostare `instances` sul numero di istanze da creare. È sempre possibile aumentare e ridurre il numero di istanze in seguito.

### Distribuire usando l'interfaccia utente Web di DC/OS

  1. Visitare la pagina Marathon all'indirizzo http://localhost/marathon, dopo avere configurato il [tunnel SSH](container-service-connect.md), quindi fare clic su `Create Appliction`.
  2. Nella finestra di dialogo `New Application` fare clic su `JSON Mode` nell'angolo superiore destro.
  3. Incollare il codice JSON precedente nell'editor.
  4. Fare clic su `Create Appliction`

### Distribuire usando l'interfaccia della riga di comando di DC/OS

Per distribuire questa applicazione con l'interfaccia della riga di comando di DC/OS, è sufficiente copiare il codice JSON precedente in un file denominato `hello-web.json`, quindi eseguire:

```bash
dcos marathon app add hello-web.json
```

## Servizio di bilanciamento del carico di Azure

Per impostazione predefinita, Azure Load Balancer espone le porte 80, 8080 e 443. Se si usa una di queste tre porte, come nell'esempio precedente, non è necessario eseguire alcuna operazione. Dovrebbe essere possibile accedere al nome FQDN del servizio di bilanciamento del carico dell'agente e, ogni volta che si esegue l'aggiornamento, si accederà a uno dei tre server Web in modalità round robin. Se tuttavia si usa una porta diversa, è necessario aggiungere una regola di round robin e un probe nel servizio di bilanciamento del carico per la porta usata. A questo scopo, immettere nell'[interfaccia della riga di comando di Azure](../xplat-cli-azure-resource-manager.md) i comandi `azure lb rule create` e `azure lb probe create`. Questa operazione può essere eseguita anche nel portale di Azure.


## Scenari aggiuntivi

È possibile avere uno scenario in cui si usano domini diversi per esporre servizi diversi, ad esempio:

mydomain1.com -> Azure LB:80 -> marathon-lb:10001 -> mycontainer1:33292 mydomain2.com -> Azure LB:80 -> marathon-lb:10002 -> mycontainer2:22321

A questo scopo, vedere la pagina relativa agli [host virtuali](https://mesosphere.com/blog/2015/12/04/dcos-marathon-lb/), che illustra come associare i domini a percorsi di marathon-lb specifici.

In alternativa è possibile esporre porte diverse e modificarne il mapping al servizio corretto dietro marathon lb, ad esempio:

Azure lb:80 -> marathon-lb:10001 -> mycontainer:233423 Azure lb:8080 -> marathon-lb:1002 -> mycontainer2:33432


## Passaggi successivi

Per altre informazioni su [marathon-lb](https://dcos.io/docs/1.7/usage/service-discovery/marathon-lb/), vedere la documentazione relativa a DC/OS.

<!---HONumber=AcomDC_0713_2016-->
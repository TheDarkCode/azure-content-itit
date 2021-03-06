<properties
   pageTitle="Aggiornamento di un'app di Service Fabric mediante PowerShell | Microsoft Azure"
   description="In questo articolo vengono esaminati l'esperienza di distribuzione di un'applicazione di Service Fabric, la modifica del codice e l'implementazione di un aggiornamento tramite PowerShell."
   services="service-fabric"
   documentationCenter=".net"
   authors="mani-ramaswamy"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.devlang="dotnet"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="NA"
   ms.date="06/13/2016"
   ms.author="subramar"/>




# Aggiornamento di un'applicazione di Service Fabric mediante PowerShell

Il metodo di aggiornamento consigliato usato più frequentemente è l'aggiornamento in sequenza in modalità monitorata (Monitored). Azure Service Fabric monitora l'integrità dell'applicazione che viene aggiornata in base a un set di criteri di integrità. Dopo che le applicazioni in un dominio di aggiornamento sono state aggiornate, Service Fabric ne valuta l'integrità e determina se passare al dominio di aggiornamento successivo o se considerare l'aggiornamento come non riuscito in base ai criteri di integrità.

L'aggiornamento di un'applicazione in modalità monitorata può essere eseguito usando le API gestite o native, PowerShell o REST. Per istruzioni su come eseguire un aggiornamento usando Visual Studio, vedere [Esercitazione sull'aggiornamento di un'applicazione di Service Fabric tramite Visual Studio](service-fabric-application-upgrade-tutorial.md).

L'aggiornamento in sequenza in modalità monitorata di Service Fabric consente all'amministratore di applicazioni di configurare i criteri di valutazione dell'integrità usati da Service Fabric per determinare se l'applicazione è integra. L'amministratore può anche configurare l'azione da intraprendere se la valutazione dell'integrità non riesce, ad esempio l'esecuzione di un rollback automatico. Questa sezione descrive in dettaglio un aggiornamento monitorato per uno degli esempi di SDK che usa PowerShell.

## Passaggio 1: creare e distribuire l'esempio di oggetti visivi


Compilare e pubblicare l'applicazione facendo clic con il pulsante destro del mouse sul progetto dell'applicazione, **VisualObjectsApplication**, e selezionare il comando **Pubblica** nella voce di menu di Service Fabric come indicato di seguito. Per altre informazioni, vedere [Esercitazione sull'aggiornamento di un'applicazione di Service Fabric tramite Visual Studio](service-fabric-application-upgrade-tutorial.md). In alternativa, è possibile usare PowerShell per distribuire l'applicazione.

> [AZURE.NOTE] Per poter usare uno dei comandi di Service Fabric in PowerShell è necessario connettersi prima al cluster con il cmdlet `Connect-ServiceFabricCluster`. Si presuppone che il cluster sia stato già configurato nel computer locale. Vedere l'articolo relativo alla [configurazione dell'ambiente di sviluppo di Service Fabric](service-fabric-get-started.md).

Dopo aver compilato il progetto in Visual Studio è possibile usare il comando di PowerShell **Copy-ServiceFabricApplicationPackage** per copiare il pacchetto dell'applicazione in ImageStore. Questo passaggio viene eseguito registrando l'applicazione nel runtime di Service Fabric con il cmdlet **Register-ServiceFabricApplicationPackage**. Il passaggio finale consiste nell'avviare un'istanza dell'applicazione con il cmdlet **New-ServiceFabricApplication**. Questi tre passaggi sono analoghi all'uso della voce di menu **Distribuisci** in Visual Studio.

È ora possibile usare [Service Fabric Explorer per visualizzare il cluster e l'applicazione](service-fabric-visualizing-your-cluster.md). L'applicazione dispone di un servizio Web a cui è possibile accedere in Internet Explorer digitando [http://localhost:8081/visualobjects](http://localhost:8081/visualobjects) sulla barra degli indirizzi. Verranno visualizzati alcuni oggetti visivi mobili sullo schermo. È anche possibile usare **Get-ServiceFabricApplication** per verificare lo stato dell'applicazione.

## Passaggio 2: aggiornare l'applicazione Oggetti visivi di esempio

È possibile osservare che con la versione distribuita al passaggio 1 gli oggetti visivi non ruotano. L'applicazione può essere aggiornata in modo che gli oggetti visivi ruotino.

Selezionare il progetto VisualObjects.ActorService nella soluzione VisualObjects e aprire il file StatefulVisualObjectActor.cs. All'interno del file passare al metodo `MoveObject`, impostare come commento `this.State.Move()` e rimuovere il commento `this.State.Move(true)`. Questa modifica fa ruotare gli oggetti dopo l'aggiornamento del servizio.

È anche necessario aggiornare il file *ServiceManifest.xml* (in PackageRoot) del progetto **VisualObjects.ActorService**. Aggiornare *CodePackage* e la versione del servizio alla versione 2.0, nonché le righe corrispondenti nel file *ServiceManifest.xml*. È possibile usare l'opzione *Edit Manifest Files* (Modifica file manifesto) di Visual Studio dopo aver fatto clic con il pulsante destro del mouse sulla soluzione per apportare le modifiche al file manifesto.


Dopo aver apportato le modifiche, il manifesto dovrebbe essere simile al seguente (le parti evidenziate indicano le modifiche):

```xml
<ServiceManifestName="VisualObjects.ActorService" Version="2.0" xmlns="http://schemas.microsoft.com/2011/01/fabric" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">

<CodePackageName="Code" Version="2.0">
```

A questo punto è necessario aggiornare il file *ApplicationManifest.xml*, disponibile nel progetto **VisualObjects** della soluzione **VisualObjects**, in modo da usare la versione 2.0 del progetto **VisualObjects.ActorService** e aggiornare anche la versione dell'applicazione da 1.0.0.0 a 2.0.0.0. Le righe corrispondenti nel file *ApplicationManifest.xml* saranno ora simili alle seguenti:

```xml
<ApplicationManifestxmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" ApplicationTypeName="VisualObjects" ApplicationTypeVersion="2.0.0.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">

 <ServiceManifestRefServiceManifestName="VisualObjects.ActorService" ServiceManifestVersion="2.0" />
```


A questo punto, compilare il progetto selezionando solo il progetto **ActorService** e quindi facendo clic con il pulsante destro del mouse e scegliendo **Compila** in Visual Studio. Se si sceglie **Ricompila tutto**, potrebbe essere necessario aggiornare le versioni anche per gli altri progetti e in *ServiceManifest.xml* e *ApplicationManifest.xml*, perché il codice è stato modificato. È quindi possibile aggiungere l'applicazione aggiornata al pacchetto facendo clic con il pulsante destro del mouse sul progetto ***VisualObjectsApplication***, scegliendo il menu Service Fabric e quindi **Pacchetto**. In questo modo verrà creato un pacchetto applicazione che può essere distribuito. L'applicazione aggiornata è ora pronta per essere distribuita.


## Passaggio 3: Scegliere i criteri di integrità e i parametri di aggiornamento

È consigliabile acquisire familiarità con i [parametri di aggiornamento dell'applicazione](service-fabric-application-upgrade-parameters.md) e con il [processo di aggiornamento](service-fabric-application-upgrade.md) per conoscere i diversi parametri di aggiornamento, valori di timeout e criteri di integrità applicati. Per questa procedura dettagliata si manterranno i criteri predefiniti di valutazione dell'integrità dei servizi, con i valori consigliati, quindi tutti i servizi e tutte le istanze saranno _integri_ dopo l'aggiornamento.

Aumentare tuttavia il valore di *HealthCheckStableDuration* impostandolo su 60 secondi, in modo che i servizi siano integri per almeno 20 secondi prima che il processo di aggiornamento passi al dominio di aggiornamento successivo. Impostare inoltre *UpgradeDomainTimeout* su 1200 secondi e *UpgradeTimeout* su 3000 secondi.

Impostare infine *UpgradeFailureAction* sull'azione di rollback richiedendo in questo modo a Service Fabric di eseguire il rollback dell'applicazione alla versione precedente, in caso di problemi durante l'aggiornamento. I parametri di aggiornamento che verranno specificati all'avvio dell'aggiornamento, nel passaggio 4, saranno quindi i seguenti:

FailureAction = Rollback

HealthCheckStableDurationSec = 60

UpgradeDomainTimeoutSec = 1200

UpgradeTimeout = 3000


## Passaggio 4: preparare l'applicazione per l'aggiornamento

L'applicazione ora è compilata e pronta per l'aggiornamento. Se si apre una finestra di PowerShell come amministratore e si digita **Get-ServiceFabricApplication**, verrà indicato che è stato distribuito il tipo di applicazione 1.0.0.0 di **VisualObjects**.

Il pacchetto applicazione è archiviato nel percorso relativo seguente, dove è stato decompresso Service Fabric SDK - *Samples\\Services\\Stateful\\VisualObjects\\VisualObjects\\obj\\x64\\Debug*. In questa directory deve essere disponibile una cartella "Package", dove è archiviato il pacchetto dell'applicazione. Controllare i timestamp per assicurarsi che si tratti dell'ultima build. Potrebbe essere anche necessario modificare i percorsi in base alle esigenze.

Copiare ora il pacchetto applicazione aggiornato in ImageStore di Service Fabric (dove Service Fabric archivia i pacchetti applicazione). Il parametro *ApplicationPackagePathInImageStore* indica a Service Fabric dove è contenuto il pacchetto applicazione. L'applicazione aggiornata è stata inserita in "VisualObjects\_V2" con il comando seguente. Potrebbe essere di nuovo necessario modificare i percorsi in base alle esigenze.

```powershell
Copy-ServiceFabricApplicationPackage  -ApplicationPackagePath .\Samples\Services\Stateful\VisualObjects\VisualObjects\obj\x64\Debug\Package
-ImageStoreConnectionString fabric:ImageStore   -ApplicationPackagePathInImageStore "VisualObjects\_V2"
```

Il passaggio successivo consiste nel registrare l'applicazione in Service Fabric. A tale scopo, usare il comando seguente:

```powershell
Register-ServiceFabricApplicationType -ApplicationPathInImageStore "VisualObjects\_V2"
```

Se il comando precedente non riesce, è probabile che si debbano ricompilare tutti i servizi. Come indicato nel passaggio 2, potrebbe essere necessario aggiornare anche la versione di WebService.

## Passaggio 5: avviare l'aggiornamento dell'applicazione

È ora possibile avviare l'aggiornamento dell'applicazione con il comando seguente:

```powershell
Start-ServiceFabricApplicationUpgrade -ApplicationName fabric:/VisualObjects -ApplicationTypeVersion 2.0.0.0 -HealthCheckStableDurationSec 60 -UpgradeDomainTimeoutSec 1200 -UpgradeTimeout 3000   -FailureAction Rollback -Monitored
```


Si noti che il nome dell'applicazione è lo stesso descritto nel file *ApplicationManifest.xml*. Service Fabric usa questo nome per identificare l'applicazione che viene aggiornata. Se si imposta un valore di timeout troppo breve, è possibile che venga visualizzato un messaggio di errore che indica il problema. Vedere la sezione relativa alla risoluzione dei problemi o aumentare i valori di timeout.

Man mano che procede l'aggiornamento dell'applicazione, è possibile monitorarlo con Service Fabric Explorer oppure con il comando di PowerShell **Get-ServiceFabricApplicationUpgrade fabric:/VisualObjects**.

In pochi minuti lo stato ottenuto usando il comando di PowerShell precedente indicherà che sono stati aggiornati (completati) tutti i domini di aggiornamento. A questo punto gli oggetti visivi nella finestra del browser avranno iniziato a ruotare.

È possibile provare a cambiare le versioni e a passare dalla versione 2 alla versione 3 come esercitazione oppure addirittura dalla versione 2 alla versione 1 (è possibile effettuare l'aggiornamento dalla versione 2 alla versione 1). Esercitarsi inoltre con i timeout e i criteri di integrità per acquisire familiarità anche con questi concetti. Per la distribuzione in un cluster di Azure, i parametri usati saranno diversi da quelli usati per la distribuzione in un cluster locale. È quindi consigliabile impostare i timeout su valori prudenziali.


## Passaggi successivi

[Esercitazione sull'aggiornamento di un'applicazione di Service Fabric tramite Visual Studio](service-fabric-application-upgrade-tutorial.md) illustra l'aggiornamento di un'applicazione con Visual Studio.

Controllare l'aggiornamento dell'applicazione usando [parametri di aggiornamento](service-fabric-application-upgrade-parameters.md).

Rendere compatibili gli aggiornamenti dell'applicazione imparando a usare la [serializzazione dei dati](service-fabric-application-upgrade-data-serialization.md).

Informazioni su come usare funzionalità avanzate durante l'aggiornamento dell'applicazione facendo riferimento ad [Argomenti avanzati](service-fabric-application-upgrade-advanced.md).

Risolvere problemi comuni negli aggiornamenti dell'applicazione facendo riferimento ai passaggi elencati in [Risoluzione dei problemi di aggiornamento delle applicazioni](service-fabric-application-upgrade-troubleshooting.md).

<!---HONumber=AcomDC_0615_2016-->
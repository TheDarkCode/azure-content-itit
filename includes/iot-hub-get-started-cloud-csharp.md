## Creare un'identità del dispositivo

In questa sezione si scriverà un'app console di Windows che consente di creare una nuova identità del dispositivo nel registro delle identità dell'hub IoT. Un dispositivo non può connettersi all'hub IoT a meno che non sia presente una voce nel registro delle identità del dispositivo. Fare riferimento alla sezione **Registro identità del dispositivo** della [Guida per sviluppatori di hub IoT][lnk-devguide-identity] per ulteriori informazioni. Quando si esegue questa applicazione console, viene generato un ID dispositivo univoco e una chiave con cui il dispositivo può identificarsi quando invia messaggi da dispositivo a cloud all'hub IoT.

1. In Visual Studio aggiungere un nuovo progetto desktop di Windows classico in Visual C# usando il modello di progetto **Applicazione console**. Assicurarsi che la versione di .NET Framework sia 4.5.1 o successiva. Denominare il progetto **CreateDeviceIdentity**.

	![][10]

2. In Esplora soluzioni fare clic con il pulsante destro del mouse sul progetto **CreateDeviceIdentity** e quindi scegliere **Gestisci pacchetti NuGet**.

3. Nella finestra **Gestione pacchetti NuGet** scegliere **Sfoglia**, cercare **microsoft.azure.devices**, fare clic su **Installa** per installare il pacchetto **Microsoft.Azure.Devices** e accettare le condizioni per l'utilizzo.

	![][11]

4. Viene scaricato e installato il pacchetto NuGet [Microsoft Azure IoT Service SDK][lnk-nuget-service-sdk] e viene aggiunto un riferimento a tale pacchetto e alle relative dipendenze.

4. Aggiungere le istruzione `using` seguenti all'inizio del file **Program.cs**:

		using Microsoft.Azure.Devices;
        using Microsoft.Azure.Devices.Common.Exceptions;

5. Aggiungere i campi seguenti alla classe **Program**, sostituendo i valori dei segnaposto con la stringa di connessione dell'hub IoT creato nella sezione precedente:

		static RegistryManager registryManager;
        static string connectionString = "{iothub connection string}";

6. Aggiungere il metodo seguente alla classe **Program**:

		private static async Task AddDeviceAsync()
        {
            string deviceId = "myFirstDevice";
            Device device;
            try
            {
                device = await registryManager.AddDeviceAsync(new Device(deviceId));
            }
            catch (DeviceAlreadyExistsException)
            {
                device = await registryManager.GetDeviceAsync(deviceId);
            }
            Console.WriteLine("Generated device key: {0}", device.Authentication.SymmetricKey.PrimaryKey);
        }

	Questo metodo crea una nuova identità del dispositivo con l'ID **myFirstDevice**. Se questo ID dispositivo è già disponibile nel registro, il codice recupera semplicemente le informazioni sul dispositivo esistenti. L'app visualizzerà quindi la chiave primaria per l'identità. Questa chiave verrà usata dal dispositivo simulato per connettersi all'hub IoT.

7. Aggiungere infine le righe seguenti al metodo **Main**:

		registryManager = RegistryManager.CreateFromConnectionString(connectionString);
        AddDeviceAsync().Wait();
        Console.ReadLine();

8. Eseguire l'applicazione e prendere nota della chiave del dispositivo.

    ![][12]

> [AZURE.NOTE] Il registro delle identità dell'hub IoT consente di archiviare solo le identità del dispositivo per abilitare l'accesso sicuro all'hub. Archivia gli ID dispositivo e le chiavi da usare come credenziali di sicurezza e un flag di attivazione/disattivazione che consente di disabilitare l'accesso per un singolo dispositivo. Se l'applicazione deve archiviare altri metadati specifici del dispositivo, dovrà usare un archivio specifico dell'applicazione. Per altre informazioni, vedere la [Guida per sviluppatori di hub IoT][lnk-devguide-identity].

## Ricezione di messaggi da dispositivo a cloud

In questa sezione si creerà un'app console di Windows che legge i messaggi da dispositivo a cloud dall'hub IoT. L'hub IoT espone un endpoint compatibile con [Hub eventi][lnk-event-hubs-overview] per abilitare la lettura dei messaggi da dispositivo a cloud. Per semplicità, questa esercitazione crea un lettore di base non adatto per una distribuzione con velocità effettiva elevata. L'esercitazione [Elaborare messaggi da dispositivo a cloud][lnk-processd2c-tutorial] illustra come elaborare i messaggi da dispositivo a cloud su vasta scala. L'esercitazione [Introduzione a Hub eventi][lnk-eventhubs-tutorial] fornisce altre informazioni su come elaborare i messaggi da Hub eventi ed è applicabile agli endpoint compatibili con l'hub eventi dell'hub IoT.

> [AZURE.NOTE] L'endpoint compatibile con Hub eventi per la lettura di messaggi da dispositivo a cloud usa sempre il protocollo AMQPS.

1. In Visual Studio aggiungere un nuovo progetto desktop di Windows classico in Visual C# usando il modello di progetto **Applicazione console**. Assicurarsi che la versione di .NET Framework sia 4.5.1 o successiva. Denominare il progetto **ReadDeviceToCloudMessages**.

    ![][10]

2. In Esplora soluzioni, fare clic con il pulsante destro del mouse sul progetto **ReadDeviceToCloudMessages** e quindi scegliere **Gestisci pacchetti NuGet**.

3. Nella finestra **Gestione pacchetti NuGet** cercare **WindowsAzure.ServiceBus**, fare clic su **Installa** e accettare le condizioni per l'utilizzo.

    Verrà quindi scaricato e installato il [bus di servizio di Azure][lnk-servicebus-nuget] e verrà aggiunto un riferimento ad esso. Questo pacchetto consente all'applicazione di connettersi all'endpoint compatibile con l'hub eventi dell'hub IoT.

4. Aggiungere le istruzione `using` seguenti all'inizio del file **Program.cs**:

        using Microsoft.ServiceBus.Messaging;
        using System.Threading;

5. Aggiungere i campi seguenti alla classe **Program**, sostituendo i valori dei segnaposto con la stringa di connessione dell'hub IoT creato nella sezione *Creare un hub IoT*:

        static string connectionString = "{iothub connection string}";
        static string iotHubD2cEndpoint = "messages/events";
        static EventHubClient eventHubClient;

6. Aggiungere il metodo seguente alla classe **Program**:

        private static async Task ReceiveMessagesFromDeviceAsync(string partition, CancellationToken ct)
        {
            var eventHubReceiver = eventHubClient.GetDefaultConsumerGroup().CreateReceiver(partition, DateTime.UtcNow);
            while (true)
            {
                if (ct.IsCancellationRequested) break;
                EventData eventData = await eventHubReceiver.ReceiveAsync();
                if (eventData == null) continue;

                string data = Encoding.UTF8.GetString(eventData.GetBytes());
                Console.WriteLine("Message received. Partition: {0} Data: '{1}'", partition, data);
            }
        }

    Questo metodo usa un'istanza **EventHubReceiver** per ricevere i messaggi da tutte le partizioni di ricezione da dispositivo a cloud dell'hub IoT. Si noti come passare un parametro `DateTime.Now` quando si crea l'oggetto **EventHubReceiver** in modo che riceva solo i messaggi inviati dopo l'avvio. Ciò è utile in un ambiente di test perché consente di visualizzare il set di messaggi corrente, ma in un ambiente di produzione il codice deve verificare l'elaborazione di tutti i messaggi. Per altre informazioni, vedere l'esercitazione [Elaborare messaggi da dispositivo a cloud dell'hub IoT][lnk-processd2c-tutorial].

7. Aggiungere infine le righe seguenti al metodo **Main**:

        Console.WriteLine("Receive messages. Ctrl-C to exit.\n");
        eventHubClient = EventHubClient.CreateFromConnectionString(connectionString, iotHubD2cEndpoint);

        var d2cPartitions = eventHubClient.GetRuntimeInformation().PartitionIds;

        CancellationTokenSource cts = new CancellationTokenSource();

        System.Console.CancelKeyPress += (s, e) =>
        {
          e.Cancel = true;
          cts.Cancel();
          Console.WriteLine("Exiting...");
        };

        var tasks = new List<Task>();
        foreach (string partition in d2cPartitions)
        {
           tasks.Add(ReceiveMessagesFromDeviceAsync(partition, cts.Token));
        }  
        Task.WaitAll(tasks.ToArray());


<!-- Links -->

[lnk-eventhubs-tutorial]: ../articles/event-hubs/event-hubs-csharp-ephcs-getstarted.md
[lnk-devguide-identity]: ../articles/iot-hub/iot-hub-devguide.md#identityregistry
[lnk-servicebus-nuget]: https://www.nuget.org/packages/WindowsAzure.ServiceBus
[lnk-event-hubs-overview]: ../articles/event-hubs/event-hubs-overview.md

[lnk-nuget-service-sdk]: https://www.nuget.org/packages/Microsoft.Azure.Devices/
[lnk-processd2c-tutorial]: ../articles/iot-hub/iot-hub-csharp-csharp-process-d2c.md

<!-- Images -->
[10]: ./media/iot-hub-getstarted-cloud-csharp/create-identity-csharp1.png
[11]: ./media/iot-hub-getstarted-cloud-csharp/create-identity-csharp2.png
[12]: ./media/iot-hub-getstarted-cloud-csharp/create-identity-csharp3.png

<!---HONumber=AcomDC_0413_2016-->
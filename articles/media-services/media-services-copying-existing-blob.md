<properties 
	pageTitle="Copia di un BLOB esistente in un asset di Servizi multimediali" 
	description="Questo argomento illustra come copiare un BLOB esistente in un asset di Servizi multimediali." 
	services="media-services" 
	documentationCenter="" 
	authors="Juliako" 
	manager="erikre" 
	editor=""/>

<tags 
	ms.service="media-services" 
	ms.workload="media" 
	ms.tgt_pltfrm="na" 
	ms.devlang="ne" 
	ms.topic="article" 
	ms.date="06/22/2016" 
	ms.author="juliako"/>

#Copia di un BLOB esistente in un asset di Servizi multimediali

Questo argomento illustra come copiare BLOB da un account di archiviazione in un nuovo asset di Servizi multimediali di Microsoft Azure.

I BLOB possono essere presenti sia in un account di archiviazione associato al proprio account di Servizi multimediali sia in un account di archiviazione non associato all'account di Servizi multimediali. Questo argomento illustra come copiare BLOB da un account di archiviazione in un asset di Servizi multimediali. Tenere presente che è possibile copiare anche in più data center. Questa operazione, tuttavia, può comportare l'addebito di costi. Per altre informazioni sui prezzi, vedere [Trasferimenti di dati](https://azure.microsoft.com/pricing/#header-11).

>[AZURE.NOTE] È sconsigliabile tentare di modificare i contenuti dei contenitori BLOB generati da Servizi multimediali senza usare le API di Servizi multimediali.

##Scaricare un esempio

È possibile ottenere ed eseguire un esempio [qui](https://azure.microsoft.com/documentation/samples/media-services-dotnet-copy-blob-into-asset/).

##Prerequisiti

- Due account di Servizi multimediali in una sottoscrizione di Azure nuova o esistente. Vedere l'argomento [Come creare un account di Servizi multimediali](media-services-create-account.md).
- Sistemi operativi: Windows 10, Windows 7, Windows 2008 R2 o Windows 8.
- .NET Framework 4.5
- Visual Studio 2010, SP1 (Professional, Premium, Ultimate, o Express) o versioni successive.

##Configurare il progetto

In questa sezione si creerà e si configurerà un progetto di applicazione console in C#.

1. Usare Visual Studio per creare una nuova soluzione contenente il progetto di applicazione console in C#.
2. Immettere CopyExistingBlobsIntoAsset per il nome e quindi fare clic su OK.
1. Usare Nuget per aggiungere riferimenti alle DLL correlate a Servizi multimediali. Nel menu principale di Visual Studio scegliere STRUMENTI -> Gestione pacchetti libreria -> Console di Gestione pacchetti. Nella finestra della console digitare Install-Package windowsazure.mediaservices e premere INVIO.
1. Aggiungere gli altri riferimenti richiesti per il progetto System.Configuration.
1. Sostituire le istruzioni using aggiunte per impostazione predefinita al file Programs.cs con le seguenti:
		
		using System;
		using System.Linq;
		using System.Configuration;
		using System.IO;
		using System.Text;
		using System.Threading;
		using System.Threading.Tasks;
		using System.Collections.Generic;
		using Microsoft.WindowsAzure.Storage;
		using Microsoft.WindowsAzure.MediaServices.Client;
		using Microsoft.WindowsAzure;
		using System.Web;
		using Microsoft.WindowsAzure.Storage.Blob;
		using Microsoft.WindowsAzure.Storage.Auth;

1. Aggiungere la sezione appSettings al file di configurazione e aggiornare i valori in base ai valori di chiave e nome dell'account di Servizi multimediali e di archiviazione.

		<appSettings>
		  <add key="MediaServicesAccountName" value="Media-Services-Account-Name"/>
		  <add key="MediaServicesAccountKey" value="Media-Services-Account-Key"/>
		  <add key="MediaServicesStorageAccountName" value="Media-Services-Storage-Account-Name"/>
		  <add key="MediaServicesStorageAccountKey" value="Media-Services-Storage-Account-Key"/>
		  <add key="ExternalStorageAccountName" value="External-Storage-Account-Name"/>
		  <add key="ExternalStorageAccountKey" value="External-Storage-Account-Key"/>
		</appSettings>


##Copiare BLOB da un account di archiviazione in un asset di Servizi multimediali

L'esempio di codice che segue effettua le attività seguenti:

1. Crea l'istanza di CloudMediaContext.
1. Crea le istanze di CloudStorageAccount: \_sourceStorageAccount e \_destinationStorageAccount.
1. Carica i file Smooth Streaming da una directory locale in un contenitore BLOB che si trova in \_sourceStorageAccount.
1. Crea un nuovo asset. Il contenitore BLOB creato per questo asset si trova in \_destinationStorageAccount.
1. Usa Azure Storage SDK per copiare i BLOB specificati nel contenitore associato all'asset.

	>[AZURE.NOTE]L'operazione di copia non genera eccezioni se il localizzatore è scaduto.

1. Dato che in questo esempio vengono copiati file Smooth Streaming, nell'esempio viene illustrato come impostare il file con estensione ism come file primario. Se ad esempio viene copiato un file con estensione mp4, il file mp4 viene impostato come file primario.
1. Crea l'URL di Smooth Streaming per il localizzatore OnDemandOrigin associato all'asset.
			
		class Program
	        {
	            // Read values from the App.config file. 
	            static string _accountName = ConfigurationManager.AppSettings["MediaServicesAccountName"];
	            static string _accountKey = ConfigurationManager.AppSettings["MediaServicesAccountKey"];
	            static string _storageAccountName = ConfigurationManager.AppSettings["MediaServicesStorageAccountName"];
	            static string _storageAccountKey = ConfigurationManager.AppSettings["MediaServicesStorageAccountKey"];
	            static string _externalStorageAccountName = ConfigurationManager.AppSettings["ExternalStorageAccountName"];
	            static string _externalStorageAccountKey = ConfigurationManager.AppSettings["ExternalStorageAccountKey"];
	
	            private static MediaServicesCredentials _cachedCredentials = null;
	            private static CloudMediaContext _context = null;
	
	            private static CloudStorageAccount _sourceStorageAccount = null;
	            private static CloudStorageAccount _destinationStorageAccount = null;
	
	            static void Main(string[] args)
	            {
	                _cachedCredentials = new MediaServicesCredentials(
	                                _accountName,
	                                _accountKey);
	                // Use the cached credentials to create CloudMediaContext.
	                _context = new CloudMediaContext(_cachedCredentials);
	
	                // In this example the storage account from which we copy blobs is not 
	                // associated with the Media Services account into which we copy blobs.
	                // But the same code will work for coping blobs from a storage account that is 
	                // associated with the Media Services account.
	                //
	                // Get a reference to a storage account that is not associated with a Media Services account
	                // (an external account).  
	                StorageCredentials externalStorageCredentials =
	                    new StorageCredentials(_externalStorageAccountName, _externalStorageAccountKey);
	                _sourceStorageAccount = new CloudStorageAccount(externalStorageCredentials, true);
	
	                //Get a reference to the storage account that is associated with a Media Services account. 
	                StorageCredentials mediaServicesStorageCredentials =
	                    new StorageCredentials(_storageAccountName, _storageAccountKey);
	                _destinationStorageAccount = new CloudStorageAccount(mediaServicesStorageCredentials, false);
	
	                // Upload Smooth Streaming files into a storage account.
	                string localMediaDir = @"C:\supportFiles\streamingfiles";
	                CloudBlobContainer blobContainer =
	                    UploadContentToStorageAccount(localMediaDir);
	
	                // Create a new asset and copy the smooth streaming files into 
	                // the container that is associated with the asset.
	                IAsset asset = CreateAssetFromExistingBlobs(blobContainer);
	
	                // Get the streaming URL for the smooth streaming files 
	                // that were copied into the asset.   
	                string urlForClientStreaming = CreateStreamingLocator(asset);
	                Console.WriteLine("Smooth Streaming URL: " + urlForClientStreaming);
	
	                Console.ReadLine();
	            }
	
	            /// <summary>
	            /// Uploads content from a local directory into the specified storage account.
	            /// In this example the storage account is not associated with the Media Services account.
	            /// </summary>
	            /// <param name="localPath">The path from which to upload the files.</param>
	            /// <returns>The container that contains the uploaded files.</returns>
	            static public CloudBlobContainer UploadContentToStorageAccount(string localPath)
	            {
	                CloudBlobClient externalCloudBlobClient = _sourceStorageAccount.CreateCloudBlobClient();
	
	                CloudBlobContainer externalMediaBlobContainer = externalCloudBlobClient.GetContainerReference("streamingfiles");
	
	                externalMediaBlobContainer.CreateIfNotExists();
	
	                // Upload files to the blob container.  
	                DirectoryInfo uploadDirectory = new DirectoryInfo(localPath);
	                foreach (var file in uploadDirectory.EnumerateFiles())
	                {
	                    CloudBlockBlob blob = externalMediaBlobContainer.GetBlockBlobReference(file.Name);
	
	                    blob.UploadFromFile(file.FullName, FileMode.Open);
	                }
	
	                return externalMediaBlobContainer;
	            }
	
	            /// <summary>
	            /// Creates a new asset and copies blobs from the specifed storage account.
	            /// </summary>
	            /// <param name="mediaBlobContainer">The specified blob container.</param>
	            /// <returns>The new asset.</returns>
	            static public IAsset CreateAssetFromExistingBlobs(CloudBlobContainer mediaBlobContainer)
	            {
	                // Create a new asset. 
	                IAsset asset = _context.Assets.Create("NewAsset_" + Guid.NewGuid(), AssetCreationOptions.None);
	
	                IAccessPolicy writePolicy = _context.AccessPolicies.Create("writePolicy",
	                    TimeSpan.FromHours(24), AccessPermissions.Write);
	                ILocator destinationLocator = _context.Locators.CreateLocator(LocatorType.Sas, asset, writePolicy);
	
	                CloudBlobClient destBlobStorage = _destinationStorageAccount.CreateCloudBlobClient();
	
	                // Get the asset container URI and Blob copy from mediaContainer to assetContainer. 
	                string destinationContainerName = (new Uri(destinationLocator.Path)).Segments[1];
	
	                CloudBlobContainer assetContainer =
	                    destBlobStorage.GetContainerReference(destinationContainerName);
	
	                if (assetContainer.CreateIfNotExists())
	                {
	                    assetContainer.SetPermissions(new BlobContainerPermissions
	                    {
	                        PublicAccess = BlobContainerPublicAccessType.Blob
	                    });
	                }
	
	                var blobList = mediaBlobContainer.ListBlobs();
	                foreach (var sourceBlob in blobList)
	                {
	                    var assetFile = asset.AssetFiles.Create((sourceBlob as ICloudBlob).Name);
	                    CopyBlob(sourceBlob as ICloudBlob, assetContainer);
	                    assetFile.ContentFileSize = (sourceBlob as ICloudBlob).Properties.Length;
	                    assetFile.Update();
	                }
	
	                asset.Update();
	
	                destinationLocator.Delete();
	                writePolicy.Delete();
	
	                // Since we copied a set of Smooth Streaming files, 
	                // set the .ism file to be the primary file. 
	                // If we, for example, copied an .mp4, then the mp4 would be the primary file. 
	                SetISMFileAsPrimary(asset);
	
	                return asset;
	            }
	
	            /// <summary>
	            /// Creates the OnDemandOrigin locator in order to get the streaming URL.
	            /// </summary>
	            /// <param name="asset">The asset that contains the smooth streaming files.</param>
	            /// <returns>The streaming URL.</returns>
	            static public string CreateStreamingLocator(IAsset asset)
	            {
	                var ismAssetFile = asset.AssetFiles.ToList().
	                    Where(f => f.Name.EndsWith(".ism", StringComparison.OrdinalIgnoreCase)).First();
	
	                // Create a 30-day readonly access policy. 
	                IAccessPolicy policy = _context.AccessPolicies.Create("Streaming policy",
	                    TimeSpan.FromDays(30),
	                    AccessPermissions.Read);
	
	                // Create a locator to the streaming content on an origin. 
	                ILocator originLocator = _context.Locators.CreateLocator(LocatorType.OnDemandOrigin, asset,
	                    policy,
	                    DateTime.UtcNow.AddMinutes(-5));
	
	                return originLocator.Path + ismAssetFile.Name + "/manifest";
	            }
	
	            /// <summary>
	            /// Copies the specified blob into the specified container.
	            /// </summary>
	            /// <param name="sourceBlob">The source container.</param>
	            /// <param name="destinationContainer">The destination container.</param>
	            static private void CopyBlob(ICloudBlob sourceBlob, CloudBlobContainer destinationContainer)
	            {
	                var signature = sourceBlob.GetSharedAccessSignature(new SharedAccessBlobPolicy
	                {
	                    Permissions = SharedAccessBlobPermissions.Read,
	                    SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24)
	                });
	
	                ICloudBlob destinationBlob = destinationContainer.GetBlockBlobReference(sourceBlob.Name);
	
	                if (destinationBlob.Exists())
	                {
	                    Console.WriteLine(string.Format("Destination blob '{0}' already exists. Skipping.", destinationBlob.Uri));
	                }
	                else
	                {
	
	                    // Display the size of the source blob.
	                    Console.WriteLine(sourceBlob.Properties.Length);
	
	                    Console.WriteLine(string.Format("Copy blob '{0}' to '{1}'", sourceBlob.Uri, destinationBlob.Uri));
	                    destinationBlob.StartCopyFromBlob(new Uri(sourceBlob.Uri.AbsoluteUri + signature));
	
	                    while (true)
	                    {
	                        // The StartCopyFromBlob is an async operation, 
	                        // so we want to check if the copy operation is completed before proceeding. 
	                        // To do that, we call FetchAttributes on the blob and check the CopyStatus. 
	                        destinationBlob.FetchAttributes();
	                        if (destinationBlob.CopyState.Status != CopyStatus.Pending)
	                        {
	                            break;
	                        }
	                        //It's still not completed. So wait for some time.
	                        System.Threading.Thread.Sleep(1000);
	                    }
	
	
	                    // Display the size of the destination blob.
	                    Console.WriteLine(destinationBlob.Properties.Length);
	
	                }
	            }
	
	            /// <summary>
	            /// Sets a file with the .ism extension as a primary file.
	            /// </summary>
	            /// <param name="asset">The asset that contains the smooth streaming files.</param>
	            static private void SetISMFileAsPrimary(IAsset asset)
	            {
	
	                //If you expect the asset to contain the .ism asset file, set the .ism file as the primary file.
	                var ismAssetFiles = asset.AssetFiles.ToList().
	                    Where(f => f.Name.EndsWith(".ism", StringComparison.OrdinalIgnoreCase)).ToArray();
	
	                // The following code assigns the first .ism file as the primary file in the asset.
	                // An asset should have one .ism file.  
	                ismAssetFiles.First().IsPrimary = true;
	                ismAssetFiles.First().Update();
	            }
	        }

 

##Percorsi di apprendimento di Media Services

[AZURE.INCLUDE [media-services-learning-paths-include](../../includes/media-services-learning-paths-include.md)]

##Fornire commenti e suggerimenti

[AZURE.INCLUDE [media-services-user-voice-include](../../includes/media-services-user-voice-include.md)]

<!---HONumber=AcomDC_0629_2016-->
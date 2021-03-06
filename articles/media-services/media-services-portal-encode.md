<properties
	pageTitle="Codificare un asset mediante Media Encoder Standard con il portale di Azure | Microsoft Azure"
	description="Questa esercitazione descrive le operazioni di codifica di un asset mediante Media Encoder Standard con il portale di Azure."
	services="media-services"
	documentationCenter=""
	authors="Juliako"
	manager="erikre"
	editor=""/>

<tags
	ms.service="media-services"
	ms.workload="media"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="08/29/2016"
	ms.author="juliako"/>


# Codificare un asset mediante Media Encoder Standard con il portale di Azure

> [AZURE.NOTE] Per completare l'esercitazione, è necessario un account Azure. Per informazioni dettagliate, vedere la pagina relativa alla [versione di valutazione gratuita di Azure](https://azure.microsoft.com/pricing/free-trial/).

Quando si usa Servizi multimediali di Azure, uno degli scenari più frequenti consiste nella distribuzione di contenuti in streaming a velocità in bit adattiva ai client. Servizi multimediali supporta le seguenti tecnologie di streaming a velocità in bit adattiva: HTTP Live Streaming (HLS), Smooth Streaming, MPEG DASH e HDS (solo per i titolari di licenza Adobe PrimeTime/Access). Per preparare i video per lo streaming a bitrate adattivo, è necessario codificare il video di origine in file a più bitrate. Per codificare i video, è consigliabile usare il codificatore **Media Encoder Standard**.

Servizi multimediali include la funzionalità per la creazione dinamica dei pacchetti, che consente di distribuire file MP4 a più bitrate nei formati MPEG DASH, HLS, Smooth Streaming or HDS, senza dover ricreare i pacchetti con questi formati di streaming. Con la creazione dinamica dei pacchetti si archiviano e si pagano solo i file in un unico formato di archiviazione e Servizi multimediali crea e fornisce la risposta appropriata in base alle richieste di un client.

Per sfruttare i vantaggi del servizio di creazione dinamica dei pacchetti, è necessario seguire questa procedura:

- Codificare il file di origine in un set di file MP4 a più bitrate. La procedura per la codifica è descritta più avanti in questa esercitazione.
- Ottenere almeno un'unità di streaming per l'endpoint di streaming da cui si pianifica la distribuzione dei contenuti. Per altre informazioni, vedere la sezione [Configurare gli endpoint di streaming](media-services-portal-vod-get-started.md#configure-streaming-endpoints).

Per ridimensionare l'elaborazione multimediale, vedere [questo](media-services-portal-scale-media-processing.md) argomento.

## Eseguire la codifica con il portale di Azure

Questa sezione descrive la procedura per la codifica di contenuti con Media Encoder Standard.

1.  Nella finestra **Impostazioni** selezionare **Asset**.
2.  Nella finestra **Asset** selezionare la risorsa che si vuole codificare.
3.  Fare clic sul pulsante **Codifica**.
4.  Nella finestra **Codifica un asset**, selezionare il processore "Media Encoder Standard" e un set di impostazioni. Ad esempio, se è noto che il video di input ha una risoluzione di 1920x1080 pixel, è possibile usare il set di impostazioni "Codec video H.264 a bitrate multiplo con risoluzione 1080p". Per altre informazioni sui set di impostazioni, vedere [questo](https://msdn.microsoft.com/library/azure/mt269960.aspx) articolo. È importante selezionare il set di impostazioni più appropriato per il video di input. Se il video disponibile è a bassa risoluzione (640x360), non usare il set di impostazioni "Codec video H.264 a bitrate multiplo con risoluzione 1080p".
	
	Per una gestione più semplice, è possibile modificare il nome dell'asset di output e il nome del processo.
		
	![Codificare gli asset](./media/media-services-portal-vod-get-started/media-services-encode1.png)
5. Fare clic su **Crea**.


##Passaggio successivo

È possibile monitorare lo stato del processo di codifica con il portale di Azure, come descritto in [questo](media-services-portal-check-job-progress.md) articolo.

##Percorsi di apprendimento di Servizi multimediali

[AZURE.INCLUDE [media-services-learning-paths-include](../../includes/media-services-learning-paths-include.md)]

##Fornire commenti e suggerimenti

[AZURE.INCLUDE [media-services-user-voice-include](../../includes/media-services-user-voice-include.md)]

<!---HONumber=AcomDC_0831_2016-->
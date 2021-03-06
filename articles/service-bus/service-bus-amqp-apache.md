<properties 
    pageTitle="Come installare Apache Qpid Proton-C in una macchina virtuale Linux | Microsoft Azure"
    description="Come creare una macchina virtuale Linux CentOS mediante Macchine virtuali di Azure e come creare e installare la libreria Apache Qpid Proton-C."
    services="service-bus"
    documentationCenter="na"
    authors="sethmanheim"
    manager="timlt"
    editor="" /> 
<tags 
    ms.service="service-bus"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="05/06/2016"
    ms.author="sethm" />

# Installare Apache Qpid Proton-C in una macchina virtuale Linux di Azure

[AZURE.INCLUDE [service-bus-selector-amqp](../../includes/service-bus-selector-amqp.md)]

Questa sezione illustra come creare una macchina virtuale Linux CentOS mediante Macchine virtuali di Azure e come scaricare, creare e installare la libreria Apache Qpid Proton-C e le associazioni di linguaggio Python e PHP. Dopo aver completato questi passaggi, sarà possibile eseguire gli esempi di Python e PHP inclusi in questa guida.

Il primo passaggio viene eseguito mediante il [portale di Azure classico][]. La schermata seguente illustra la modalità di creazione di una macchina virtuale CentOS denominata "scott-centos":

![Proton in una macchina virtuale Linux di Azure][0]

Dopo il provisioning, il portale visualizza quanto segue:

![Proton in una macchina virtuale Linux di Azure][1]

Per accedere al computer, è necessario conoscere la porta dell'endpoint per SSH. È possibile ottenere questo valore dal [portale di Azure classico][] selezionando la VM appena creata e facendo clic sulla scheda **Endpoint**. La schermata seguente illustra che la porta pubblica SSH per il computer è 57146.

![Proton in una macchina virtuale Linux di Azure][2]

È ora possibile connettersi al computer mediante SSH. Questo esempio usa lo strumento PuTTY, come nella schermata seguente:

![Proton in una macchina virtuale Linux di Azure][3]

Per le app PHP e Python, questo esempio usa le librerie client Proton di Apache. Queste librerie sono disponibili per il download dal [http://qpid.apache.org/download.html](http://qpid.apache.org/download.html). Il file Readme incluso nel pacchetto di distribuzione illustra i passaggi necessari per installare le dipendenze e creare la libreria Proton. Ecco un riepilogo della procedura:

1.  Modificare il file di configurazione yum (/etc/yum.conf) e impostare come commento l'esclusione degli aggiornamenti per le intestazioni del kernel (# exclude=kernel*). Questo passaggio è necessario per installare il compilatore GCC.

2.  Installare i pacchetti prerequisiti:

	```
	# required dependencies 
	yum install gcc cmake libuuid-devel
	
	# dependencies needed for ssl support
	yum install openssl-devel
	
	# dependencies needed for bindings
	yum install swig python-devel ruby-devel php-devel java-1.6.0-openjdk
	
	# dependencies needed for python docs
	yum install epydoc
	```

1.  Scaricare la libreria Proton:

	```
	[azureuser@this-user ~]$ wget http://apache.panu.it/qpid/proton/0.9/qpid-proton-0.9.tar.gz
	--2016-04-17 14:45:03--  http://apache.panu.it/qpid/proton/0.9/qpid-proton-0.9.tar.gz
	Resolving apache.panu.it (apache.panu.it)... 81.208.22.71
	Connecting to apache.panu.it (apache.panu.it)|81.208.22.71|:80... connected.
	HTTP request sent, awaiting response... 200 OK
	Length: 868226 (848K) [application/x-gzip]
	Saving to: ‘qpid-proton-0.9.tar.gz’
	
	qpid-proton-0.9.tar.gz                               
	
	100%[====================================================================================================================>] 847.88K   102KB/s    in 8.4s    
	
	2016-04-17 14:45:12 (101 KB/s) - ‘qpid-proton-0.9.tar.gz’ saved [868226/868226]
	```

1.  Estrarre il codice Proton dall'archivio di distribuzione:

	```
	tar xvfz qpid-proton-0.9.tar.gz
	```

1.  Compilare e installare il codice come illustrato nella procedura seguente, tratta dal file Readme:

	```
	From the directory where you found this README file:	
	
	mkdir build cd build
			
	# Set the install prefix. You may need to adjust depending on your		
	# system.		
	cmake -DCMAKE\_INSTALL\_PREFIX=/usr ..
			
	# Omit the docs target if you do not wish to build or install		
	# documentation.		
	make all docs
			
	# Note that this step will require root privileges.		
	make install
	```

Dopo avere eseguito questa procedura, la libreria Proton è installata nel computer e pronta per l'uso.

## Passaggi successivi

Per altre informazioni, vedere il collegamento seguente:

- [Panoramica di AMQP per il bus di servizio][]

[Panoramica di AMQP per il bus di servizio]: service-bus-amqp-overview.md
[0]: ./media/service-bus-amqp-apache/amqp-apache-1.png
[1]: ./media/service-bus-amqp-apache/amqp-apache-2.png
[2]: ./media/service-bus-amqp-apache/amqp-apache-3.png
[3]: ./media/service-bus-amqp-apache/amqp-apache-4.png

[portale di Azure classico]: http://manage.windowsazure.com

<!---HONumber=AcomDC_0511_2016-->
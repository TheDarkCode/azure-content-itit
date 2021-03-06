<properties 
   pageTitle="Configurare una connessione VPN tra due reti virtuali | Microsoft Azure" 
   description="Informazioni su come configurare le connessioni VPN e la risoluzione dei nomi di dominio tra due reti virtuali di Azure e come configurare la replica geografica di HBase." 
   services="hdinsight,virtual-network" 
   documentationCenter="" 
   authors="mumian" 
   manager="paulettm" 
   editor="cgronlun"/>

<tags
   ms.service="hdinsight"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="big-data" 
   ms.date="06/28/2016"
   ms.author="jgao"/>

# Configurare una connessione VPN tra due reti virtuali di Azure  

> [AZURE.SELECTOR]
- [Configurare la connettività VPN](hdinsight-hbase-geo-replication-configure-VNETs.md)
- [Configurare DNS](hdinsight-hbase-geo-replication-configure-DNS.md)
- [Configurare la replica di HBase](hdinsight-hbase-geo-replication.md)

La connettività di rete virtuale di Azure Site-To-Site usa un gateway VPN per fornire un tunnel sicuro tramite Ipsec/IKE. Le reti virtuali possono trovarsi in diverse sottoscrizioni e aree geografiche diverse. È anche possibile combinare una comunicazione tra reti virtuali con configurazioni multisito. Esistono diversi motivi per la connettività tra reti virtuali:

- Presenza e ridondanza in più aree geografiche
- Applicazioni multilivello in singole aree geografiche con alto grado di isolamento
- Comunicazione tra sottoscrizioni e organizzazioni in Azure

Per altre informazioni, vedere [Configurare una connessione tra reti virtuali](../vpn-gateway/virtual-networks-configure-vnet-to-vnet-connection.md).

Per visualizzare un video:

> [AZURE.VIDEO configure-the-vpn-connectivity-between-two-azure-virtual-networks]

Questa esercitazione fa parte della [serie][hdinsight-hbase-replication] sulla creazione della replica geografica di HBase.

- Configurare una connessione VPN tra due reti virtuali (questa esercitazione)
- [Configurare il server DNS tra due reti virtuali di Azure][hdinsight-hbase-geo-replication-dns]
- [Configurare la replica geografica di HBase][hdinsight-hbase-geo-replication]

Il diagramma seguente illustra le due reti virtuali create in questa esercitazione:

![Grafico della rete virtuale di replica di HBase in HDInsight][img-vnet-diagram]
 

##Prerequisiti
Prima di iniziare questa esercitazione, è necessario disporre di quanto segue:

- **Una sottoscrizione di Azure**. Vedere [Ottenere una versione di prova gratuita di Azure](https://azure.microsoft.com/documentation/videos/get-azure-free-trial-for-testing-hadoop-in-hdinsight/).

- **Workstation con Azure PowerShell**.

	Prima di eseguire script PowerShell, assicurarsi di essere connessi alla sottoscrizione di Azure usando il seguente cmdlet:

		Add-AzureAccount

	Se si dispone di più sottoscrizioni di Azure, usare il seguente cmdlet per impostare la sottoscrizione corrente:

		Select-AzureSubscription <AzureSubscriptionName>
		
	[AZURE.INCLUDE [upgrade-powershell](../../includes/hdinsight-use-latest-powershell.md)]


>[AZURE.NOTE] I nomi dei servizi Azure e i nomi delle macchine virtuali devono essere univoci. Il nome usato in questa esercitazione è Contoso-[servizio di Azure/nome macchina virtuale]-[EU/US]. Ad esempio, Contoso-VNet-EU è la rete virtuale di Azure nel data center in Europa settentrionale; Contoso-DNS-US è la macchina virtuale del server DNS nel data center degli Stati Uniti orientali. È necessario indicare dei nomi a scelta.
 

##Creare due reti virtuali di Azure



**Per creare una rete virtuale denominata Contoso-VNet-EU in Europa settentrionale**

1.	Accedere al [portale di Azure classico][azure-portal].
2.	Fare clic su **NUOVO**, **SERVIZI DI RETE**, **RETE VIRTUALE**, **CREAZIONE PERSONALIZZATA**.
3.	Digitare:

	- **NOME**: Contoso-VNet-EU
	- **LOCALITÀ**: Europa settentrionale

		In questa esercitazione vengono usati i data center di Europa settentrionale e Stati Uniti orientali. È possibile scegliere il proprio data center.
4.	Digitare:

	- **SERVER DNS**: (lasciare vuoto)
	
		È necessario il proprio server DNS per la risoluzione dei nomi all'interno di reti virtuali. Per altre informazioni su quando usare la risoluzione dei nomi fornita da Azure e quando usare il proprio server DNS, vedere [Risoluzione dei nomi (DNS)](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md). Per istruzioni su come configurare la risoluzione dei nomi tra reti virtuali, vedere [Configurare DNS tra due reti virtuali di Azure][hdinsight-hbase-dns].
  
	- **Configurare una VPN Point-to-Site**: (deselezionato)

		Point-to-site non è valido per questo scenario.

 	- **Configurare una VPN Site-to-Site**: (deselezionato)
 	
		Verrà configurata la connessione VPN da sito a sito per la rete virtuale di Azure nel data center degli Stati Uniti orientali.
5.	Digitare:

	- 	**IP INIZIALE SPAZIO DI INDIRIZZI**: 10.1.0.0
	- 	**CIDR SPAZIO DI INDIRIZZI**: /16
	- 	**Subnet-1 STARTING IP**: 10.1.0.0
	- 	**CIDR Subnet-1**: /24

	Lo spazio degli indirizzi non può sovrapporsi con la rete virtuale degli Stati Uniti.

**Per creare una rete virtuale denominata Contoso-VNet-EU in Europa occidentale**

- Ripetere l'ultima procedura con i valori seguenti:

	- **NOME**: Contoso-VNet-US
	- **LOCALITÀ**: Stati Uniti orientali
	 
	- **SERVER DNS**: (lasciare vuoto)
	- **Configurare una VPN Point-to-Site**: (deselezionato)
	- **Configurare una VPN Site-to-Site**: (deselezionato)
	 
	- **IP INIZIALE SPAZIO DI INDIRIZZI**: 10.2.0.0
	- **CIDR SPAZIO DI INDIRIZZI**: /16
	- **Subnet-1 STARTING IP**: 10.2.0.0
	- **CIDR Subnet-1**: /24

















##Configurare una connessione VPN tra due reti virtuali

###Creare reti locali

Quando si crea una configurazione tra reti virtuali, è necessario configurare ciascuna rete virtuale in modo che si identifichino reciprocamente come sito di rete locale. In questa sezione si configurerà ciascuna rete virtuale come rete locale. Le reti locali condividono gli stessi spazi di indirizzi IP con la rete virtuale corrispondente.

![Configurare la connettività Site-to-Site della VPN di Azure - reti locali di Azure][img-vnet-lnet-diagram]


**Per creare una rete locale denominata Contoso-LNet-EU che corrisponda allo spazio di indirizzi della rete Contoso-VNet-EU**

1. Dal portale di Azure classico, fare clic su **NUOVO**, **SERVIZI DI RETE**, **RETE VIRTUALE**, **AGGIUNGI RETE LOCALE**.
3. Digitare:

	- **NAME**: Contoso-LNet-EU
	- **INDIRIZZO IP DISPOSITIVO VPN**: 192.168.0.1 (questo indirizzo verrà aggiornato in seguito)

		In genere, si utilizzerà l'indirizzo IP esterno effettivo di un dispositivo VPN. Per le configurazioni da rete virtuale a rete virtuale verrà usato l’indirizzo IP del gateway VPN. Dato che non sono stati ancora creati i gateway VPN per le due reti virtuali, immettere un indirizzo IP arbitrario e tornare a risolvere il problema.
4.	Digitare:

	- **IP INIZIALE SPAZIO DI INDIRIZZI:** 10.1.0.0
	- **CIDR SPAZIO DI INDIRIZZI:** /16
	
	Deve corrispondere esattamente all'intervallo specificato in precedenza per Contoso-VNet-EU.

**Per creare una rete locale denominata Contoso-LNet-US che corrisponda allo spazio di indirizzi della rete Contoso-VNet-US**

- Ripetere l'ultima procedura con i parametri seguenti:

	- **NAME**: Contoso-LNet-US
	- **INDIRIZZO IP DISPOSITIVO VPN**: 192.168.0.1 (questo indirizzo verrà aggiornato in seguito)
	 
	- **IP INIZIALE SPAZIO DI INDIRIZZI**: 10.2.0.0
	- **CIDR SPAZIO DI INDIRIZZI**: /16


###Creare gateway VPN

Questa configurazione consiste in due parti: innanzitutto verrà configurata una connessione Site-to-Site della rete virtuale a una rete locale, quindi verrà creata una VON con routing dinamico. Per la connettività tra reti virtuali sono necessari gateway VPN di Azure e VPN con routing dinamico. Le VPN con routing statico di Azure non sono supportate.

**Per configurare la connessione Site-to-Site tra Contoso-VNet-EU e Contoso-LNet-US**

1.	Dal portale di Azure classico, fare clic su **RETI** nel riquadro a sinistra,
2.	Fare clic su **Contoso-VNet-EU**.
3.	Fare clic sulla scheda **CONFIGURA**.
4.	Selezionare **Connetti alla rete locale**.
5.	In **RETE LOCALE**, selezionare **Contoso-LNet-US**.
6.	Nella sezione relativa agli spazi di indirizzi della rete virtuale, fare clic su **Aggiungi subnet gateway**.
7.	Fare clic su **SAVE**.
8.	Fare clic su **OK** per confermare.


**Per creare un gateway VPN per Contoso-VNet-EU**

1.	Dal portale di Azure classico, fare clic sulla scheda **DASHBOARD**.
4.	Fare clic su **CREA GATEWAY** nella parte inferiore della pagina e quindi su **Routing dinamico**.
5.	Fare clic su **Sì** per confermare. Si noti la rappresentazione grafica del gateway nella pagina diventa gialla con il testo Creazione del gateway. La creazione del gateway richiede in genere circa 15 minuti.

	Quando lo stato del gateway diventa Connessione in corso, l'indirizzo IP di ogni gateway sarà visibile nel Dashboard. Annotare l'indirizzo IP corrispondente a ciascuna rete virtuale, avendo cura di non confonderli. Si tratta degli indirizzi IP che verranno usati durante la modifica degli indirizzi IP segnaposto per il dispositivo VPN nelle reti locali.

6.	Creare una copia dell'**INDIRIZZO IP DEL GATEWAY**. Verrà usato per configurare l'indirizzo IP del gateway VPN per Contoso-VNet-EU nella sezione successiva.

**Per creare un gateway VPN per Contoso-VNet-EU**

- Ripetere le ultime due procedure per configurare la connettività Site-to-Site tra Contoso-VNet-US e Contoso-LNet-EU e quindi creare un gateway VPN per Contoso-VNet-US. Al termine, si otterrà l'indirizzo IP del gateway VPN per Contoso-VNet-US.


### Impostare gli indirizzi IP del dispositivo VPN per le reti locali
Nell'ultima sezione verrà creato un gateway VPN per ciascuna rete virtuale. Sono disponibili gli indirizzi IP dei gateway VPN, quindi è ora possibile tornare a configurare gli indirizzi IP del dispositivo VPN nella rete locale.

**Per configurare l'indirizzo IP del dispositivo VPN per Contoso-LNet-EU**

1.	Dal portale di Azure classico, fare clic su **RETI** nel riquadro a sinistra.
2.	Fare clic su **RETI LOCALI** nella parte superiore.
3.	Fare clic su **Contoso-LNet-EU**, quindi fare clic su **MODIFICA** nella parte inferiore.
4.	Aggiornare **INDIRIZZO IP DISPOSITIVO VPN**. Si tratta dell'indirizzo che si ottiene dalla scheda DASHBOARD della rete Contoso-VNET-EU.
5.	Fare clic sul pulsante a destra.
6.	Fare quindi clic sul pulsante con il segno di spunta.

**Per configurare l'indirizzo IP del dispositivo VPN per Contoso-LNet-US**

- Ripetere l'ultima procedura per configurare l'indirizzo IP del dispositivo VPN per Contoso-LNet-US.

###Impostare le chiavi del gateway della rete virtuale

I gateway di rete virtuale usano una chiave condivisa per autenticare le connessioni tra le reti virtuali. Non è possibile configurare la chiave dal portale di Azure classico. È necessario usare PowerShell o .NET SDK.

**Per impostare le chiavi**

1. Dalla workstation, aprire **Windows PowerShell ISE** o la console di Windows PowerShell.
2. Aggiornare i parametri nello script seguente ed eseguirlo:

		Add-AuzreAccount
		Select-AzureSubscription -[AzureSubscriptionName]
		Set-AzureVNetGatewayKey -VNetName ContosoVNet-EU -LocalNetworkSiteName Contoso-LNet-US -SharedKey A1b2C3D4
		Set-AzureVNetGatewayKey -VNetName ContosoVNet-US -LocalNetworkSiteName Contoso-LNet-EU -SharedKey A1b2C3D4 


##Controllare la connessione VPN 

Senza le macchine virtuali distribuite alle reti virtuali, è possibile usare il grafico visuale della rete virtuale nella pagina Dashboard della rete virtuale sul portale di Azure classico per controllare lo stato di connessione:

![Stato di connessione VPN della rete virtuale di replica di HBase in HDInsight][img-vpn-status]
  



##Passaggi successivi

In questa esercitazione si è appreso come configurare una connessione VPN tra due reti virtuali di Azure. Gli altri due articoli della serie trattano gli argomenti seguenti:

- [Configurare DNS tra due reti virtuali di Azure][hdinsight-hbase-geo-replication-dns]
- [Configurare la replica geografica di HBase][hdinsight-hbase-geo-replication]



[hdinsight-hbase-geo-replication-dns]: hdinsight-hbase-geo-replication-configure-dns.md
[hdinsight-hbase-geo-replication]: hdinsight-hbase-geo-replication.md


[azure-purchase-options]: http://azure.microsoft.com/pricing/purchase-options/
[azure-member-offers]: http://azure.microsoft.com/pricing/member-offers/
[azure-free-trial]: http://azure.microsoft.com/pricing/free-trial/
[azure-portal]: https://portal.azure.com

[powershell-install]: ../install-configure-powershell.md



[hdinsight-hbase-replication]: hdinsight-hbase-geo-replication.md
[hdinsight-hbase-dns]: hdinsight-hbase-geo-replication-configure-DNS.md


[img-vnet-diagram]: ./media/hdinsight-hbase-geo-replication-configure-VNets/HDInsight.HBase.VPN.diagram.png
[img-vnet-lnet-diagram]: ./media/hdinsight-hbase-geo-replication-configure-VNets/HDInsight.HBase.VPN.LNet.diagram.png
[img-vpn-status]: ./media/hdinsight-hbase-geo-replication-configure-VNets/HDInsight.HBase.VPN.status.png

<!---HONumber=AcomDC_0629_2016-->
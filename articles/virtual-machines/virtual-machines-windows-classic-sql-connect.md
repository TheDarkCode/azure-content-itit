<properties
	pageTitle="Connettersi a una macchina virtuale di SQL Server (Classica) | Microsoft Azure"
	description="Informazioni su come connettersi a SQL Server in esecuzione su una macchina virtuale di Azure. In questo argomento viene usato il modello di distribuzione classica. Gli scenari variano a seconda della configurazione di rete e della posizione del client."
	services="virtual-machines-windows"
	documentationCenter="na"
	authors="rothja"
	manager="jhubbard"
	tags="azure-service-management"/>
<tags
	ms.service="virtual-machines-windows"
	ms.devlang="na"
	ms.topic="article"
	ms.tgt_pltfrm="vm-windows-sql-server"
	ms.workload="infrastructure-services"
	ms.date="06/23/2016"
	ms.author="jroth" />

# Connettersi a una macchina virtuale di SQL Server in Azure (distribuzione classica)

> [AZURE.SELECTOR]
- [Gestione risorse](virtual-machines-windows-sql-connect.md)
- [Classico](virtual-machines-windows-classic-sql-connect.md)

## Panoramica

Questo argomento descrive la modalità di connessione all'istanza di SQL Server in esecuzione su una macchina virtuale di Azure. Illustra alcuni [scenari di connettività generali](#connection-scenarios) e quindi descrive la [procedura dettagliata per la configurazione della connettività di SQL Server in una macchina virtuale di Azure](#steps-for-configuring-sql-server-connectivity-in-an-azure-vm).

[AZURE.INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-classic-include.md)] Se si usano macchine virtuali di Resource Manager, vedere [Connect to a SQL Server Virtual Machine on Azure using Resource Manager](virtual-machines-windows-sql-connect.md) (Connettersi a una macchina virtuale SQL Server su Azure tramite Resource Manager).

## Scenari di connessione

La modalità di connessione di un client a SQL Server in esecuzione in una macchina virtuale varia a seconda della posizione del client e della configurazione tra il computer e la rete. Tali scenari includono:

- [Configurare e connettersi a più macchine virtuali di SQL Server nello stesso servizio cloud di Azure](#connect-to-sql-server-in-the-same-cloud-service)
- [Connettersi a SQL Server tramite Internet](#connect-to-sql-server-over-the-internet)
- [Connettersi a SQL Server nella stessa rete virtuale](#connect-to-sql-server-in-the-same-virtual-network)

>[AZURE.NOTE] Prima di connettersi con uno di questi metodi, è necessario seguire i [passaggi in questo articolo per configurare la connettività](#steps-for-configuring-sql-server-connectivity-in-an-azure-vm).

### Configurare e connettersi a più macchine virtuali di SQL Server nello stesso servizio cloud di Azure

È possibile creare più macchine virtuali nello stesso servizio cloud. Per comprendere meglio questo scenario, vedere [Come connettere le macchine virtuali con una rete virtuale o un servizio cloud](virtual-machines-windows-classic-connect-vms.md#connect-vms-in-a-standalone-cloud-service). In questo caso un client su una macchina virtuale tenta di connettersi a SQL Server in esecuzione su un'altra macchina virtuale nello stesso servizio cloud.

In questo scenario è possibile connettersi usando il **nome** della macchina virtuale (anche visualizzato come **Nome computer** o **hostname** nel portale). Questo è il nome specificato durante la creazione della macchina virtuale. Ad esempio, se la macchina virtuale SQL è stata denominata **mysqlvm**, una macchina virtuale client nello stesso servizio cloud potrebbe usare la stringa di connessione seguente per connettersi:

	"Server=mysqlvm;Integrated Security=false;User ID=<login_name>;Password=<your_password>"

### Connettersi a SQL Server tramite Internet

Se si desidera connettersi al motore di database di SQL Server tramite Internet, è necessario creare un endpoint della macchina virtuale per le comunicazioni TCP in ingresso. In questo passaggio di configurazione di Azure, il traffico della porta TCP in ingresso viene indirizzato a una porta TCP accessibile alla macchina virtuale.

Per connettersi tramite Internet è necessario usare il nome DNS e il numero di porta dell'endpoint della macchina virtuale (configurati più avanti in questo articolo). Per trovare il nome DNS, passare al portale di Azure e selezionare **Macchine virtuali (versione classica)**. Selezionare quindi la macchina virtuale. Il **Nome DNS** è visualizzato nella sezione **Panoramica**.

Si consideri ad esempio una macchina virtuale classica denominata **mysqlvm** con nome DNS **mysqlvm7777.cloudapp.net** ed endpoint di macchina virtuale **57500**. Supponendo che la connettività sia configurata correttamente, la stringa di connessione seguente può essere usata per accedere alla macchina virtuale da qualsiasi posizione in Internet:

	"Server=mycloudservice.cloudapp.net,57500;Integrated Security=false;User ID=<login_name>;Password=<your_password>"

Sebbene in questo modo venga abilitata la connettività per i client tramite Internet, ciò non significa che chiunque può connettersi all'istanza di SQL Server. I client esterni dovranno disporre del nome utente e della password corretti. Per una maggiore sicurezza, non usare la nota porta 1433 per l'endpoint pubblico della macchina virtuale. Se possibile, è consigliabile aggiungere un ACL all'endpoint per limitare il traffico ai soli client autorizzati. Per istruzioni sull'uso di ACL con gli endpoint, vedere [Gestire l'elenco di controllo di accesso su un endpoint](virtual-machines-windows-classic-setup-endpoints.md#manage-the-acl-on-an-endpoint).

>[AZURE.NOTE] È importante notare che quando si usa questa tecnica per comunicare con SQL Server, tutti i dati in uscita dai data center di Azure sono soggetti ai normali [prezzi di trasferimenti dei dati in uscita](https://azure.microsoft.com/pricing/details/data-transfers/).

### Connettersi a SQL Server nella stessa rete virtuale

La [rete virtuale](../virtual-network/virtual-networks-overview.md) supporta scenari aggiuntivi. È possibile connettere le macchine virtuali nella stessa rete virtuale, anche se si trovano in servizi cloud diversi. Con una [VPN da sito a sito](../vpn-gateway/vpn-gateway-site-to-site-create.md) è possibile creare un'architettura ibrida che connette le macchine virtuali a computer e reti locali.

Le reti virtuali consentono inoltre di aggiungere le macchine virtuali di Azure a un dominio. Si tratta dell'unico modo per usare l'autenticazione di Windows per SQL Server. Gli altri scenari di connessione richiedono l'autenticazione SQL con nomi utente e password.

Se si desidera configurare un ambiente di dominio e l'autenticazione di Windows, non è necessario seguire la procedura descritta in questo articolo per configurare l'endpoint pubblico o l'autenticazione SQL e gli account di accesso. In questo scenario è possibile connettersi all'istanza di SQL Server specificando il nome della macchina virtuale SQL Server nella stringa di connessione. Nell'esempio seguente si presuppone che sia stata configurata anche l'autenticazione di Windows e che all'utente sia stato concesso l'accesso all'istanza di SQL Server.

	"Server=mysqlvm;Integrated Security=true"

## Procedura per la configurazione della connettività di SQL Server in una macchina virtuale di Azure

I passaggi seguenti illustrano come connettersi all'istanza di SQL Server su Internet mediante SQL Server Management Studio (SSMS). Tuttavia, gli stessi passaggi si applicano per rendere accessibile la macchina virtuale di SQL Server per le applicazioni in esecuzione sia in locale che in Azure.

Prima di poter eseguire la connessione all'istanza di SQL Server da un’altra VM o da Internet, è necessario completare le seguenti attività, come descritto nelle seguenti sezioni:

- [Creare un endpoint TCP per la macchina virtuale](#create-a-tcp-endpoint-for-the-virtual-machine)
- [Aprire le porte TCP in Windows Firewall](#open-tcp-ports-in-the-windows-firewall-for-the-default-instance-of-the-database-engine)
- [Configurare SQL Server per l'ascolto sul protocollo TCP](#configure-sql-server-to-listen-on-the-tcp-protocol)
- [Configurare SQL Server per l'autenticazione in modalità mista](#configure-sql-server-for-mixed-mode-authentication)
- [Creare gli account di accesso di SQL Server](#create-sql-server-authentication-logins)
- [Determinare il nome DNS della macchina virtuale](#determine-the-dns-name-of-the-virtual-machine)
- [Eseguire la connessione al motore di database da un altro computer](#connect-to-the-database-engine-from-another-computer)

Il percorso di connessione è riepilogato nel seguente diagramma:

![Connessione a una macchina virtuale di SQL Server](../../includes/media/virtual-machines-sql-server-connection-steps/SQLServerinVMConnectionMap.png)

[AZURE.INCLUDE [Connettersi a SQL Server in una macchina virtuale - Endpoint TCP classico](../../includes/virtual-machines-sql-server-connection-steps-classic-tcp-endpoint.md)]

[AZURE.INCLUDE [Connettersi a SQL Server in una macchina virtuale](../../includes/virtual-machines-sql-server-connection-steps.md)]

[AZURE.INCLUDE [Connettersi a SQL Server in una macchina virtuale - Procedura classica](../../includes/virtual-machines-sql-server-connection-steps-classic.md)]

## Passaggi successivi

Se si intende anche usare gruppi di disponibilità AlwaysOn per la disponibilità elevata e il ripristino di emergenza, è consigliabile implementare un listener. I client del database si connettono al listener anziché connettersi direttamente a una delle istanze di SQL Server. Il listener indirizza i client alla replica primaria nel gruppo di disponibilità. Per altre informazioni, vedere l'articolo relativo alla [configurazione di un listener di ILB per gruppi di disponibilità AlwaysOn in Azure](virtual-machines-windows-classic-ps-sql-int-listener.md).

È importante esaminare tutte le procedure consigliate sulla sicurezza per SQL Server in esecuzione in una macchina virtuale di Azure. Per altre informazioni, vedere [Considerazioni relative alla sicurezza per SQL Server in Macchine virtuali di Azure](virtual-machines-windows-sql-security.md).

[Esplorare il percorso di apprendimento](https://azure.microsoft.com/documentation/learning-paths/sql-azure-vm/) per SQL Server nelle macchine virtuali di Azure.

Per altri argomenti relativi all'esecuzione di SQL Server nelle macchine virtuali di Azure, vedere [SQL Server in Macchine virtuali di Azure](virtual-machines-windows-sql-server-iaas-overview.md).

<!---HONumber=AcomDC_0629_2016-->
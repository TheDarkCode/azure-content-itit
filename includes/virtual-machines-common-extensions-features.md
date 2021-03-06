



Per informazioni dettagliate sugli agenti VM e sul relativo ruolo nel supporto delle estensioni delle macchine virtuali, vedere [Informazioni generali sull'agente VM e sulle estensioni VM](../articles/virtual-machines/virtual-machines-windows-classic-manage-extensions.md).

##Estensioni di macchina virtuale

Le estensioni di VM implementano la maggior parte delle funzionalità critiche da usare con le macchine virtuali, incluse le funzionalità di base quali la reimpostazione delle password, la configurazione di RDP e molto altro ancora. Poiché vengono aggiunte in continuazione nuove estensioni, il numero di possibili funzionalità supportate dalle VM in Azure è in continua crescita. Per impostazione predefinita, alcune estensioni di VM di base vengono installate quando si crea una macchina virtuale dalla Raccolta immagini, incluse **IaaSDiagnostics** (attualmente solo per VM Windows), **VMAccess** e **BGInfo** (attualmente solo per VM Windows). Non tutte le estensioni, tuttavia, sono implementate in Windows e Linux in un momento specifico, a causa del flusso continuo di aggiornamenti delle funzionalità e di nuove estensioni.

##Connettività e gestione di base

Le estensioni seguenti sono essenziali per abilitare, riabilitare o disabilitare la connettività di base alle VM create e in esecuzione.

|Nome estensione VM|Descrizione della funzionalità|Altre informazioni
|---|---|---|
|VMAccessAgent (Windows)|Creare, aggiornare e reimpostare le informazioni utente e le configurazioni delle connessioni RDP.|[Windows](../articles/virtual-machines/virtual-machines-windows-classic-extensions-customscript.md)
|VMAccessForLinux (Linux)|Creare, aggiornare e reimpostare le informazioni utente e le configurazioni delle connessioni SSH.|[Linux](https://github.com/Azure/azure-linux-extensions/tree/master/VMAccess)

##Gestione della distribuzione e della configurazione

Le estensioni seguenti supportano diversi scenari e funzionalità di gestione della distribuzione e della configurazione. Vedere anche la sezione relativa alle operazioni e alla gestione delle macchine virtuali più avanti.

|Nome estensione VM|Descrizione della funzionalità|Altre informazioni|
|---|---|---|
|**MSEnterpriseApplication**|Implementa funzionalità per il supporto da parte di Windows System Center.|[Ruoli macchina virtuale di System Center 2012 R2](http://social.technet.microsoft.com/wiki/contents/articles/18274.system-center-2012-r2-virtual-machine-role-authoring-guide-resource-extension-package.aspx)|
|**Octopus Deploy** (basata sull'estensione DSC)|Supporta la distribuzione automatica di applicazioni Web ASP.NET e Servizi Windows in ambienti di sviluppo, test e produzione.|[Introduzione a Octopus Deploy](http://docs.octopusdeploy.com/display/OD/Getting%20started)|
|**Responsabile del rilascio di Visual Studio** (basata sull'estensione DSC)|Supporta la distribuzione continua con Visual Studio.|[Automatizzare le distribuzioni con Release Management](https://msdn.microsoft.com/Library/vs/alm/Release/overview)|
|**CentosChefClient**|||
|**ChefClient**|Crea un client Chef su Windows. È possibile usare anche l'estensione DSC, illustrata più avanti.|[Chef e Microsoft Azure](https://www.getchef.com/solutions/azure/)|
|**LinuxChefClient**|||
|**DockerExtension**|Installa il daemon Docker per supportare i comandi Docker remoti.|[Come usare l'estensione della macchina virtuale Docker](../articles/virtual-machines/virtual-machines-linux-dockerextension.md). Per informazioni più dettagliate, vedere il [Manuale dell'utente dell'estensione della macchina virtuale Docker](https://github.com/Azure/azure-docker-extension/blob/master/README.md).|
|**DSC**|Estensione PowerShell DSC (Desired State Configuration).|[Estensione Azure PowerShell DSC (Desired State Configuration)](http://blogs.msdn.com/b/powershell/archive/2014/08/07/introducing-the-azure-powershell-dsc-desired-state-configuration-extension.aspx)|
|**PuppetEnterpriseAgent**|Implementa le funzionalità di Puppet Enterprise. |[Puppet in Azure](http://puppetlabs.com/solutions/microsoft)|
|**CustomScriptExtension** (Windows)**CustomScriptForLinux** (Linux)|Richiama script personalizzati sulla VM in qualsiasi momento, ovvero all'avvio o durante il ciclo di vita.|[Estensione di script personalizzati](../articles/virtual-machines/virtual-machines-windows-classic-extensions-customscript.md) | [Linux](https://github.com/Azure/azure-linux-extensions/tree/master/CustomScript)|
|**AzureCATExtensionHandler**|Utilizza i dati di diagnostica raccolti da **IaaSDiagnostics** e da poche altre origini dati quali le [Metriche di Analisi archiviazione di Azure](https://msdn.microsoft.com/library/azure/hh343270.aspx) e li trasforma in un set di dati aggregato, ideale per l'utilizzo da parte del processo del controllo host SAP.|[Monitoraggio avanzato di Azure per SAP](https://azure.microsoft.com/blog/2014/06/04/azure-enhanced-monitoring-for-sap/)|

##Sicurezza e protezione

Le estensioni in questa sezione forniscono funzionalità di sicurezza essenziali per le VM di Azure.

|Nome estensione VM|Descrizione della funzionalità|Altre informazioni|
|---|---|---|
|**CloudLinkSecureVMWindowsAgent**|Offre ai clienti di Microsoft Azure la possibilità di crittografare i dati delle macchine virtuali in un'infrastruttura condivisa multi-tenant e di controllare completamente le chiavi di crittografia per i dati crittografati nell'infrastruttura di archiviazione di Azure.|[Protezione delle macchine virtuali di Microsoft Azure tramite la crittografia BitLocker e del sistema operativo nativo](http://www.cloudlinktech.com/azure)|
|**McAfeeEndpointSecurity**|Protegge la VM da software dannoso.|[McAfee](https://www.mcafeeasap.com/MarketingContent/default.aspx)|
|**TrendMicroDSA**|Abilita il supporto della piattaforma Trend Micro Deep Security per fornire soluzioni per rilevamento e prevenzione delle intrusioni, firewall, antimalware, reputazione Web, ispezione dei log e monitoraggio dell'integrità.|[Come installare e configurare Trend Micro Deep Security come servizio in una macchina virtuale di Azure](../articles/virtual-machines/virtual-machines-windows-classic-install-trend.md)|
|**PortalProtectExtension**|Protegge da minacce all'ambiente Microsoft SharePoint.|[Protezione della distribuzione SharePoint in Azure](http://blog.trendmicro.com/securing-sharepoint-deployment-azure/)|
|**IaaSAntimalware**|Microsoft Antimalware per Servizi cloud e Macchine virtuali di Azure è una funzionalità di protezione in tempo reale che consente di identificare e rimuovere virus, spyware e altro software dannoso e offre avvisi configurabili per i casi in cui software dannoso o indesiderato prova l'installazione o l'esecuzione nel sistema.|[Microsoft Antimalware per Servizi cloud e Macchine virtuali di Azure](../articles/azure-security-antimalware.md)|
|**SymantecEndpointProtection**|Symantec Endpoint Protection 12.1.4 consente di ottenere sicurezza e prestazioni in sistemi fisici e virtuali.|[Come installare e configurare Symantec Endpoint Protection in una macchina virtuale di Azure](../articles/virtual-machines/virtual-machines-windows-classic-install-symantec.md)

##Operazioni e gestione delle macchine virtuali

Supporta le funzionalità e il comportamento relativi alle operazioni e alla gestione. Vedere anche la sezione precedente Gestione della distribuzione e della configurazione.

|**Nome estensione VM**|Descrizione della funzionalità|Altre informazioni|
|---|---|---|
|**AzureVmLogCollector**|È possibile usare l'estensione **AzureVMLogCollector** su richiesta per eseguire una raccolta occasionale di log da una o più macchine virtuali di Servizio cloud, da ruoli Web e ruoli di lavoro, e trasferire i file raccolti in un account di archiviazione di Azure, senza accedere in modalità remota ad alcuna macchina virtuale. |[Estensione AzureLogCollector](../articles/virtual-machines/virtual-machines-windows-log-collector-extension.md)|
|**IaaSDiagnostics**|Abilita, disabilita e configura Diagnostica di Azure e viene usata anche da **AzureCATExtensionHandler** per supportare il monitoraggio di SAP.|[Monitoraggio delle Macchine virtuali di Microsoft Azure con l'estensione Diagnostica di Azure](https://azure.microsoft.com/blog/2014/09/02/windows-azure-virtual-machine-monitoring-with-wad-extension/)|
|**OSPatchingForLinux**|Consente agli amministratori di VM di Azure di automatizzare gli aggiornamenti al sistema operativo delle VM con configurazioni personalizzate. È possibile usare l'estensione OSPatching per configurare gli aggiornamenti al sistema operativo per le macchine virtuali. È ad esempio possibile specificare con che frequenza e quando installare le patch del sistema operativo, specificare quali patch installare e configurare il comportamento di riavvio dopo gli aggiornamenti|[Post di blog sull'estensione OSPatching](https://azure.microsoft.com/blog/2014/10/23/automate-linux-vm-os-updates-using-ospatching-extension/). Vedere anche il file Leggimi e il codice sorgente in Github nella pagina relativa all'[Estensione OSPatching](https://github.com/Azure/azure-linux-extensions).|

##Sviluppo e debug

Queste estensioni VM sono elencate qui per completezza, poiché forniscono supporto per funzionalità correlate a Visual Studio, ma non devono essere usate direttamente.

|Nome estensione VM|Descrizione della funzionalità|Altre informazioni|
|---|---|---|
|**VS14CTPDebugger**|Supporta il debug remoto da VS mediante Azure SDK 2.4|[Debug remoto in Visual Studio](https://msdn.microsoft.com/library/y7f5zaaa.aspx)|
|**VS2013Debugger**|Supporta il debug remoto da VS mediante Azure SDK 2.4||
|**VS2012Debugger**|Supporta il debug remoto da VS mediante Azure SDK 2.4||
|**RemoteDebugVS14CTP**|Supporta il debug remoto da VS mediante Azure SDK 2.3||
|**RemoteDebugVS2013**|Supporta il debug remoto da VS mediante Azure SDK 2.3||
|**RemoteDebugVS2012**|Supporta il debug remoto da VS mediante Azure SDK 2.3||
|**WebDeployForVSDevTest**|Installa e configura IIS e Distribuzione Web in Windows Server. La rimozione o la disabilitazione non sono supportate.|

##Funzionalità varie

Queste estensioni forniscono il supporto per altre funzionalità delle VM che potrebbero essere utili.

|Nome estensione VM|Descrizione della funzionalità|Altre informazioni|
|---|---|---|
|**BGInfo**|Quando si usa RDP, presenta sul desktop un'immagine consolidata delle informazioni utili sul server.|[Estensione BGInfo](https://msdn.microsoft.com/library/mt589195.aspx)|
|**HpcVmDrivers**|Installa, configura e mantiene i driver del dispositivo di rete (RDMA) d’accesso alla memoria diretto remoto in una macchina virtuale di dimensioni A8 o A9 che esegue il server di Windows 2012 R2 il server di Windows 2012. Abilita le macchine virtuali A8 o A9 con cluster ad utilizzare la rete RDMA durante l'esecuzione di applicazioni MPI parallele.|[Informazioni sulle istanze A8, A9, A10 e A11 a elevato utilizzo di calcolo](../articles/virtual-machines/virtual-machines-windows-a8-a9-a10-a11-specs.md)

<!---HONumber=AcomDC_0608_2016-->
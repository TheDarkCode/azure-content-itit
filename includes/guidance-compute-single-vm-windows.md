Questo articolo illustra un insieme di procedure consolidate per l'esecuzione di una macchina virtuale (VM) Windows in Azure, con particolare attenzione agli aspetti di scalabilità, disponibilità, gestibilità e sicurezza.

> [AZURE.NOTE] Azure offre due diversi modelli di distribuzione, ovvero [Azure Resource Manager][resource-manager-overview] e la distribuzione classica. Questo articolo usa Azure Resource Manager, consigliato da Microsoft per le nuove distribuzioni.

Non è consigliabile usare una singola VM per carichi di lavoro di produzione perché non esiste alcun contratto di servizio che disciplini il tempo di attività delle singole VM in Azure. Per ottenere il contratto di servizio, è necessario distribuire più VM in un [set di disponibilità][availability-set]. Per altre informazioni, vedere [Running multiple Windows VMs on Azure][multi-vm] \(Esecuzione di più VM Windows in Azure).

## Diagramma dell'architettura

Il provisioning di una VM in Azure coinvolge un altri componenti mobili oltre alla VM in sé. Sono presenti elementi di calcolo, rete e archiviazione.

![[0]][0]

- **Gruppo di risorse.** Un [_gruppo di risorse_][resource-manager-overview] è un contenitore in cui risiedono le risorse correlate. Creare un gruppo di risorse in cui includere le risorse per questa VM.

- **VM**. È possibile effettuare il provisioning di una macchina virtuale da un elenco di immagini pubblicate oppure da un file di disco rigido virtuale (VHD) caricato nell'archivio BLOB di Azure.

- **Disco del sistema operativo.** Il disco del sistema operativo è un VHD archiviato nell'[Archiviazione di Azure][azure-storage]. Viene quindi salvato in modo permanente anche se il computer host si arresta.

- **Disco temporaneo.** La VM viene creata con un disco temporaneo, ovvero l'unità `D:` in Windows. Questo disco viene archiviato in un'unità fisica nel computer host. _Non_ viene salvato nell'Archiviazione di Azure ed è possibile che venga eliminato durante i riavvii e altri eventi del ciclo di vita della VM. Usare questo disco solo per dati temporanei, ad esempio file di paging o di scambio.

- **Dischi dati.** Un [disco dati][data-disk] è un VHD persistente usato per i dati dell'applicazione. I dischi dati vengono archiviati nell'Archiviazione di Azure, come il disco del sistema operativo.

- **Rete virtuale (VNet) e subnet.** Ogni VM in Azure viene distribuita in una rete virtuale (VNet), che viene a sua volta suddivisa in subnet.

- **Indirizzo IP pubblico.** Un indirizzo IP pubblico è necessario per comunicare con la macchina virtuale, ad esempio tramite Desktop remoto.

- **Interfaccia di rete (NIC)**. La scheda di interfaccia di rete consente alla VM di comunicare con la rete virtuale.

- **Gruppo di sicurezza di rete (NSG)**. Il [gruppo di sicurezza di rete][nsg] viene usato per consentire/negare il traffico di rete verso la subnet. Un NSG può essere associato a una singola scheda di interfaccia di rete o a una subnet. Se lo si associa a una subnet, le regole dell’NSG si applicano a tutte le VM nella subnet.
 
- **Diagnostica.** La registrazione diagnostica è essenziale per la gestione e la risoluzione dei problemi della macchina virtuale.

## Consigli

### Indicazioni per le VM

- È consigliabile usare le serie GS e DS, a meno che non si disponga di un carico di lavoro specializzato, ad esempio di high-performance computing. Per informazioni dettagliate, vedere [Dimensioni delle macchine virtuali in Azure][virtual-machine-sizes]. Quando si sposta un carico di lavoro esistente in Azure, per iniziare scegliere le dimensioni di VM più simili a quelle dei server locali. Misurare quindi le prestazioni del carico di lavoro effettivo in relazione agli aspetti di CPU, memoria e operazioni di input/output (IOPS) del disco e regolare le dimensioni secondo necessità. Inoltre, se sono necessarie più schede di interfaccia di rete, tenerne presente il limite per ogni dimensione.

- Quando si effettua il provisioning della VM e di altre risorse, è necessario specificare una località. È in genere consigliabile scegliere la località più vicina agli utenti interni o ai clienti. È tuttavia possibile che le dimensioni di VM non siano tutte disponibili in tutte le località. Per informazioni dettagliate, vedere [Servizi in base all'area][services-by-region]. Per elencare le dimensioni delle VM disponibili in una determinata località, eseguire il comando seguente dell'interfaccia della riga di comando di Azure:

    ```
    azure vm sizes --location <location>
    ```

- Per informazioni sulla scelta di un'immagine di VM pubblicata, vedere [Selezionare immagini di VM Linux con l'interfaccia della riga di comando di Azure][select-vm-image].

### Indicazioni per il disco e l'archiviazione

- Per ottimizzare le prestazioni I/O del disco, si consiglia di usare [Archiviazione Premium][premium-storage], che archivia i dati in unità SSD (Solid State Drive). I costi dipendono dalle dimensioni del disco sottoposto a provisioning. Anche IOPS e velocità effettiva dipendono dalle dimensioni del disco. Quando si effettua il provisioning di un disco è quindi consigliabile tenere in considerazione tutti e tre i fattori, ovvero capacità, IOPS e velocità effettiva.

- Un account di archiviazione può supportare da una a 20 VM.

- Aggiungere uno o più dischi dati. Quando si crea un nuovo VHD, il disco non è formattato. Accedere alla VM per formattare il disco.

- Se sono presenti molti dischi dati, occorre prestare attenzione ai limiti totali di I/O dell'account di archiviazione. Per altre informazioni, vedere [Sottoscrizione di Azure e limiti, quote e vincoli dei servizi][vm-disk-limits].

- Per prestazioni ottimali, creare un account di archiviazione separato per i log di diagnostica. Un account di archiviazione con ridondanza locale standard è sufficiente per i log di diagnostica.

- Quando possibile, installare applicazioni su un disco dati, anziché sul disco del sistema operativo. Tuttavia, per alcune applicazioni legacy potrebbe essere necessario installare componenti nell'unità C:. In tal caso, è possibile [ridimensionare il disco del sistema operativo][resize-os-disk] tramite PowerShell.

### Indicazioni per la rete

- L'indirizzo IP pubblico può essere dinamico o statico. Per impostazione predefinita, è dinamico.

    - Riservare un [indirizzo IP statico][static-ip] se è necessario un indirizzo IP fisso che non subirà modifiche, ad esempio se è necessario creare un record A in DNS o se è necessario che l'indirizzo IP sia consentito.

    - È inoltre possibile creare un nome di dominio completo (FQDN) per l'indirizzo IP. È quindi possibile registrare un [record CNAME][cname-record] in DNS che punta al nome FQDN. Per altre informazioni, vedere [Creare un nome di dominio completo nel portale di Azure][fqdn].

- Tutti i gruppi di sicurezza di rete contengono un set di [regole predefinite][nsg-default-rules], inclusa una regola che blocca tutto il traffico Internet in ingresso. Le regole predefinite non possono essere eliminate, ma altre regole possono eseguirne l'override. Per consentire il traffico Internet, creare regole per indirizzare il traffico in ingresso a determinate porte, ad esempio la porta 80 per il traffico HTTP.

- Per abilitare Desktop remoto, aggiungere una regola all'NSG per consentire il traffico in arrivo sulla porta TCP 3389.

## Considerazioni sulla scalabilità

- Per aumentare o ridurre le prestazioni di una VM, è possibile [modificare le dimensioni della VM][vm-resize].

- Per scalare orizzontalmente, inserire due o più VM in un set di disponibilità nell'ambito di un servizio di bilanciamento del carico. Per informazioni dettagliate, vedere [Running multiple VMs on Azure][multi-vm] \(Esecuzione di più VM Windows in Azure).

## Considerazioni sulla disponibilità

- Come detto in precedenza, non esiste alcun contratto di servizio per una singola VM. Per ottenere il contratto di servizio, è necessario distribuire più VM in un set di disponibilità.

- È possibile che la VM sia interessata da attività di [manutenzione pianificata][planned-maintenance] o [manutenzione non pianificata][manage-vm-availability]. È possibile usare i [log di riavvio della VM][reboot-logs] per determinare se un riavvio della macchina virtuale è stato provocato da attività di manutenzione pianificata.

- Il backup dei VHD viene eseguito dall'[Archiviazione di Azure][azure-storage] e ne viene eseguita la replica per finalità di durabilità e disponibilità.

- Per proteggersi dalla perdita accidentale di dati durante le operazioni normali, ad esempio a causa di un errore dell'utente, è consigliabile implementare anche i backup temporizzati usando [snapshot di BLOB][blob-snapshot] o un altro strumento.

## Considerazioni sulla gestibilità

- **Gruppi di risorse.** Posizionare in uno stesso [gruppo di risorse][resource-manager-overview] le risorse strettamente associate che condividono lo stesso ciclo di vita. I gruppi di risorse consentono di distribuire e monitorare le risorse in gruppo, distribuendo i costi per ogni gruppo di risorse. È inoltre possibile eliminare un intero set di risorse, operazione molto utile nelle distribuzioni di test. Assegnare alle risorse nomi significativi. In tal modo, è più semplice individuare una specifica risorsa e comprenderne il ruolo. Vedere [Recommended Naming Conventions for Azure Resources][naming conventions] \(Convenzioni di denominazione consigliate per le risorse di Azure).

- **Diagnostica delle VM.** Abilitare il monitoraggio e la diagnostica, tra cui le metriche di base sull'integrità, i log relativi all'infrastruttura di diagnostica e la [diagnostica di avvio][boot-diagnostics]. La diagnostica di avvio permette di diagnosticare gli errori di avvio quando la VM passa a uno stato non avviabile. Per altre informazioni, vedere [Abilitare il monitoraggio e la diagnostica][enable-monitoring]. Usare l'estensione [Raccolta di log di Azure][log-collector] per raccogliere i log della piattaforma Azure e caricarli nell'Archiviazione di Azure.

    Per abilitare la diagnostica, eseguire il comando seguente dell'interfaccia della riga di comando:

    ```text
    azure vm enable-diag <resource-group> <vm-name>
     ```

- **Arresto di una VM.** Azure distingue tra gli stati "Arrestato" e "Deallocato". Vengono applicati addebiti se lo stato della VM è "Arrestato". Non viene applicato alcun addebito se la VM è stata deallocata.

    Usare il comando seguente dell'interfaccia della riga di comando per deallocare una VM:

    ```text
    azure vm deallocate <resource-group> <vm-name>
    ```

    Anche il pulsante **Arresta** nel portale di Azure consente di deallocare la VM. Se, tuttavia, l'arresto viene effettuato tramite il sistema operativo ad accesso eseguito, la VM viene arrestata ma _non_ deallocata, quindi gli addebiti continueranno a essere effettuati.

- **Eliminazione di una VM.** Se si elimina una VM, i VHD non vengono eliminati. È quindi possibile eliminare in modo sicuro la macchina virtuale senza perdere dati. Verranno tuttavia applicati comunque addebiti per l'archiviazione. Per eliminare il VHD, eliminare il file dall'[archivio BLOB][blob-storage].

  Per impedire l'eliminazione accidentale, usare un [blocco di risorsa][resource-lock] per bloccare l'intero gruppo di risorse o le singole risorse, ad esempio la VM.

## Considerazioni relative alla sicurezza

- Usare il [Centro sicurezza di Azure][security-center] per ottenere una visualizzazione centrale dello stato di sicurezza delle risorse di Azure. Il Centro sicurezza monitora potenziali problemi di sicurezza, ad esempio aggiornamenti del sistema e antimalware, e fornisce un'immagine completa dello stato della sicurezza della distribuzione.

    - Il Centro sicurezza è configurato per ogni sottoscrizione di Azure. Abilitare la raccolta dei dati di sicurezza come descritto in [Usare il Centro sicurezza].

    - Quando la raccolta dei dati è abilitata, il Centro sicurezza analizza automaticamente tutte le macchine virtuali create nell'ambito della sottoscrizione.

- **Gestione delle patch.** Se abilitato, Centro sicurezza PC controlla se mancano aggiornamenti critici e della sicurezza. Usare le [impostazioni di Criteri di gruppo][group-policy] nella VM per abilitare gli aggiornamenti automatici del sistema.

- **Antimalware.** Se abilitato, Centro sicurezza PC controlla se è installato il software antimalware. È inoltre possibile utilizzare Centro sicurezza PC per installare il software antimalware all'interno del portale di Azure.

- Usare il [controllo degli accessi in base al ruolo][rbac] \(RBAC) per controllare l'accesso alle risorse di Azure da distribuire. Il controllo degli accessi in base al ruolo consente di assegnare i ruoli di autorizzazione ai membri del proprio team DevOps. Ad esempio, il ruolo di lettura permette di visualizzare le risorse di Azure, ma non di crearle, gestirle o eliminarle. Alcuni ruoli sono specifici di determinati tipi di risorse di Azure. Ad esempio, il ruolo di Collaboratore Macchina virtuale consente di riavviare o deallocare una VM, reimpostare la password di amministratore, creare una nuova VM e così via. Altri [ruoli RBAC predefiniti][rbac-roles] che potrebbero essere utili per questa architettura di riferimento includono [Utente DevTest Labs][rbac-devtest] e [Collaboratore Rete][rbac-network]. Oltre a poter assegnare un utente a più ruoli, è possibile creare ruoli personalizzati per autorizzazioni ancora più dettagliate.

    > [AZURE.NOTE] Il controllo degli accessi in base al ruolo non limita le azioni eseguibili da un utente registrato in una VM. Le autorizzazioni sono determinate dal tipo di account sul sistema operativo guest.

- Per reimpostare la password dell'amministratore locale, eseguire il comando `vm reset-access` dell'interfaccia della riga di comando di Azure.

    ```text
    azure vm reset-access -u <user> -p <new-password> <resource-group> <vm-name>
    ```

- Per verificare le azioni di provisioning e altri eventi della VM, usare i [log di controllo][audit-logs].

- Se si necessitano di crittografare i dischi del sistema operativo e i dischi dati, usare [Crittografia dischi di Azure ][disk-encryption].

## Componenti della soluzione

È disponibile uno script di soluzione di esempio ([Deploy-ReferenceArchitecture.ps1][solution-script]) che può essere usato per implementare l'architettura conforme alle indicazioni descritte in questo articolo. Questo script usa i modelli di [Azure Resource Manager][ARM-Templates]. I modelli sono disponibili come un insieme di elementi costitutivi fondamentali, ognuno dei quali esegue un'azione specifica, ad esempio la creazione di una rete virtuale o la configurazione di un gruppo di sicurezza di rete. Lo scopo dello script è di gestire la distribuzione del modello.

I modelli sono governati da parametri, che sono contenuti in file JSON separati. È possibile modificare i parametri in questi file per configurare la distribuzione in base alle proprie esigenze. Non è necessario modificare i modelli stessi. Gli schemi degli oggetti nel file dei parametri non devono essere modificati.

Quando si modificano i modelli, creare oggetti che rispettano le convenzioni di denominazione descritte in [Recommended Naming Conventions for Azure Resources][naming conventions] \(Convenzioni di denominazione consigliate per le risorse di Azure).

Lo script fa riferimento al file dei parametri seguenti per creare le VM e l'infrastruttura circostante:

- **[virtualNetwork.parameters.json][vnet-parameters]**. Questo file definisce le impostazioni di rete virtuale, ad esempio il nome, lo spazio di indirizzi, la subnet e gli indirizzi di qualsiasi server DNS richiesto. Gli indirizzi della subnet devono essere inglobati dallo spazio di indirizzi della rete virtuale.

	<!-- source: https://github.com/mspnp/reference-architectures/blob/master/guidance-compute-single-vm/parameters/windows/virtualNetwork.parameters.json#L4-L21 -->
	```json
  "parameters": {
    "virtualNetworkSettings": {
      "value": {
        "name": "ra-single-vm-vnet",
        "resourceGroup": "ra-single-vm-rg",
        "addressPrefixes": [
          "172.17.0.0/16"
        ],
        "subnets": [
          {
            "name": "ra-single-vm-sn",
            "addressPrefix": "172.17.0.0/24"
          }
        ],
        "dnsServers": [ ]
      }
    }
  }
	```

- **[networkSecurityGroups.parameters.json][nsg-parameters]**. Questo file contiene le definizioni dei gruppi di sicurezza di rete (NSG) e delle relative regole. Il parametro `name` nel blocco `virtualNetworkSettings` specifica la rete virtuale a cui è associato il gruppo di sicurezza di rete. Il parametro `subnets` nel blocco `networkSecurityGroupSettings` identifica tutte le subnet che applicano le regole del gruppo di sicurezza di rete nella rete virtuale. Deve trattarsi di elementi definiti nel file **virtualNetwork.parameters.json**.

	La regola di sicurezza predefinita illustrata nell'esempio consente all'utente di connettersi alla VM tramite una connessione desktop remoto (RDP). È possibile aprire porte aggiuntive o negare l'accesso a porte specifiche aggiungendo altri elementi alla matrice `securityRules`.

	<!-- source: https://github.com/mspnp/reference-architectures/blob/master/guidance-compute-single-vm/parameters/windows/networkSecurityGroups.parameters.json#L4-L36 -->
	```json
  "parameters": {
    "virtualNetworkSettings": {
      "value": {
        "name": "ra-single-vm-vnet",
        "resourceGroup": "ra-single-vm-rg"
      }
    },
    "networkSecurityGroupsSettings": {
      "value": [
        {
          "name": "ra-single-vm-nsg",
          "subnets": [
            "ra-single-vm-sn"
          ],
          "networkInterfaces": [
          ],
          "securityRules": [
            {
              "name": "RDPAllow",
              "direction": "Inbound",
              "priority": 100,
              "sourceAddressPrefix": "*",
              "destinationAddressPrefix": "*",
              "sourcePortRange": "*",
              "destinationPortRange": "3389",
              "access": "Allow",
              "protocol": "Tcp"
            }
          ]
        }
      ]
    }
  }
	```

- **[virtualMachineParameters.json][vm-parameters]**. Questo file definisce le impostazioni della VM , inclusi il nome e le dimensioni, le credenziali di sicurezza per l'utente amministratore, i dischi da creare e gli account di archiviazione per contenere questi dischi.

	È necessario specificare un'immagine nella sezione `imageReference`. I valori riportati di seguito creano una VM con la build più recente di Windows Server 2012 R2 Datacenter. È possibile utilizzare la seguente interfaccia della riga di comando di Azure per ottenere un elenco di tutte le immagini di Windows disponibili in un'area (nell'esempio viene utilizzata l'area westus):

	```text
	azure vm image list westus MicrosoftWindowsServer WindowsServer
	```

	Il parametro `subnetName` nella sezione `nics` specifica la subnet per la VM. Analogamente, il parametro `name` in `virtualNetworkSettings` identifica la VM da usare. I valori devono corrispondere al nome della subnet e della rete virtuale definite nel file **virtualNetwork.parameters.json**.

	È possibile creare più VM attraverso la condivisione di un account di archiviazione o con i propri account di archiviazione modificando le impostazioni nella sezione `buildingBlockSettings`. Se si creano più VM, è necessario specificare anche il nome di un set di disponibilità da usare o creare nella sezione `availabilitySet`.

	<!-- source: https://github.com/mspnp/reference-architectures/blob/master/guidance-compute-single-vm/parameters/windows/virtualMachine.parameters.json#L4-L64 -->
	```json
   "parameters": {
    "virtualMachinesSettings": {
      "value": {
        "namePrefix": "ra-single-vm",
        "computerNamePrefix": "cn",
        "size": "Standard_DS1_v2",
        "osType": "windows",
        "adminUsername": "testuser",
        "adminPassword": "AweS0me@PW",
        "sshPublicKey": "",
        "osAuthenticationType": "password",
        "nics": [
          {
            "isPublic": "true",
            "subnetName": "ra-single-vm-sn",
            "privateIPAllocationMethod": "dynamic",
            "publicIPAllocationMethod": "dynamic",
            "enableIPForwarding": false,
            "dnsServers": [
            ],
            "isPrimary": "true"
          }
        ],
        "imageReference": {
          "publisher": "MicrosoftWindowsServer",
          "offer": "WindowsServer",
          "sku": "2012-R2-Datacenter",
          "version": "latest"
        },
        "dataDisks": {
          "count": 2,
          "properties": {
            "diskSizeGB": 128,
            "caching": "None",
            "createOption": "Empty"
          }
        },
        "osDisk": {
          "caching": "ReadWrite"
        },
        "extensions": [ ],
        "availabilitySet": {
          "useExistingAvailabilitySet": "No",
          "name": ""
        }
      }
    },
    "virtualNetworkSettings": {
      "value": {
        "name": "ra-single-vm-vnet",
        "resourceGroup": "ra-single-vm-rg"
      }
    },
    "buildingBlockSettings": {
      "value": {
        "storageAccountsCount": 1,
        "vmCount": 1,
        "vmStartIndex": 0
      }
    }
  }
	```

## Distribuzione della soluzione

Questa soluzione presume l'esistenza dei prerequisiti seguenti:

- L'utente dispone di una sottoscrizione Azure che consente di creare gruppi di risorse.

- L'utente ha scaricato e installato la build più recente di Azure PowerShell. Per le istruzioni, vedere [qui][azure-powershell-download].

Per eseguire lo script che distribuisce la soluzione:

1. Creare una cartella contenente le sottocartelle denominate `Scripts` e `Templates`.

2. Nella cartella Modelli, creare un'altra sottocartella e rinominarla Windows.

3. Scaricare il file [Deploy-ReferenceArchitecture.ps1][solution-script] nella cartella Script.

4. Scaricare i seguenti file nella cartella Modelli/Windows:

	- [virtualNetwork.parameters.json][vnet-parameters]

	- [networkSecurityGroups.parameters.json][nsg-parameters]

	- [virtualMachineParameters.json][vm-parameters]

5. Modificare il file Deploy-ReferenceArchitecture.ps1 nella cartella Script e modificare la riga seguente per specificare il gruppo di risorse che deve essere creato o utilizzato per contenere la VM e le risorse create dallo script:

	<!-- source: https://github.com/mspnp/reference-architectures/blob/master/guidance-compute-single-vm/Deploy-ReferenceArchitecture.ps1#L37 -->
	```powershell
	$resourceGroupName = "ra-single-vm-rg"
	```
6. Modificare ogni file JSON nella cartella Modelli/Windows per impostare i parametri relativi a rete virtuale, gruppo di sicurezza di rete e VM, come descritto nella sezione Componenti della soluzione.

	>[AZURE.NOTE] Assicurarsi di impostare il parametro `resourceGroup` nella sezione `virtualNetworkSettings` del file virtualMachineParameters.json affinché corrisponda a quanto specificato nel file dello script Deploy-ReferenceArchitecture.ps1.

7. Aprire una finestra di PowerShell, accedere alla cartella Script ed eseguire il comando seguente:

	```powershell
	.\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> Windows
	```

	Sostituire `<subscription id>` con l'ID della sottoscrizione di Azure.

	Per `<location>`, specificare un'area di Azure, come `eastus` o `westus`.

8. Al termine dello script, utilizzare il portale di Azure per verificare che la creazione della rete, del gruppo di sicurezza di rete e della VM sia riuscita perfettamente.

## Passaggi successivi

Per poter applicare il [Contratto di Servizio per Macchine virtuali][vm-sla], è necessario distribuire due o più istanze in un set di disponibilità. Per altre informazioni, vedere [Running multiple VMs on Azure][multi-vm] \(Esecuzione di più VM in Azure).

<!-- links -->

[audit-logs]: https://azure.microsoft.com/blog/analyze-azure-audit-logs-in-powerbi-more/
[availability-set]: ../articles/virtual-machines/virtual-machines-windows-create-availability-set.md
[azure-cli]: ../articles/virtual-machines-command-line-tools.md
[azure-storage]: ../articles/storage/storage-introduction.md
[blob-snapshot]: ../articles/storage/storage-blob-snapshots.md
[blob-storage]: ../articles/storage/storage-introduction.md
[boot-diagnostics]: https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/
[cname-record]: https://en.wikipedia.org/wiki/CNAME_record
[data-disk]: ../articles/virtual-machines/virtual-machines-windows-about-disks-vhds.md
[disk-encryption]: ../articles/security/azure-security-disk-encryption.md
[enable-monitoring]: ../articles/azure-portal/insights-how-to-use-diagnostics.md
[fqdn]: ../articles/virtual-machines/virtual-machines-windows-portal-create-fqdn.md
[group-policy]: https://technet.microsoft.com/it-IT/library/dn595129.aspx
[log-collector]: https://azure.microsoft.com/blog/simplifying-virtual-machine-troubleshooting-using-azure-log-collector/
[manage-vm-availability]: ../articles/virtual-machines/virtual-machines-windows-manage-availability.md
[multi-vm]: ../articles/guidance/guidance-compute-multi-vm.md
[naming conventions]: ../articles/guidance/guidance-naming-conventions.md
[nsg]: ../articles/virtual-network/virtual-networks-nsg.md
[nsg-default-rules]: ../articles/virtual-network/virtual-networks-nsg.md#default-rules
[planned-maintenance]: ../articles/virtual-machines/virtual-machines-windows-planned-maintenance.md
[premium-storage]: ../articles/storage/storage-premium-storage.md
[rbac]: ../articles/active-directory/role-based-access-control-what-is.md
[rbac-roles]: ../articles/active-directory/role-based-access-built-in-roles.md
[rbac-devtest]: ../articles/active-directory/role-based-access-built-in-roles.md#devtest-lab-user
[rbac-network]: ../articles/active-directory/role-based-access-built-in-roles.md#network-contributor
[reboot-logs]: https://azure.microsoft.com/blog/viewing-vm-reboot-logs/
[resize-os-disk]: ../articles/virtual-machines/virtual-machines-windows-expand-os-disk.md
[Resize-VHD]: https://technet.microsoft.com/it-IT/library/hh848535.aspx
[Resize virtual machines]: https://azure.microsoft.com/blog/resize-virtual-machines/
[resource-lock]: ../articles/resource-group-lock-resources.md
[resource-manager-overview]: ../articles/resource-group-overview.md
[security-center]: https://azure.microsoft.com/services/security-center/
[select-vm-image]: ../articles/virtual-machines/virtual-machines-windows-cli-ps-findimage.md
[services-by-region]: https://azure.microsoft.com/regions/#services
[static-ip]: ../articles/virtual-network/virtual-networks-reserved-public-ip.md
[storage-price]: https://azure.microsoft.com/pricing/details/storage/
[Usare il Centro sicurezza]: ../articles/security-center/security-center-get-started.md#use-security-center
[virtual-machine-sizes]: ../articles/virtual-machines/virtual-machines-windows-sizes.md
[vm-disk-limits]: ../articles/azure-subscription-service-limits.md#virtual-machine-disk-limits
[vm-resize]: ../articles/virtual-machines/virtual-machines-linux-change-vm-size.md
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_0/
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
[solution-script]: https://github.com/mspnp/reference-architectures/tree/master/guidance-compute-single-vm/Deploy-ReferenceArchitecture.ps1
[vnet-parameters]: https://github.com/mspnp/reference-architectures/tree/master/guidance-compute-single-vm/parameters/windows/virtualNetwork.parameters.json
[nsg-parameters]: https://github.com/mspnp/reference-architectures/tree/master/guidance-compute-single-vm/parameters/windows/networkSecurityGroups.parameters.json
[vm-parameters]: https://github.com/mspnp/reference-architectures/tree/master/guidance-compute-single-vm/parameters/windows/virtualMachine.parameters.json
[azure-powershell-download]: https://azure.microsoft.com/documentation/articles/powershell-install-configure/
[0]: ./media/guidance-blueprints/compute-single-vm.png "Singola architettura VM di Windows in Azure"

<!---HONumber=AcomDC_0831_2016-->
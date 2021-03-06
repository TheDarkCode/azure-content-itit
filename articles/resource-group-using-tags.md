<properties
	pageTitle="Uso dei tag per organizzare le risorse di Azure | Microsoft Azure"
	description="Mostra come applicare i tag per organizzare le risorse per la fatturazione e la gestione."
	services="azure-resource-manager"
	documentationCenter=""
	authors="tfitzmac"
	manager="timlt"
	editor="tysonn"/>

<tags
	ms.service="azure-resource-manager"
	ms.workload="multiple"
	ms.tgt_pltfrm="AzurePortal"
	ms.devlang="na"
	ms.topic="article"
	ms.date="08/16/2016"
	ms.author="tomfitz"/>


# Uso dei tag per organizzare le risorse di Azure

La Gestione risorse consente di organizzare logicamente le risorse tramite l'applicazione di tag. I tag sono costituiti da coppie chiave/valore che identificano le risorse con le proprietà definite. Per contrassegnare le risorse come appartenenti alla stessa categoria, applicare lo stesso tag a tali risorse.

Quando si visualizzano le risorse con un tag specifico, vengono visualizzate le risorse da tutti i gruppi di risorse. Non si è limitati unicamente alle risorse che appartengono allo stesso gruppo di risorse, cosa che consente di organizzare le risorse in modo indipendente rispetto alle relazioni di distribuzione. I tag possono essere utili quando è necessario organizzare le risorse per la fatturazione o la gestione.

Ogni tag aggiunto a una risorsa o a un gruppo di risorse viene aggiunto automaticamente alla tassonomia a livello di sottoscrizione. È anche possibile precompilare tale tassonomia con nomi e valori di tag che si vogliono usare in futuro come risorse provviste di tag.

Ciascuna risorsa o gruppo di risorse può avere un massimo di 15 tag. Il nome del tag è limitato a 512 caratteri e il valore del tag è limitato a 256 caratteri.

> [AZURE.NOTE] È possibile applicare dei tag solo alle risorse che supportano le operazioni di gestione delle risorse. Se è stata creata una macchina virtuale, una rete virtuale o un'archiviazione tramite il modello di distribuzione classica, ad esempio tramite il portale di Azure classico, non è possibile applicare un tag a tale risorsa. Per supportare l'assegnazione di tag, distribuire nuovamente tali risorse tramite Gestione risorse. Tutte le altre risorse supportano l'assegnazione di tag.

## Modelli

Per applicare un tag a una risorsa durante la distribuzione, è sufficiente aggiungere l'elemento **tags** alla risorsa che si intende distribuire e specificare il nome e il valore del tag. Non è necessario che il nome e il valore del tag esistano già nella sottoscrizione. È possibile specificare fino a 15 tag per ogni risorsa.

L'esempio seguente illustra un account di archiviazione con un tag.

    "resources": [
        {
            "type": "Microsoft.Storage/storageAccounts",
            "apiVersion": "2015-06-15",
            "name": "[concat('storage', uniqueString(resourceGroup().id))]",
            "location": "[resourceGroup().location]",
            "tags": {
                "dept": "Finance"
            },
            "properties": 
            {
                "accountType": "Standard_LRS"
            }
        }
    ]

Attualmente Gestione risorse non supporta l'elaborazione di un oggetto per i nomi e i valori del tag. In alternativa, è possibile passare un oggetto per i valori di tag, ma è comunque necessario specificare i nomi dei tag, come illustrato nell'esempio seguente.

    {
      "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "tagvalues": {
          "type": "object",
          "defaultValue": {
            "dept": "Finance",
            "project": "Test"
          }
        }
      },
      "resources": [
      {
        "apiVersion": "2015-06-15",
        "type": "Microsoft.Storage/storageAccounts",
        "name": "examplestorage",
        "tags": {
          "dept": "[parameters('tagvalues').dept]",
          "project": "[parameters('tagvalues').project]"
        },
        "location": "[resourceGroup().location]",
        "properties": {
          "accountType": "Standard_LRS"
        }
      }]
    }


## di Microsoft Azure

[AZURE.INCLUDE [resource-manager-tag-resource](../includes/resource-manager-tag-resources.md)]

## PowerShell

[AZURE.INCLUDE [resource-manager-tag-resources](../includes/resource-manager-tag-resources-powershell.md)]

## Interfaccia della riga di comando di Azure

I tag risultano direttamente sulle risorse e sui gruppi di risorse. Per visualizzare i tag esistenti, è sufficiente recuperare un gruppo di risorse e le relative risorse con **azure group show**.

    azure group show -n tag-demo-group
    
Verranno restituiti i metadati relativi al gruppo di risorse, inclusi gli eventuali tag applicati.
    
    info:    Executing command group show
    + Listing resource groups
    + Listing resources for the group
    data:    Id:                  /subscriptions/{guid}/resourceGroups/tag-demo-group
    data:    Name:                tag-demo-group
    data:    Location:            westus
    data:    Provisioning State:  Succeeded
    data:    Tags: Dept=Finance;Environment=Production
    data:    Resources:
    data:
    data:      Id      : /subscriptions/{guid}/resourceGroups/tag-demo-group/providers/Microsoft.Sql/servers/tfsqlserver
    data:      Name    : tfsqlserver
    data:      Type    : servers
    data:      Location: eastus2
    data:      Tags    : Dept=Finance;Environment=Production
    ...

Per ottenere i tag solo per il gruppo di risorse, usare un'utilità JSON come [jq](http://stedolan.github.io/jq/download/).

    azure group show -n tag-demo-group --json | jq ".tags"
    
Verranno restituiti i tag per tale gruppo di risorse.
    
    {
      "Dept": "Finance",
      "Environment": "Production" 
    }

È possibile visualizzare i tag per una particolare risorsa usando **azure resource show**.

    azure resource show -g tag-demo-group -n tfsqlserver -r Microsoft.Sql/servers -o 2014-04-01-preview --json | jq ".tags"
    
Verranno restituiti i tag per tale risorsa.
    
    {
      "Dept": "Finance",
      "Environment": "Production"
    }
    
Nell'esempio seguente viene illustrato come recuperare tutte le risorse che dispongono di un valore e un nome di tag.

    azure resource list --json | jq ".[] | select(.tags.Dept == "Finance") | .name"
    
Verranno restituiti i nomi delle risorse con tale tag.
    
    "tfsqlserver"
    "tfsqlserver/tfsqldata"

I tag vengono aggiornati nel loro complesso. Per aggiungere un tag a una risorsa che ne dispone già, recuperare tutti i tag esistenti che si desidera conservare. Per impostare i valori dei tag per un gruppo di risorse, usare **azure group set** e fornire tutti i tag per il gruppo di risorse.

    azure group set -n tag-demo-group -t Dept=Finance;Environment=Production;Project=Upgrade
    
Viene restituito un riepilogo del gruppo di risorse con i nuovi tag.
    
    info:    Executing command group set
    ...
    data:    Name:                tag-demo-group
    data:    Location:            westus
    data:    Provisioning State:  Succeeded
    data:    Tags: Dept=Finance;Environment=Production;Project=Upgrade
    ...
    
È possibile elencare i tag esistenti nella sottoscrizione con **azure tag list** e aggiungere un tag con **azure tag create**. Per rimuovere un tag dalla tassonomia per la sottoscrizione, per prima cosa rimuovere il tag dalle eventuali risorse. Dopodiché, rimuovere il tag con **azure tag delete**.

## API REST

Sia il portale che PowerShell usano l'[API REST di Gestione risorse](https://msdn.microsoft.com/library/azure/dn848368.aspx) dietro le quinte. Se è necessario integrare l'assegnazione di tag in un altro ambiente, è possibile ottenere i tag con un'operazione GET sull'ID di risorsa e aggiornare il set di tag con una chiamata PATCH.


## Tag e fatturazione

Per i servizi supportati, è possibile utilizzare i tag per raggruppare i dati di fatturazione. Ad esempio, [Macchine virtuali integrato con Gestione risorse di Azure](./virtual-machines/virtual-machines-windows-compare-deployment-models.md) consente di definire e applicare i tag per organizzare l'utilizzo della fatturazione per le macchine virtuali. Se si eseguono più macchine virtuali per organizzazioni diverse, è possibile utilizzare i tag per raggruppare l'utilizzo in base al centro di costo. È inoltre possibile utilizzare i tag per classificare i costi in base all'ambiente di runtime; ad esempio, l'utilizzo di fatturazione per le macchine virtuali in esecuzione nell'ambiente di produzione.

Le informazioni sui tag possono essere recuperate tramite l'[API di utilizzo della risorsa di Azure e della Rate Card](billing-usage-rate-card-overview.md) o il file di utilizzo con valori delimitati da virgole (CSV). Il file di utilizzo può essere scaricato dal [portale degli account di Azure](https://account.windowsazure.com/) o dal [portale EA](https://ea.azure.com). Per altre informazioni sull'accesso a livello di codice alle informazioni sulla fatturazione, vedere [Ottenere informazioni dettagliate sul consumo delle risorse di Microsoft Azure](billing-usage-rate-card-overview.md). Per le operazioni API REST, vedere [Riferimento API REST alla fatturazione di Azure](https://msdn.microsoft.com/library/azure/1ea5b323-54bb-423d-916f-190de96c6a3c).

Quando si scarica il CSV di utilizzo per i servizi che supportano i tag di fatturazione, i tag vengono visualizzati nella colonna **Tag**. Per ulteriori informazioni, vedere [Informazioni sulla fattura di Microsoft Azure](billing-understand-your-bill.md).

![Vedere i tag della fatturazione](./media/resource-group-using-tags/billing_csv.png)

## Passaggi successivi

- È possibile applicare restrizioni e convenzioni all’interno della sottoscrizione con criteri personalizzati. Il criterio definito potrebbe richiedere che per tutte le risorse sia impostato un determinato tag. Per altre informazioni, vedere [Usare i criteri per gestire le risorse e controllare l'accesso](resource-manager-policy.md).
- Per un'introduzione all'uso di Azure PowerShell durante la distribuzione delle risorse, vedere [Uso di Azure PowerShell con Gestione risorse di Azure](./powershell-azure-resource-manager.md).
- Per un'introduzione all'uso dell'interfaccia della riga di comando di Azure durante la distribuzione delle risorse, vedere [Uso dell'interfaccia della riga di comando di Azure per Mac, Linux e Windows con Gestione risorse di Azure](./xplat-cli-azure-resource-manager.md).
- Per un'introduzione all'uso del portale, vedere [Uso del portale di Azure per gestire le risorse di Azure](./azure-portal/resource-group-portal.md)

<!---HONumber=AcomDC_0817_2016-->
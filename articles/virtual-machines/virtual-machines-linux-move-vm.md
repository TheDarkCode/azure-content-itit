<properties
	pageTitle="Spostare una VM di Linux | Microsoft Azure"
	description="Spostare una VM di Linux in un'altra sottoscrizione o in un altro gruppo di risorse di Azure nel modello di distribuzione di Resource Manager."
	services="virtual-machines-linux"
	documentationCenter=""
	authors="cynthn"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>  

<tags
	ms.service="virtual-machines-linux"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="08/08/2016"
	ms.author="cynthn"/>  

	


# Spostare una VM di Linux in un'altra sottoscrizione o in un altro gruppo di risorse

Questo articolo illustra come spostare una VM di Linux tra gruppi di risorse o sottoscrizioni. Lo spostamento di una VM tra sottoscrizioni può essere comodo se è stata creata una VM in una sottoscrizione personale e ora si desidera spostarla alla sottoscrizione dell'azienda.

> [AZURE.NOTE] Nell'ambito dello spostamento vengono creati nuovi ID risorsa. Una volta spostata la VM, è necessario aggiornare strumenti e script in modo che usino i nuovi ID risorsa.


## Usare l'interfaccia della riga di comando di Azure per spostare una VM 

Per spostare correttamente una VM, è necessario spostare la VM e tutte le relative risorse di supporto. Usare il comando **azure group show** per elencare tutte le risorse in un gruppo di risorse e i relativi ID. Può essere utile inviare tramite pipe l'output di questo comando in un file, in modo da poter copiare e incollare gli ID in comandi successivi.

	azure group show <resourceGroupName>

Per spostare la VM e le relative risorse in un altro gruppo di risorse, usare il comando dell'interfaccia della riga di comando **azure resource move**. L'esempio seguente mostra come spostare una VM e la maggior parte delle risorse comuni richieste. Viene usato il parametro **-i** e poi passato un elenco delimitato da virgole (senza spazi) degli ID per le risorse da spostare.

	
    vm=/subscriptions/<sourceSubscriptionID>/resourceGroups/<sourceResourceGroup>/providers/Microsoft.Compute/virtualMachines/<vmName>
	nic=/subscriptions/<sourceSubscriptionID>/resourceGroups/<sourceResourceGroup>/providers/Microsoft.Network/networkInterfaces/<nicName>
	nsg=/subscriptions/<sourceSubscriptionID>/resourceGroups/<sourceResourceGroup>/providers/Microsoft.Network/networkSecurityGroups/<nsgName>
	pip=/subscriptions/<sourceSubscriptionID>/resourceGroups/<sourceResourceGroup>/providers/Microsoft.Network/publicIPAddresses/<publicIPName>
	vnet=/subscriptions/<sourceSubscriptionID>/resourceGroups/<sourceResourceGroup>/providers/Microsoft.Network/virtualNetworks/<vnetName>
	diag=/subscriptions/<sourceSubscriptionID>/resourceGroups/<sourceResourceGroup>/providers/Microsoft.Storage/storageAccounts/<diagnosticStorageAccountName>
	storage=/subscriptions/<sourceSubscriptionID>/resourceGroups/<sourceResourceGroup>/providers/Microsoft.Storage/storageAccounts/<storageAcountName>  	
	
	azure resource move --ids $vm,$nic,$nsg,$pip,$vnet,$storage,$diag -d "<destinationResourceGroup>"
	
Per spostare la VM e le relative risorse in una sottoscrizione diversa, aggiungere il parametro **--destination-subscriptionId &#60;destinationSubscriptionID&#62;** per specificare la sottoscrizione di destinazione.

Se si lavora dal prompt dei comandi in un computer Windows, è necessario aggiungere **$** davanti ai nomi delle variabili quando vengono dichiarate. Questo prefisso non è necessario in Linux.

Viene richiesto di confermare che si vuole spostare la risorsa specificata. Digitare **Y** per confermare lo spostamento delle risorse.
	

[AZURE.INCLUDE [virtual-machines-common-move-vm](../../includes/virtual-machines-common-move-vm.md)]

## Passaggi successivi

È possibile spostare molti tipi diversi di risorse tra gruppi di risorse e sottoscrizioni. Per altre informazioni, vedere [Spostare le risorse in un gruppo di risorse o una sottoscrizione nuovi](../resource-group-move-resources.md).

<!---HONumber=AcomDC_0810_2016-->
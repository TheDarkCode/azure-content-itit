<properties
	pageTitle="Spostare database tra server, tra sottoscrizioni e all'interno e all'esterno di Azure."
	description="Azioni rapide per copiare, spostare ed eseguire la migrazione di dati e database nel database SQL di Azure."
	services="sql-database"
	documentationCenter=""
	authors="v-shysun"
	manager="felixwu"
	editor=""/>

<tags
	ms.service="sql-database"
	ms.workload="data-management"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="06/03/2016"
	ms.author="v-shysun"/>

# Spostare database tra server, tra sottoscrizioni e all'interno e all'esterno di Azure

[AZURE.INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]
##Per spostare un database in un server diverso nella stessa sottoscrizione
- Nel [portale di Azure](https://portal.azure.com) fare clic su **Database SQL**, selezionare un database dall'elenco e quindi fare clic su **Copia**. Per altre informazioni, vedere [Copiare un database SQL di Azure](sql-database-copy.md).

## Per spostare un database tra sottoscrizioni
- Nel [portale di Azure](https://portal.azure.com) fare clic su **SQL Server** e quindi selezionare dall'elenco il server che ospita il database. Fare clic su **Sposta**, quindi selezionare le risorse da spostare e la sottoscrizione in cui spostarle.

## Per eseguire la migrazione di un database SQL in Azure
- Determinare la compatibilità del database e quindi selezionare il metodo di migrazione appropriato in base alle esigenze. Seguire le linee guida e le opzioni disponibili in [Migrazione di un database SQL Server al database SQL nel cloud](sql-database-cloud-migrate.md).

## Per creare una copia di un database da usare all'esterno di Azure
- [Esportare un file BACPAC.](sql-database-export.md)

<!---HONumber=AcomDC_0608_2016-->
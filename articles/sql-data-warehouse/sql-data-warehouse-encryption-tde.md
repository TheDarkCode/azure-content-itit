<properties
   pageTitle="Introduzione a Transparent Data Encryption (TDE) di SQL Data Warehouse | Microsoft Azure"
   description="Introduzione a Transparent Data Encryption (TDE) di SQL Data Warehouse"
   services="sql-data-warehouse"
   documentationCenter=""
   authors="ronortloff"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.workload="data-management"
   ms.tgt_pltfrm="na"
   ms.devlang="na"
   ms.topic="article"
   ms.date="08/29/2016" 
   ms.author="rortloff;barbkess;sonyama"/>

# Introduzione a Transparent Data Encryption (TDE) di SQL Data Warehouse

> [AZURE.SELECTOR]
- [Proteggere un database in SQL Data Warehouse](sql-data-warehouse-overview-manage-security.md)
- [Introduzione al rilevamento delle minacce](sql-data-warehouse-security-threat-detection.md)
- [Introduzione a Transparent Data Encryption (TDE) di SQL Data Warehouse](sql-data-warehouse-encryption-tde.md)
- [Introduzione a Transparent Data Encryption (TDE)](sql-data-warehouse-encryption-tde-tsql.md)
- [Servizio di controllo di Azure SQL Data Warehouse](sql-data-warehouse-auditing-overview.md)
- [SQL Data Warehouse - Supporto client di livello inferiore per controllo e maschera dati dinamica](sql-data-warehouse-auditing-downlevel-clients.md)


La funzionalità Transparent Data Encryption (TDE) di SQL Data Warehouse di Azure consente di proteggersi da attività dannose eseguendo in tempo reale la crittografia e la decrittografia dei database, dei backup associati e dei file di log delle transazioni inattivi, senza dover apportare modifiche all'applicazione.

TDE esegue la crittografia dell'archiviazione di un intero database usando una chiave simmetrica detta "chiave di crittografia del database". Nel database SQL la chiave di crittografia del database è protetta da un certificato server incorporato. Il certificato server incorporato è univoco per ogni server di database SQL. Microsoft ruota automaticamente questi certificati almeno ogni 90 giorni. L’algoritmo di crittografia utilizzato da SQL Data Warehouse è AES-256. Per una descrizione generale della funzionalità TDE, vedere [Transparent Data Encryption (TDE)].

##Abilitazione della crittografia

Per abilitare TDE per un SQL Data Warehouse, seguire questa procedura:

1. Aprire il database nel [portale di Azure](https://portal.azure.com)
2. Nel pannello del database fare clic sul pulsante **Impostazioni**
3. Selezionare l’opzione ![][1] di **Crittografia dati trasparente**
4. Selezionare l’impostazione ![][2] su **Attiva**
5. Selezionare **Salva** ![][3]

##Disabilitazione della crittografia

Per disabilitare TDE per un SQL Data Warehouse, seguire questa procedura:

1. Aprire il database nel [portale di Azure](https://portal.azure.com)
2. Nel pannello del database fare clic sul pulsante **Impostazioni**
3. Selezionare l’opzione ![][1] di **Crittografia dati trasparente**
4. Selezionare l’impostazione ![][4] su **Disattiva**
5. Selezionare **Salva** ![][5]

##Viste a gestione dinamica della crittografia

La crittografia può essere confermata con le seguenti viste a gestione dinamica:

- [sys.databases]
- [sys.dm\_pdw\_nodes\_database\_encryption\_keys]

<!--MSDN references-->
[Transparent Data Encryption (TDE)]: https://msdn.microsoft.com/library/bb934049.aspx
[sys.databases]: http://msdn.microsoft.com/library/ms178534.aspx
[sys.dm\_pdw\_nodes\_database\_encryption\_keys]: https://msdn.microsoft.com/library/mt203922.aspx

<!--Image references-->
[1]: ./media/sql-data-warehouse-security-tde/sql-data-warehouse-security-tde-portal-settings.png
[2]: ./media/sql-data-warehouse-security-tde/sql-data-warehouse-security-tde-portal-settings-on.png
[3]: ./media/sql-data-warehouse-security-tde/sql-data-warehouse-security-tde-portal-settings-save.png
[4]: ./media/sql-data-warehouse-security-tde/sql-data-warehouse-security-tde-portal-settings-off.png
[5]: ./media/sql-data-warehouse-security-tde/sql-data-warehouse-security-tde-portal-settings-save2.png

<!--Link references-->

<!---HONumber=AcomDC_0907_2016-->
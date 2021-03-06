<properties
	pageTitle="Soluzione di valutazione della configurazione di Log Analytics | Microsoft Azure"
	description="La soluzione di valutazione della configurazione di Log Analytics fornisce informazioni dettagliate sullo stato corrente dell'infrastruttura server di System Center Operations Manager se si usano agenti Operations Manager o un gruppo di gestione di Operations Manager."
	services="log-analytics"
	documentationCenter=""
	authors="bandersmsft"
	manager="jwhit"
	editor=""/>

<tags
	ms.service="log-analytics"
	ms.workload="na"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="05/04/2016"
	ms.author="banders"/>

# Soluzione di valutazione della configurazione di Log Analytics

La soluzione di valutazione della configurazione di Log Analytics consente di individuare potenziali problemi di configurazione dei server tramite avvisi e consigli della Knowledge Base.

Questa soluzione richiede System Center Operations Manager. La soluzione di valutazione della configurazione non è disponibile se si usa solo agenti connessi direttamente.

La visualizzazione di alcune informazioni nella soluzione di valutazione della configurazione richiede il plug-in Silverlight per il browser in uso.

>[AZURE.NOTE] La soluzione di valutazione della configurazione è disponibile solo per aree di lavoro dell'area Stati Uniti orientali e non verrà aggiunta in altre aree. Le funzionalità della soluzione di valutazione della configurazione verranno sostituite da funzioni di valutazione specifiche dei carichi di lavoro, quali Active Directory Assessment e SQL Server Assessment.

![Valutazione della configurazione - Riquadro](./media/log-analytics-configuration-assessment/oms-config-assess-tile.png)

I dati di configurazione vengono raccolti dai server monitorati e quindi inviati al servizio OMS nel cloud per poter essere elaborati. Viene applicata la logica ai dati ricevuti, quindi questi ultimi vengono registrati nel servizio cloud. I dati elaborati per i server vengono visualizzati per le aree seguenti:

- **Avvisi:** mostra gli avvisi proattivi relativi alla configurazione generati per i server monitorati. Gli avvisi vengono generati da regole definite dal Servizio Supporto Tecnico Clienti Microsoft sulla base delle procedure consigliate derivanti dall'esperienza sul campo.
- **Consigli della Knowledge Base**: elenca gli articoli della Microsoft Knowledge Base consigliati per i carichi di lavoro rilevati nell'infrastruttura. Gli articoli vengono suggeriti automaticamente in base alla configurazione tramite Machine Learning.
- **Server e carichi di lavoro analizzati**: elenca i server e i carichi di lavoro monitorati da OMS.

![Valutazione della configurazione - Dashboard](./media/log-analytics-configuration-assessment/oms-config-assess-dash01.png)

### Tecnologie che è possibile analizzare con la soluzione di valutazione della configurazione

La soluzione di valutazione della configurazione di OMS analizza i carichi di lavoro seguenti:

- Windows Server 2012 e Microsoft Hyper-V Server 2012
- Windows Server 2008 e Windows Server 2008 R2, inclusi:
    - Active Directory
	- Host Hyper-V
	- Sistema operativo generale
- SQL Server 2008 e versioni successive
    - Motore di database di SQL Server
- Microsoft SharePoint 2010
- Microsoft Exchange Server 2010 e Microsoft Exchange Server 2013
- Microsoft Lync Server 2013 e Lync Server 2010
- System Center 2012 SP1 – Virtual Machine Manager

Per SQL Server sono supportate per l'analisi le edizioni seguenti a 32 bit e a 64 bit:

- SQL Server 2016 - Tutte le edizioni
- SQL Server 2014 - Tutte le edizioni
- SQL Server 2008 e 2008 R2 - Tutte le edizioni

Il motore di database di SQL Server viene analizzato in tutte le edizioni supportate. È inoltre supportata l'edizione a 32 bit di SQL Server eseguita nell'implementazione WOW64.

## Installazione e configurazione della soluzione
Usare le informazioni seguenti per installare e configurare la soluzione.

- Per la soluzione di valutazione della configurazione è necessario Operations Manager.
- In particolare, è necessario disporre di un agente Operations Manager su ciascun computer di cui si intende valutare la configurazione.
- Aggiungere la soluzione di valutazione della configurazione all'area di lavoro di OMS usando la procedura descritta nell'articolo [Aggiungere soluzioni di Log Analytics dalla Raccolta soluzioni](log-analytics-add-solutions.md). Non è richiesta alcuna ulteriore configurazione.

## Informazioni dettagliate sulla raccolta dei dati per la valutazione della configurazione

La tabella seguente illustra i metodi di raccolta dei dati e altre informazioni dettagliate sul modo in cui vengono raccolti i dati per la valutazione della configurazione.

| piattaforma | Agente diretto | Agente SCOM | Archiviazione di Azure | SCOM obbligatorio? | Dati dell'agente SCOM inviati con il gruppo di gestione | frequenza della raccolta |
|---|---|---|---|---|---|---|
|Windows|![No](./media/log-analytics-configuration-assessment/oms-bullet-red.png)|![Sì](./media/log-analytics-configuration-assessment/oms-bullet-green.png)|![No](./media/log-analytics-configuration-assessment/oms-bullet-red.png)| ![Sì](./media/log-analytics-configuration-assessment/oms-bullet-green.png)|![Sì](./media/log-analytics-configuration-assessment/oms-bullet-green.png)| due volte al giorno|


## Avvisi di valutazione della configurazione
È possibile visualizzare e gestire gli avvisi generati dalla soluzione di valutazione della configurazione nella pagina degli avvisi. Gli avvisi indicano il problema rilevato, la causa e la soluzione da adottare. Forniscono inoltre informazioni sulle impostazioni di configurazione dell'ambiente che potrebbero generare problemi di prestazioni.

![vista avvisi](./media/log-analytics-configuration-assessment/oms-config-assess-alerts01.png)

>[AZURE.NOTE] Gli avvisi della soluzione di valutazione della configurazione sono diversi dagli altri avvisi di Log Analytics e, per poterli visualizzare, è necessario il plug-in Silverlight per il browser in uso.

Quando si seleziona un elemento o una categoria di elementi nella pagina degli avvisi, viene visualizzato un elenco di server o carichi di lavoro, con i relativi avvisi.

![elenco avvisi](./media/log-analytics-configuration-assessment/oms-config-assess-alerts-view-config.png)

Ogni avviso include un collegamento a un articolo della Microsoft Knowledge Base, in cui sono disponibili informazioni aggiuntive sull'avviso.

>[AZURE.TIP] Per impostazione predefinita, il numero massimo di avvisi visualizzati è 2.000. Per visualizzarne un numero superiore, fare clic sulla barra di notifica sopra l'elenco degli avvisi.

È possibile fare clic su un elemento qualsiasi dell'elenco per visualizzare l'articolo della Knowledge Base che aiuta a identificare e risolvere la causa del problema che ha generato l'avviso.

![articolo KB](./media/log-analytics-configuration-assessment/oms-config-assess-alerts-details-kb.png)

È possibile intervenire sull regole di avviso per ignorare specifiche regole o classi di regole.

![gestione delle regole di avviso](./media/log-analytics-configuration-assessment/oms-config-assess-alert-rules.png)

## Consigli della Knowledge Base
Se si sceglie di visualizzare i consigli della Knowledge Base, vengono presentati i risultati di una ricerca nei log con l'elenco degli articoli della Microsoft Knowledge Base consigliati per i carichi di lavoro e i computer in uso, contenenti informazioni aggiuntive sull'avviso.

![risultati della ricerca per consigli della Knowledge Base](./media/log-analytics-configuration-assessment/oms-config-assess-knowledge-recommendations.png)

## Server e carichi di lavoro analizzati
Se si sceglie di visualizzare i consigli della Knowledge Base, vengono presentati i risultati di una ricerca nei log con l'elenco di tutti i server e i carichi di lavoro noti a OMS tramite Operations Manager.

![server e carichi di lavoro](./media/log-analytics-configuration-assessment/oms-config-assess-servers-workloads.png)

## Passaggi successivi

- Per visualizzare informazioni dettagliate sulla valutazione della configurazione, usare [Ricerche nei log in Log Analytics](log-analytics-log-searches.md).

<!---HONumber=AcomDC_0518_2016-->
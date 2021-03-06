<properties
   pageTitle="Concetti relativi alla progettazione per Azure AD Connect | Microsoft Azure"
   description="Questo argomento illustra alcune aree di progettazione dell'implementazione."
   services="active-directory"
   documentationCenter=""
   authors="AndKjell"
   manager="stevenpo"
   editor=""/>

<tags
   ms.service="active-directory"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="Identity"
   ms.date="06/27/2016"
   ms.author="andkjell"/>

# Azure AD Connect: Concetti relativi alla progettazione
L'obiettivo di questo argomento consiste nell'illustrare le aree da esaminare durante la progettazione dell'implementazione di Azure AD Connect. Si tratta di un'analisi approfondita di determinate aree e questi concetti vengono illustrati brevemente anche in altri argomenti.

## sourceAnchor
L'attributo sourceAnchor viene definito come *un attributo immutabile durante il ciclo di vita di un oggetto*. Identifica in modo univoco un oggetto come corrispondente all'oggetto locale in Azure AD. L'attributo viene definito anche **immutableId** e i due nomi sono usati in modo intercambiabile.

Il termine immutabile, ovvero non modificabile, è importante in questo argomento. Poiché il valore di questo attributo non può essere modificato dopo la configurazione, è essenziale scegliere una progettazione in grado di supportare lo scenario desiderato.

L'attributo viene usato per gli scenari seguenti:

- Quando viene compilato un nuovo motore di sincronizzazione o quando il motore viene ricompilato dopo uno scenario di ripristino di emergenza, questo attributo collegherà gli oggetti esistenti in Azure AD con gli oggetti locali.
- Se si passa da un modello di identità solo cloud a un modello di identità sincronizzato, questo attributo consentirà la "corrispondenza rigida" degli oggetti esistenti in Azure AD con gli oggetti locali.
- Se si usa la federazione, questo attributo viene usato insieme a **userPrincipalName** nell'attestazione per identificare in modo univoco un utente.

Questo argomento esaminerà sourceAnchor solo relativamente agli utenti. Le stesse regole sono applicabili a tutti i tipi di oggetto, ma in genere presentano problemi solo per gli utenti.

### Selezione di un attributo sourceAnchor valido
Il valore dell'attributo deve rispettare le regole seguenti:

- La lunghezza del testo deve essere inferiore a 60 caratteri
    - I caratteri diversi da a-z, A-Z o 0-9 verranno codificati e conteggiati come 3 caratteri
- Non deve contenere un carattere speciale: &#92; ! # $ % & * + / = ? ^ &#96; { }| ~ < > ( ) ' ; : , [ ] " @ _
- Deve essere globalmente univoco
- Deve essere una stringa, un valore intero o un numero binario
- Non deve essere basato sul nome dell'utente, queste modifiche
- Non devono fare distinzione tra maiuscole e minuscole e devono evitare valori che possono variare in base alle maiuscole/minuscole
- Deve essere assegnato quando viene creato l'oggetto.

Se l'attributo sourceAnchor selezionato non è di tipo stringa, Azure AD Connect userà Base64Encode per il valore dell'attributo per assicurare che non vengano visualizzati caratteri speciali. Se si usa un altro server federativo, assicurarsi che il server sia in grado di applicare Base64Encode all'attributo.

L'attributo sourceAnchor rispetta la distinzione tra maiuscole e minuscole. Un valore quale "DavideMilano" è diverso da "davidemilano".

Se è presente una singola foresta locale, l'attributo da usare sarà **objectGUID**. Questo attributo deve essere usato anche con le impostazioni rapide in Azure AD Connect e viene usato anche da DirSync.

Se sono presenti più foreste e gli utenti non vengono spostati tra foreste e domini nella stessa foresta, è possibile usare comunque l'attributo **objectGUID**.

Se si spostano gli utenti tra foreste e domini, sarà necessario trovare un attributo che non verrà modificato o che possa essere spostato con gli utenti durante lo spostamento. Un approccio consigliato consiste nell'introdurre un attributo sintetico. È possibile usare un attributo che include un elemento analogo a un GUID. Un nuovo GUID viene creato e assegnato all'utente durante la creazione di un oggetto. Una regola personalizzata può essere creata nel server del motore di sincronizzazione per creare questo valore in base all’**objectGUID** e aggiornare l'attributo selezionato in ADDS. Quando si sposta l'oggetto, assicurarsi di copiare anche il contenuto di questo valore.

Un'altra soluzione consiste nello scegliere un attributo esistente che si sa che non cambierà. Uno degli attributi più comunemente usati è **employeeID**. Se si prende in considerazione un attributo che conterrà lettere, assicurarsi che non vi sia alcuna possibilità che le lettere maiuscole e/o minuscole usate per il valore dell'attributo vengano modificate. Gli attributi non validi che non devono essere usati includono gli attributi con il nome dell'utente. In caso di matrimonio o divorzio è possibile che il nome venga modificato e ciò non è consentito per questo attributo. Per questo motivo non è nemmeno possibile selezionare attributi quali **userPrincipalName**, **mail** e **targetAddress** nell'installazione guidata di Azure AD Connect. Questi attributi conterranno anche il carattere @, che non è consentito in sourceAnchor.

### Modifica dell'attributo sourceAnchor
Il valore dell'attributo sourceAnchor non può essere modificato dopo la creazione dell'oggetto in Azure AD e la sincronizzazione dell'identità.

Le restrizioni seguenti sono quindi applicabili ad Azure AD Connect:

- L'attributo sourceAnchor può essere configurato solo durante l'installazione iniziale. Se si esegue di nuovo l'installazione guidata, questa opzione sarà di sola lettura. Per modificarla, sarà necessario eseguire la disinstallazione e la reinstallazione.
- Se si installa un altro server di Azure AD Connect, sarà necessario selezionare lo stesso attributo sourceAnchor usato in precedenza. Se si passa ad Azure AD Connect dopo avere usato DirSync, sarà necessario usare **objectGUID**, perché questo è l'attributo usato da DirSync.
- Se il valore di sourceAnchor viene modificato dopo l'esportazione dell'oggetto in Azure AD, il servizio di sincronizzazione Azure AD Connect genererà un errore e non consentirà altre modifiche all'oggetto prima della risoluzione del problema e della reimpostazione del valore precedente di sourceAnchor nella directory di origine.

## Accesso ad Azure AD

Durante l'integrazione della directory locale con Azure AD, è importante capire come le impostazioni di sincronizzazione possono influire sul modo in cui l'utente esegue l'autenticazione. Azure AD usa userPrincipalName o UPN per autenticare l'utente. Tuttavia, quando si sincronizzano gli utenti è necessario scegliere con attenzione l'attributo da usare per userPrincipalName.

### Scegliere l'attributo per userPrincipalName

Quando si seleziona l'attributo per specificare il valore di UPN da usare in Azure è necessario verificare quanto segue:

* I valori di attributo devono essere conformi alla sintassi UPN (RFC 822), ovvero devono essere nel formato username@domain.
* Il suffisso nei valori deve corrispondere a uno dei domini personalizzati verificati in Azure AD

Nelle impostazioni rapide come attributo deve essere scelto userPrincipalName. Se tuttavia si ritiene che l'attributo userPrincipalName non contenga il valore che gli utenti dovrebbero usare per l'accesso ad Azure, è necessario scegliere **Installazione personalizzata** e indicare l'attributo appropriato.

### Stato del dominio personalizzato e UPN
È importante verificare che esista un dominio verificato per il suffisso UPN.

John è un utente di contoso.com. Si vuole fare in modo che John usi l'UPN locale john@contoso.com per l'accesso ad Azure dopo che è stata eseguita la sincronizzazione degli utenti con la directory di Azure AD azurecontoso.onmicrosoft.com. A tale scopo, è necessario aggiungere e verificare contoso.com come dominio personalizzato in Azure AD per poter avviare la sincronizzazione degli utenti. Se il suffisso UPN di John, ad esempio contoso.com, non corrisponde a un dominio verificato in Azure AD, il suffisso UPN verrà sostituito con azurecontoso.onmicrosoft.com e John dovrà usare john@azurecontoso.onmicrosoft.com per accedere ad Azure.

### Domini locali non instradabili e UPN per Azure AD
Alcune organizzazioni usano domini non instradabili, ad esempio contoso.local, o domini semplici con etichetta singola come contoso. In Azure AD non sarà possibile verificare un dominio non instradabile. Azure AD Connect è in grado eseguire la sincronizzazione con un solo dominio verificato in Azure AD. Quando si crea una directory di Azure AD, viene creato un dominio instradabile che diventa dominio predefinito per Azure AD, ad esempio contoso.onmicrosoft.com. Si rende quindi necessario verificare eventuali altri domini instradabili nello stesso scenario, nel caso in cui non si voglia eseguire la sincronizzazione con il dominio .onmicrosoft.com predefinito.

Per altre informazioni sull'aggiunta e la verifica dei domini, vedere [Aggiungere un nome di dominio personalizzato ad Azure Active Directory](active-directory-add-domain.md).

Azure AD Connect rileva se l'esecuzione avviene in un ambiente di dominio non instradabile e avvisa che è opportuno non proseguire con le impostazioni rapide. Se si opera in un dominio non instradabile, è probabile che anche l'UPN degli utenti abbia un suffisso non instradabile. Ad esempio, se si opera in contoso.local, Azure AD Connect suggerisce di usare le impostazioni personalizzate anziché le impostazioni rapide. Con le impostazioni personalizzate sarà possibile specificare l'attributo che deve essere usato come nome UPN per l'accesso ad Azure dopo la sincronizzazione degli utenti con Azure AD. Per altre informazioni, vedere **Selezione dell'attributo per il nome dell'entità utente in Azure AD**.

## Passaggi successivi
Altre informazioni su [Integrazione delle identità locali con Azure Active Directory](active-directory-aadconnect.md).

<!---HONumber=AcomDC_0629_2016-->
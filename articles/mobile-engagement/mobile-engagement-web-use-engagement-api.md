<properties
	pageTitle="API di Azure Mobile Engagement SDK per Web | Microsoft Azure"
	description="Ultimi aggiornamenti e procedure relativi all'SDK per Web per Azure Mobile Engagement"
	services="mobile-engagement"
	documentationCenter="mobile"
	authors="piyushjo"
	manager="erikre"
	editor="" />

<tags
	ms.service="mobile-engagement"
	ms.workload="mobile"
	ms.tgt_pltfrm="web"
	ms.devlang="js"
	ms.topic="article"
	ms.date="06/07/2016"
	ms.author="piyushjo" />

# Usare l'API di Azure Mobile Engagement in un'applicazione Web

Questo documento è un'aggiunta al documento [Integrare Azure Mobile Engagement in un'applicazione Web](mobile-engagement-web-integrate-engagement.md). Offre informazioni approfondite su come usare l'API di Azure Mobile Engagement per segnalare le statistiche dell'applicazione.

L'API di Mobile Engagement viene messa a disposizione dall'oggetto `engagement.agent`. L'alias predefinito di Azure Mobile Engagement Web SDK è `engagement`. È possibile ridefinire l'alias dalla configurazione dell'SDK.

## Concetti relativi a Mobile Engagement

Le parti seguenti approfondiscono le informazioni contenute nell'articolo [Concetti relativi ad Azure Mobile Engagement](mobile-engagement-concepts.md) per la piattaforma Web.

### `Session` e `Activity`

Se l'utente resta inattivo per più di alcuni secondi tra due attività, la sequenza di attività dell'utente viene divisa in due sessioni distinte. Questi pochi secondi vengono chiamati timeout della sessione.

Se l'applicazione Web non dichiara la fine delle attività utente chiamando la funzione `engagement.agent.endActivity`, il server di Mobile Engagement terminerà automaticamente la sessione utente entro tre minuti dalla chiusura della pagina dell'applicazione. Questo comportamento viene definito timeout della sessione del server.

### `Crash`

I report automatici di eccezioni JavaScript non rilevate non vengono creati per impostazione predefinita. È tuttavia possibile segnalare manualmente gli arresti anomali usando la funzione `sendCrash`. Vedere la sezione sulla segnalazione degli arresti anomali.

## Segnalazione di attività

La segnalazione delle attività utente include il momento in cui un utente avvia una nuova attività e il momento in cui l'utente termina l'attività corrente.

### L'utente avvia una nuova attività

	engagement.agent.startActivity("MyUserActivity");

È necessario chiamare `startActivity()` ogni volta che l'attività dell'utente cambia. La prima chiamata a questa funzione avvia una nuova sessione utente.

### L'utente termina l'attività corrente

	engagement.agent.endActivity();

È necessario chiamare `endActivity()` almeno una volta quando l'utente termina l'ultima attività. In questo modo si indica a Mobile Engagement Web SDK che l'utente è attualmente inattivo e che la sessione utente deve essere chiusa allo scadere del timeout. Se si chiama `startActivity()` prima dello scadere del timeout, la sessione viene semplicemente ripresa.

Data l'assenza di una chiamata affidabile per il momento in cui la finestra dello strumento di navigazione viene chiusa, è spesso difficile o impossibile rilevare la fine delle attività utente all'interno di ambienti Web. Per questo motivo il server di Mobile Engagement termina automaticamente la sessione utente entro tre minuti dalla chiusura della pagina dell'applicazione.

## Segnalazione di eventi

La segnalazione di eventi include eventi di sessione ed eventi autonomi.

### Eventi di sessione

Gli eventi di sessione vengono in genere usati per segnalare le azioni eseguite da un utente durante la sua sessione.

**Esempio senza dati aggiuntivi:**

	loginButton.onclick = function() {
	  engagement.agent.sendSessionEvent('login');
	  // [...]
	}

**Esempio con dati aggiuntivi:**

	loginButton.onclick = function() {
	  engagement.agent.sendSessionEvent('login', {user: 'alice'});
	  // [...]
	}

### Eventi autonomi

A differenza degli eventi di sessione, gli eventi autonomi possono verificarsi all'esterno del contesto di una sessione.

A tale scopo, usare ``engagement.agent.sendEvent`` invece di ``engagement.agent.sendSessionEvent``.

## Segnalazione di errori

La segnalazione di errori include errori di sessione ed errori autonomi.

### Errori di sessione

Gli errori di sessione vengono in genere usati per segnalare gli errori che hanno impatto sull'utente durante la sua sessione.

**Esempio senza dati aggiuntivi:**

	var validateForm = function() {
	  // [...]
	  if (password.length < 6) {
	    engagement.agent.sendSessionError('password_too_short');
	  }
	  // [...]
	}

**Esempio con dati aggiuntivi:**

	var validateForm = function() {
	  // [...]
	  if (password.length < 6) {
	    engagement.agent.sendSessionError('password_too_short', {length: 4});
	  }
	  // [...]
	}

### Errori autonomi

A differenza degli errori di sessione, gli errori autonomi possono verificarsi all'esterno del contesto di una sessione.

A tale scopo, usare `engagement.agent.sendError` invece di `engagement.agent.sendSessionError`.

## Segnalazione di processi

La segnalazione di processi include la segnalazione di errori ed eventi che si verificano durante un processo e la segnalazione di arresti anomali.

**Esempio:**

Per monitorare una richiesta AJAX, usare il codice seguente:

	// [...]
	xhr.onreadystatechange = function() {
	  if (xhr.readyState == 4) {
	  // [...]
	    engagement.agent.endJob('publish');
	  }
	}
	engagement.agent.startJob('publish');
	xhr.send();
	// [...]

### Segnalazione di errori durante un processo

Gli errori possono essere correlati a un processo in esecuzione invece che alla sessione utente corrente.

**Esempio:**

Per segnalare un errore in caso di esito negativo di una richiesta AJAX:

	// [...]
	xhr.onreadystatechange = function() {
	  if (xhr.readyState == 4) {
	    // [...]
	    if (xhr.status == 0 || xhr.status >= 400) {
	      engagement.agent.sendJobError('publish_xhr', 'publish', {status: xhr.status, statusText: xhr.statusText});
	    }
	    engagement.agent.endJob('publish');
	  }
	}
	engagement.agent.startJob('publish');
	xhr.send();
	// [...]

### Segnalazione di eventi durante un processo

Gli eventi possono essere correlati a un processo in esecuzione invece che alla sessione utente corrente grazie alla funzione `engagement.agent.sendJobEvent`.

Questa funzione opera esattamente come la funzione `engagement.agent.sendJobError`.

### Segnalazione di arresti anomali

Usare la funzione `sendCrash` per segnalare manualmente gli arresti anomali.

L'argomento `crashid` è una stringa che identifica il tipo di arresto anomalo. L'argomento `crash` è in genere l'analisi dello stack dell'arresto anomalo sotto forma di stringa.

	engagement.agent.sendCrash(crashid, crash);

## Parametri aggiuntivi

È possibile collegare dati arbitrari a un evento, errore, attività o processo.

Questi dati possono essere qualsiasi oggetto JSON, ma non una matrice o un tipo primitivo.

**Esempio:**

	var extras = {"video_id": 123, "ref_click": "http://foobar.com/blog"};
	engagement.agent.sendEvent("video_clicked", extras);

### Limiti

I limiti che si applicano ai parametri aggiuntivi riguardano le aree delle espressioni regolari per chiavi, tipi di valore e dimensioni.

#### Chiavi

Ogni chiave dell'oggetto deve corrispondere all'espressione regolare seguente:

	^[a-zA-Z][a-zA-Z_0-9]*

Questo significa che le chiavi devono iniziare con almeno una lettera seguita da lettere, cifre o caratteri di sottolineatura (\_).

#### Valori

I valori sono limitati a tipi string, number e boolean.

#### Dimensione

I dati aggiuntivi sono limitati a 1.024 caratteri per chiamata, dopo che la chiamata è stata codificata in JSON da Mobile Engagement Web SDK.

## Segnalazione di informazioni sull'applicazione

È possibile segnalare manualmente le informazioni di traccia o qualsiasi altra informazione specifica dell'applicazione con la funzione `sendAppInfo()`.

Si noti che queste informazioni possono essere inviate in modo incrementale. Verrà mantenuto solo l'ultimo valore per una chiave specifica per un dispositivo specifico.

Come per i dati aggiuntivi degli eventi, è possibile usare qualsiasi oggetto JSON per astrarre le informazioni sull'applicazione. Si noti che le matrici o gli oggetti secondari vengono trattati come stringhe flat, usando la serializzazione JSON.

**Esempio:**

Di seguito è riportato un esempio di codice per inviare il sesso e la data di nascita dell'utente:

	var appInfos = {"birthdate":"1983-12-07","gender":"female"};
	engagement.agent.sendAppInfo(appInfos);

### Limiti

I limiti che si applicano alle informazioni sull'applicazione riguardano le aree delle espressioni regolari per chiavi e dimensioni.

#### Chiavi

Ogni chiave dell'oggetto deve corrispondere all'espressione regolare seguente:

	^[a-zA-Z][a-zA-Z_0-9]*

Questo significa che le chiavi devono iniziare con almeno una lettera seguita da lettere, cifre o caratteri di sottolineatura (\_).

#### Dimensione

Le informazioni sull'applicazione sono limitate a 1.024 caratteri per chiamata, dopo che la chiamata è stata codificata in JSON da Mobile Engagement Web SDK.

Nell'esempio precedente, il codice JSON inviato al server è lungo 44 caratteri:

	{"birthdate":"1983-12-07","gender":"female"}

<!---HONumber=AcomDC_0713_2016-->
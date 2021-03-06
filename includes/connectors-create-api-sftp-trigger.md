Aggiungere un trigger.

1. Immettere *sftp* nella casella di ricerca della finestra di progettazione di App per la logica, quindi selezionare il trigger **SFTP - Quando viene aggiunto o modificato un file**  
![Immagine di trigger SFTP 1](./media/connectors-create-api-sftp/trigger-1.png)  
- Il controllo **Quando viene aggiunto o modificato un file** apre  
![Immagine di trigger SFTP 2](./media/connectors-create-api-sftp/trigger-2.png)  
- Selezionare **...** sul lato destro del controllo per aprire il controllo di selezione della cartella  
![Immagine di trigger SFTP 3](./media/connectors-create-api-sftp/action-1.png)  
- Selezionare **SFTP** per scegliere la cartella principale da monitorare per la presenza di file nuovi o modificati. Tenere presente che ora la cartella principale viene visualizzata nel controllo **Cartella**.  
![Immagine di trigger SFTP 4](./media/connectors-create-api-sftp/action-2.png)  

A questo punto, l'app per la logica è stata configurata con un trigger che avvierà l'esecuzione di altri trigger e altre azioni nel flusso di lavoro quando un file viene modificato o creato nell'apposita cartella SFTP.

>[AZURE.NOTE]Affinché sia funzionale, l'app per la logica deve contenere almeno un trigger e un'azione. Seguire i passaggi nella sezione successiva per aggiungere un'azione.

<!---HONumber=AcomDC_0727_2016-->

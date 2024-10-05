---
lab:
  title: Hinzufügen zu einem Index mithilfe der Push-API
---

# Hinzufügen zu einem Index mithilfe der Push-API

Hier erfahren Sie, wie Sie mithilfe von C#-Code einen Azure KI-Suche-Index erstellen und Dokumente in diesen Index hochladen.

In dieser Übung klonen Sie eine vorhandene C#-Lösung und führen sie aus, um die optimale Batchgröße zum Hochladen von Dokumenten zu ermitteln. Anschließend verwenden Sie diese Batchgröße und laden Dokumente mithilfe eines Threadingansatzes effektiv hoch.

> **Hinweis** Um diese Übung abschließen zu können, benötigen Sie ein Microsoft Azure-Abonnement. Wenn Sie noch kein Abonnement besitzen, können Sie sich unter [https://azure.com/free](https://azure.com/free?azure-portal=true) für eine kostenlose Testversion registrieren.

## Einrichten Ihrer Azure-Ressourcen

Um Zeit zu sparen, wählen Sie diese ARM-Vorlage aus, um Ressourcen zu erstellen, die Sie später in der Übung benötigen:

1. [Bereitstellen von Ressourcen in Azure](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoftLearning%2Fmslearn-knowledge-mining%2Fmain%2FLabfiles%2F07-exercise-add-to-index-use-push-api%20lab-files%2Fazuredeploy.json) – Wählen Sie diesen Link aus, um Ihre Azure AI-Ressourcen zu erstellen.
    ![Screenshot: Optionen, die beim Bereitstellen von Ressourcen in Azure angezeigt werden](../media/07-media/deploy-azure-resources.png)
1. Klicken Sie unter **Ressourcengruppe** auf **Neu erstellen**, und nennen Sie die Ressourcengruppe **cog-search-language-exe**.
1. Wählen Sie unter **Region** eine [unterstützte Region](/azure/ai-services/language-service/custom-text-classification/service-limits#regional-availability) aus, die sich in Ihrer Nähe befindet.
1. Das **Ressourcenpräfix** muss global eindeutig sein. Geben Sie ein Präfix aus zufälligen Zahlen und Kleinbuchstaben ein (z. B. **acs118245**).
1. Wählen Sie unter **Standort** dieselbe Region aus wie zuvor.
1. Klicken Sie auf **Überprüfen + erstellen**.
1. Klicken Sie auf **Erstellen**.
1. Wenn die Bereitstellung abgeschlossen ist, klicken Sie auf **Zu Ressourcengruppe wechseln**, um alle erstellten Ressourcen anzuzeigen.

    ![Screenshot aller bereitgestellten Azure-Ressourcen.](../media/07-media/azure-resources-created.png)

### Kopieren von REST-API-Informationen für einen Azure KI-Suche-Dienst

1. Wählen Sie in der Liste der Ressourcen den von Ihnen erstellten Suchdienst aus. Im obigen Beispiel ist dies **acs118245-search-service**.
1. Kopieren Sie den Suchdienstnamen in eine Textdatei.

    ![Screenshot: Schlüsselabschnitt eines Suchdienstes.](../media/07-media/search-api-keys-exercise-version.png)
1. Wählen Sie auf der linken Seite **Schlüssel** aus, und kopieren Sie dann den **primären Administratorschlüssel** in dieselbe Textdatei.

### Herunterladen eines Beispielcodes

Öffnen Sie Azure Cloud Shell, indem Sie das Cloud Shell-Symbol im oberen Fensterbereich des Azure-Portals auswählen.
> **Hinweis** Wenn Sie aufgefordert werden, ein Azure Storage-Konto zu erstellen, wählen Sie **Speicher erstellen** aus.

1. Nachdem der Ladevorgang abgeschlossen ist, klonen Sie das folgende Beispielcode-Repository, indem Sie Folgendes in Ihrer Cloud Shell-Instanz ausführen:

    ```powershell
    git clone https://github.com/Azure-Samples/azure-search-dotnet-scale.git samples
    ```

1. Wechseln Sie in das neu erstellte Verzeichnis, indem Sie Folgendes ausführen:

    ```powershell
    cd samples
    ```

1. Führen Sie anschließend Folgendes aus:

    ```powershell
    code ./optimize-data-indexing/v11
    ```

1. Dadurch wird der Code-Editor in Cloud Shell im Ordner `/optimize-data-indexing/v11` geöffnet.

    ![Screenshot: Anzeige der Setupbenachrichtigungen in VS Code.](../media/07-media/setup-visual-studio-code-solution.png)
1. Erweitern Sie im Navigationsmenü auf der linken Seite den Ordner **OptimizeDataIndexing**, und wählen Sie dann die Datei **appsettings.json** aus.

    ![Screenshot: Inhalt der appsettings.json-Datei.](../media/07-media/update-app-settings.png)
1. Fügen Sie ihren Suchdienstnamen und den primären Administratorschlüssel ein.

    ```json
    {
      "SearchServiceUri": "https://acs118245-search-service.search.windows.net",
      "SearchServiceAdminApiKey": "YOUR_SEARCH_SERVICE_KEY",
      "SearchIndexName": "optimize-indexing"
    }
    ```

    Die Datei mit den Einstellungen sollte in etwa wie oben gezeigt aussehen.
1. Speichern Sie Ihre Änderungen, indem Sie **STRG+S** drücken.
1. Wählen Sie die Datei **OptimizeDataIndexing.csproj** aus. <!-- Added this and the next two steps in case we can't update the file in the repo that holds these (seems to be separate from the other labs)-->
1. Ändern Sie in der fünften Zeile `<TargetFramework>netcoreapp3.1</TargetFramework>` in `<TargetFramework>net7.0</TargetFramework>`. <!--- can be removed if no longer needed based on the above-->
1. Speichern Sie Ihre Änderungen, indem Sie **STRG+S** drücken.<!--- can be removed if no longer needed based on the above-->
1. Geben Sie im Terminal `cd ./optimize-data-indexing/v11/OptimizeDataIndexing` ein, und drücken Sie dann die **EINGABETASTE**, um in das richtige Verzeichnis zu wechseln.
1. Wählen Sie die **Program.cs**-Datei aus. Geben Sie dann im Terminal `dotnet run` ein, und drücken Sie die **EINGABETASTE**.

    ![Screenshot: Ausführung der App in VS Code mit einer Ausnahme.](../media/07-media/debug-application.png)
Die Ausgabe zeigt, dass in diesem Fall die beste Batchgröße 900 Dokumente umfasst, da sie 3,688 MB pro Sekunde erreicht.

### Bearbeiten des Codes zum Implementieren von Threading und einer Backoff- und Wiederholungsstrategie

Code wurde auskommentiert und ist bereit dafür, die App so zu ändern, dass Threads zum Hochladen von Dokumenten in den Suchindex verwendet werden können.

1. Stellen Sie sicher, dass Sie **Program.cs** ausgewählt haben.

    ![Screenshot: Anzeige der Program.cs-Datei in VS Code.](../media/07-media/edit-program-code.png)
1. Kommentieren Sie die Zeilen 38 und 39 wie folgt aus:

    ```csharp
    //Console.WriteLine("{0}", "Finding optimal batch size...\n");
    //await TestBatchSizesAsync(searchClient, numTries: 3);
    ```

1. Heben Sie die Auskommentierung der Zeilen 41 bis 49 auf.

    ```csharp
    long numDocuments = 100000;
    DataGenerator dg = new DataGenerator();
    List<Hotel> hotels = dg.GetHotels(numDocuments, "large");

    Console.WriteLine("{0}", "Uploading using exponential backoff...\n");
    await ExponentialBackoff.IndexDataAsync(searchClient, hotels, 1000, 8);

    Console.WriteLine("{0}", "Validating all data was indexed...\n");
    await ValidateIndexAsync(indexClient, indexName, numDocuments);
    ```

    Der Code, der die Batchgröße und die Anzahl der Threads steuert, lautet `await ExponentialBackoff.IndexDataAsync(searchClient, hotels, 1000, 8)`. Die Batchgröße beträgt 1.000, und es gibt acht Threads.

    ![Screenshot mit dem vollständigen bearbeiteten Code.](../media/07-media/thread-code-ready.png)
    Der Code sollte wie das obenstehende Beispiel aussehen.

1. Speichern Sie Ihre Änderungen, indem Sie **STRG**+**S** drücken.
1. Wählen Sie Ihr Terminal aus, und drücken Sie dann eine beliebige Taste, um den laufenden Prozess zu beenden, falls noch nicht geschehen.
1. Führen Sie `dotnet run` im Terminal aus.

    ![Screenshot mit den abgeschlossenen Meldungen in der Konsole.](../media/07-media/upload-hundred-thousand-documents.png)
    Die App startet acht Threads, immer wenn ein Thread das Schreiben einer neuen Nachricht in die Konsole abgeschlossen hat:

    ```powershell
    Finished a thread, kicking off another...
    Sending a batch of 1000 docs starting with doc 57000...
    ```

    Nach dem Hochladen von 100.000 Dokumenten schreibt die App eine Zusammenfassung (dies kann eine Weile dauern):

    ```powershell
    Ended at: 9/1/2023 3:25:36 PM
    
    Upload time total: 00:01:18:0220862
    Upload time per batch: 780.2209 ms
    Upload time per document: 0.7802 ms
    
    Validating all data was indexed...
    
    Waiting for service statistics to update...
    
    Document Count is 100000
    
    Waiting for service statistics to update...
    
    Index Statistics: Document Count is 100000
    Index Statistics: Storage Size is 71453102
    
    ``````

Durchsuchen Sie den Code in der `TestBatchSizesAsync`-Prozedur, um zu erfahren, wie der Code die Leistung der Batchgröße überprüft.

Durchsuchen Sie den Code in der `IndexDataAsync`-Prozedur, um zu sehen, wie der Code das Threading verwaltet.

Durchsuchen Sie den Code im `ExponentialBackoffAsync`, um zu sehen, wie der Code eine exponentielle Backoff-Wiederholungsstrategie implementiert.

Sie können das Azure-Portal durchsuchen und dort überprüfen, ob die Dokumente dem Index hinzugefügt wurden.

![Screenshot: Suchindex mit 100000 Dokumenten.](../media/07-media/check-search-service-index.png)

### Bereinigung

Nachdem Sie die Übung abgeschlossen haben, löschen Sie alle nicht länger benötigten Ressourcen. Beginnen Sie mit dem Code, der auf Ihrem Computer geklont ist. Löschen Sie dann die Azure-Ressourcen.

1. Wählen Sie im **Azure-Portal** die Option „Ressourcengruppen“ aus.
1. Wählen Sie die Ressourcengruppe aus, die Sie für diese Übung erstellt haben.
1. Wählen Sie die Option **Ressourcengruppe löschen**. 
1. Bestätigen Sie den Löschvorgang, und wählen Sie dann **Löschen** aus.
1. Wählen Sie die Ressourcen aus, die Sie nicht mehr benötigen, und klicken Sie auf **Löschen**.

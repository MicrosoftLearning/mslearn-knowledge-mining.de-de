---
lab:
  title: Erstellen eines benutzerdefinierten Skills für Azure KI-Suche
  module: Module 12 - Creating a Knowledge Mining Solution
---

# Erstellen eines benutzerdefinierten Skills für Azure KI-Suche

Azure KI-Suche verwendet eine Anreicherungspipeline von KI-Skills, um KI-generierte Felder aus Dokumenten zu extrahieren und in einen Suchindex einzufügen. Es gibt einen umfassenden Satz integrierter Skills, die Sie verwenden können, aber wenn Sie eine bestimmte Anforderung haben, die von diesen Skills nicht erfüllt wird, können Sie einen benutzerdefinierten Skill erstellen.

In dieser Übung erstellen Sie einen benutzerdefinierten Skill, der die Häufigkeit einzelner Wörter in einem Dokument tabelliert, um eine Liste der fünf am häufigsten verwendeten Wörter zu generieren und einer Suchlösung für Margie‘s Travel – einem fiktiven Reisebüro – hinzuzufügen.

## Vorbereiten der Entwicklung einer App in Visual Studio Code

Sie entwickeln Ihre Such-App mit Visual Studio Code. Die Codedateien für Ihre App wurden in einem GitHub-Repository bereitgestellt.

> **Tipp**: Wenn Sie das Repository **mslearn-knowledge-mining** bereits geklont haben, öffnen Sie es in Visual Studio Code. Führen Sie andernfalls die folgenden Schritte aus, um es in Ihrer Entwicklungsumgebung zu klonen.

1. Starten Sie Visual Studio Code.
1. Öffnen Sie die Palette (UMSCHALT+STRG+P), und führen Sie einen **Git: Clone**-Befehl aus, um das Repository `https://github.com/MicrosoftLearning/mslearn-knowledge-mining` in einen lokalen Ordner zu klonen (der Ordner ist beliebig).
1. Nachdem das Repository geklont wurde, öffnen Sie den Ordner in Visual Studio Code.
1. Warten Sie, während zusätzliche Dateien zur Unterstützung der C#-Codeprojekte im Repository installiert werden.

    > **Hinweis**: Wenn Sie aufgefordert werden, erforderliche Ressourcen zum Erstellen und Debuggen hinzuzufügen, wählen Sie **Jetzt nicht** aus.

## Erstellen von Azure-Ressourcen

> **Hinweis**: Wenn Sie die Übung zum **[Erstellen einer Azure KI-Suche-Lösung](01-azure-search.md)** bereits abgeschlossen haben und diese Azure-Ressourcen noch in Ihrem Abonnement verfügbar sind, können Sie diesen Abschnitt überspringen und mit dem Abschnitt **Erstellen einer Suchlösung** beginnen. Führen Sie andernfalls die folgenden Schritte aus, um die erforderlichen Azure-Ressourcen bereitzustellen.

1. Öffnen Sie in einem Webbrowser das Azure-Portal unter `https://portal.azure.com`, und melden Sie sich mit dem Microsoft-Konto an, das Ihrem Azure-Abonnement zugeordnet ist.
2. Suchen Sie in der oberen Suchleiste nach *Azure KI Services*, wählen Sie **Azure KI Services** aus und erstellen Sie eine Azure KI Services Multi-Service-Kontoressource mit den folgenden Einstellungen:
    - **Abonnement:** *Geben Sie Ihr Azure-Abonnement an.*
    - **Ressourcengruppe**: *Wählen Sie eine Ressourcengruppe aus, oder erstellen Sie eine (wenn Sie ein eingeschränktes Abonnement verwenden, sind Sie möglicherweise nicht berechtigt, eine neue Ressourcengruppe zu erstellen. Verwenden Sie dann die bereitgestellte Gruppe.)*
    - **Region:** *Wählen Sie aus verfügbaren Regionen, die sich geografisch in Ihrer Nähe befinden.*
    - **Name**: *Geben Sie einen eindeutigen Namen ein.*
    - **Tarif**: Standard S0.
1. Wechseln Sie nach der Bereitstellung zur Ressource, und notieren Sie sich auf der Seite **Übersicht** die **Abonnement-ID** und den **Standort**. Sie benötigen diese Werte zusammen mit dem Namen der Ressourcengruppe in den nachfolgenden Schritten. 
1. Erweitern Sie in Visual Studio Code den Ordner **Labfiles/02-search-skill**, und wählen Sie **setup.cmd** aus. Sie verwenden dieses Batchskript, um die Befehle der Azure-Befehlszeilenschnittstellen (CLI) auszuführen, die zum Erstellen der benötigten Azure-Ressourcen erforderlich sind.
1. Klicken Sie mit der rechten Maustaste auf den Ordner **02-search-skill**, und wählen Sie die Option **Open in Integrated Terminal** (In integriertem Terminal öffnen) aus.
1. Geben Sie im Terminalfenster den folgenden Befehl ein, um eine authentifizierte Verbindung mit Ihrem Azure-Abonnement herzustellen.

    ```powershell
    az login --output none
    ```

8. Melden Sie sich bei Ihrem Azure-Abonnement an, wenn Sie dazu aufgefordert werden. Kehren Sie dann zu Visual Studio Code zurück, und warten Sie, bis der Anmeldevorgang abgeschlossen ist.
9. Führen Sie den folgenden Befehl aus, um Azure-Speicherorte auflisten.

    ```powershell
    az account list-locations -o table
    ```

10. Suchen Sie in der Ausgabe nach dem Wert **Name**, der dem Standort Ihrer Ressourcengruppe entspricht (für *USA, Osten* ist der entsprechende Name beispielsweise *eastus).*
11. Ändern Sie im Skript **setup.cmd** die Deklarationen der Variablen **subscription_id**, **resource_group** und **location** mit den entsprechenden Werten für Ihre Abonnement-ID, den Namen der Ressourcengruppe und den Standortnamen. Speichern Sie anschließend die Änderungen.
12. Geben Sie im Terminal für den Ordner **02-search-skill** den folgenden Befehl ein, um das Skript auszuführen:

    ```powershell
    ./setup
    ```

    > **Hinweis:** Wenn das Skript fehlschlägt, stellen Sie sicher, dass Sie es mit den richtigen Variablennamen gespeichert haben, und versuchen Sie es erneut.

13. Wenn das Skript abgeschlossen ist, überprüfen Sie die angezeigte Ausgabe, und notieren Sie sich die folgenden Informationen zu Ihren Azure-Ressourcen (diese Werte werden Sie später benötigen):
    - Speicherkontoname
    - Speicherverbindungszeichenfolge
    - Suchdienstendpunkt
    - Administratorschlüssel für Suchdienst
    - Abfrageschlüssel für Suchdienst

14. Aktualisieren Sie im Azure-Portal die Ressourcengruppe, und überprüfen Sie, ob sie das Azure Storage-Konto, die Azure KI Services-Ressource und die Azure KI-Suche-Ressource enthält.

## Erstellen einer Suchlösung

Nachdem Sie nun über die erforderlichen Azure-Ressourcen verfügen, können Sie eine Suchlösung erstellen, die aus den folgenden Komponenten besteht:

- Eine **Datenquelle**, die auf die Dokumente in Ihrem Azure-Speichercontainer verweist.
- Ein **Skillset**, das eine Anreicherungspipeline von Qualifikationen definiert, um KI-generierte Felder aus den Dokumenten zu extrahieren.
- Ein **Index**, der einen durchsuchbaren Satz von Dokumentdatensätzen definiert.
- Ein **Indexer**, der die Dokumente aus der Datenquelle extrahiert, das Skillset anwendet und den Index auffüllt.

In dieser Übung verwenden Sie die REST-Schnittstelle von Azure KI-Suche, um diese Komponenten durch Übermitteln von JSON-Anforderungen zu erstellen.

1. Erweitern Sie in Visual Studio Code im Ordner **02-search-skill** den Ordner **create-search**, und wählen Sie **data_source.json** aus. Diese Datei enthält eine JSON-Definition für eine Datenquelle namens **margies-custom-data**.
2. Ersetzen Sie den Platzhalter **YOUR_CONNECTION_STRING** durch die Verbindungszeichenfolge für Ihr Azure-Speicherkonto, die etwa wie folgt aussehen sollte:

    ```
    DefaultEndpointsProtocol=https;AccountName=ai102str123;AccountKey=12345abcdefg...==;EndpointSuffix=core.windows.net
    ```

    *Sie finden die Verbindungszeichenfolge auf der Seite mit den **Zugriffsschlüsseln** für Ihr Speicherkonto im Azure-Portal.*

3. Speichern und schließen Sie die aktualisierte JSON-Datei.
4. Öffnen Sie im Ordner **create-search** die Datei **skillset.json**. Diese Datei enthält eine JSON-Definition für ein Skillset namens **margies-custom-skillset**.
5. Ersetzen Sie oben in der Skillsetdefinition im **cognitiveServices**-Element den Platzhalter **YOUR_AI_SERVICES_KEY** durch einen der Schlüssel für Ihre Azure KI Services-Ressourcen.

    *Sie finden diese Schlüssel auf der Seite **Schlüssel und Endpunkt** für Ihre Azure KI Services-Ressource im Azure-Portal.*

6. Speichern und schließen Sie die aktualisierte JSON-Datei.
7. Öffnen Sie im Ordner **create-search** die Datei **index.json**. Diese Datei enthält eine JSON-Definition für einen Index namens **margies-custom-index**.
8. Überprüfen Sie den JSON-Code für den Index, und schließen Sie die Datei, ohne Änderungen vorzunehmen.
9. Öffnen Sie im Ordner **create-search** die Datei **indexer.json**. Diese Datei enthält eine JSON-Definition für einen Indexer namens **margies-custom-indexer**.
10. Überprüfen Sie den JSON-Code für den Indexer, und schließen Sie die Datei, ohne Änderungen vorzunehmen.
11. Öffnen Sie im Ordner **create-search** die Datei **create-search.cmd**. Dieses Batchskript verwendet das Hilfsprogramm cURL, um die JSON-Definitionen an die REST-Schnittstelle für Ihre Azure KI-Suche-Ressource zu übermitteln.
12. Ersetzen Sie die Platzhalter für die Variablen **YOUR_SEARCH_URL** und **YOUR_ADMIN_KEY** durch die **URL** und einen der **Administratorschlüssel** für Ihre Azure KI-Suche-Ressource.

    *Sie finden diese Werte auf der Seite mit der **Übersicht** und der Seite mit den **Schlüsseln** für Ihre Azure KI-Suche-Ressource im Azure-Portal.*

13. Speichern Sie die aktualisierte Batchdatei.
14. Klicken Sie mit der rechten Maustaste auf den Ordner **create-search**, und wählen Sie die Option **Open in Integrated Terminal** (In integriertem Terminal öffnen) aus.
15. Geben Sie im Terminalbereich für den Ordner **create-search** den folgenden Befehl ein, um das Batchskript auszuführen.

    ```powershell
    ./create-search
    ```

16. Wählen Sie nach Abschluss des Skripts im Azure-Portal auf der Seite für Ihre Azure KI-Suche-Ressource die Seite **Indexer** aus, und warten Sie, bis der Indizierungsprozess abgeschlossen ist.

    *Sie können **Aktualisieren** auswählen, um den Fortschritt des Indizierungsvorgang zu verfolgen. Es kann eine Minute dauern, bis der Vorgang abgeschlossen ist.*

## Durchsuchen des Index

Nachdem Sie nun über einen Index verfügen, können Sie ihn durchsuchen.

1. Wählen Sie oben auf dem Blatt für Ihre Azure KI-Suche-Ressource die Option **Suchexplorer** aus.
2. Geben Sie im Suchexplorer im Feld **Abfragezeichenfolge** die folgende Zeichenfolge ein, und wählen Sie dann **Suchen** aus.

    ```
    search=London&$select=url,sentiment,keyphrases&$filter=metadata_author eq 'Reviewer' and sentiment eq 'positive'
    ```

    Diese Abfrage ruft die Elemente **url**, **sentiment** und **keyphrases** für alle Dokumente ab, in denen *London* erwähnt wird, die von *Reviewer* erstellt wurden und deren **sentiment**-Bezeichnung positiv ist (also positive Rezensionen, die London erwähnen).

## Erstellen einer Azure-Funktion für einen benutzerdefinierten Skill

Die Suchlösung enthält eine Reihe integrierter KI-Skills, die den Index mit Informationen aus den Dokumenten anreichern, z. B. die Stimmungsbewertungen und Listen von Schlüsselbegriffen aus der vorherigen Aufgabe.

Sie können den Index weiter verbessern, indem Sie benutzerdefinierte Skills erstellen. Beispielsweise kann es hilfreich sein, die Wörter, die in jedem Dokument am häufigsten verwendet werden, zu identifizieren, aber es gibt keinen integrierten Skill, der diese Funktionalität bietet.

Um die Wortzählungsfunktion als benutzerdefinierten Skill zu implementieren, erstellen Sie eine Azure-Funktion in Ihrer bevorzugten Sprache.

> **Hinweis:** In dieser Übung erstellen Sie eine einfache Node.JS-Funktion mithilfe der Codebearbeitungsfunktionen im Azure-Portal. In einer Produktionslösung würden Sie in der Regel eine Entwicklungsumgebung wie Visual Studio Code verwenden, um eine Funktions-App in Ihrer bevorzugten Sprache (z. B. C#, Python, Node.JS oder Java) zu erstellen und diese im Rahmen eines DevOps-Prozesses in Azure zu veröffentlichen.

1. Erstellen Sie im Azure-Portal auf der Seite **Start** eine neue **Funktions-App**-Ressource mit den folgenden Einstellungen:
    - **Abonnement:** *Ihr Abonnement*
    - **Ressourcengruppe**: *Die gleiche Ressourcengruppe wie Ihre Azure KI-Suche-Ressource*
    - **Name der Funktions-App**: *Ein eindeutiger Name*
    - **Veröffentlichen**: Code
    - **Laufzeitstapel**: Node.js
    - **Version:** 18 LTS
    - **Region:** *Die gleiche Region wie Ihre Azure KI-Suche-Ressource*

2. Warten Sie, bis die Bereitstellung abgeschlossen ist, und wechseln Sie dann zur bereitgestellten Funktions-App-Ressource.
3. Wählen Sie auf der Seite **Übersicht** die Option **Im Azure-Portal erstellen** aus, um eine neue Funktion mit den folgenden Einstellungen zu erstellen:
    - **Einrichten einer Entwicklungsumgebung**:
        - **Entwicklungsumgebung**: Im Portal entwickeln
    - **Vorlage auswählen**:
        - **Vorlage**: HTTP-Trigger
    - **Vorlagendetails**:
        - **Neue Funktion**: wordcount
        - **Autorisierungsstufe:** Funktion
4. Warten Sie, bis die *wordcount*-Funktion erstellt wird. Wählen Sie dann auf der Seite die Registerkarte **Programmieren und testen** aus.
5. Ersetzen Sie den Standardfunktionscode durch folgenden Code:

```javascript
module.exports = async function (context, req) {
    context.log('JavaScript HTTP trigger function processed a request.');

    if (req.body && req.body.values) {

        vals = req.body.values;

        // Array of stop words to be ignored
        var stopwords = ['', 'i', 'me', 'my', 'myself', 'we', 'our', 'ours', 'ourselves', 'you', 
        "youre", "youve", "youll", "youd", 'your', 'yours', 'yourself', 
        'yourselves', 'he', 'him', 'his', 'himself', 'she', "shes", 'her', 
        'hers', 'herself', 'it', "its", 'itself', 'they', 'them', 
        'their', 'theirs', 'themselves', 'what', 'which', 'who', 'whom', 
        'this', 'that', "thatll", 'these', 'those', 'am', 'is', 'are', 'was',
        'were', 'be', 'been', 'being', 'have', 'has', 'had', 'having', 'do', 
        'does', 'did', 'doing', 'a', 'an', 'the', 'and', 'but', 'if', 'or', 
        'because', 'as', 'until', 'while', 'of', 'at', 'by', 'for', 'with', 
        'about', 'against', 'between', 'into', 'through', 'during', 'before', 
        'after', 'above', 'below', 'to', 'from', 'up', 'down', 'in', 'out', 
        'on', 'off', 'over', 'under', 'again', 'further', 'then', 'once', 'here', 
        'there', 'when', 'where', 'why', 'how', 'all', 'any', 'both', 'each', 
        'few', 'more', 'most', 'other', 'some', 'such', 'no', 'nor', 'not', 
        'only', 'own', 'same', 'so', 'than', 'too', 'very', 'can', 'will',
        'just', "dont", 'should', "shouldve", 'now', "arent", "couldnt", 
        "didnt", "doesnt", "hadnt", "hasnt", "havent", "isnt", "mightnt", "mustnt",
        "neednt", "shant", "shouldnt", "wasnt", "werent", "wont", "wouldnt"];

        res = {"values":[]};

        for (rec in vals)
        {
            // Get the record ID and text for this input
            resVal = {recordId:vals[rec].recordId, data:{}};
            txt = vals[rec].data.text;

            // remove punctuation and numerals
            txt = txt.replace(/[^ A-Za-z_]/g,"").toLowerCase();

            // Get an array of words
            words = txt.split(" ")

            // count instances of non-stopwords
            wordCounts = {}
            for(var i = 0; i < words.length; ++i) {
                word = words[i];
                if (stopwords.includes(word) == false )
                {
                    if (wordCounts[word])
                    {
                        wordCounts[word] ++;
                    }
                    else
                    {
                        wordCounts[word] = 1;
                    }
                }
            }

            // Convert wordcounts to an array
            var topWords = [];
            for (var word in wordCounts) {
                topWords.push([word, wordCounts[word]]);
            }

            // Sort in descending order of count
            topWords.sort(function(a, b) {
                return b[1] - a[1];
            });

            // Get the first ten words from the first array dimension
            resVal.data.text = topWords.slice(0,9)
              .map(function(value,index) { return value[0]; });

            res.values[rec] = resVal;
        };

        context.res = {
            body: JSON.stringify(res),
            headers: {
            'Content-Type': 'application/json'
        }

        };
    }
    else {
        context.res = {
            status: 400,
            body: {"errors":[{"message": "Invalid input"}]},
            headers: {
            'Content-Type': 'application/json'
        }

        };
    }
};
```

6. Speichern Sie die Funktion, und öffnen Sie dann den Bereich **Testen/Ausführen**.
7. Ersetzen Sie im Bereich **Testen/Ausführen** den vorhandenen **Text** durch den folgenden JSON-Code. Dieser entspricht dem Schema, das ein Azure KI-Suche-Skill erwartet, mit dem Datensätze mit Daten für mindestens ein Dokument zur Verarbeitung übermittelt werden:

    ```json
    {
        "values": [
            {
                "recordId": "a1",
                "data":
                {
                "text":  "Tiger, tiger burning bright in the darkness of the night.",
                "language": "en"
                }
            },
            {
                "recordId": "a2",
                "data":
                {
                "text":  "The rain in spain stays mainly in the plains! That's where you'll find the rain!",
                "language": "en"
                }
            }
        ]
    }
    ```

8. Klicken Sie auf **Ausführen**, und sehen Sie sich den Inhalt der von der Funktion zurückgegebenen HTTP-Antwort an. Er entspricht dem Schema, das Azure KI-Suche erwartet, wenn ein Skill angewandt wird, mit dem eine Antwort für jedes Dokument zurückgegeben wird. In diesem Fall besteht die Antwort aus maximal 10 Begriffen in jedem Dokument in absteigender Reihenfolge ihrer Häufigkeit:

    ```json
    {
        "values": [
        {
            "recordId": "a1",
            "data": {
                "text": [
                "tiger",
                "burning",
                "bright",
                "darkness",
                "night"
                ]
            }
        },
        {
            "recordId": "a2",
            "data": {
                "text": [
                    "rain",
                    "spain",
                    "stays",
                    "mainly",
                    "plains",
                    "thats",
                    "youll",
                    "find"
                ]
            }
        }
        ]
    }
    ```

9. Schließen Sie den Bereich **Testen/Ausführen**, und klicken Sie auf dem Blatt der **wordcount**-Funktion auf **Funktions-URL abrufen**. Kopieren Sie dann die URL für den Standardschlüssel in die Zwischenablage. Sie wird in der nächsten Prozedur benötigt.

## Hinzufügen des benutzerdefinierten Skills zur Suchlösung

Nun müssen Sie Ihre Funktion als benutzerdefinierten Skill in das Skillset der Suchlösung einfügen und die erzeugten Ergebnisse einem Feld im Index zuordnen. 

1. Öffnen Sie in Visual Studio Code die Datei **update-skillset.json** im Ordner **02-search-skill/update-search**. Sie enthält die JSON-Definition eines Skillsets.
2. Überprüfen Sie die Skillsetdefinition. Sie enthält dieselben Skills wie zuvor und dazu noch einen neuen **WebApiSkill**-Skill namens **get-top-words**.
3. Bearbeiten Sie die Definition des Skills **get-top-words**, um den **uri**-Wert auf die URL für Ihre Azure-Funktion festzulegen (die Sie im vorherigen Verfahren in die Zwischenablage kopiert haben), und ersetzen Sie damit **YOUR-FUNCTION-APP-URL**.
4. Ersetzen Sie oben in der Skillsetdefinition im **cognitiveServices**-Element den Platzhalter **YOUR_AI_SERVICES_KEY** durch einen der Schlüssel für Ihre Azure KI Services-Ressourcen.

    *Sie finden diese Schlüssel auf der Seite **Schlüssel und Endpunkt** für Ihre Azure KI Services-Ressource im Azure-Portal.*

5. Speichern und schließen Sie die aktualisierte JSON-Datei.
6. Öffnen Sie die Datei **update-index.json** im Ordner **update-search**. Diese Datei enthält die JSON-Definition für den Index **margies-custom-index** mit einem zusätzlichen Feld namens **top_words** am unteren Rand der Indexdefinition.
7. Überprüfen Sie den JSON-Code für den Index, und schließen Sie die Datei, ohne Änderungen vorzunehmen.
8. Öffnen Sie die Datei **update-indexer.json** im Ordner **update-search**. Diese Datei enthält eine JSON-Definition für **margies-custom-indexer** mit einer zusätzlichen Zuordnung für das Feld **top_words**.
9. Überprüfen Sie den JSON-Code für den Indexer, und schließen Sie die Datei, ohne Änderungen vorzunehmen.
10. Öffnen Sie die Datei **update-search.cmd** im Ordner **update-search**. Dieses Batchskript verwendet das Hilfsprogramm cURL, um die aktualisierten JSON-Definitionen an die REST-Schnittstelle für Ihre Azure KI-Suche-Ressource zu übermitteln.
11. Ersetzen Sie die Platzhalter für die Variablen **YOUR_SEARCH_URL** und **YOUR_ADMIN_KEY** durch die **URL** und einen der **Administratorschlüssel** für Ihre Azure KI-Suche-Ressource.

    *Sie finden diese Werte auf der Seite mit der **Übersicht** und der Seite mit den **Schlüsseln** für Ihre Azure KI-Suche-Ressource im Azure-Portal.*

12. Speichern Sie die aktualisierte Batchdatei.
13. Klicken Sie mit der rechten Maustaste auf den Ordner **update-search**, und wählen Sie die Option **Open in Integrated Terminal** (In integriertem Terminal öffnen) aus.
14. Geben Sie im Terminalbereich für den Ordner **update-search** den folgenden Befehl ein, um das Batchskript auszuführen.

    ```powershell
    ./update-search
    ```

15. Wählen Sie nach Abschluss des Skripts im Azure-Portal auf der Seite für Ihre Azure KI-Suche-Ressource die Seite **Indexer** aus, und warten Sie, bis der Indizierungsprozess abgeschlossen ist.

    *Sie können **Aktualisieren** auswählen, um den Fortschritt des Indizierungsvorgang zu verfolgen. Es kann eine Minute dauern, bis der Vorgang abgeschlossen ist.*

## Durchsuchen des Index

Nachdem Sie nun über einen Index verfügen, können Sie ihn durchsuchen.

1. Wählen Sie oben auf dem Blatt für Ihre Azure KI-Suche-Ressource die Option **Suchexplorer** aus.
2. Ändern Sie im Suchexplorer die Ansicht in die **JSON-Ansicht**, und übermitteln Sie dann die folgende Suchabfrage:

    ```json
    {
      "search": "Las Vegas",
      "select": "url,top_words"
    }
    ```

    Diese Abfrage ruft die Felder **url** und **top_words** für alle Dokumente ab, in denen *Las Vegas* erwähnt wird.

## Weitere Informationen

Weitere Informationen zum Erstellen benutzerdefinierter Skills für Azure KI-Suche finden Sie in der [Azure KI-Suche-Dokumentation](https://docs.microsoft.com/azure/search/cognitive-search-custom-skill-interface).

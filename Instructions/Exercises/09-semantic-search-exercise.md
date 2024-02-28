---
lab:
  title: Einrichten von Semantischer Sortierer
---

# Einrichten von Semantischer Sortierer

> **Hinweis**: Um dieses Lab abzuschließen, benötigen Sie ein [Azure-Abonnement](https://azure.microsoft.com/free?azure-portal=true), in dem Sie über Administratorzugriff verfügen. Für diese Übung wird der Dienst **Azure KI-Suche** mit einem abrechnungsfähigen Tarif benötigt.

In dieser Übung werden Sie Semantischer Sortierer einem Index hinzufügen und für eine Abfrage verwenden.

## Aktivieren von Semantischer Sortierer

1. Öffnen Sie das Azure-Portal, und melden Sie sich an.
1. Wählen Sie **Alle Ressourcen** und dann Ihren Suchdienst aus.
1. Wählen Sie im Navigationsbereich **Semantischer Sortierer (Vorschau)** aus.
1. Wählen Sie unter **Verfügbarkeit** in der Option **Free** die Option **Plan auswählen**aus.

![Screenshot des Dialogfelds „Semantischer Sortierer“](../media/semantic-search/semanticsearch.png)

## Importieren eines Beispielindexes

1. Kehren Sie zur Seite **Übersicht** Ihres Suchdiensts zurück.
1. Klicken Sie auf **Daten importieren**.

    ![Screenshot der Schaltfläche „Daten importieren“.](../media/semantic-search/importdata.png)

1. Wählen Sie unter **Datenquelle** die Option **Beispiele** aus.
1. Wählen Sie **hotels-sample** aus und dann **Weiter: Kognitive Skills hinzufügen (optional)** aus.
1. Wählen Sie **Springen zu: Zielindex anpassen** aus.
1. Klicken Sie auf **Next: Erstellen eines Indexers**.
1. Klicken Sie auf **Submit** (Senden).

## Konfigurieren der semantischen Rangfolge

Nachdem Sie einen Suchindex und Semantischer Sortierer aktiviert haben, können Sie die semantische Sortierung konfigurieren. Sie benötigen einen Suchclient, der Vorschau-APIs für die Abfrageanforderung unterstützt. Sie können den Such-Explorer im Azure-Portal, in der Postman-App, im Azure-SDK für .NET oder im Azure-SDK für Python verwenden. In dieser Übung werden Sie den Such-Explorer im Azure-Portal verwenden.

Führen Sie die folgenden Schritte aus, um die semantische Rangfolge zu konfigurieren:

1. Wählen Sie auf der Navigationsleiste in **Suchverwaltung** die Option **Indizes** aus.

    ![Screenshot der Schaltfläche „Indizes“.](../media/semantic-search/indexes.png)

1. Wählen Sie Ihren Index aus.
1. Wählen Sie **semantischen Konfigurationen** und dann **Semantische Konfiguration hinzufügen** aus.
1. Geben Sie in **Name** den Wert **hotels-conf** ein.
1. Wählen Sie im **Feld „Titel“** die Option **HotelName** aus.
1. Wählen Sie unter **Inhaltsfelder** im **Feld „Name“** die Option **Beschreibung** aus.
1. Wiederholen Sie den vorherigen Schritt für die folgenden Felder:
    - **Kategorie**
    - **Adresse/Stadt**
1. Wählen Sie unter **Schlüsselwortfelder** im **Feld „Name“** die Option **Tags** aus.
1. Wählen Sie **Speichern**.
1. Wählen Sie auf der Indexseite **Speichern** aus.
1. Wählen Sie **Suchexplorer** aus.
1. Wählen Sie **Ansicht** und dann **JSON-Ansicht** aus.
1. Geben Sie im JSON-Abfrageeditor den folgenden Text ein:

    ```json
        {
         "queryType": "semantic",
         "queryLanguage" : "en-us",
         "search": "all hotels near the water" , 
         "semanticConfiguration": "hotels-conf" , 
         "searchFields": "",
         "speller": "lexicon" , 
         "answers": "extractive|count-3",
         "count": true
        }
    ```

1. Wählen Sie **Suchen**.
1. Überprüfen Sie die Ergebnisse der Abfrage.

## Bereinigung

Wenn Sie den Azure KI-Suche-Dienst nicht mehr benötigen, sollten Sie die Ressource aus Ihrem Azure-Abonnement löschen, um die Kosten zu reduzieren.

>**Hinweis** Durch das Löschen Ihres Azure KI-Suche-Diensts wird sichergestellt, dass Ihr Abonnement nicht mehr für Ressourcen belastet wird. Ihnen wird jedoch ein kleiner Betrag für die Datenspeicherung in Rechnung gestellt, solange der Speicher in Ihrem Abonnement vorhanden ist. Wenn Sie die Erkundung des Cognitive Search-Diensts abgeschlossen haben, können Sie den Cognitive Search-Dienst und die zugehörigen Ressourcen löschen. Wenn Sie jedoch andere Labs in dieser Reihe abschließen möchten, müssen Sie ihn neu erstellen.
> So löschen Sie die Ressourcen:
> 1. Öffnen Sie im [Azure-Portal](https://portal.azure.com?azure-portal=true ) auf der Seite **Ressourcengruppen** die Ressourcengruppe, die Sie beim Erstellen Ihres Cognitive Search-Diensts angegeben haben.
> 1. Klicken Sie auf **Ressourcengruppe löschen**, geben Sie den Ressourcengruppennamen ein, um zu bestätigen, dass Sie ihn löschen möchten, und klicken Sie dann auf **Löschen**.

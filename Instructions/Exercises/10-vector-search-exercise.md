---
lab:
  title: Verwenden der REST-API zum Ausführen von Vektorsuchabfragen
---

# Verwenden der REST-API zum Ausführen von Vektorsuchabfragen

In dieser Übung richten Sie Ihr Projekt ein, erstellen einen Index, laden Ihre Dokumente hoch und führen Abfragen aus.

Sie benötigen folgendes, um diese Übung erfolgreich auszuführen:

- Die [Postmann](https://www.postman.com/downloads/) Anwendung
- Ein Azure-Abonnement
- Azure KI Search-Service
- Die Postman-Beispielsammlung befindet sich in diesem Repository - *Vector-Search-Quickstart.postman_collection v1.0 json*.

> **Hinweis** Weitere Informationen zur Postman-App finden Sie bei Bedarf [hier](https://learn.microsoft.com/en-us/azure/search/search-get-started-rest).

## Einrichten des Projekts

Richten Sie zunächst Ihr Projekt ein, indem Sie die folgenden Schritte ausführen:

1. Notieren Sie sich die **URL **und den **Schlüssel** aus Ihrem Azure KI Search-Dienst.

    ![Abbildung des Speicherorts für Dienstname und Schlüssel.](../media/vector-search/search keys.png)

1. Laden Sie die [Postman-Beispielsammlung](https://github.com/MicrosoftLearning/mslearn-knowledge-mining/blob/main/Labfiles/10-vector-search/Vector%20Search.postman_collection%20v1.0.json) herunter.
1. Öffnen Sie Postman und importieren die Sammlung, indem Sie die Schaltfläche **Importieren** auswählen und den Sammlungsordner in das Feld ziehen und ablegen.

    ![Abbildung des Dialogfelds „Importieren“](../media/vector-search/import.png)

1. Wählen Sie die Schaltfläche **Kopie** aus, um eine Kopie der Sammlung zu erstellen und einen eindeutigen Namen hinzuzufügen.
1. Klicken Sie mit der rechten Maustaste auf ihren Sammlungsnamen, und wählen Sie **Bearbeiten** aus.
1. Wählen Sie die Registerkarte **Variablen** aus, und geben Sie die folgenden Werte mithilfe des Suchdiensts und den Indexnamen ihres Azure KI Search-Diensts ein:

    ![Diagramm zeigt ein Beispiel für Variableneinstellungen](../media/vector-search/variables.png)

1. Speichern Sie Ihre Änderungen, indem Sie die Schaltfläche **Speichern** auswählen.

Sie sind jetzt bereit, Ihre Anforderungen an den Azure KI Search-Dienst zu senden.

## Erstellen eines Indexes

Erstellen Sie als Nächstes Ihren Index in Postman:

1. Wählen Sie **PUT Erstellen/Aktualisieren des Indexes** aus dem seitlichen Menü aus.
1. Aktualisieren Sie die URL mit Ihrem **search-service-name**, dem **index-name** und der **api-version**, die Sie sich zuvor notiert haben.
1. Wählen Sie die Registerkarte **Textkörper** aus, um die Antwort anzuzeigen.
1. Legen Sie den **index-name** mit dem Wert Ihres Indexnamens aus Ihrer URL fest, und wählen Sie **Senden** aus.

Sie sollten einen Statuscode vom Typ **200** sehen, der auf eine erfolgreiche Anforderung hinweist.

## Hochladen von Dokumenten

In der Anforderung „Dokumente hochladen“ sind 108 Dokumente enthalten, jedes enthält eine vollständige Gruppe von Einbettungen für die Felder **titleVector** und **contentVector**.

1. Wählen Sie **POST Hochladen von Dokumenten** aus dem Seitenmenü aus.
1. Aktualisieren Sie die URL mit Ihrem **search-service-name**, dem **index-name** und der **api-version** wie zuvor.
1. Wählen Sie die Registerkarte **Textkörper** aus, um die Antwort anzuzeigen, und wählen Sie **Senden** aus.

Sie sollten ein Statuscode vom Typ **200** sehen, der anzeigt, dass Ihre Anforderung erfolgreich war.

## Ausführen von Abfragen

1. Versuchen Sie nun, die folgenden Abfragen im Seitenmenü auszuführen. Um dies zu tun, stellen Sie sicher, dass Sie die URL jedes Mal wie zuvor aktualisieren und eine Anforderung senden, indem Sie **Senden** auswählen:

    - Einzelvektorsuche
    - Einzelvektorsuche mit Filter
    - Einfache Hybridsuche
    - Einfache Hybridsuche mit Filter
    - Feldübergreifende Suche
    - Suche mit mehreren Abfragen

1. Wählen Sie die Registerkarte **Textkörper** aus, um die Antwort zu sehen und die Ergebnisse anzuzeigen.

Für eine erfolgreiche Anforderung sollten einen Statuscode vom Typ **200** sehen.

### Bereinigung

Nachdem Sie die Übung abgeschlossen haben, löschen Sie alle nicht länger benötigten Ressourcen. Beginnen Sie mit dem Code, der auf Ihrem Computer geklont ist. Löschen Sie dann die Azure-Ressourcen.

---
lab:
  title: Debuggen von Suchproblemen
---

# Debuggen von Suchproblemen

Sie haben Ihre Suchlösung erstellt, aber es gibt einige Warnungen im Indexer.

In dieser Übung erstellen Sie eine Azure KI-Suche-Lösung, importieren einige Beispieldaten, und beheben dann eine Warnung im Indexer.

> **Hinweis**: Um diese Übung abschließen zu können, benötigen Sie ein Microsoft Azure-Abonnement. Wenn Sie noch keines haben, können Sie sich unter [https://azure.com/free](https://azure.com/free?azure-portal=true) für eine kostenlose Testversion registrieren.

## Erstellen Ihrer Suchlösung

Bevor Sie mit einer Debugsitzung beginnen können, müssen Sie einen Azure Cognitive Search-Dienst erstellen.

1. [Bereitstellen von Ressourcen in Azure](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoftLearning%2Fmslearn-knowledge-mining%2Fmain%2FLabfiles%2F08-debug-search%2Fazuredeploy.json) – Wählen Sie diesen Link aus, um alle Ressourcen bereitzustellen, die Sie im Azure-Portal benötigen.

    ![Screenshot: ARM-Bereitstellungsvorlage mit eingegebenen Feldern.](../media/08-media/arm-template-deployment.png)

1. Klicken Sie unter **Ressourcengruppe** auf die Option **Neu erstellen**.
1. Geben Sie **acs-cognitive-search-exercise** ein.
1. Wählen Sie die **Region** aus, die Ihnen am nächsten liegt.
1. Geben Sie für das **Ressourcenpräfix** den Wert **acslearnex** ein, und fügen Sie eine zufällige Kombination aus Zahlen oder Zeichen hinzu, um sicherzustellen, dass der Speichername eindeutig ist.
1. Wählen Sie für „Standort“ dieselbe Region aus, die Sie oben verwendet haben.
1. Klicken Sie im unteren Bereich auf **Überprüfen und erstellen**.
1. Warten Sie, bis die Ressource bereitgestellt wurde, und wählen Sie dann **Zu Ressourcengruppe wechseln** aus.

## Importieren von Beispieldaten

Da Ihre Ressourcen erstellt wurden, können Sie jetzt Ihre Quelldaten importieren.

1. Wählen Sie in den aufgelisteten Ressourcen den Suchdienst aus.

1. Wählen Sie im Bereich **Übersicht** die Option **Daten importieren** aus.

      ![Screenshot: Menü „Daten importieren“ ausgewählt.](../media/08-media/import-data.png)

1. Wählen Sie im Bereich „Daten importieren“ die Datenquelle **Samples** aus.

      ![Screenshot: vervollständigte Felder.](../media/08-media/import-data-selection-screen-small.png)

1. Wählen Sie **hotels-sample** aus der Liste der Beispiele aus.
1. Wählen Sie **Weiter: Kognitive Skills hinzufügen (optional)** aus.
1. Erweitern Sie den Abschnitt **Anreicherungen hinzufügen**.

    ![Screenshot: Optionen zum Hinzufügen von Anreicherungen.](../media/08-media/add-enrichments.png)

1. Wählen Sie **Kognitive Fähigkeiten für Text** aus.
1. Wählen Sie **Weiter: Zielindex anpassen** aus.
1. Übernehmen Sie die Standardwerte, und wählen Sie **Weiter: Indexer erstellen** aus.
1. Klicken Sie auf **Submit** (Senden).

## Verwenden einer Debugsitzung zum Beheben von Warnungen in Ihrem Indexer

Der Indexer beginnt nun damit, 50 Dokumente zu erfassen. Wenn Sie jedoch den Status des Indexers überprüfen, sehen Sie, dass Warnungen vorhanden sind.

![Screenshot: 150 Warnungen im Indexer.](../media/08-media/indexer-warnings.png)

1. Wählen Sie **Debugsitzung** im linken Bereich aus.

1. Wählen Sie **+ Debugsitzung hinzufügen** aus.

1. Wählen Sie für die Storage-Verbindungszeichenfolge **Vorhandene Verbindung verwenden** und dann Ihr Speicherkonto aus.

    ![Screenshot: Neue Debugsitzung, Auswahl einer Verbindung.](../media/08-media/connect-storage.png)
1. Wählen Sie **+ Container** aus, um einen neuen Container hinzuzufügen. Benennen Sie ihn **acs-debug-storage**.

    ![Screenshot: Erstellen eines Speichercontainers.](../media/08-media/create-storage-container.png)

1. Legen Sie die **Anonyme Zugriffsebene** auf **Container(anonymer Lesezugriff für Container und Blobs)** fest.

    > **Hinweis:** Möglicherweise müssen Sie anonymen Blobzugriff aktivieren, um diese Option auszuwählen. Wechseln Sie dazu im Speicherkonto zu **Konfiguration**, legen Sie **Anonymen Blobzugriff zulassen** auf **Aktiviert** fest, und wählen Sie dann **Speichern** aus.

1. Klicken Sie auf **Erstellen**.
1. Wählen Sie Ihren neuen Container in der Liste aus, und wählen Sie dann **Auswählen** aus.
1. Wählen Sie **hotel-sample-indexer** als **Indexervorlage** aus.
1. Wählen Sie **Sitzung speichern** aus.

    Das Abhängigkeitsdiagramm zeigt Ihnen, dass für jedes Dokument ein Fehler zu drei Skills vorhanden ist.
    ![Ein Screenshot zeigt die drei Fehler für ein angereichertes Dokument.](../media/08-media/warning-skill-selection.png)

1. Wählen Sie **V3** aus.
1. Wählen Sie im Bereich „Skills details“ (Skilldetails) **Fehler/Warnungen (1)** aus.
1. Erweitern Sie die Spalte **Meldung**, damit die Details angezeigt werden.

    Die Details lauten:

    *Ungültiger Sprachcode „(Unbekannt)“. Unterstützte Sprachen: ar,cs,da,de,en,es,fi,fr,hu,it,ja,ko,nl,no,pl,pt-BR,pt-PT,ru,sv,tr,zh-Hans. Weitere Informationen finden Sie unter https://aka.ms/language-service/language-support.*

    Wenn Sie sich nun wieder das Abhängigkeitsdiagramm ansehen, weist der Skill „Spracherkennung“ Ausgaben für die drei Skills mit Warnungen auf. Die Skilleingabe, die den Fehler verursacht, ist `languageCode`.

1. Wählen Sie im Abhängigkeitsdiagramm **Spracherkennung** aus.

    ![Screenshot: Skilleinstellungen für die Sprachenerkennung.](../media/08-media/language-detection-error.png)
    Beachten Sie in der JSON-Datei für Skilleinstellungen, dass `HotelId` als Feld für die Ableitung der Sprache verwendet wird.

    Dieses Feld verursacht den Fehler, da der Skill die Sprache nicht aus einer ID ableiten kann.

## Beheben der Warnung im Indexer

1. Wählen Sie unter „Eingaben“ die Option **Quelle** aus, und ändern Sie das Feld in `/document/Description`.
    ![Screenshot: Bildschirm für Skill zur Sprachenerkennung, Skill festgelegt.](../media/08-media/language-detection-fix.png)
1. Wählen Sie **Speichern** aus.
1. Klicken Sie auf **Run** (Ausführen).

    ![Screenshot: Notwendigkeit einer Ausführung nach Aktualisierung eines Skills.](../media/08-media/rerun-debug-session.png)

    Der Indexer sollte keine Fehler oder Warnungen mehr haben. Das Skillset kann jetzt aktualisiert werden.

1. Wählen Sie **Änderungen committen...** aus.

    ![Screenshot: Problem behoben.](../media/08-media/error-fixed.png)
1. Klicken Sie auf **OK**.

1. Jetzt müssen Sie sicherstellen, dass Ihr Skillset an eine Azure KI Services-Ressource angefügt ist, da Sie sonst auf die Basisquote stoßen und der Indexer ein Timeout erleidet. Wählen Sie dazu im linken Bereich **Skillsets** aus, und wählen Sie dann Ihr **hotels-sample-skillset** aus.

    ![Ein Screenshot zeigt die Skillset-Liste.](../media/08-media/update-skillset.png)
1. Wählen Sie **KI-Dienst verbinden** aus, und wählen Sie dann die KI-Dienstressource in der Liste aus.

    ![Ein Screenshot zeigt die Azure KI Services-Ressource zum Anfügen an das Skillset.](../media/08-media/skillset-attach-service.png)
1. Wählen Sie **Speichern**.

1. Führen Sie nun Ihren Indexer aus, um die Dokumente mit den festen KI-Anreicherungen zu aktualisieren. Wählen Sie dazu im linken Bereich **Indexers** aus, wählen Sie **hotels-sample-indexer** und dann **Ausführen** aus.  Wenn die Ausführung abgeschlossen ist, sollten Sie sehen, dass die Warnungen jetzt null (0) sind.

    ![Screenshot: Alle Probleme behoben.](../media/08-media/warnings-fixed-indexer.png)

### Bereinigung

 Nachdem Sie nun die Übung abgeschlossen und die Erkundung der Azure KI-Suche-Dienste beendet haben, löschen Sie die Azure-Ressourcen, die Sie während der Übung erstellt haben. Am einfachsten ist es, die Ressourcengruppe **acs-cognitive-search-exercise** zu löschen.

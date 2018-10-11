# Document Manager

The DocumentManager class is the central workhorse and communication hub for StatTag.  It acts as the bridge between Microsoft Word events (triggered through Add-In events), and other events the user initiates within StatTag (e.g., creating a new tag).

## Helper Classes

DocumentManager makes use of the following classes for aspects of its work:

* TagManager
* StatsManager
* SettingsManager

## Add-In
### DocumentChange
When the document change event is fired by Microsoft Word, the Document Manager is alerted via a call to SetActiveDocument.  This, in turn, alerts any listeners of the change via the ActiveDocumentChanged event.
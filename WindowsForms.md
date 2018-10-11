# Windows - Form Communication

Within the Windows version of StatTag, as of v4.0 we started using a modeless dialog to manage all of our tags.  That requires additional considerations for how we communicate between that tag management form and all of our other objects (documents, Add-In, etc.)

*This pertains to the Windows version of StatTag only*

## Tag Manager
The TagManager dialog is the modeless dialog we use as our primary source of interaction within StatTag.

### Input
The Tag Manager dialog is created with a reference to the singleton Document Manager instance used by the add-in.  This allows the dialog to have a reference to all code files and their tags.  Using delegates, the Tag Manager dialog is alerted any time that something in the Document Manager changes.  Access to the Document Manager is done in a thread-safe manner.

### Communication
The Tag Manager is alerted to the following events from the Document Manager:

* ActiveDocumentChanged
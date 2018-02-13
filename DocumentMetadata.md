# Document Metadata
When you associate a code file with a Microsoft Word document, StatTag adds two entries to the [Document Variables](https://msdn.microsoft.com/en-us/library/microsoft.office.tools.word.document.variables.aspx) collection.

The reason for two different metadata variables is a side-effect of the implementation of StatTag.  Originally, the "StatTag Configuration" variable was created, but as a JSON array could not be extended with other values.  To address this, the "StatTag Metadata" variable was added in v3.1.

## StatTag Configuration
The "StatTag Configuration" variable is a JSON serialized array of [CodeFile](https://github.com/StatTag/StatTag/blob/master/Core/Models/CodeFile.cs) entries, stored as a string.

**Example:**

```
[
  {
    "StatisticalPackage" : "SAS",
    "FilePath":"C:\Data\Analysis.sas",
    "LastCached":null
  }
]
```

More information about each field is available in the [CodeFile](https://github.com/StatTag/StatTag/blob/master/Core/Models/CodeFile.cs) source code.

If there are no code files associated with a Word document, the variable will not be set by StatTag (or, if it is set, it will be removed).

## StatTag Metadata
The "StatTag Metadata" variable is a serialized JSON [DocumentMetadata](https://github.com/StatTag/StatTag/blob/master/Core/Models/DocumentMetadata.cs) object, also stored as a string.

**Example:**

```
[
  {
    "StatTagVersion": "StatTag v3.1.0",
    "RepresentMissingValues": "StatPackageDefault",
    "CustomMissingValue": "[X]",
    "MetadataFormatVersion": "1.0.0",
    "TagFormatVersion": "1.0.0"
  }
]
```

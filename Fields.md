# Handling of Fields in StatTag

## Overview
StatTag makes heavy use of Word Fields for its functionality.

## Processing Fields
Originally for a given document we were iterating over the document.Fields collection.  This only provided a list of the fields in the main text of the document, which ignored text boxes and shapes.

The common code pattern we use to process fields ensures we not only get all of the fields in the various places where text can exist (StoryRanges within a Word document), but also from embedded Shapes which can contain textboxes.


```
// First iterate over all of the shapes
var shapes = document.Shapes;
foreach (var shape in shapes.OfType<Microsoft.Office.Interop.Word.Shape>())
{
  if (shape != null && shape.TextFrame != null && shape.TextFrame.TextRange != null)
  {
    var fields = shape.TextFrame.TextRange.Fields;
    ProcessFieldsCollection(fields);  // <-- Your specific function goes here
    Marshal.ReleaseComObject(fields);
  }
}

// Then iterate over all of the story ranges - this will include text areas as well as text boxes.
foreach (var story in document.StoryRanges.OfType<Range>())
{
  var fields = story.Fields;
  ProcessFieldsCollection(fields);  // <-- Your specific function goes here
  Marshal.ReleaseComObject(fields);
}
```
# skyve-cookbook
Examples and code samples for using the [Skyve](http://skyve.org/) framework.

#### Ordering a collection in a data table
Collections can be ordered or unordered, where an ordered collection allows the user to define the sort of items in the list via drag and drop. Ordering on a collection can be enabled by adding the `ordered="true"` attribute to the collection declaration.

Collections can also have a default sorting by specifying the sort columns in the collection declaration in the document:
```xml
<ordering>
  <order by="binding1" sort="ascending" />
  <order by="binding2" sort="ascending" />
</ordering>
```

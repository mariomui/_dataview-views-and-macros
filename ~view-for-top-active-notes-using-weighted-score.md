---
DOC_VERSION: v1.0.1-tanuws
tag: _meta
MUID: MUID-665
---
# -

[[~view-for-unused-MUIDs]]

## About

```dataview
TASK WHERE file.name = this.file.name and !completed
```

This partial view uses my weighted score algorithm to mathematically ascertain my most worked on notes.

* Usage:
  * Use this view to make sure you don't overly work on one area of your studies

* [ ] Resolve performance in query.
  * Symptoms include laggy UI behavior #_todo/42-priority-high--/to-improve-meta-qol
  * [ ] Calculate actual lag times for the table generated by [[~view-for-top-active-notes-using-weighted-score]]

* Core Fields Legend:
  * *File* links to entire file
  * *Public Api* links to public facing content
  * *Score* uses the algorithm to express a single score.
    * Dimensionalities are inlinks and [[,aka-Outlinks#=]]

# =

* ℹ Outlinks are valued more heavily than inlinks when creating the Score.
* ⚙ DATAVIEW DQL POWERED
```dataview
TABLE file_link_path as "Public API", weighted as Score, file.frontmatter.MUID as MUID, length(file.inlinks) as "inlinks#", length(file.outlinks) as "outlinks#"
FLATTEN (
    link(
        join([file.path,"#="],""), "="
    )
) as file_link_path
FLATTEN round(
    (
      ( length(file.inlinks) * number("0.5") ) + 
      ( length(file.outlinks) * number("0.9") )
    ),
    2 
  ) as "weighted"
FLATTEN string(
   join(
    list(
     econtains(file.etags, "#meta"), econtains(file.etags,"#wip")
    ), [""]
   )
  ) as tagged_tuple
WHERE econtains(list("falsetrue", "falsefalse"), tagged_tuple) and weighted > 2
SORT weighted desc
LIMIT 30
```

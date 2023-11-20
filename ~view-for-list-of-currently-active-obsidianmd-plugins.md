---
TEMPLATE_VERSION: v1.0.5_note-refactor-template
MUID: MUID-1113
CREATION_DATE: 2023-06-14
tags: _wip
UMID:
---

```toc
```

# -

![[~view-for-local-tasks-using-a-progress-bar-MUID-698#=|nlk]]

```dataview
task where file.name = this.file.name and !completed
```

```dataview
task where file.name = this.file.name and completed
```


## About

Dataview is by far the best plugin.


# =


*`= this.file.name`*
~~~dataviewjs
const {plugins} = app.plugins

const sortPredicate = (a,b) => {
  return a.manifest.name < b.manifest.name ? - 1 : 1
}
const dataMatrix = Object.values(plugins).sort(sortPredicate)
    .map(({manifest}, idx) => {
    return ([
        formatWrap(
            manifest.name,
            10, 
            {cancel: true}
        ),
        formatWrap(
            manifest.description || "N/A", 
            60, 
            {cancel: true}
        )
    ])
})
function formatWrap(desc, len, config = {cancel: false}) {

    const result = [];
    let line = desc[0];
    for (let i = 1 ; i < desc.length; i++) {
        line += desc[i];
        if (config.cancel) {
            continue;
        }
        const shouldNewLine = (i % len) === 0;
        if (i !== 0 && shouldNewLine) {
            result.push(line);
            line = "";
        }
    }
    if (result.length === 0) return [line];
    return result.join("<br />")
}

const tab = dv.markdownTable(["name", "desc"],dataMatrix)
dv.paragraph(tab)
~~~

---

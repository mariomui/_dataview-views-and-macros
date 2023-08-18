---
VERSION: v0.0.2
tag: _wip
ID: 
SHORT_NAME: view-for-reading-time
---

# -

![[~view-for-local-tasks-using-a-progress-bar-TCODEID-2#=|nlk]]

```dataview
task where file.name = this.file.name and !completed
```

```dataview
task where file.name = this.file.name and completed
```
# About: 

* [ ] Fix separation of concerns

# =

~~~dataviewjs
renderContent(getDomText.call(this))

// UI business Logic
function getDomText() {
  const nodes = this.app
    .statusBar
    ?.containerEl
    ?.childNodes
  for (const node of nodes) {
    const isTargetNode = node.classList
      .contains(
      'plugin-obsidian-reading-time'
      )
    if (isTargetNode) {
      return node?.textContent
    }
  }
}
// UI
function renderContent(
  minutesRead,
  option = {thresh: 1}
) {
  const isBelowThresh = Number(
    minutesRead.split('').first()
  ) <= option.thresh;

  dv.paragraph(
    isBelowThresh ? '0 min' : minutesRead
  );
}
~~~

# ---Transient

- [ ] Rename the partial views folder to something more appropriate
  - #\_todo/a-musing/on-naming-transcluded-codelets
  - These aren't partials because while they can be used by templater they are more transcluded. they are more like imported dynamic code and not templated code aka partials (in the sense that handlebars uses them. There are similarities as well as startk diffferences.
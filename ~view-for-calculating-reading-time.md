---
VERSION: v0.0.2
tag: _wip
ID: 
SHORT_NAME: view-for-reading-time
---

# -

## Meta

![[~view-for-local-tasks-using-a-progress-bar-MUID-698#=|nlk]]

```dataview
task where file.name = this.file.name and !completed
```

```dataview
task where file.name = this.file.name and completed
```

## About

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

# ---Transient Sandbox

v0.0.0

- [ ] Rename the partial views folder to something more appropriate
  - #_todo/to-muse/upon-codelet-naming
  - These aren't partials because while they can be used by templater they are more transcluded. they are more like imported dynamic code and not templated code aka [[partial,etc]] (in the sense that handlebars uses them. There are similarities as well as stark diffferences.

---

# ---Transient Local Archive

## LA--code--reading time v1.0

```js
~~~dataviewjs

renderContent(getDomText.call(this))

// UI business Logic
function getDomText() {
  const nodes = this.app.statusBar
    ?.containerEl
    ?.childNodes

  for (const node of nodes) {
    const isTargetNode = node.classList
      .contains('plugin-obsidian-reading-time')

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
```

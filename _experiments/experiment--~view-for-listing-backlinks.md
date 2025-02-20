---
TEMPLATE_VERSION: v1.0.7_note-refactor-template
MUID: MUID-1904
CREATION_DATE: 2023-12-17
tags: _wip 
UMID: 
---
# -

## Meta
![[~view-for-local-tasks-using-a-progress-bar,nb.-MUID-698#=|nlk]]

```dataview
task where file.name = this.file.name and !completed
```

```dataview
task where file.name = this.file.name and completed
```

## About

### Reference

![[~view-for-referencing-current-jumpid#=|nlk]]

* †

# =

*`= this.file.name`*

Look inside the number of backlinks (that mention the current file) and contains ,AKA in its filename) 
```dataviewjs
const {metadataCache,workspace, vault} = this.app
const {parseBase} = createParserHelpers()
workspace.onLayoutReady(main.bind(this))
function main() {
  const vf = workspace.getActiveFile()
  const {data} = metadataCache.getBacklinksForFile(vf)
  if (!data) return;
  const mapByParseBase = (path) => parseBase(path)
  const _data = Object.keys(data).map(mapByParseBase);
  console.log({data})
  const filtereds = _data.filter(datum => datum.contains(',aka'));

  (async function (genRenderListItems,renderTitle) {
    renderTitle("Generate List of Backlinks (^aka)")
    const $l = await genRenderListItems(filtereds)
    console.log({$l})
  })(
    genRenderListItems.bind(this), 
    renderTitle.bind(this)
  )
}

// renderers
function renderTitle(title) {
  const $p = dv.paragraph(title.toUpperCase())
  $p.setAttribute("style", "text-decoration: underline")
}
async function genRenderListItems(listItems) {
  const links = listItems.map((l) => {
    return metadataCache.getFirstLinkpathDest(l,"")
  }).map((t) => {
    return dv.fileLink(t.path,t.base)
  })
  console.log({links})
  const $l = await dv.list(links);
  return $l
}
// util
function createParserHelpers() {
  const parseFn = vault.adapter.path.parse;
  return {
    parseBase: (path, spec = "base") => {
      return parseFn(path)[spec]
    }
  }
}
```

---

# ---Transient
---
tags: _meta 
DOC_VERSION: v1.0.5
---
# -

- [ ] Code [[~view-for-referencing-current-jumpid]] to place its versioning number directly into the ui-code. #_todo/to-code ➕ 2025-04-04
  - Current use of inline dataview is a stopgap solution. Please update the js code iteslf.
- This [[Partial-dataview,vis-Noteshippo,]] is meant to create a codified name for the links that jump from a quoted piece of work to the link providing the original source of the material. 
	- [[list-of-fictive-universe-affixes,by-romanized-to-norse-rune-tuple,nb.-writeshippo-title-level-affix]]
- [x] 💀 Use ~~ad-finem~~ instead of b.t. for simpler more defined one way classification. ✅ 2025-04-04
  - Consider Replacing "ad-finem" with `[[belonging-to,ad-finem-Note-Taking]][[,aka-b.t.]]`
    - 💀🔑 Belonging to sounds like it belongs to but it doesn't have the particular meaning. A list of conventions is less belonging to and more for the purpose of implementing or embettring. I like ad finem more.
  - ! All nomenclature referred here is obsolete

1.05 Remove Header
1.04 Add layout ready and replace dv.paragaraph with this.container api.
v1.03 remove the weird dash to facilitate text search
v1.0.2

## 20-Inlink

[[_writing _¬]]
> [!info]- Get Macros that Consume v0.0.1 `=this.MUID` 
`= join( map( filter( this.file.inlinks, (link) => icontains(meta(link).path, "macro") ) , (link) => "• " + link ), "<br>")`


# =

₀ ₁ ₂ ₃ ₄ ₅ ₆ ₇ ₈ ₉ 
**`= this.file.name + "-" + this.file.frontmatter.DOC_VERSION`**
```dataviewjs
const {workspace} = this.app
const container = this.container
workspace.onLayoutReady(main.bind(this))

function main() {
  container.createDiv({
    text: calcJumpId()
  })
}

function calcJumpId() {
    const subNumbers = "₀₁₂₃₄₅₆₇₈₉".split('')
    const { configuration: { 
      configuration : { idDefinitions } } 
    } = app
        .plugins.plugins['incremental-id'];

    const {currentIteration: jidx} = idDefinitions
        .find(findByNameAndCurrentIteration);
    function findByNameAndCurrentIteration({
      prefix, 
      currentIteration
    }) {
      return prefix === 'JUMPID' && currentIteration;
    }
    return `ⱼᵤₘₚ${String(jidx).split("")
        .map(Number).map(
          (num,idx) => subNumbers[num]).join('')}`;
}

```

# ---Transient

### ÷LA--code--v1

~~~js
```
```dataviewjs
function main() {
    const subNumbers = "₀₁₂₃₄₅₆₇₈₉".split('')
    const { configuration: { configuration : { idDefinitions } } } = app
        .plugins.plugins['incremental-id'];
        
    const {currentIteration: jidx} = idDefinitions
        .find(
            ({name, currentIteration}) => name === 'JUMPID' && currentIteration
        );
    return `ⱼᵤₘₚ₋${String(jidx).split("")
        .map(Number).map((num,idx) => subNumbers[num]).join('')}`;
}

try {
    dv.paragraph(main());
} catch (err) { console.log({err}) }
```
```
~~~

## ÷LA--code--v0

- [ ] Use vscode to diff if version below is different from v1

```js

~~~dataviewjs
function main() {
    const subNumbers = "₀₁₂₃₄₅₆₇₈₉".split('')
    const { configuration: { configuration : { idDefinitions } } } = app
        .plugins.plugins['incremental-id'];
        
    const {currentIteration: jidx} = idDefinitions
        .find(
            ({name, currentIteration}) => name === 'JUMPID' && currentIteration
        );
    return `ⱼᵤₘₚ₋${String(jidx).split("")
        .map(Number).map((num,idx) => subNumbers[num]).join('')}`;
}

try {
    dv.paragraph(main());
} catch (err) { console.log({err}) }
~~~
```

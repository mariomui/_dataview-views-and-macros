---
tag: _meta 
PARTIAL_VERSION: v1.0.4
---
# -

This partial is meant to create a codified name for the links that jump from a quoted piece of work to the link providing the original source of the material. [[list-of-conventions-for-encoding-metadata-at-the-note-title-level,ad-finem-Novel-Writing]]
- [x] Use ad-finem instead of b.t. for simpler more defined one way classification.
  - Consider Replacing "ad-finem" with [[belonging-to,ad-finem-Note-Taking]][[,aka-b.t.]]
    - 🔑 Belonging to sounds like it belongs to but it doesn't have the particular meaning. A list of conventions is less belonging to and more for the purpose of implementing or embettring. I like ad finem more.

1.04 Add layout ready and replace dv.paragaraph with this.container api.
v1.03 remove the weird dash to facilitate text search
v1.0.2

# =

## Latest Reference ID Link String v1.0.3

₀ ₁ ₂ ₃ ₄ ₅ ₆ ₇ ₈ ₉
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

### Archive

#### v1

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

## v0?

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

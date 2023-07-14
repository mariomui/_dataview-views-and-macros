---
tag: _meta 
PARTIAL_VERSION: v1.0.4
---
# -

This partial is meant to create a codified name for the links that jump from a quoted piece of work to the link providing the original source of the material. [[§meta-sub-sup-scripts-for-reference-and-universe-prefixing#=]]

1.04 Add layout ready and replace dv.paragaraph with this.container api.
v1.03 remove the weird dash to facilitate text search
v1.0.2

# =

## Latest Reference ID Link String v1.0.3

₀ ₁ ₂ ₃ ₄ ₅ ₆ ₇ ₈ ₉
```dataviewjs

this.app.workspace.onLayoutReady(main.bind(this))

function main() {
  this.container.createDiv({
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
        .find(
            ({name, currentIteration}) => name === 'JUMPID' && currentIteration
        );
    return `ⱼᵤₘₚ${String(jidx).split("")
        .map(Number).map(
          (num,idx) => subNumbers[num]).join('')}`;
}



```

# ---Transient Sandbox

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

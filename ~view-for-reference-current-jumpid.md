---
tag: _meta 
---
# -

This partial is meant to create a codified name for the links that jump from a quoted piece of work to the link providing the original source of the material. [[§meta-sub-sup-scripts-for-reference-and-universe-prefixing#=]]

v1.03 remove the weird dash to facilitate text search
v1.0.2

# =

## Latest Reference ID Link String v1.0.3
₀ ₁ ₂ ₃ ₄ ₅ ₆ ₇ ₈ ₉
```dataviewjs
function main() {
    const subNumbers = "₀₁₂₃₄₅₆₇₈₉".split('')
    const { configuration: { configuration : { idDefinitions } } } = app
        .plugins.plugins['incremental-id'];
        
    const {currentIteration: jidx} = idDefinitions
        .find(
            ({name, currentIteration}) => name === 'JUMPID' && currentIteration
        );
    return `ⱼᵤₘₚ${String(jidx).split("")
        .map(Number).map((num,idx) => subNumbers[num]).join('')}`;
}

try {
    dv.paragraph(main());
} catch (err) { console.log({err}) }
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
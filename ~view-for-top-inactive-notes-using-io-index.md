---
VERSION: v1.0.5_note-refactor-template
MUID: MUID-944
CREATION_DATE: 2023-05-30 
MODIFICATION_DATE: 2023-05-30
tag: _wip 
UMID: 

---

```toc
```

# -

```dataview
TASK 
WHERE file.name = this.file.name
AND !completed
```

## : About
- [ ] Remove templated from. Use more expressive field names.
### :: Reference
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
    return `ⱼᵤₘₚ₋${String(jidx).split("")
        .map(Number).map((num,idx) => subNumbers[num]).join('')}`;
}

try {
    dv.paragraph(main());
} catch (err) { console.log({err}) }
```

* $+$

# =

## = Dashboard top 30 inactive note orphans
*`= this.file.name`*


USAGE: Find the top 30 most **INACTIVE** notes, 
REASON: Prioritize untouched notes
![[~view-for-inactive-note-collection#=]]



# ---Transient Sandbox

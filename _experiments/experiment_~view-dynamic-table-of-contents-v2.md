---
TEMPLATE_VERSION: v1.0.7_note-refactor-template
MUID: MUID-1709
CREATION_DATE: 2023-11-15
tags: _wip 
UMID: 
---
# -

![[~view-for-local-tasks-using-a-progress-bar-MUID-698#=|nlk]]

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

- [ ] Is this code different than [[~view-dynamic-table-of-contents]]? ➕ 2023-11-15
*`= this.file.name`*

```dataviewjs
const {workspace,metadataCache} = this.app
const vf = workspace.getActiveFile()
const mdc = metadataCache.getFileCache(vf)
let stack = []
let flag = true
mdc.headings.map(mapByHeading)

function mapByHeading({heading,level}, idx) {
  const content = `* ${heading}`;
  if (level === 1 && flag) {
    stack.push(`* \\${heading}`);
  }
  if (level > 1 && flag) {
    const _content = createNestedHeadingContent(level - 1, content);
    stack.push(_content)
  }

  return heading

  function createNestedHeadingContent(sandwichCnt, content) {
    for (let i = 0; i < sandwichCnt; i++) {
      content = Array(content);
    }
    return content;
  }
}
console.log(stack)

dv.list(stack)
```

---

# ---Transient
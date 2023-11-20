---
TEMPLATE_VERSION: v1.0.0-default-meta
MUID: MUID-1196
CREATION_DATE: 2023-06-22 
tag: _meta 
UMID: 
DOC_VERSION: v0.0.1
---

---
# -
## About

Collects all the MUIDs not used. Harcoded search goes from 600 to currently used ID no.


# =
Unused MUIDs
```dataviewjs
//const vfiles = this.app.vault.getAllLoadedFiles();
const iidp = this.app.plugins.plugins["incremental-id"]

const vfiles = this.app.vault.getAllLoadedFiles().filter(removeAllFolderPredicate);
function removeAllFolderPredicate(file) {
    return !file?.children
}

const prefix = "MUID"
const maxlimit = getCurrentIteration(prefix)
const upperlimit = 200
const lowerlimit = Math.max(upperlimit - 100, 0)

main.call(this,vfiles);
function main(vfiles) {
    const dict = {};
    for (const vfile of vfiles) {
        const MUID = getUIFromVFile.call(this, vfile);
        if (MUID) {
            dict[MUID] = true;
        }
    }

    const unusedIds = getUnusedIds(dict, {
        upperlimit, 
        prefix,
        lowerlimit
    });

    const matrix = transformToColMatrix(
        unusedIds, 13
    ) 
    doRenderTable(matrix);
}

function doRenderTable(matrix) {
    const colCnt = matrix.first().length;
    dv.table(Array(colCnt).fill(null)
        .map(() => "•"), matrix)
}

function transformToColMatrix(items, col = 1) {
    const matrix = [];
    let accrue = []
    for (let [idx,item] of items.entries()) {
        if (idx !== 0 && (idx % col) === 0) {
            matrix.push(accrue.slice(idx - col))
        }
        accrue.push(item);
    }
    return matrix
}

function getCurrentIteration(prefix) {
    const defs = iidp.configuration
        .configuration.idDefinitions
    
    const def = defs.find((def) => def.prefix === prefix)
    const {currentIteration } = def
    return currentIteration
}

function getUIFromVFile(vfile) {
    const {frontmatter} = this.app
        .metadataCache.getCache(vfile.path);
    if (frontmatter?.["MUID"]) {
        return frontmatter.MUID
    }
    return null;
}

function getUnusedIds(dict, config = {
    prefix: "MUID",
    upperlimit: 200,
    lowerlimit: 50
}) {
    const {prefix, upperlimit, lowerlimit} = config;
    const muids = Array(upperlimit-lowerlimit)
        .fill(null)
        .map((a, idx) => {
        return (`${prefix}-${idx + lowerlimit}`)    
    })
    const unusedIds = [];
    for (const muid of muids) {
        if (!dict[muid]) {
            unusedIds.push(muid)
        }
    }
    return unusedIds
}

```


<%= /**
* v0.0.1 
  * Begin process to extract the upper limit and lower limit into knows
  * transfer learnings from experiment over to the About section
**/ -%>
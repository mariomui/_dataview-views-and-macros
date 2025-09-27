---
CREATION_DATE: "2023-06-22"
DOC_VERSION: v0.0.3
MUID: MUID-1196
TEMPLATE_VERSION: 0.0.0
cssclasses:
  - dvjs-no-overflow
tags:
  - _meta
---

---
# -
### About

* Collects all the MUIDs not used. Harcoded search goes from 600 to currently used ID no.
- [ ] Write code to keep track of unique css classes used. ➕ 2025-05-05 #_todo/to-code 

## 20-Inlink

> [!abstract]- %%  %% Automated List of Reference Inlinks (v0.0.5)
> * ℹ Commit/design logs are located in this [[,aka-MUID-150|experiment note]]. 
> > `= join( map( sort( map( filter(this.file.inlinks, (link) => meta(link).path != this.file.path), (x) => [ split(meta(x).path, "/")[length(split(meta(x).path, "/")) - 1], x ] ) ), (b) => "• " + choice( length(b[0]) > 28, link( b[1], truncate( regexreplace(b[0], "(-of|of|the|-the|-for|-that|https-|ee)", ""), length( regexreplace(b[0], "(-of|of|the|-the|-for|-that|https-|ee)", "") ) * 0.75 ) ), link(b[1], regexreplace(b[0], "\.md$", "")) ) ), "<br>" )`


# =

* @ Unused MUIDs
```dataviewjs
//const vfiles = this.app.vault.getAllLoadedFiles();
const iidp = this.app.plugins.plugins["incremental-id"]

const vfiles = this.app.vault.getAllLoadedFiles().filter(removeAllFolderPredicate);
function removeAllFolderPredicate(file) {
    return !file?.children
}

// # KNOBS
const PREFIX = "MUID"
const maxlimit = getCurrentIteration("MUID")
const UPPER_LIMIT = 400;
const LOWER_LIMIT = 100;
const MAX_COUNT = 20;
// Math.max(upper_limit - 100, 0)
const COLUMNS_TO_RENDER = 6
const abstract_note_affixes = ["macro-","~view-","~viewfn-","-template"]
// bootstrap
main.call(this,vfiles);

function checkIfBasenameContains( basename, targets) {
	return targets.reduce((chain, target) => {
		const logic = basename.contains(target);
		return logic || chain;		
	}, false);
}
// workshorse
function main(vfiles) {
    const dict = {};
    for (const vfile of vfiles) {
        const muid = getUIFromVFile.call(this, vfile);
        if (
	        checkIfBasenameContains(
		        vfile.basename,
		        abstract_note_affixes
				)
			) {
         const matches = vfile.basename
          .matchAll(/MUID-\d+/g)
         for (const match of matches) {
	        dict[match] = true;
         }
        }
        if (muid) {
				dict[muid] = true;
        }
    }

    const unusedIds = getUnusedIds(dict, {
        upper_limit: UPPER_LIMIT, 
        prefix: PREFIX,
        lower_limit: LOWER_LIMIT,
        max_count: MAX_COUNT
    });

    const matrix = transformToColMatrix(
        unusedIds, COLUMNS_TO_RENDER
    ) 
    doRenderTable(matrix);
    dv.list([
      `Codelet searches for at least *${MAX_COUNT} items* between MUID-${UPPER_LIMIT} and MUID-${LOWER_LIMIT}`, 
      `*${unusedIds.length}* items scoured.`,
      `Current MUID is: *MUID-${getCurrentIteration(PREFIX)}*`
    ].map(x => `• ${x} <br>`))
}

function doRenderTable(matrix) {
    const colCnt = matrix?.first()?.length;
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

// i am definitely gonna factorize my functions from now on. I like having configgers and defaults. 
function getUnusedIds(dict, config = {
    prefix: "MUID",
    upper_limit: 200,
    lower_limit: 50,
    max_count: 50,
}) {
  let count = 0;
    const {prefix, upper_limit, lower_limit, max_count} = config;
    console.log({config})
    // create 150 items, 
    const muids = Array(upper_limit - lower_limit)
        .fill(null)
        .map((a, idx) => {
        return (`${prefix}-${idx + lower_limit}`)    
    })
    const unusedIds = [];
    for (const muid of muids) {

        if (!dict[muid]) {
            unusedIds.push(muid)
        }
        if (unusedIds.length >= max_count) {
          return unusedIds;
        }
    }
    return unusedIds
}

```

<%* /** Readme
* 🚦If the file basename contains `macro-`, it will also scan the filename for the MUID to knock off the list of reusable muids.
  * Possible 🐛 might occur with a file called "macro-econcomics,muid-1111" <--low possibility
  * [ ] Consider factorizing all the functions so I can have stable defaults ➕ 2025-05-01 #_todo/to-code 
* # On performance:
	* The reason why it's performant is because it is somewhat O(N).
	* The pointer travels through the cache of vfiles (folders removed), and extracts the MUID from the frontmatter and specific title-targetted files. 
	* To create the UI element, a scalar list is created of a specific section of MUIDs, say MUID-50, MUID-51, MUID-52. 
	* This list is iterated against the dictionary. 
		* There may be futher optimization if the filtering pass is included in the vfile iteration.
**/_%>
<%* /** Commit Log
* v
	* Add -template, and ~viewfn-,view to the type of notes scoured 
* v0.0.3 *2025-05-01*
  * Refactor code.
  * bump version
  * Update match MUID algorithm to take into account multiple MUIDs in filename.
  * also scraped the MUID off the filenames of the the files that have "macro-" in them. This lets macros have a unique id without placing it in the frontmatter (avoids templater conflicts)
* v0.0.2
  * render number of unused ids
* v0.0.1
  * Begin process to extract the upper limit and lower limit into knows
  * transfer learnings from experiment over to the About section
**/ _%>

[[macro-for-insert-of-all-inlink-endpoint,uti.-inline-dql,cf.-MUID-128]

---
tags: _meta
DOC_VERSION: v1.0.3
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

- [ ] Consider removing [[cf]] from [[Partial-dataview,vis-Noteshippo,]] because i do not see another person use the term partial dataview.

This note is a [[Partial-dataview,vis-Noteshippo,]].  This [[partial,et-alia]]'s' job is to show the exact files listed inside a subfolder sans files of any tag of a lower than itself.

> [!info] This [[Partial-dataview,vis-Noteshippo,]] has replaced the need for [[deprecating_tag-page-template]]

# =


```dataviewjs

const {default: obs} = this.app.plugins.plugins['templater-obsidian'].templater.current_functions_object.obsidian

const {vault, workspace, metadataCache, fileManager} = this.app

const vf = vault.getAbstractFileByPath(this.currentFilePath)
const frontmatter = metadataCache.getFileCache(vf).frontmatter
const DOC_VERSION = frontmatter.DOC_VERSION || "v.N/A"
const BUTTON_TITLE = `Refresh ${DOC_VERSION} ${vf.basename}` 

workspace.onLayoutReady(main.bind(this));

function main() {
    const {path} = this.app.workspace.getActiveFile();
    const {frontmatter} = this.app
        .metadataCache.getCache(path);
    const alias = frontmatter?.Aliases?.[0]
    const $button = dv.el("button", BUTTON_TITLE);
    function clickHandler(alias, DOC_VERSION) {
      createDashboard(alias, DOC_VERSION);
      new obs.Notice(alias, 7000)
    }
    $button.onclick = () => {
      alias && clickHandler(alias, DOC_VERSION);
    }
}



function createDashboard(alias = "#_",DOC_VERSION) {

    dv.execute(`
        TABLE WITHOUT ID link(file.link, file.name) as "${DOC_VERSION} Root Tag",
        tags
        FROM ${alias}
        FLATTEN join(file.etags," ") as tags
        FLATTEN join(
            filter(
                file.etags, 
                (x) => startswith(
                    x,join(
                        ["${alias}","/"],
                        ""
                    )
                )
            )
        ) as umbrellaTags
        WHERE length(umbrellaTags) = 0
        SORT file.ctime desc
    `)
} 
```

# ---Transient Archive
##  v1.0.3 v2023-12-27

```js

const {default: obs} = this.app.plugins.plugins['templater-obsidian'].templater.current_functions_object.obsidian
const PARTIAL_VERSION = "v1.0.2";
const {vault} = this.app
const {name: codelet_name} = vault.adapter.path.parse(this.currentFilePath)

const BUTTON_TITLE = `Refresh ${PARTIAL_VERSION} ${codelet_name}` 

main.call(this);

function main() {
    const {path} = this.app.workspace.getActiveFile();
    const {frontmatter} = this.app
        .metadataCache.getCache(path);
    const alias = frontmatter?.Aliases?.[0]

    if (!this.marioConfig) {
        this.marioConfig = {
            ...this.marioConfig, 
            dvel:dv.el("button", BUTTON_TITLE),
            handle: () => {
                createDashboard(alias);
                new obs.Notice(alias, 7000)
            }
        }
        const {dvel, handle} = this.marioConfig;
        dvel.addEventListener(
            'click',
            handle
        );
        
        alias && createDashboard(alias);
        return;
    } 
    const {dvel, handle} = this.marioConfig;
    if (dvel && handle) {
        return dvel.removeListener(handle)
    }
}



function createDashboard(alias = "#_") {

    dv.execute(`
        TABLE WITHOUT ID link(file.link, file.name) as "${PARTIAL_VERSION} Root Tag",
        tags
        FROM ${alias}
        FLATTEN join(file.etags," ") as tags
        FLATTEN join(
            filter(
                file.etags, 
                (x) => startswith(
                    x,join(
                        ["${alias}","/"],
                        ""
                    )
                )
            )
        ) as umbrellaTags
        WHERE length(umbrellaTags) = 0
        SORT file.ctime desc
    `)
} 
```

## CEATION_DATE: 2023-06-04

- [ ] Check if the following codelet  is a duplicate of v1.0.0
  - There are slight differences that require folliwng up such as the inline dv query in js. 🤔 Might not be that important to register.
  - Consider scavenging for techniques then archive/commit then version. Then delete?

```js
~~~dataviewjs
const {default: obs} = this.app.plugins.plugins['templater-obsidian'].templater.current_functions_object.obsidian

if (!this.marioConfig) {
    this.marioConfig = {
        ...this.marioConfig, 
        dvel:dv.el("button", "Refresh"),
        handle: () => new obs.Notice('', 7000)
    }
    this.marioConfig.dvel.addEventListener(
        'click',
        this.marioConfig.handle
    );
    
    const alias = dv.current?.().aliases?.[0];
    if (alias) createDashboard(alias);
} else if (this.marioConfig.dvel && this.marioConfig.handle) {
    this.marioConfig.dvel
        .removeListener(this.marioConfig.handle)

}


function createDashboard(alias = "#_") {
    dv.execute(`
        table join(file.etags,"") as tags
        from ${alias}
        FLATTEN join(
            filter(
                file.etags , 
                (x) => startswith(x,alias)
            )
        ) as maps
        WHERE length(maps) = 0
        sort file.ctime desc
    `)
} 
~~~
```

## v1.0.0

* Desc:
  * A prototype showcasing code generating the transclusion-ready view
* Features:
  * Refresh button

```js
~~~dataviewjs

const {default: obs} = this.app.plugins.plugins['templater-obsidian'].templater.current_functions_object.obsidian

main.call(this);

function main() {
    const {path} = this.app.workspace.getActiveFile();
    const {frontmatter} = this.app
        .metadataCache.getCache(path);
    const alias = frontmatter?.Aliases?.[0]
    
    if (!this.marioConfig) {
        this.marioConfig = {
            ...this.marioConfig, 
            dvel:dv.el("button", "Refresh"),
            handle: () => {
                createDashboard(alias);
                new obs.Notice(alias, 7000)
            }
        }
        const {dvel, handle} = this.marioConfig;
        dvel.addEventListener(
            'click',
            handle
        );
        
        alias && createDashboard(alias);
        return;
    } 
    if (this.marioConfig.dvel && this.marioConfig.handle) {
        this.marioConfig.dvel
            .removeListener(this.marioConfig.handle)
        return;
    }
}



function createDashboard(alias = "#_") {
    // new obs.Notice(alias,'500')
    dv.execute(`
        TABLE WITHOUT ID join(file.etags,"\n") as "v1.0 Root Tag"
        FROM ${alias}
        FLATTEN join(
            filter(
                file.etags, 
                (x) => startswith(
                    x,join(
                        ["${alias}","/"],
                        ""
                    )
                )
            )
        ) as umbrellaTags
        WHERE length(umbrellaTags) = 0
        sort file.ctime desc
    `)
} 
~~~
```

## v0.0.1

- [ ] list all types of stub note naming in Andy's note taking system #_todo/to-differential/on-note-naming
* 💣
  * FROM clause input is hard coded
  * Utilizing `this.file.name` to obtain the tag name is not dynamic enough
    * When the tag page changes, the file name that is associated to the tag does not change. 👀 [[tag-wrangler-plugin-for-obsidianmd]] Only the Aliases does.

```js
~~~dataview
TABLE WITHOUT ID link(file.name) as "Root Tag", file.etags
FROM #_meta
FLATTEN (
    join(list("#", this.file.name, "/"), "")
) as startsWithTerm
FLATTEN join(
    filter(
        file.etags , (x) => startswith(x,  startsWithTerm)
    )
) as maps
WHERE length(maps) = 0
sort file.ctime desc
~~~
```

## v0.0.0

```js
~~~dataview
table file.etags, startsWithTerm
FROM #_coding
FLATTEN (
    join(list("#", this.file.name, "/"), "")
) as startsWithTerm
FLATTEN join(
    filter(
        file.etags , (x) => startswith(x,  startsWithTerm)
    )
) as maps
WHERE length(maps) = 0
sort file.ctime desc
~~~
```

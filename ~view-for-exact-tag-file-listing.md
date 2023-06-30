---
tag: _meta 
PARTIAL_VERSION: v1.0.2
---

# =

```dataviewjs
const {default: obs} = this.app.plugins.plugins['templater-obsidian'].templater.current_functions_object.obsidian
const PARTIAL_VERSION = "v1.0.2";

const BUTTON_TITLE = `Refresh ${PARTIAL_VERSION}` 

main.call(this);

function main() {
    const {path} = this.app.workspace.getActiveFile();
    const {frontmatter} = this.app
        .metadataCache.getCache(path);
    const alias = frontmatter?.Aliases?.[0]
    console.log(this.marioConfig, "MM")
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
        sort file.ctime desc
    `)
} 
```

---
# ---Transient Sandbox

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
- [ ] list all types of stub note naming in Andy's note taking system #_todo/to-research-note-naming 
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
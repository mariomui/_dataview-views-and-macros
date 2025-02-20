---
tags:
  - _meta
DOC_VERSION: v1.0.5
MUID: MUID-105
CREATION_DATE: 2024-06-03
---
# -

## 10-Meta
![[~view-for-local-tasks-using-a-progress-bar,nb.-MUID-698#=|nlk]]

```dataview
task where file.name = this.file.name and !completed
```

```dataview
task where file.name = this.file.name and completed
```

## About

This note is a [[Partial-dataview,vis-Noteshippo,]].  This [[partial,etc]]'s' job is to show the exact files listed inside a subfolder sans files of any tag of a lower than itself.

The code in particular is focused on scraping the alias and then getting the tag listing

* ! Root Tags such as `_lcsh` do not work because of recent Obsidian changes. Nested tags still work.

> [!info] This [[Partial-dataview,vis-Noteshippo,]] has replaced the need for [[ø--tag-page-template]]

# =

> [!info] Show files with the exact tag level and not any further nested tag

**file_basename**: *`= this.file.name`* doc-`=this.DOC_VERSION` `= this.MUID`/`=this.heading`/`=this.UMID`/

```dataviewjs

const {default: obs} = this.app.plugins.plugins['templater-obsidian'].templater.current_functions_object.obsidian

const {vault, workspace, metadataCache, fileManager} = this.app

const vf = vault.getAbstractFileByPath(this.currentFilePath)
const frontmatter = metadataCache.getFileCache(vf).frontmatter
const DOC_VERSION = frontmatter?.DOC_VERSION || "v.N/A"
const BUTTON_TITLE = `Refresh ${DOC_VERSION} ${vf.basename}` 

// might as well understand pjeby's around

// var uninstall1 = around(anObject, {
//     someMethod(oldMethod) {
//         return function(...args) {
//             console.log("wrapper 1 before someMethod", args);
//             const result = oldMethod && oldMethod.apply(this, args);
//             console.log("wrapper 1 after someMethod", result);
//             return result;
//         }
//     }
// });
// around loops through the object keys and does aroundActual onto the function. 
/*
createWrapper is function sometMethod(oldMethod) {
    return function (...args) {
        
    }
}
therefore createWrapper has to wrap the oldMedhod.
*/



workspace.onLayoutReady(main.bind(this));

function main() {
    const {path} = this.app.workspace.getActiveFile();
    const {frontmatter} = this.app
      .metadataCache.getCache(path);
    console.log({frontmatter})
    const alias = frontmatter?.Aliases?.[0] || frontmatter?.aliases?.[0]
    const $button = dv.el("button", BUTTON_TITLE + ":" + alias);
    function clickHandler(alias, DOC_VERSION) {
      createDashboard.call(this,alias, DOC_VERSION);
      new obs.Notice(alias, 7000)
    }
    $button.onclick = () => {
      if (!alias) {
        return console.error(`${alias} is missing!`)
      }
      alias && clickHandler.call(this,alias, DOC_VERSION);
    }
}

function createDashboard(alias = "#_",DOC_VERSION) {

    const _space = " "
    const lowerBoundRowsCnt = 10;

    const query_string = `
        TABLE WITHOUT ID link(file.link, file.name) as "${DOC_VERSION} Root Tag",
        tags
        FROM ${alias}
        FLATTEN join(file.etags," ") as tags`
        + _space + // seperate out the most dynamic portion because of possible interporlation errors.
        `FLATTEN econtains(file.etags, ${wrapInQuotes(alias)}) as isContain` 
        + _space + 
        `WHERE isContain = true`
        + _space + 
        `SORT file.ctime desc`
        + _space + 
        `LIMIT ${lowerBoundRowsCnt}`

  dv.execute.call(this,query_string);

    const renderedRowsCount = this.container
      .getElementsByTagName("tr").length;
} 

function wrapInQuotes(alias) {
    const ret = `\"${alias}\"`
    return ret;
}

// experimental utils
function createSignal(signal) {
    let _signal = signal;
    const signalTuple = [getSignal, setSignal]
    return signalTuple;
    function getSignal() {
        return _signal;
    }
    function setSignal(cb) {
        if (typeof cb === "function") {
            _signal = cb(_signal)
            return getSignal();
        }
        _signal = cb
        return getSignal();
    }
}
function around(obj, methodName, createOverride) {
    // the old one is in the instnacne.
    const methodOnInstance = obj?.[methodName] 


    if (!methodOnInstance ) {
        throw new Error('poo')
    }

    obj[methodName] = createOverride(methodOnInstance)

    return uninstall 
    
    function uninstall() {
        return (obj)[methodName] = methodOnInstance
    }
}

```



# ---Transient Commit Log

- v1.0.5
  - Use econtains and etags in the main query so that exact file tag search can be used because e stands for exact matches.
* v1.04
  * Archive the dataview-reliant version of [[~view-for-exact-tag-file-listing]]
  * Remove umbrella tag and DQL filtering due to changes in ObsidianMD tag structure (they removed the back slashes present in any solo tag)
  * Setup prelim pagination prototype
    * pagination bar
    * reactivity
      * Create an example of solidjs createSignal
    * Hook into ui loop to grab the row count at runtime.
      * Create an example of monkey around without the ability to hijack prototypes
        * 🔑 [[monkey-around,vis-Coding-toolbox]] for hijacking prototypes requires access to the class constructor which i don't have.
          * [[new,vis-Web-api,vis-Coding-javascript,]]
# ---Transient Archive

## v1.0.4 archived@v2024-04-09

~~~
```dataviewjs

const {default: obs} = this.app.plugins.plugins['templater-obsidian'].templater.current_functions_object.obsidian

const {vault, workspace, metadataCache, fileManager} = this.app

const vf = vault.getAbstractFileByPath(this.currentFilePath)
const frontmatter = metadataCache.getFileCache(vf).frontmatter
const DOC_VERSION = frontmatter?.DOC_VERSION || "v.N/A"
const BUTTON_TITLE = `Refresh ${DOC_VERSION} ${vf.basename}` 

// might as well understand pjeby's around

// var uninstall1 = around(anObject, {
//     someMethod(oldMethod) {
//         return function(...args) {
//             console.log("wrapper 1 before someMethod", args);
//             const result = oldMethod && oldMethod.apply(this, args);
//             console.log("wrapper 1 after someMethod", result);
//             return result;
//         }
//     }
// });
// around loops through the object keys and does aroundActual onto the function. 
/*
createWrapper is function sometMethod(oldMethod) {
    return function (...args) {
        
    }
}
therefore createWrapper has to wrap the oldMedhod.
*/
function around(obj, methodName, createOverride) {
    // the old one is in the instnacne.
    const methodOnInstance = obj?.[methodName] 


    if (!methodOnInstance ) {
        throw new Error('poo')
    }

    obj[methodName] = createOverride(methodOnInstance)

    return uninstall 
    
    function uninstall() {
        return (obj)[methodName] = methodOnInstance
    }
}


workspace.onLayoutReady(main.bind(this));

function main() {
    const {path} = this.app.workspace.getActiveFile();
    const {frontmatter} = this.app
      .metadataCache.getCache(path);
    console.log({frontmatter})
    const alias = frontmatter?.Aliases?.[0] || frontmatter?.aliases?.[0]
    const $button = dv.el("button", BUTTON_TITLE + ":" + alias);
    function clickHandler(alias, DOC_VERSION) {
      createDashboard.call(this,alias, DOC_VERSION);
      new obs.Notice(alias, 7000)
    }
    $button.onclick = () => {
      if (!alias) {
        return console.error(`${alias} is missing!`)
      }
      alias && clickHandler.call(this,alias, DOC_VERSION);
    }
}

function createDashboard(alias = "#_",DOC_VERSION) {


    const _space = " "
    const lowerBoundRowsCnt = 10;
    // const [counter,setCounter] = createSignal(0);



    const query_string = `
        TABLE WITHOUT ID link(file.link, file.name) as "${DOC_VERSION} Root Tag",
        tags
        FROM ${alias}
        FLATTEN join(file.etags," ") as tags`
        + _space + // seperate out the most dynamic portion because of possible interporlation errors.
        `FLATTEN contains(file.tags, ${wrapInQuotes(alias)}) as isContain` 
        + _space + 
        `WHERE isContain = true`
        + _space + 
        `SORT file.ctime desc`
        + _space + 
        `LIMIT ${lowerBoundRowsCnt}`

    // const prot = Object.getPrototypeOf(dv.api).constructor.prototype


    //   let uninstall = around(dv.api, "table", function (old) {
    //       return function(...args) {
    //           console.log(...args)
    //           console.log(old)
    //           old.apply(this, args)
    //       }
    //   })
      dv.execute.call(this,query_string);

      const renderedRowsCount = this.container
        .getElementsByTagName("tr").length;


      // setTimeout(() => console.log({renderedRowsCount}))

    //   uninstall();
    //   console.log(dv.api.table)

    // dv.table(["a"], [[1,2,3]])


    // for (let i = 5; i > 0; i--) {
    
    //     setCounter((counter() + 1))
    // }
    // console.log(counter())
} 

function wrapInQuotes(alias) {
    const ret = `\"${alias}\"`
    return ret;
}

// fake solidjs
function createSignal(signal) {
    let _signal = signal;
    const signalTuple = [getSignal, setSignal]
    return signalTuple;
    function getSignal() {
        return _signal;
    }
    function setSignal(cb) {
        if (typeof cb === "function") {
            _signal = cb(_signal)
            return getSignal();
        }
        _signal = cb
        return getSignal();
    }
}


```
~~~

## v1.0.3 v2023-12-27

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
        tags,
        umbrellaTags
        FROM ${alias}
        FLATTEN join(file.etags," ") as tags
        FLATTEN join(
            filter(
                file.etags, 
                (x) => startswith(
                    x,join(
                        ["${alias}",""],
                        ""
                    )
                )
            )
        ) as umbrellaTags
        LIMIT 5
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

* 💣
  * FROM clause input is hard coded
  * Utilizing `this.file.name` to obtain the tag name is not dynamic enough
    * When the tag page changes, the file name that is associated to the tag does not change. 👀 [[tag-wrangler-plugin,bt-ObsidianMD-app,]] Only the Aliases does.

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

# ---Transient

- [obsidian-note-gallery/src/main.ts at 3e6239b2a9e4439a2f93edde103921c0544d5603 · pashashocky/obsidian-note-gallery · GitHub](https://github.com/pashashocky/obsidian-note-gallery/blob/3e6239b2a9e4439a2f93edde103921c0544d5603/src/main.ts#L127)
  - There is no way to grab the constructor for the EmbeddedClass component other than to patch Component.addChild
  - When addChild has been used, then the Embedded Class constructor can be store.
  - There are other ways patch around this but it feels dumb and requires too much insider knowledge to understand.


> [!info] This is a good way to use regex to only get an exact file tag listing but the EmbeddedClass and the EmbeddedLeafInitializer are not exposed by obsidian
 
- [ ] What does EmbeddedLeafInitializer do? ➕ 2024-05-11 #_todo/to-muse/upon-obsidianmd-plugin-authoring/regarding-embedded-query-api
  - ℹ The EmbeddedLeafInitializer class is not exposed by [[ObsidianMD-app,]]

```query
line: /#_writing-bit(?!\/)/
```
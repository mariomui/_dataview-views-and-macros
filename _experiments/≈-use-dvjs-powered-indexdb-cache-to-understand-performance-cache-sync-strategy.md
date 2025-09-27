---
CREATION_DATE: 2023-08-05
MUID: MUID-1406
PROJECT_PARENT: 
TEMPLATE_VERSION: "[[∑--experiment-template]]"
tags:
  - _misc/_wip
heading: Plug-ins (Computer programs)--Research
uri: https://id.loc.gov/authorities/subjects/sh98001621
broader:
  - "[[Utilities (Computer programs)]]"
---

# -

## 00-Meta

> [!info]- Progress Bar v0.0.3
> > ![[~view-for-local-tasks-using-a-progress-bar,nb.-MUID-698#=|olk]]
> ```dataview
> task where file.name = this.file.name and !completed
> ```
> > 
> ```dataview
> task where file.name = this.file.name and completed
> ```


### 10÷About

* 🤔 State a lifecyle process to help myself think.
  * 🔰
    * [[reference-note,etc]] -> [[stub-note,etc]] -> [[,aka-library-specced-note]] -> [[claim-note,etc]]
  * 🔚
    * What happens when the [[reference-note,etc]] also requires coding?
    * TLINE:  Use [[interim--question-spec,vis-Noteshippo-taxonomy,]] instead of workboard-note to hold code and learnings.
      * [[experiment_use-dvjs-powered-indexdb-cache-to-understand-performance-cache-sync-strategy-1691247592373.jpeg]]

### 11÷Reference

* †

## 20-Inlink

> [!abstract]- %%  %% Automated List of Reference Inlinks (v0.0.5)
> * ℹ Commit/design logs are located in this [[,aka-MUID-150|experiment note]]. 
> > `= join( map( sort( map( filter(this.file.inlinks, (link) => meta(link).path != this.file.path), (x) => [ split(meta(x).path, "/")[length(split(meta(x).path, "/")) - 1], x ] ) ), (b) => "• " + choice( length(b[0]) > 28, link( b[1], truncate( regexreplace(b[0], "(-of|of|the|-the|-for|-that|https-|ee)", ""), length( regexreplace(b[0], "(-of|of|the|-the|-for|-that|https-|ee)", "") ) * 0.75 ) ), link(b[1], regexreplace(b[0], "\.md$", "")) ) ), "<br>" )`


# =


**base_filepath-v0.0.2**: *`= this.file.path`* doc-`= this.DOC_VERSION` / ids: `= this.MUID`,`= this.UMID` / lcsh: `= this.heading` / updated on: `= dateformat(this.file.mday, "yyyy-LL-dd")` / file-size: `= round(this.file.size/1024,2)` KB

* [[≈-use-dvjs-powered-indexdb-cache-to-understand-performance-cache-sync-strategy#10÷About|10÷About]]
* [[≈-use-dvjs-powered-indexdb-cache-to-understand-performance-cache-sync-strategy#LR--TOC|TOC]]

## Value

- [[indexedDb,vis-Coding,]] is used by both [[+C10,Obsidianmd-app,]] and [[Dataview-plugin,bt.-Obsidianmd-app,]] to store/cache results to improve fetch performance. Understanding it might allow me to directly access the database without caching objects myself.

## Logic

* @ Prerequisites
  * # How Does The Data Look Like?
    * ## Databases Supplied By Obsidian
      * [[≈-use-dvjs-powered-indexdb-cache-to-understand-performance-cache-sync-strategy#LR--codeblock--metadatastore where the metadata is kept|LR--codeblock--metadatastore where the metadata is kept]]
        * `metadata` and `file` are the store names supplied by [[+C10,Obsidianmd-app,]]
    * ## Databases Supplied By Dataview
      * [[≈-use-dvjs-powered-indexdb-cache-to-understand-performance-cache-sync-strategy#LR--codeblock--how To To Open The Dataview Indexdb Cache|LR--codeblock--how To To Open The Dataview Indexdb Cache]]
        * It's a little more complicated. See [[≈-how-to-use-dvjs-powered-dataviewcache,uti.-indexDb]] for an deeper understanding
- @ Differential Between Obsidian And Datview IndexedDb Stores
  - The [[Dataview-plugin,bt.-Obsidianmd-app,]] store is a api response unlike obsidian db api
- ## Directed Code Experiments
  * # Dataview Plugin Exploration
    * [[≈-how-to-use-dvjs-powered-dataviewcache,uti.-indexDb]]
      * This experiment dives into the indexdb cache used by dataview for direct access bypassing dataview apis.
  * # Dataviewjs Powered Snippet
    * [[≈-use-dvjs-powered-indexdb-cache-to-understand-performance-cache-sync-strategy#LR--code--practical Example Of Walking Through Indexdb|LR--code--practical Example Of Walking Through Indexdb]]

## Experiment

- The following is a sandbox of experiments to understand how the obsidianmd app cache works

- the structure is a little different than a simple object db, you go through it like it's a field or cursor and it pops out items that might necessarily be what you want. the entire db is serial in nature. I'm not sure its worth it to learn. It's 10x more code.

```dataviewjs
const {
  workspace, vault, metadataCache
} = this.app;

workspace.onLayoutReady(bootstrap.bind(this))

function bootstrap() {
  (async function(ctx) {
    const {db} = metadataCache;
    const metadataStore = getReadOnlyIDBStore(
      db, 'metadata'
    );
    const fileStore = getReadOnlyIDBStore(
      db, 'file'
    );

    const pop = createPop()
    await iterateStore(
      metadataStore, 
      5, 
      (datum) => {
        pop(datum)
      }
    );
    const pops = pop('mario');
    renderList.call(
      ctx,
      pops.map(mapByField)
    )
    
    function mapByField({field}) {
      return field;
    }
      
    function createPop() {
      const pops = []
      return function pop(datum = null) {
        if (datum === 'mario') {
          return pops;
        }
        pops.push(datum)
      }
    }


    // console.log({pops, metadataStore});
    console.log(await metadataStore.getAll())
  })(this)
  
}

// # UI renderer
function renderList(pops) {
  const md = dv.markdownList.call(this, pops);
  workspace.onLayoutReady(() => {
    dv.paragraph(md);
  })
}

// # UTIL
async function iterateStore(
  store, 
  count,
  cb = console.log
) {
    const cursor = await store.openCursor()    
    
    let i = count;
    for (const field in cursor.value) {  
      cb({
        field, 
        data: cursor.value[field]
      });  
      i--;
    }
    if (count > 0) {
      await cursor.continue()
    } else {
      return;
    }
}
function getReadOnlyIDBStore(
  db, store_name
) {
  const tx = db.transaction(
      store_name, 'readonly'
  );  
  const store = tx
    .objectStore(store_name);  
  return store;
}

// they took out the get all function this doesnt work to get all data from store.
async function genGetAll(store) {
  if (!store?.getAll) return Promise.resolve();
  return await store.getAll();
}
```

- Use the indexdb api to use a [[∑--ß-118-emphasis-in-Writing-Mythcreants-https-mythcreants-com-blog-podcasts-118-emphasis-in-writing|turing]]-like cursor to move across the metadataCache



# ---Transient

* TLINE: *How to do performance metrics on IndexedDB questions?*
  * Tool
    * [ ] [The Trace Event Profiling Tool (about:tracing)](https://www.chromium.org/developers/how-tos/trace-event-profiling-tool/) #_todo/to-study/on-chrome/regarding-performance-measuring
  * Procedure
    * [Speeding up IndexedDB reads and writes | Read the Tea Leaves](https://nolanlawson.com/2021/08/22/speeding-up-indexeddb-reads-and-writes/)
  * Batched Cursor Technique
    * [(#6031) - faster IDB changes() with batched cursor by nolanlawson · Pull Request #6033 · pouchdb/pouchdb · GitHub](https://github.com/pouchdb/pouchdb/pull/6033) #_todo/42-priority-high--/to-study/on-indexdb-querying/regarding-improving-performance
  * Resources
    * [Working with IndexedDB](https://web.dev/indexeddb/)
      * example code


# ---Transient Jobs

![[~viewfn-for-sluicing-header-links-for-citations,nb.-MUID-1560#=|?search_term=---Transient Local&t=nlknoui-scroll]]


# ---Transient Local Resources

## LR--codeblock--how To To Open The Dataview Indexdb Cache

```js
const APP_ID = this.app.appId
const dbName = "dataview/cache/" + APP_ID
const DBOpenRequest = window.indexedDB.open(dbName)
```

## LR--codeblock--metadatastore Where The Metadata Is Kept

```js
  const {db} = metadataCache;
  const metadataStore = getReadOnlyIDBStore(
    db, 'metadata'
  );
```

## LR--code--practical Example Of Walking Through Indexdb

* ? Does getting the file this way directly from the indexdb more performant? possibly not.
```dataviewjs
const {
  workspace, vault, metadataCache
} = this.app;

workspace.onLayoutReady(bootstrap.bind(this))

function bootstrap() {
  (async function(ctx) {
    const {db} = metadataCache;
    const metadataStore = getReadOnlyIDBStore(
      db, 'metadata'
    );
    const fileStore = getReadOnlyIDBStore(
      db, 'file'
    );
    const metaStore = await metadataStore.getAll()
    
    const item = metaStore.find((datum) => {
      return datum?.frontmatter?.MUID === "MUID-568"
    })
    console.log({item})
  })(this) 
}

function getReadOnlyIDBStore(
  db, store_name
) {
  const tx = db.transaction(
      store_name, 'readonly'
  );  
  const store = tx
    .objectStore(store_name);  
  return store;
}
``
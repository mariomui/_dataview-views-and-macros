---
CREATION_DATE: 2023-12-05
MUID: MUID-1851
PROJECT_PARENT: "[[+π,learn-dataview,vis-Coding]]"
TEMPLATE_VERSION: v1.0.7_note-refactor-template
tags:
  - _misc/_wip
heading: Plug-ins (Computer programs)--Development
uri: https://id.loc.gov/authorities/subjects/sh98001621
broader:
  - "[[Utilities (Computer programs)]]"
---
# -

## 00-Meta
![[~view-for-local-tasks-using-a-progress-bar,nb.-MUID-698#=|nlk]]

```dataview
task where file.name = this.file.name and !completed
```

```dataview
task where file.name = this.file.name and completed
```

### 10÷About

This [[,aka-experiment-specced-note]] details the experiment to mess with the dataview cache as controlled by [[indexedDb,vis-Coding,]]

[[,aka-experiment-specced-note]]

[[experiment-symbol,uti.-≈,bt.-Noteshippo-title-level-affix,]]
### 11÷Reference

![[~view-for-referencing-current-jumpid#=|nlk]]

* †

# =


**base_filepath-v0.0.6**: `= choice( contains(this.file.folder, this.file.name), link(this.file.path), join(["*",this.file.path,"*"], ""))` doc-`= this.DOC_VERSION` / ids: `= this.MUID`,PP:`= this.PROJECT_PARENT` / lcsh: `= link(this.heading)`

dataview specific 
- This code shows how to walk the indexdb cache populated by dataview.
```dataviewjs
const {
  workspace, vault, metadataCache
} = this.app;
const APP_ID = this.app.appId
const dbName = "dataview/cache/" + APP_ID
workspace.onLayoutReady(bootstrap.bind(this))
async function genDBOpen(dbname) {
  const DBOpenRequest = window.indexedDB.open(dbName)
  return new Promise((resolve,reject) => {
    DBOpenRequest.onerror = (event) => {
      reject({
        error: event,
        desc: "Error loading database",
        payload: null
      })
    };
    DBOpenRequest.onsuccess = (event) => {
      const success_msg = "Database initialized";
      resolve({
        error: null,
        desc: success_msg,
        payload: DBOpenRequest.result
      })
    }
  })
}
function bootstrap() {
  (async function(ctx) {

    const request = await genDBOpen(dbName)
    if (request.error) {
      console.error({request})
      return null
    }
    const db = request.payload;
    console.log({db}, "dvdb")
    const metadataStore = getReadOnlyIDBStore(
      db, 'keyvaluepairs'
    );
    console.log('all',await metadataStore.getAll())
    await iterateStore(metadataStore,5, (pops) => {
      const keys = Object.keys(pops)
      const entry = Object.entries(pops.data)
      console.log({entry})
      dv.list(entry)
      renderList.call(
        ctx,
        keys
      )
    })

      
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
    const _cursor = await store.openCursor()    
    _cursor.onsuccess = async () => {
  
      const {result} = _cursor

      cb(result.value)
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

---

# ---Transien
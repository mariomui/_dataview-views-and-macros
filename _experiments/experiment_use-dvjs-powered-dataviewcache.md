---
TEMPLATE_VERSION: v1.0.7_note-refactor-template
MUID: MUID-1851
CREATION_DATE: 2023-12-05
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


*`= this.file.name`*

dataview specific
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

# ---Transient
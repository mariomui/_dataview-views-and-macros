---
tag: _meta _coding/partial
VERSION: v1.00
ID: TCODEID-1
SHORT_NAME: garden-note-by-lottery
UMID: 
---
# -

```dataview
task where file.name = this.file.name and !completed
```

- [x] Add pending results?  
- [ ] Update demo with new content, and create warning service for staleness.
  - Old demo are not stampdated and thereby exposed to drift.
  - Is it worth the time to create a syncing solution whereby code on this document creates a stale alert automatically?
    - The [[~view-for-note-strength-alert]] codelet is 50% of the way there...
- [ ] Instead of shaping the data so that it is transposed, i should think about it in terms of data. It should be in rows. The UI, well, that should be taken care of by UI transformations.
## About

See [[demo-of-partial-view-for-lotterizing-note-work-slate.gif|demo video]] for details on usage.


# =

```dataviewjs

const {default: obs} = this.app.plugins.plugins['templater-obsidian'].templater.current_functions_object.obsidian

const {workspace, vault, metadataCache} = this.app;
const sourceVirtualFile = vault.getAbstractFileByPath(
  this.currentFilePath
);
const sourceFrontmatter = metadataCache.getFileCache(
  sourceVirtualFile
);

const {VERSION, SHORT_NAME} = sourceFrontmatter.frontmatter
const _SHORT_NAME = SHORT_NAME ? (SHORT_NAME || '').toUpperCase().at(0) + SHORT_NAME.slice(1) : "";
// # data
const folder_names = [
    "B_churn_box",
    "A_sources",
    "B_seeds",
    "/"
];
const button_title = `${_SHORT_NAME} ${VERSION}`;
const click_text = "Obsidian Powered text";
const time = 700;

// figgers must be before bootstrap
const folderAliasFig = {
    "/": "Root"
}

const createGetter = (fig = {}) => (key, fallback) => {
    return fig[key] ?? fallback
} ;
const getFolderAlias = createGetter(folderAliasFig);

// # bootup
workspace.onLayoutReady(bootstrap.bind(this))
function bootstrap() {
  main.call(
      this,
      button_title, 
      function hc() {
          if (!this?.fig) {
              const el = 
                  generateTable(this.app,folder_names);
              this.fig = Object.assign({}, {
                  should: false,
                  el,
              })
              return el;
          } else {
              this.container.lastChild.remove()
              const nel = generateTable(
                  this.app, folder_names
              ); 
          }
      }.bind(this)
  );
}

// # business logic
 function main(
    button_title,
    handleClick
) {


    const btn = new obs
        .ButtonComponent(this.container)
        .setButtonText(button_title)
        .onClick(() => handleClick());
    this.fig = {
        btn,
        el: handleClick()
    }
}

// # helper
function generateTable(app,folder_names) {
    const els = [];
    const fls = [];
    for (const folder_name of folder_names) {
        const {max, _acf} = prepam(
            app.vault, 
            folder_name
        );
        const { path, name } = _acf[randomNum(0, max)]         
        els.push(
            renderLink(
                manuLinkFromFolderName(
                    path, 
                    getFolderAlias(
                        folder_name, 
                        folder_name
                    ) 
                )
            )
        )
        fls.push(
            manuNameAndCountFromFolderName(
                name, max
            )
        )
    }
    const _fls = fls.reduce((chain, {name, filesCnt}) => {
        const [a,b] = chain;
        a.push(name)
        b.push(filesCnt)
        return chain;
    }, [[],[]])
    
    const headers = Array(els.length)
        .fill(null).map(() => "º");
    const mdt = dv.markdownTable(
        headers, [els, _fls[0], _fls[1]]
    );
   return dv.paragraph(mdt);
}

function prepam(vault, folder_name) {
    const abf = vault
        .getAbstractFileByPath(folder_name)
    if (abf?.children) {
        const acf = abf.children.filter(
            ({extension}) => extension === "md"
        )
        const max = acf.length
        const _acf = acf.slice().sort()

        return {
            _acf,
            max,
        }
    }
}
function manuLinkFromFolderName(
    path,
    folder_name
) {
    return dv.fileLink(
        path,
        false,
        folder_name
    )
}
function manuNameAndCountFromFolderName(
    name,
    max,
) {
    return {name, filesCnt : max}
}

function randomNum(min, max) {
    return Math.floor(Math.random() * (max - min)) + min
}
function renderLink(link) {
    return link;
}


```
# ---Transient
---
tags:
  - _meta
DOC_VERSION: v1.0.3
CODELET_SHORTNAME: garden-note-by-lottery
UMID: 
MUID: MUID-129
---
# -

[[~view-for-unused-MUIDs]]

```dataview
task where file.name = this.file.name and !completed
```

## 10-About

See [[demo-of-partial-view-for-lotterizing-note-work-slate.gif|demo video]] for details on usage.

* [ ]  Convert [[~view-for-lotterizing-note-work-slate-TCODEID-1]] into transcluded code. ( possible inputs would be folders I want? I would prolly want to hook into templater for that.)
  * Is there a way to instance transcluded code? 🔑 Not htat i know of.

- [ ] Remove dependencies to dv if possible. ➕ 2025-02-04 #_todo/80-longterm--/to-code 


# =


```dataviewjs
const { default: obs } =
  this.app.plugins.plugins["templater-obsidian"].templater
    .current_functions_object.obsidian;

const { workspace, vault, metadataCache, fileManager } = this.app;
const sourceVirtualFile = vault.getAbstractFileByPath(
  this.currentFilePath,
);
const sourceFrontmatter =
  metadataCache.getFileCache(sourceVirtualFile);

const { DOC_VERSION: VERSION, CODELET_SHORTNAME } = sourceFrontmatter.frontmatter;

const _CODELET_SHORTNAME = CODELET_SHORTNAME
  ? (CODELET_SHORTNAME || "").toUpperCase().at(0) + CODELET_SHORTNAME.slice(1)
  : "";
// # data
const folder_names = ["B_churn_box", "A_sources", "B_seeds", "/"];
const button_title = `${_CODELET_SHORTNAME} ${VERSION}`;
const click_text = "Obsidian Powered text";
const time = 700;

// figgers must be before bootstrap
const folderAliasFig = {
  "/": "Root",
};

const createGetter =
  (fig = {}) =>
  (key, fallback) => {
    return fig[key] ?? fallback;
  };
const getFolderAlias = createGetter(folderAliasFig);

// # bootup
workspace.onLayoutReady(bootstrap.bind(this));
function bootstrap() {
  // this.container.lastChild.remove();
  main.call(
    this,
    button_title,
    handleRegenerateUiOnClick.bind(this)
  );
  const el = generateTable(this.app, folder_names);
}


function handleRegenerateUiOnClick() {
  this.container.lastChild.remove();
  const el = generateTable(this.app, folder_names);
  return el;
}

// # business logic
function main(button_title, handleClick) {
  const btn = new obs.ButtonComponent(this.container)
    .setButtonText(button_title)
    .onClick(() => handleClick());
}

// # helper
function generateTable(app, folder_names) {
  const els = [];
  const fls = [];
  for (const folder_name of folder_names) {
    const { max, _acf } = prepam(app.vault, folder_name);
    const { path, name } = _acf[randomNum(0, max)];
    els.push(
      renderLink(
        manuLinkFromFolderName(
          path,
          getFolderAlias(folder_name, folder_name),
        ),
      ),
    );
    fls.push(manuNameAndCountFromFolderName(name, max));
  }
  const _fls = fls.reduce(
    (chain, { name, filesCnt }) => {
      const [a, b] = chain;
      a.push(name);
      b.push(filesCnt);
      return chain;
    },
    [[], []],
  );

  const headers = Array(els.length)
    .fill(null)
    .map(() => "º");
  const [filenames, filesCnts] = _fls;
  const vfs = filenames.map((filename) => metadataCache.getFirstLinkpathDest(filename));
  const markdownLinks = vfs.map((vf) => fileManager.generateMarkdownLink(vf, vf.basename)).map(wrapPredicate)

  const mdt = dv.markdownTable(headers, [els, markdownLinks, filesCnts]);
  return dv.paragraph(mdt);

  function wrapPredicate(markdownLink) {
     return `\<span style\="--link-color\:#A3C1C9 !important"\>${markdownLink}<\span>`
  }
}

function prepam(vault, folder_name, ext = "md") {
  const abf = vault.getAbstractFileByPath(folder_name);
  if (abf?.children) {
    const acf = abf.children.filter(
      ({ extension }) => extension === ext,
    );
    const max = acf.length;
    const _acf = acf.slice().sort();

    return {
      _acf,
      max,
    };
  }
}
function manuLinkFromFolderName(path, folder_name) {
  return dv.fileLink(path, false, folder_name);
}
function manuNameAndCountFromFolderName(name, max) {
  return { name, filesCnt: max };
}

function randomNum(min, max) {
  return Math.floor(Math.random() * (max - min)) + min;
}
function renderLink(link) {
  return link;
}

```

# ---Transient


v1.0.0
````

```dataviewjs
const { default: obs } =
  this.app.plugins.plugins["templater-obsidian"].templater
    .current_functions_object.obsidian;

const { workspace, vault, metadataCache } = this.app;
const sourceVirtualFile = vault.getAbstractFileByPath(
  this.currentFilePath,
);
const sourceFrontmatter =
  metadataCache.getFileCache(sourceVirtualFile);

const { DOC_VERSION: VERSION, CODELET_SHORTNAME } = sourceFrontmatter.frontmatter;
const _CODELET_SHORTNAME = CODELET_SHORTNAME
  ? (CODELET_SHORTNAME || "").toUpperCase().at(0) + CODELET_SHORTNAME.slice(1)
  : "";
// # data
const folder_names = ["B_projects"];
const button_title = `${_CODELET_SHORTNAME} ${VERSION}`;
const click_text = "Obsidian Powered text";
const time = 700;

// figgers must be before bootstrap
const folderAliasFig = {
  "/": "Root",
};

const createGetter =
  (fig = {}) =>
  (key, fallback) => {
    return fig[key] ?? fallback;
  };
const getFolderAlias = createGetter(folderAliasFig);

// # bootup
workspace.onLayoutReady(bootstrap.bind(this));
function bootstrap() {
  main.call(
    this,
    button_title,
    function hc() {
      if (!this?.fig) {
        const el = generateTable(this.app, folder_names);
        this.fig = Object.assign(
          {},
          {
            should: false,
            el,
          },
        );
        return el;
      } else {
        this.container.lastChild.remove();
        const nel = generateTable(this.app, folder_names);
      }
    }.bind(this),
  );
}

// # business logic
function main(button_title, handleClick) {
  const btn = new obs.ButtonComponent(this.container)
    .setButtonText(button_title)
    .onClick(() => handleClick());
  this.fig = {
    btn,
    el: handleClick(),
  };
}

// # helper
function generateTable(app, folder_names) {
  const els = [];
  const fls = [];
  for (const folder_name of folder_names) {
    const { max, _acf } = prepam(app.vault, folder_name);
    const { path, name } = _acf[randomNum(0, max)];
    els.push(
      renderLink(
        manuLinkFromFolderName(
          path,
          getFolderAlias(folder_name, folder_name),
        ),
      ),
    );
    fls.push(manuNameAndCountFromFolderName(name, max));
  }
  const _fls = fls.reduce(
    (chain, { name, filesCnt }) => {
      const [a, b] = chain;
      a.push(name);
      b.push(filesCnt);
      return chain;
    },
    [[], []],
  );

  const headers = Array(els.length)
    .fill(null)
    .map(() => "º");
  const mdt = dv.markdownTable(headers, [els, _fls[0], _fls[1]]);
  return dv.paragraph(mdt);
}

function prepam(vault, folder_name) {
  const abf = vault.getAbstractFileByPath(folder_name);
  if (abf?.children) {
    const acf = abf.children.filter(
      ({ extension }) => extension === "md",
    );
    const max = acf.length;
    const _acf = acf.slice().sort();

    return {
      _acf,
      max,
    };
  }
}
function manuLinkFromFolderName(path, folder_name) {
  return dv.fileLink(path, false, folder_name);
}
function manuNameAndCountFromFolderName(name, max) {
  return { name, filesCnt: max };
}

function randomNum(min, max) {
  return Math.floor(Math.random() * (max - min)) + min;
}
function renderLink(link) {
  return link;
}
```
````

# ---Transient Commit Log

[[transient-commit-log-endpoint,bt.-Noteshippo-heading-api,]]

* v1.0.3 *2025-02-04*
  * Make the links under the folder clickable.
  * Remove the TCODE from yaml and from title
  * use secondary inline color for markdownTable markdownlinks
    * light teal `#A3C1C9`
* v1.0.2 Fix bug where button removed the element without repopulating
  * Related topic. using global objects to handle a once object is no bueno.

## Ω

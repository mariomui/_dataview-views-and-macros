---
tag: _wip
VERSION: v0.0.1-TCODEID-5
---

# -

This [[experiment-note]] explores the substitution of the renderValue function with my own so that i can attach to the domFragment.
Key files are genRenderVfs, which starts the process
genListAsDomV2 and customRenderFiles are extraneous middleman.

genRenderValue is the recursive ui impl which has the same implementation as dataview only stripped down to list components.

⚠ There is a refreshing bug that happens.

> [!warning] Requires obsidian apis from templater 
> https://github.com/SilentVoid13/Templater/blob/487805b5ad1fd7fbc145040ed82b4c41fc2c48e2/src/core/Templater.ts#L79C14-L79C14
> * **TLINE**: *Populate Startup Template with dummy file in order to access obs api through templater plugin even when within the dataviewjs environ.*
>   * parse_template populates current_functions_object with the obs module
>   * Startup Templates is the ui setting that calls parse template
>   * ![[experiment_list-a-folders-contents-and-add-to-dom-1692346851509.jpeg]]

# =

```dataviewjs
const {vault, plugins, workspace, fileManager, metadataCache} = this.app;
const {default: obs} = plugins.plugins['templater-obsidian'].templater.current_functions_object.obsidian

const current_file_path = dv.currentFilePath

workspace.onLayoutReady(main.bind(this))

function main() {
  //knobs
  // const vf = workspace.getActiveFile()
  // const folder_path = vf?.parent?.path
  const folder_path = vault
    .getAbstractFileByPath(
      current_file_path
    ).parent?.path;

  if (!folder_path) return;

  //workhorse
  (async function(ctx, genRenderVfs) {
    // start scope
    { var _files = [],
          recursedVfs = [];

      try {
        var recursedVfs = await getRecursiveList(folder_path)

        const {files} = await genListByFolderPath(
          folder_path
        );
        var _files = _files.concat(files)
      } catch(err) {

        return err;
      }
      await genRenderVfs(recursedVfs)
    } // end scope

    return;
  })(
    this,
    genRenderVfs.bind(this),
  );
}


async function genRenderVfs(vfs) {

  for (const vf of vfs) {

    if (vf.hasOwnProperty("children")) {
      const {files} = await genListByFolderPath(vf.path)
      const file_paths = files;

      await genRenderListTuple.call(
        this,[vf.name, file_paths]
      );
    }
  }
  return;
}




/**
@param val{string|Array}
@param
@return void
**/
async function genRenderListTuple(listTuple) {
  const $frag = document.createDocumentFragment();
  await genRenderValue.call(this,
    listTuple,
    $frag,
    true,
    "list",
    0,
    this.component
  )
  return this.container.append($frag)
}

function renderFiles(file_paths) {

  const links = file_paths
    .map((file_path) => {
      const vf = vault.getAbstractFileByPath(
        file_path
      );
      /* return metadataCache.fileToLinktext(
        vf, ""
      );
      */ // dumb api that generates the shortname from a vfile
      return fileManager.generateMarkdownLink(vf, "")
    })

  dv.list(links);
}
async function genListByFolderPath(folder_path) {
    const list = await vault.adapter
      .list(obs.normalizePath(folder_path))
    return list
}
async function getRecursiveList(path) {
  const vfs = []
  const tfolder = vault.getAbstractFileByPath(path)
  const reclist = await obs.Vault.recurseChildren(tfolder, cb)
  return vfs;

  function cb(vf) {
    vfs.push(vf);
  }
}

function isArray(candidate) {
  return Object.prototype.toString.call(candidate) === '[object Array]';
}


///////

/**
        child, //value
        li,  // container
        isExpandList, // alwways expand list really
        "list", // context
         depth + 1 // escape hatch
**/
async function genRenderValue(
  val,
  container,
  isExpandList,
  context,
  depth,
  component
) {
  if (depth > 2) return;
  if (typeof val === "string") {
    await genRenderCompactMarkdown(
      val,
      container,
      "",
      component
    )
    return;
  }

  if (isArray(val)) {

    const clss = context === "list" ?
      "dataview-result-list-ul" :
      "dataview-result-list-root-ul";
    const cls = {
      cls: [
        "dataview",
        "dataview-ul",
        clss
      ]
    }
    let list = container.createEl(
      "ul",
      cls
    );
    for (let child of val) {
      const rootStyle = {
        attr: {
          style: "list-style-type: none"
        }
      };
      const style = isArray(child) ? rootStyle : {};
      let li = list.createEl("li", manuLiCss(style));
      const context = isArray(child) ? "list" : "";
      await genRenderValue(
        child, //value
        li,  // container
        isExpandList, // is rest of list
        context,
        depth + 1, // escape hatch
        component
      );
    }
  }
}

function manuLiCss(config = {}) {
 return {
   cls: "dataview-result-list-li",
   ...config
 }
}

// i hate this fucntion.
async function genRenderCompactMarkdown(
    markdown, // string,
    container, // HTMLElement,
    sourcePath, // string,
    component // Component
) {
  let subcontainer = container.createSpan();
  await obs.MarkdownRenderer.renderMarkdown(
    markdown, subcontainer, sourcePath, component
  );

}
```

# ---Transient

![[#Figure 1 Lists all files recursively of containing folder|nlk]]

```js
function getFileNameFromPath(file_path) {
  const { name } = vault.adapter.path.parse(filepath);
  return name;
}
```

# ---Transient Scrap Paper

       value: any,
        container: HTMLElement,
        component: Component,
        filePath: string,
        inline: boolean = false

```dataviewjs
const file_path = dv.currentFilePath;
DataviewAPI.renderValue(
  [
    " * [ ] hi",
    [
      " * [ ] ![[experiment_list-a-folders-contents-and-add-to-dom]]",
      [
        " * [ ] a",
        [
          " * [ ] d"," * [ ] e"
        ]
      ]
    ]
  ], this.container, this.component, file_path, true)
```

# ---Transient Local Resources

## Figure 1: Lists all files recursively of containing folder

![[experiment_list-a-folders-contents-and-add-to-dom-1692136016163.jpeg]]

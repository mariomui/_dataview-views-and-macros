---
tag: _wip 
VERSION: v0.0.1
---

# -

This [[experiment-note]] explores the substitution of the renderValue function with my own so that i can attach to the domFragment.
Key files are genRenderVfs, which starts the process
genListAsDomV2 and customRenderFiles are extraneous middleman.

genRenderValue is the recursive ui impl which has the same implementation as dataview only stripped down to list components.

⚠ There is a refreshing bug that happens.

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
  .getAbstractFileByPath(current_file_path).parent?.path;
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
        console.error(err);
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

      await genListAsDomV2.call(this, vf.name, files)
    }
  }
  return;
}

async function genListAsDomV2(header,datums) {
  await customRenderFiles.call(this,[header,datums])
}

/**
@param val{string|Array}
@param 
@return void
**/
async function customRenderFiles(file_paths) {
  const $frag = document.createDocumentFragment();
  await genRenderValue.call(this,
    file_paths, 
    $frag,
    true,
    "list",
    0,
    new obs.Component()
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

function getFileNameFromPath(file_path) {
  const {name} = vault.adapter.path.parse(filepath)
  return name;
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
      {cls}
    );
    for (let child of val) {
      let li = list.createEl("li", manuLiCss());
      await genRenderValue(
        child, //value 
        li,  // container
        isExpandList, // is rest of list
        "list", // context
        depth + 1, // escape hatch
        component
      );
    }
  }
}

function manuLiCss() {
 return { cls: "dataview-result-list-li" }
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
# ---Transient Local Resources

## Figure 1: Lists all files recursively of containing folder
![[experiment_list-a-folders-contents-and-add-to-dom-1692136016163.jpeg]]
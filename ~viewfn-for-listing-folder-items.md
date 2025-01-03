---
TEMPLATE_VERSION: v1.0.4
TEMPLATE_SOURCE: "[[10--nascent-spec-template]]"
tags:
  - _wip
UMID: 
MUID: MUID-126
DOC_VERSION: v.0.0.1
---

# -

## About

This [[Partial-dataview,vis-Noteshippo,]] uses [[custom-transclusion-parameters,cf.-Kanzi,vis-ObisidianMD-app,]] to scrape the contents of a folder, regardless of whatever files maybe hidden.

[[~view-for-unused-MUIDs]]

# =

**file_basename**: *`= this.file.name`* doc-`=this.DOC_VERSION`

- Usage: ?isDynamic=1 (search current file's parents and list)

```dataviewjs
const {vault, plugins, workspace, fileManager, metadataCache} = this.app;
const {default: obs} = plugins.plugins['templater-obsidian'].templater.current_functions_object.obsidian

const current_file_path = dv.currentFilePath


workspace.onLayoutReady(main.bind(this))


function manuFig() {
  return {
    vault,
    getEmbedsFromVf,
    extractTargetEmbed,
    manuParams,
    parseStringToMap,
  }
}

function main() {
  const getCurrentfilepath = () => this.currentFilePath;
  
  //knobs
  const vf = workspace.getActiveFile()
  const folder_path = vf?.parent?.path
  
  // const folder_path = vault.getAbstractFileByPath(current_file_path).parent?.path;
  
  const providing_path = getCurrentfilepath();
  const providing_folder_path = vault
    .getAbstractFileByPath(providing_path).parent?.path;
  
  if (!folder_path || !providing_folder_path) return;

  // scrape custom transclusion parameters
  
  const argMap = extractParams(
    providing_path,
    workspace.getActiveFile(),
    manuFig()
  );
  const isDynamic = argMap?.isDynamic === "1";
  
  console.log({argMap, isDynamic, folder_path, providing_folder_path});
  
  //workhorse
  (async function(ctx, genRenderVfs) {
    // start scope 
    { var _files = [],
          recursedVfs = []; 
      
      try {
        const _folder_path = isDynamic ?  folder_path : providing_folder_path;
        var recursedVfs = await getRecursiveList(_folder_path)

        const {files} = await genListByFolderPath(
          _folder_path
        );
        console.log({files})
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


/**
        child, //value 
        li,  // container
        isExpandList, // alwways expand list really
        "list", // context
         depth + 1 // escape hatch
         
@desc recursive renderer of lists.
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
/**
@desc LIterally attaches a span literal to the subcontainer right away.
**/
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

function manuLiCss() {
 return { cls: "dataview-result-list-li" }
}

// Extract Params Utility Code *2024-07-19*
function manuParams() {
  return {
    search_term: "",
    regex_flag: "g"
  }
}
function extractParams(
  current_filepath,
  vf,
  strategies = {
    vault,
    getEmbedsFromVf,
    extractTargetEmbed,
    manuParams,
    parseStringToMap,
  }
) {
  const {
    vault,
    getEmbedsFromVf,
    extractTargetEmbed,
    manuParams,
    parseStringToMap,
  } = strategies;

  const embeds = getEmbedsFromVf(vf)
  const {name} = vault.adapter.path
    .parse(current_filepath);
  const embed = extractTargetEmbed(
    name,
    embeds
  );
  const displayText = embed?.displayText;


  if (!displayText) {
    return manuParams()
  };

  const argMap = parseStringToMap(
    displayText
  );
  return {
    ...manuParams(),
    ...argMap
  };
  // /end params extraction
}


// helpers
function getEmbedsFromVf(vf) {
  return metadataCache
    .getFileCache(vf)?.embeds || [];
}

function extractTargetEmbed(
  embed_name,
  embeds
) {
  const embed = embeds?.find(
  (embed) => {
    const parsed = obs.parseLinktext(
      embed.link
    );
    return parsed.path === embed_name;
  });
  return embed;
}

function parseStringToMap(str) {
  { var parsed = {};
    try {
      parsed = vault.adapter
        ?.url
        ?.parse(str, true);
    } catch (err) {

      return {
        err
      }
    }
    return parsed?.query || {};
  }
  return {};
}
```


# ---Transient

[[common-utility-code,vis-Dataviewjs,etc]]


[[transient-doc-log-endpoint,bt.-Noteshippo-heading-api,]]

# --Transient Doc Log

- v0.0.1 *2024-07-19*
  - I am continuitng to use doc log because it just makes sense not to do anything like code version. If i ever have something that isn't code or a doc, what am i going to do, name it someting else? Better to just rename [[transient-commit-log-endpoint,bt.-Noteshippo-heading-api,]] as Template log.
    - [ ] Understand commit log and see if i can partition templating logs away from noteshippo system ➕ 2024-07-19 #_todo/52-priority-low--/to-muse 
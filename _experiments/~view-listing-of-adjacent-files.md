---
CREATION_DATE: 2024-03-29
DOC_VERSION: v0.0.0
MUID: MUID-2258
TEMPLATE_VERSION: v1.0.6
TEMPLATE_SOURCE: "[[10--blank-no-api-template]]"
UMID: 
aliases: 
tags:
  - _wip
---

# -

## About

> [!info] %%  %% Goal
>

# =

**filename:** `=this.file.path`

```dataviewjs
const CODELET = {
  VERSION: "v1.0.0",
  TITLE: "LIST-FILES-WITHIN-FOLDER-NOTE"
}
const {workspace} = this.app;
const vf = workspace.getActiveFile();


workspace.onLayoutReady(
  main.bind(this)
);

function main() {
  (async function (ctx) {
    const prefix = workspace
      .getActiveFile().parent.getParentPrefix()
    const {
      files: file_paths 
    } = await ctx.app.vault
        .adapter.list(prefix)
    console.log({prefix})
    const data = file_paths
      .map(
        (file_path) => {
          if (!file_path.endsWith(".md")) return false;
          const link = getWikiLinkByFilePath(
            ctx, 
            file_path
          );
          return Array(link) // vertical columned.
        }  
      )
      .filter(Boolean)
    
    await genRenderTable(ctx, data)
  })(this)
}

// ui domain
async function genRenderTable(ctx, data, limit = 50) {
  const _data = data.length > limit ? data.slice(0,50) : data;

  return await dv.table(
   [`${CODELET.TITLE} ${CODELET.VERSION} :: ${_data.length}/${data.length}`],
   _data 
  )
}

// business domain
function getWikiLinkByFilePath(
  ctx, file_path, config = { embed: "=" }
) {
  const vf = getVirtualFileByPath(ctx, file_path)
  return ctx.app.fileManager
    .generateMarkdownLink(
      vf, 
      vf.path, 
      `${vf.path}#${config.embed}`, 
      vf.basename
  )
}
function getVirtualFileByPath(ctx, file_path) {
  console.log({file_path})
  return ctx.app.vault.getAbstractFileByPath(file_path)
}


```

# ---Transient Local Resources

# ---Transient Local Archive

## LA--code--version with heavy reliance to the dataview ui 

```dataviewjs
const vf = this.app.vault.getAbstractFileByPath(dv.currentFilePath)
const parentPath = `"${vf.parent.path}"`;
const list = dv.pages(parentPath).map(mapByFileNameToLink).groupBy((link) => {
  return link.path.split('/').slice(0,2).join('/')
})

function mapByFileNameToLink({file}) {
  return dv.fileLink(file.path,false, file.name.length > 46 ? file.name.substring(0,46)+ "...." : file.name)
}

const matrix = list.map((l) => [l.key, l.rows]) 

DataviewAPI.renderValue(matrix,this.container,this.component,dv.currentFilePath, true)
```
---
TEMPLATE_VERSION: v1.0.7_note-refactor-template
MUID: MUID-2184
CREATION_DATE: 2024-02-25
tags: _wip 
UMID: 
---
# -

![[~view-for-local-tasks-using-a-progress-bar,nb.-MUID-698#=|nlk]]

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
**base_filepath-v0.0.2**: *`= this.file.path`* doc-`= this.DOC_VERSION` / ids: `= this.MUID`,`= this.UMID` / lcsh: `= this.heading` / updated on: `= dateformat(this.file.mday, "yyyy-LL-dd")` / file-size: `= round(this.file.size/1024,2)` KB


*`= this.file.name`*

```dataviewjs
const {vault, plugins, workspace, fileManager, metadataCache} = this.app;
const {default: obs} = plugins.plugins['templater-obsidian'].templater.current_functions_object.obsidian

const current_file_path = dv.currentFilePath

workspace.onLayoutReady(main.bind(this))

function main() {
  (genMain)()
  async function genMain() {
    const dir_path = getNearestRootPathFromFilePath(current_file_path);
    const vfs = await genRecursiveList(dir_path)

    const fms = vfs.map((vf) => {
      return metadataCache.getFileCache(vf).frontmatter
    })


    genRenderList(list)
  }
}

function getNearestRootPathFromFilePath(current_file_path) {
  const folder_path = vault
    .getAbstractFileByPath(
      current_file_path
    ).parent?.path;

  if (!folder_path) return;
  return folder_path
}
async function genRecursiveList(path) {
  const vfs = []
  if (!path) return vfs;
  
  const tfolder = vault.getAbstractFileByPath(path)
  const reclist = await obs.Vault.recurseChildren(tfolder, cb)
  return vfs;

  function cb(vf) {
    vfs.push(vf);
  }
}
```

---

# ---Transient
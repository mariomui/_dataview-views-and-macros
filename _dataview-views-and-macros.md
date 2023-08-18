---
tag: _meta/folder_page 
alias: _dataview-views-and-macros
---
# -

- [ ] Add a versioning comment for every [[Partial-dataview]] housed in this folder
* #_todo/to-muse/on-meta-filename-syncing
  * [ ] Alias or aliases do not change when folder change.
  * [ ] Folder pages change with the folder renaming 

* #_todo/to-muse/on-naming-a-meta-note
  - [ ] All tag pages and folder pages aside from the top level  A_ B_ C_ D_...Z_ will be prefixed with an underscore. 

## About

This [[folder-page]] describes the files that should be housed inside this folder. It describes the types of files that should be classified as a [[Partial-dataview]] , or a [[macro]]. Mostly, the files are a sandbox for pages with views powered by [[Dataview-plugin,ad-finem-ObsidianMD]]. 

Any subfolders will also be described here.

Static pages are post processed and will be shared. 

The code stays off the primary note but the transcluded preview is available. This works well for dynamically generate hotkey pages.


> [!note]
> This folder is git synced

# =

Normal dataviewjs way of listing files in a folder.
*Hampered cuz dv only lists files and not folders.*

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

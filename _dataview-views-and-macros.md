---
tag: _meta/folder_page 
alias: _dataview-views-and-macros
---
# -

- [ ] Do not Frontcase the affixes in the note titles because these are theoretically concept handles.
  - Note-Taking could very easily be a note named Note-taking so consistency is much better.
- [ ] Replace all mentions of Note-Taking with MNote Taking System and create a note pertaining to that.
  - convert all files named `[[Note-taking-system,cf.-Mario-Mui,]]` into [[,aka-Note-taking-system,cf.-Mario-Mui]]
- [ ] Add a versioning comment for every [[Partial-dataview,vis-Noteshippo]] housed in this folder
* #_todo/to-muse/on-noteshippo
  * [ ] Alias or aliases do not change when folder change.
  * [ ] Folder pages change with the folder renaming 

* #_todo/to-muse/on-noteshippo/regarding-meta-note-naming-rules
  - [ ] All tag pages and folder pages aside from the top level  A_ B_ C_ D_...Z_ will be prefixed with an underscore. 
- [ ] Insert media detailing usage of Partial dataview into each [[Partial-dataview,vis-Noteshippo]]'s [[private-header,b.t.-Anchor-heading-api]][[,aka-private-H1,ad-finem-Note-Taking]]

## About

This [[folder-page,vis-Noteshippo,]] describes the files that should be housed inside this folder. It describes the types of files that should be classified as a [[Partial-dataview,vis-Noteshippo]] , or a [[macro]]. Mostly, the files are a sandbox for pages with views powered by [[Dataview-plugin,ad-finem-ObsidianMD]]. 

Any subfolders will also be described here.

Static pages are post processed and will be shared. 

The code stays off the primary note but the transcluded preview is available. This works well for dynamically generate hotkey pages.


> [!note]
> This folder is git synced

# =

%% Begin Waypoint %%
- **[[_experiments]]**
- [[~deprecated_button-for-ytranscript]]
- [[~interim_view-for-recent-reference-link-to-note-title-transform]]
- [[~view-for-big5-character-radar-chart]]
- [[~view-for-broken-links]]
- [[~view-for-calculating-reading-time]]
- [[~view-for-creating-absolute-links-for-citations]]
- [[~view-for-custom-hot-keys-for-code-block-from-selection-plugin]]
- [[~view-for-custom-hot-keys-for-obsidian-timestamp-plugin-MUID-701]]
- [[~view-for-exact-tag-file-listing]]
- [[~view-for-inactive-note-collection]]
- [[~view-for-inactive-notes-count]]
- [[~view-for-listing-highlighted-text-in-current-file]]
- [[~view-for-local-tasks-using-a-progress-bar-MUID-698]]
- [[~view-for-lotterizing-note-work-slate-TCODEID-1]]
- [[~view-for-oldest-files-in-system-TCODEID-3]]
- [[~view-for-referencing-current-jumpid]]
- [[~view-for-top-active-notes-using-weighted-score]]
- [[~view-for-top-inactive-notes-using-io-index]]
- [[~view-for-unused-MUIDs]]
- [[~viewfn-for-creating-absolute-links-for-citations]]
- [[~viewfn-for-pending-notes]]
- [[~viewfn-for-shadow-note-search-by-file-name-TCODEID-4]]
- **projects**

%% End Waypoint %%

- [ ] Extract ⤵ to learnings from listing files
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

# ---Transient
Normal dataviewjs way of listing files in a folder.
*Hampered cuz dv only lists files and not folders.*
- [ ] Extract this codelet  into a partial dataview named "interim dataviewjs waypoint version" #_todo/to-extract/on-a-codelet/regarding-waypointing-notes-in-a-folder 
  - See [[experiment_list-a-folders-contents-and-add-to-dom-TCODEID-5]] for details
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

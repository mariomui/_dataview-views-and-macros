---
TEMPLATE_VERSION: v1.0.5_note-refactor-template
MUID: MUID-1401
CREATION_DATE: 2023-08-04 
tag: _wip 
UMID: 
VERSION: v1.0.1-note-search-TCODEID-4

---
# -

```dataview
TASK 
WHERE file.name = this.file.name
AND !completed
```

## About

This is a [[note-template]] designed to template extracted content.
The code used here might be more suitable for a plugin.

After trying [[Obsidian-text-expander-plugin,ad-finem-ObsidianMD]], the ability to create a query without resulting to pulling it from the search bar is missing.
* Cons
  * Hard codes the file names exposing vulnerableness to stale data.
  * Mutates the file note (displeases me greatly)

### Commit Log

v1.0.1 is somewhere floating hard coded out there in the [[notesphere]]

### Planned feature

- [ ] Add url parsing for the querying dynamic params

### Reference

* †

# =

## = TITLE

*`= this.file.name`*

* [ ] #_todo/priority-w/to-extract/on-a-dataviewjs-codelet/regarding-better-shadow-note-search Extract code below ⤵
  * Add in code for transclusion params

- [ ] Is there a way to export the function extractParams as dataview module using viewState? #_todo/a-muse/on-an-obsidianmd-api-codelet/regarding-global-code-sharing

```dataviewjs
const {
  workspace, metadataCache, vault
} = this.app;

workspace.onLayoutReady(main.call(this))

function main() {
  return function () {
    const providing_path = dv
      .currentFilePath;
    const argMap = extractParams(
      providing_path,
      workspace.getActiveFile()
    )
  
    const pvf = vault
      .getAbstractFileByPath(
        providing_path
      );
    const {
      frontmatter: fm
    } = metadataCache
      .getFileCache(pvf);
    const VERSION = fm?.VERSION || "NA";
    const num = argMap?.n ?? 5;
    
    dv.paragraph(num);
    
    const data = transformData();
    render([VERSION].concat(data));
  }
  function transformData() {
    const uls = metadataCache
      .unresolvedLinks;
    const term = "wip-"
    const keys = [];
    const keysMap = {}
    for (const ulkey in uls) {
      for (const key in uls[ulkey]) {
        if (
          key.startsWith(term) &&
         !keysMap?.[key]
       ) {
          keysMap[key] = true;
          keys.push(key)
        }
      }
    }
    return keys;
  }
  function render(strs) {
    dv.list(strs)
  }
}
function manuParams() {
  return { n: 5 }
}
function extractParams(
  current_filepath, 
  vf
) {
  // v1.01-extractParams

  const embeds = getEmbedsFromVf(vf)
  const {name} = vault.adapter.path
    .parse(current_filepath);
  const embed = extractTargetEmbed(
    name, 
    embeds
  );
  const displayText = embed?.displayText;
  
  console.log(displayText)
  if (!displayText) {
    return manuParams()
  };
  
  const argmap = parseStringToMap(
    displayText
  );
  const limit = argmap["-n"]
  // /end params extraction 

  if (limit) {
    return {
      n: Number(limit)
    };
  }
  return manuParams();

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
    const parts = str
      .trim().split(/\s+/); 
    const map = {}; 
    for (
      let i = 0;
      i < parts.length;
      i += 2
    ) {
      const key = parts[i]; 
      const value = parts[i + 1]; 
      if (key && value) { 
        map[key] = value; 
      } 
    }
    return map; 
  }
}
```

---

# ---Transient Sandbox

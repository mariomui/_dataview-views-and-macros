---
DOC_VERSION: v1.0.5_note-refactor-template
MUID: MUID-1401
CREATION_DATE: 2023-08-04
tags:
  - _wip
UMID: 
VERSION: v1.0.3-shadow-note-search-TCODEID-4
---

# -

```dataview
TASK 
WHERE file.name = this.file.name
AND !completed
```

## About

This is a [[Partial-dataview,vis-Noteshippo,]] designed to search [[,aka-shadow-note]]s for a specific term using regex.

* Rough structure is ⤵
```ts
interface TCODEID4 {
    search_term: "",
    regex_flag: "g"
}
TCODEID4: (queryString: TCODEID4) => void
```

![[~viewfn-for-shadow-note-search-by-file-name,nb.-MUID-1401#Usage Guide|olk]]

The code used here might be more suitable for a plugin.

After trying [[Obsidian-text-expander-plugin,b.t.-ObsidianMD]], the ability to create a query without resulting to pulling it from the search bar is missing.
* Cons
  * Hard codes the file names exposing vulnerableness to stale data.
  * Mutates the file note (displeases me greatly)

### Commit Log

v1.0.1 is somewhere floating hard coded out there in the [[notesphere,cf.-Kanzi,]]

### Planned feature

- [x] Add url parsing for the querying dynamic params

### Reference

* †

# =

`= this.file.name`

![[~viewfn-for-shadow-note-search-by-file-name,nb.-MUID-1401#Usage Guide|olk]]

```dataviewjs
const {default: obs} = this.app.plugins.plugins['templater-obsidian'].templater.current_functions_object.obsidian;
const {
  workspace, metadataCache, vault
} = this.app;
const {adapter} = vault;

const getCFP = () => this.currentFilePath;

workspace.onLayoutReady(main.call(this))

function main() {
  return function () {
    const providing_path = getCFP()
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
    
    try {

      const data = transformData(argMap);
      renderList(
        `==${VERSION}==`,
        data
      );
    } catch(err) {
      console.log({err})
    }
  }
  function transformData(paramMap) {
    const {
      search_term,
      regex_flag
    } = paramMap
    const uls = metadataCache
      .unresolvedLinks;
    const isInvalid = ["", null, undefined].includes(search_term);
    if (isInvalid) return [];
    try {
      const regex = new RegExp(search_term, regex_flag);
      return queryShadowNotes(
        uls,
        regex
      )
    } catch(err) {
      return new Error(JSON.stringify(err))
    }

  }
  function queryShadowNotes(
    uls = [],
    regex
  ) {
    const shadowNotes = [];
    const uniq = {}

    for (const file_name in uls) {
      for (const shadow_note in uls[file_name]) {
        
        if ( regex.exec(shadow_note) && !uniq?.[shadow_note] ) {
          uniq[shadow_note] = true;
          shadowNotes.push(shadow_note);
        }
      }
    }
    return shadowNotes;
  }
  function renderList(
    header,
    datums
  ) {
    const md = dv
      .markdownList(
        [header,...datums]
      );
    const _md = md
      .split('\n').map((m,i) => {
        if (i > 0) {
          return "  " + m;
        }
        return m
      })
    dv.paragraph(_md.join('\n'))
  }
}

// # Extraction Code vX.X.X. 
// impossible to version when inside another codelet.
function manuParams() {
  return {
    search_term: "",
    regex_flag: "g"
  }
}
function extractParams(
  current_filepath, 
  vf
) {

  const embeds = getEmbedsFromVf(vf)
  const {name} = vault.adapter.path
    .parse(current_filepath);
  const embed = extractTargetEmbed(
    name, 
    embeds
  );
  const displayText = embed?.displayText;
  
  console.log({displayText})
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
        parsed = adapter
          ?.url
          ?.parse(str, true);
      } catch (err) {
        console.log({err})
        return {
          err
        }   
      }
      return parsed?.query || {};
    }
    return {};
  }
}
```

---
# ---Transient Local Resources

## Usage Guide

* API is ⤵
  * `[[` `= this.file.name` `|?search_term=`**note**`&regex_flag=gm`]]`

# ---Transient Sandbox

* [x] #_todo/42-priority-high--/to-extract/on-a-dataviewjs-codelet/regarding-better-shadow-note-search Extract code below ⤵
  * Add in code for transclusion params

- [ ] Is there a way to export the function extractParams as dataview module using viewState? #_todo/to-muse/on-obsidianmd-app/regarding-a-codelet/regarding-global-code-sharing
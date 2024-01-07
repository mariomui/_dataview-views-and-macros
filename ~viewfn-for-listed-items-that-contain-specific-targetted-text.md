---
CREATION_DATE: 2023-12-22
DOC_VERSION: v0.0.2
MUID: MUID-1925
TEMPLATE_VERSION: v1.0.5_default-template
UMID: "[[UMID-305a25a5-a286-4d38-8e86-19019c23e63c]]"
alias: 
tags:
  - _wip
---

# -

![[~view-for-local-tasks-using-a-progress-bar-MUID-698#=|olk]]

```dataview
TASK WHERE file.name = this.file.name AND !completed
```
```dataview
TASK WHERE file.name = this.file.name AND completed
```

## About

* Hover over me and snatch the output of note title algorithm:[[~view-for-recent-reference-link-to-note-title-transform]]

### Reference

![[~view-for-referencing-current-jumpid#=|nlk]]

* †

# =



> [!info] Embed `![viewfn...|?search_term=...]` into your note.
> ℹ Replace `...` with the list item text you want to target

*`= this.file.name`*


```dataviewjs
const { plugins, workspace, vault, metadataCache, fileManager } =
  this.app;

const { default: obs } =
  plugins.plugins["templater-obsidian"].templater
    .current_functions_object.obsidian;

const getCurrentfilepath = () => this.currentFilePath;

workspace.onLayoutReady(bootstrap.bind(this));
function bootstrap() {

    // business domain
    const providing_path = getCurrentfilepath();
    const argMap = extractParams(
      providing_path,
      workspace.getActiveFile(),
    );
    
    const pvf = vault.getAbstractFileByPath(providing_path);
    const { frontmatter: fm } = metadataCache.getFileCache(pvf);

    const dvCurrent = dv.page(this.app.workspace.getActiveFile()?.path) || dv.current()
    const childrenTextTuples = dvCurrent.file.lists.values.map(({text, children}) => ({children, text}))

    const datums = [];
    const {search_term: target} = argMap;
    if (target === "") {
      renderText("* > [!warning] Param search is empty")
      return console.error("params are empty")
    }
    walk(childrenTextTuples, datums, 0, {target})

    workspace.onLayoutReady(() => {
    
      renderText("* ! Render list items that are tagged with " + target);
      renderList(datums)
    })
}


function renderText(text) {
  dv.paragraph(text)
}
function renderList(datums) {
  dv.list(datums)
}
// walk an array
function manuWalkFig() {
 return ({target: `## ! `})
} 
function walk(list, c, level, fig = manuWalkFig()) {
  if (level > 4) {
    return;
  }
  const config = {
    ...(manuWalkFig()),
    ...fig
  }
  if (config.target === "") return;
  for(let li of list) {
    const d = [];
    if ((li.text.indexOf(`${config.target}`) > -1)) {
      d.push(li.text)
      if (Array.isArray(li.children) && li.children.length) {
        walk(li.children, d, level + 1, config)
      }

    }
    if (level > 0) {
      d.push(li.text);
    }
    if (d.length) {
      c.push(d)
    }
  }
}

// # Extraction Code See MUID-1634 for versioning 
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
        parsed = vault.adapter
          ?.url
          ?.parse(str, true);
      } catch (err) {

        return {
          err
        }
      }
      const parsedQuery = parsed?.query || {};
      const search_term = decodeURIComponent(parsed.href.match(/(?<=\=)[^&]+/).at(0) || "");
      
      
      if (parsed.href.includes("#")) {
        return {
          ...parsedQuery,
          search_term
        }
      }
      return parsedQuery
    }
    return {};
  }
}
```


# ---Transient Commit Log

* v0.0.1 
  * Solves the problem of non uri compatitble letters liek # and !. These cannot be fed into the queryString so the solution currently is to create an if statement to manually parse them out. 
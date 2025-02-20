---
CREATION_DATE: 2023-12-22
DOC_VERSION: v0.0.5
MUID: MUID-1925
TEMPLATE_VERSION: 0.0.0
UMID: "[[UMID-305a25a5-a286-4d38-8e86-19019c23e63c]]"
aliases: 
tags:
  - _wip
TEMPLATE_SOURCE: "[[10--blank-no-api-template]]"
---

# -

![[~view-for-local-tasks-using-a-progress-bar,nb.-MUID-698#=|olk]]

```dataview
TASK WHERE file.name = this.file.name AND !completed
```
```dataview
TASK WHERE file.name = this.file.name AND completed
```

## About



# =


> [!info] hover for [[~viewfn-for-listed-items-that-contain-specific-targetted-text,nb.-MUID-1925#LR--instruction--how to use|how to use]] 

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

    const DOC_VERSION = fm?.DOC_VERSION || "";
    
    const dvCurrent = dv.page(this.app.workspace.getActiveFile()?.path) || dv.current()
    const childrenTextTuples = dvCurrent.file.lists.values.map(({text, children}) => ({children, text}))

    const datums = [];
    const {search_term: target} = argMap;

    // escape
    if (target === "") {
      renderText(`* > ${fm.MUID} [!warning] Param search is empty ${formatDocv(DOC_VERSION)}`)
      return console.error("params are empty")
    }

    // search and extract
    walk(childrenTextTuples, datums, 0, {target})

    // ui
    workspace.onLayoutReady(() => {
    
      renderText(`* ! ${fm.MUID} Render list items that are tagged with ${target} ${formatDocv(DOC_VERSION)}`);
      renderList(datums)
    })
}


function formatDocv(doc_version) {
  return `••• **docv:** ${doc_version}`
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

# ---Transient Local Resources

## LR--instruction--how to use

> [!info] Embed `![viewfn...|?search_term=...]` into your note.
> ℹ Replace `...` with the list item text you want to target

*`= this.file.name`*

# ---Transient Commit Log

- v0.0.5 *2024-06-21*
  - Use nb instead of cf because cf means to refer to an expert, and nb is more of a catchall. The addition of MUID in the title is differentiate all the hard copy instances of the same code.
    - having hard copies of these instances means that this code should be a plugin.
  - Add doc version ot ui explainer text
  - Bold docv instead because of possible encapsulation areas when targgeted text contains an openended asterisk
    - 🔎🐛  search_text= * 
      - This causes the italicization parser to parse the closes text and not the docv.
- v0.0.4 *2024-06-06*
  - Add frontmatter MUID to codelet render
* v0.0.3 *2024-05-28*
  * create hoverable instructions
  * [[transient-commit-log-endpoint,bt.-Noteshippo-heading-api,]]
* v0.0.1 
  * Solves the problem of non uri compatitble letters liek # and !. These cannot be fed into the queryString so the solution currently is to create an if statement to manually parse them out. 
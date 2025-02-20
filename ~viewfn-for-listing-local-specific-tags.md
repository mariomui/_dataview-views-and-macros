---
TEMPLATE_VERSION: v1.0.7_note-refactor-template
MUID: MUID-1922
CREATION_DATE: 2023-12-22
tags: _wip 
DOC_VERSION: v0.0.0
UMID: "[[UMID-305a25a5-a286-4d38-8e86-19019c23e63c]]"
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


There is a heavy reliance to dv.file 

one day i have to excise this so that I can grab it from the indexedDb. 
The puprose of this partial is to dynamically list the list items using a tag specified in the [[custom-transclusion-parameters,cf.-Kanzi,vis-ObisidianMD-app,]]
### Reference

> [!info] [[~view-for-referencing-current-jumpid]]

* †

# =

> [!warning] This only lists current tags in the note and doesn't search globally

> [!info] Embed `![viewfn...|?search_term=_cv...]` into your note.
> ℹ Replace `#cv` with the tag you want to list

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
    const {search_term: tagTarget} = argMap;

    walk(childrenTextTuples, datums, 0, {tagTarget})
    
    
    renderText(`* ! Render list items that are tagged with ${tagTarget}`);
    renderList(datums)
}

function withTagTarget(tagTarget, cb) {
  cb(tagTarget)
}

function renderText(text) {
  dv.paragraph(text)
}
function renderList(datums) {
  dv.list(datums)
}
// walk an array
function manuWalkFig() {
 return ({tagTarget: `_cv/`})
} 
function walk(list, c, level, fig = manuWalkFig()) {
  const config = {
    ...(manuWalkFig()),
    ...fig
  }
  for(let li of list) {
    const d = [];
    if ((li.text.indexOf(`#${config.tagTarget}`) > -1) || (level > 0)) {
      d.push(li.text)
      if (Array.isArray(li.children) && li.children.length) {
        walk(li.children, d, level + 1, config)
      }
      if (d.length) {
        c.push(d)
      }
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
      return parsed?.query || {};
    }
    return {};
  }
}
```

---

# ---Transient
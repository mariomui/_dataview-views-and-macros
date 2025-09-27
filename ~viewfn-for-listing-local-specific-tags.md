---
TEMPLATE_VERSION: v1.0.7_note-refactor-template
MUID: MUID-1922
CREATION_DATE: 2023-12-22
tags: _misc/_wip
DOC_VERSION: v0.0.0
UMID: "[[π-design-custom-transclusion-parameters-codelets]]"
---
# -
## 00-Meta

> [!info]+ Progress Bar
> > ![[~view-for-local-tasks-using-a-progress-bar,nb.-MUID-698#=|olk]]
> ```dataview
> task where file.name = this.file.name and !completed
> ```
> > 
> ```dataview
> task where file.name = this.file.name and completed
> ```
### 10÷About


- There is a heavy reliance to dv.file 
- [ ] Consider replacing reliance to dv file with indexedDb. ➕ 2025-05-05 #_todo/to-muse 
- The purpose of this partial is to dynamically list the list items using a tag specified in the [[custom-transclusion-parameters,bt.-Noteshippo-terminology,]]
### 11÷Reference

> [!info] [[~view-for-referencing-current-jumpid]]

* †

## 20-Inlink

> [!abstract]- %%  %% Automated List of Reference Inlinks (v0.0.5)
> * ℹ Commit/design logs are located in this [[,aka-MUID-150|experiment note]]. 
> > `= join( map( sort( map( filter(this.file.inlinks, (link) => meta(link).path != this.file.path), (x) => [ split(meta(x).path, "/")[length(split(meta(x).path, "/")) - 1], x ] ) ), (b) => "• " + choice( length(b[0]) > 28, link( b[1], truncate( regexreplace(b[0], "(-of|of|the|-the|-for|-that|https-|ee)", ""), length( regexreplace(b[0], "(-of|of|the|-the|-for|-that|https-|ee)", "") ) * 0.75 ) ), link(b[1], regexreplace(b[0], "\.md$", "")) ) ), "<br>" )`


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
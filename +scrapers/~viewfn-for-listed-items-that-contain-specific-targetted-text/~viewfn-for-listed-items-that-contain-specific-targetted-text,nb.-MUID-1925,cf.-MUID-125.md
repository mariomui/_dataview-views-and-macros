---
CREATION_DATE: "2023-12-22"
DOC_VERSION: v0.0.7
MUID: MUID-1925
PROJECT_PARENT: "[[π-design-custom-transclusion-parameters-codelets]]"
TEMPLATE_SOURCE: "[[10--blank-no-api-template]]"
TEMPLATE_VERSION: 0.0.0
aliases: 
cssclasses:
  - dvjs-no-overflow
tags:
  - _misc/_wip
---

# -

## 00-Meta

> [!info]- Progress Bar v0.0.3
> > ![[~view-for-local-tasks-using-a-progress-bar,nb.-MUID-698#=|olk]]
> ```dataview
> task where file.name = this.file.name and !completed
> ```
> >
> ```dataview
> task where file.name = this.file.name and completed
> ```

### 10÷About

## 20-Inlink

> [!abstract]- %%  %% Automated List of Reference Inlinks (v0.0.5)
> * ℹ Commit/design logs are located in this [[,aka-MUID-150|experiment note]].
> > `= join( map( sort( map( filter(this.file.inlinks, (link) => meta(link).path != this.file.path), (x) => [ split(meta(x).path, "/")[length(split(meta(x).path, "/")) - 1], x ] ) ), (b) => "• " + choice( length(b[0]) > 28, link( b[1], truncate( regexreplace(b[0], "(-of|of|the|-the|-for|-that|https-|ee)", ""), length( regexreplace(b[0], "(-of|of|the|-the|-for|-that|https-|ee)", "") ) * 0.75 ) ), link(b[1], regexreplace(b[0], "\.md$", "")) ) ), "<br>" )`

# =

> [!info] hover for [[~viewfn-for-listed-items-that-contain-specific-targetted-text,nb.-MUID-1925,cf.-MUID-125#LR--instruction--how to use|how to use]]; If parsing for &, please write %26
```dataviewjs
const { plugins, workspace, vault, metadataCache, fileManager } =
  this.app;

const { default: obs } =
  plugins.plugins["templater-obsidian"].templater
    .current_functions_object.obsidian;

const getCurrentfilepath = () => this.currentFilePath;

workspace.onLayoutReady(bootstrap.bind(this));
function bootstrap() {

    const genRenderCompactMarkdown = createGenRenderCompactMarkdown.call(this, {obs});
    // genRenderValue and genderListTuple are separated for no reason.
    const genRenderValue = createGenRenderValue.call(this, {genRenderCompactMarkdown})
    // i need access to genRenderValue to fiddle with the dials.
    const genRenderListTuple = createGenRenderListTuple.call(this,{genRenderValue})
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
    workspace.onLayoutReady(async () => {
    
      renderText(`* ! ${fm.MUID} Render list items that are tagged with "${target}" ${formatDocv(DOC_VERSION)}. ${datums.length} Root items.`);
      
      console.log({datums,childrenTextTuples})
      // renderList(datums)
      // on future versions of genRenderList, if we can return the fragment, then we
      // could have very efficient appending onto the dom.

      await genRenderListTuple(
        cheapFlatten(datums)
      )
    })
}


// ## Custom DVJS Rendering Replacements
function cheapFlatten(datums) {
  const normedDatums = [];
  for (let datum of datums) {
    if (datum.length === 1) {
      normedDatums.push(...datum)
    } else {
      normedDatums.push(datum.first(), ...datum.slice(1))
    }
  }
  return normedDatums;
  /* walked datums looks like this; I need it flattened.
  const example = [ 
    ["label"],
    [ "nested",["uno"] ],
    [ "labeldos", [ "uno" ] ]
  ]
  it's a conditional flatten. very weird. But it's fast and it nests and it's cheap especially when i'm only nesting 2 layers.
  const desired = [
    "unnested_root1",
    "nested_root1", 
    [nested_root1_item1]
    nested_root2, 
    [nested_root2_item1]
  ]
  */
}

function createGenRenderListTuple(fig) {
  const {genRenderValue} = fig;
  /** 
   * @param {Array<[string, string[]]>} listTuple
   * @param {DocumentFragment} container
   * @param {boolean} isExpandList
   * @param {string} context // context means is it a list or not.
   * @param {number} depth // how far it goes.
   * @param {Component} component // has all the obs goodies needed for markdownRender
  **/ 

  return (async function _genRenderListTuple(listTuple, type = "list") {
    const $frag = document.createDocumentFragment();
    await genRenderValue.call(this,
      listTuple,
      $frag,
      true,
      type,
      -1,
      this.component
    )
    return this.container.append($frag)
  }).bind(this);
}
// obs is the only toolbox that is global
function createGenRenderCompactMarkdown(fig = {}) {
  if (!fig.obs) throw new Error("no obs!")
  return async function _genRenderCompactMarkdown(
    markdown, // string,
    container, // HTMLElement,
    sourcePath, // string,
    component // Component
  ) {
    let subcontainer = container.createSpan();
    await obs.MarkdownRenderer.renderMarkdown(
      markdown, subcontainer, sourcePath, component
    );
  }
}
// This is a recursive UI rendering fuction for creating lists.
function createGenRenderValue(fig) {
  const manuLiCss = (config = {}) => ({cls: "", ...config })
  const outsideFig = {
    isArray: Array.isArray,
  };
  const {genRenderCompactMarkdown, isArray} = {
    ...fig, 
    ...outsideFig,
  };
  return async function _genRenderValue(
    val,
    container,
    isExpandList,
    context,
    depth,
    component
  ) {
    if (depth > 2) return;
    if (typeof val === "string") {
      await genRenderCompactMarkdown(
        val,
        container,
        "",
        component
      )
      return;
    }

    if (isArray(val)) {

      const clss = context === "list" ?
        "dataview-result-list-ul" :
        "dataview-result-list-root-ul";
      const cls = {
        cls: [
          "dataview",
          "dataview-ul",
          clss
        ]
      }
      let list = container.createEl(
        "ul",
        cls
      );
      for (let child of val) {
        const rootStyle = {
          attr: {
            // style: "list-style-type: disc",
          }
        };
        const style = isArray(child) ? rootStyle : rootStyle;
        let li = list.createEl("li", manuLiCss(style));
        const context = isArray(child) ? "list" : "";
        await _genRenderValue(
          child, //value
          li,  // container
          isExpandList, // is rest of list
          context,
          depth + 1, // escape hatch
          component
        );
      }
    }
  }
}
// # Utils

function formatDocv(doc_version) {
  return `••• **docv:** ${doc_version}`
}

// ## Utils::UI
function renderText(text) {
  dv.paragraph(text)
}
function renderList(datums) {
  dv.list(datums)
}

// ## Utils::Scraping
// walk an array
function manuWalkFig() {
 return ({target: `## ! `})
} 
function walk(list, c, level, fig = manuWalkFig()) {
  if (level > 3) {
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
      // console.log({parsedQuery,parsed})
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

- v0.0.7 
	- testest out whether or not the recursive walk being 4 levels is a bit too much. (use 3 as the thresh hold for escape)
* v0.0.6 *2025-05-09*
  * Updated code based on [[≈-list-a-folders-contents-and-add-to-dom,uti.-javascript,nb.-MUID-125]]. 
    * Updated the genRenderListTuple function but its still janky. Limitations have not been tested. However, it is super fast. There needs to be some refactoring but it has replaced dv.list in terms of very small lists. 
      * [ ] Create a project note to document the changes for MUID-125 and MUID-1925 #_todo/to-document/upon-obsidianmd-codelets/regarding-improving-list-item-rendering ➕ 2025-05-09
  * Remove bullet point from embed. Recently bullets now have a margin of 35px that gets added in an embed. It's very annoying.
* v0.0.5 *2024-06-21*
  * Use nb instead of cf because cf means to refer to an expert, and nb is more of a catchall. The addition of MUID in the title is differentiate all the hard copy instances of the same code.
    * having hard copies of these instances means that this code should be a plugin.
  * Add doc version ot ui explainer text
  * Bold docv instead because of possible encapsulation areas when targgeted text contains an openended asterisk
    * 🔎🐛  search_text= *
      * This causes the italicization parser to parse the closes text and not the docv.
* v0.0.4 *2024-06-06*
  * Add frontmatter MUID to codelet render
* v0.0.3 *2024-05-28*
  * create hoverable instructions
  * [[transient-commit-log-endpoint,bt.-Noteshippo-heading-api,]]
* v0.0.1
  * Solves the problem of non uri compatitble letters liek # and !. These cannot be fed into the queryString so the solution currently is to create an if statement to manually parse them out
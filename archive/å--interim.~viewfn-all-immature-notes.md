---
TEMPLATE_VERSION: v1.0.10
TEMPLATE_SOURCE: "[[10--nascent-spec-template]]"
tags:
  - _misc/_wip
MUID: 
CREATION_DATE: 2025-01-30
nextPageIndex: 0
paginateByCnt: 10
DOC_VERSION: v.0.0.1
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

### 10-About

- ! Do the todos but the code here itself needs to be extracted to [[common-utility-code,vis-Dataviewjs,etc]]

* [x] Add custom transclusion to this partial view so that it can input the folder dynamically #_todo/to-enhance ➕ 2025-01-30 ✅ 2025-04-03

- [ ] Change `todo-add` to `todo-add-entry` #_todo/to-fix/upon-noteshippo-tag-nomenclature ➕ 2025-04-03
	* [x] what does `#_todo/to-add`  mean? #_todo/70-done--/to-muse ✅ 2025-04-03
		* It means to add an entry into a [[,aka-library-specced-note]]
		* It has some overlap with other [[,aka-cv]]ed verbs: muse process and document. 
			* [ ] Document todo maturity cycle [[ideation-lifecycle,uti.-Noteshippo-task-subsystem,]] ➕ 2025-04-03 #_todo/to-document/upon-noteshippo/regarding-ideation-process
				* [[task-system,bt.-Noteshippo-subsystem,]]
				* [[≠--todo-system,bt.-Noteshippo-subsystem,]]
				* Document is when i dont have a note for it (therefore, the note link is usually stubbed in the same task). 
				* Process is to think about it on an extant document. 
				* Muse is when I want to think about it but I don't have a need to create a note. 
				* Add is specifically for library entry notes.
					* 🤔 consider renaming `#_todo/to-add`  to `#todo/to-add-entry`
- [ ] Remove the term vernacular and use nomenclature instead #_todo/80-longterm--/to-normalize ➕ 2025-04-03
	* [[obsolete-vernacular-symbol,uti.-≠,bt.-Noteshippo-pan-level-flag,]]

## 20-Inlink

> [!abstract]- %%  %% Automated List of Reference Inlinks (v0.0.5)
> * ℹ Commit/design logs are located in this [[,aka-MUID-150|experiment note]].
> > `= join( map( sort( map( filter(this.file.inlinks, (link) => meta(link).path != this.file.path), (x) => [ split(meta(x).path, "/")[length(split(meta(x).path, "/")) - 1], x ] ) ), (b) => "• " + choice( length(b[0]) > 28, link( b[1], truncate( regexreplace(b[0], "(-of|of|the|-the|-for|-that|https-|ee)", ""), length( regexreplace(b[0], "(-of|of|the|-the|-for|-that|https-|ee)", "") ) * 0.75 ) ), link(b[1], regexreplace(b[0], "\.md$", "")) ) ), "<br>" )`

# =

**base_filepath-v0.0.6**: `= choice( contains(this.file.folder, this.file.name), link(this.file.path), join(["*",this.file.path,"*"], ""))` doc-`= this.DOC_VERSION` / ids: `= this.MUID`,PP:`= this.PROJECT_PARENT` / lcsh: `= link(this.heading)`

```dataviewjs

const { workspace, vault, metadataCache, fileManager} = this.app;
const {default: obs} = this.app.plugins.plugins['templater-obsidian'].templater.current_functions_object.obsidian

const getCurrentfilepath = () => this.currentFilePath;

const paginationFieldName = "nextPageIndex"
const LIMIT = 5; // 5 citums , every flash article must have at least
const WIP_TAGNAME = "_misc/_wip";
const SEARCHED_FOLDER = "A_sources/_flash-articles"


async function genDiffManager(file_path, tag) {
  const link = metadataCache.getFirstLinkpathDest(file_path,"")
  console.log({link,file_path})
  const vf = vault.getAbstractFileByPath(file_path);
 
  // logic
  const pathCacheTuple = await genReadFilesInAbstractFolder(vf)
  const lteMarkdownLinks = pathCacheTuple.filter(_filterTuplesByCitumCountPredicate)
  const propersCnt = pathCacheTuple.length - lteMarkdownLinks.length;
 

 
  async function genReadFilesInAbstractFolder(
   abstractFolder
  ) {
   return await Promise.all(
     abstractFolder.children.map(
       (vf) => [vf.basename, metadataCache
       .getCache(vf.path)]
     )
   )
  }
 
  function _filterTuplesByCitumCountPredicate(tuples) {
    const [path, cache] = tuples;
    const hasWip = cache?.frontmatter?.tags?.includes(tag)

    if (!hasWip || !cache?.headings) {
     return false;
    }
    const cnt = cache.headings.reduce((chain, item) => {
     if (!item?.heading) return false;
     if (item.heading.startsWith("LC--")) {
       return chain += 1
     }
     return chain;
    }, 0)

    return cnt <= LIMIT;
  }

  return [pathCacheTuple, lteMarkdownLinks, propersCnt]
}

workspace.onLayoutReady(async () => {

  
  let initialIndex = 0;
  let paginationCnt = 15;
  const metadataEditor = workspace.getActiveFileView().metadataEditor
  const serializeYaml = metadataEditor.serialize;
  const synchronizeYaml = metadataEditor.synchronize;

  const fm = metadataEditor.serialize();

 function manuParams() {
   return {
     sf: SEARCHED_FOLDER,
     tag: WIP_TAGNAME
   }
 }

 function manuFig() {
   return {
     vault,
     getEmbedsFromVf,
     extractTargetEmbed,
     manuParams,
     parseStringToMap,
   }
 }
  // business domain
  const providing_path = getCurrentfilepath();
  const argMap = extractParams(
    providing_path,
    workspace.getActiveFile(),
    manuFig()
  );
	// get out arly if no values detected
	if (Object.values(argmap).every((x) => !x)) return;
  
  const [
    pathCacheTuple, lteMarkdownLinks, propersCnt
  ] = await genDiffManager(argMap.sf, argMap.tag);

  // create config data drivers
  const mainCfg = {
    initialIndex: fm?.[paginationFieldName] || 0,
    items: lteMarkdownLinks.map((lte) => `[[${lte.first()}]]`),
    paginationCnt: Number(fm?.paginateByCnt) || paginationCnt,
    paginationFieldName,
    renderButton: renderButton.bind(this),
    serializeYaml: serializeYaml.bind(metadataEditor),
    synchronizeYaml: synchronizeYaml.bind(metadataEditor),
    lteMarkdownLinks, 
    propersCnt
  }

 // Running logic a little earlier.
  main.call(this, mainCfg)


})


// bootstrap
function main(cfg) {
  genMain.call(this, cfg)
}

async function genMain(cfg) {
  // ## provider
  const providing_path = getCurrentfilepath();
  const pvf = vault.getAbstractFileByPath(providing_path);

  // ## consumer
  const cvf = workspace.getActiveFile();

  // logic helpers


  // ## Throw on Setup Error
  if (!renderButton) throw new Error("rendering function missing");

  try {
  // ## MAIN RENDER
    myRender.call(this, cfg);
  } catch(e) {
    throw new Error(JSON.stringify({e, etext: "error problems"}))
  }
  renderDetails(lteMarkdownLinks, propersCnt, LIMIT)
}

// Render utility for pagination
function myRender(cfg) {
  const itemsLen = cfg.items?.length ?? 0;
  const CMD = {
    NEXT: "next",
    PREV: "prev"
  }

  cfg.renderButton(
    CMD.PREV, 
    _onRender.bind(this)
  )
  cfg.renderButton(
    CMD.NEXT, 
    _onRender.bind(this)
  )

  _onRender.call(this, CMD.SAME)


  function _onRender(cmd) {
    const fm = cfg.serializeYaml();
    let _initialIndex;

    _initialIndex = cfg.initialIndex;
    if (
      fm.hasOwnProperty(cfg.paginationFieldName)
    ) {
      _initialIndex = fm[cfg.paginationFieldName]
    }
    let lastIndex = 0;
    const slices = [];

    if (cmd === CMD.NEXT) {
      lastIndex = (_initialIndex + cfg.paginationCnt) > itemsLen ? itemsLen : (_initialIndex + cfg.paginationCnt);
      slices.splice(
        0,0,
        ...cfg.items.slice(_initialIndex, lastIndex)
      )
    }
    if (cmd === CMD.PREV) {

      const rewindedIndex = _initialIndex - (2 * cfg.paginationCnt); // 40 - 15 = 35 35
      _initialIndex = rewindedIndex <= 0 ? 0 : rewindedIndex;
      lastIndex = _initialIndex + cfg.paginationCnt;
      slices.splice(
        0,0,
        ...cfg.items.slice(_initialIndex, lastIndex)
      );
      // console.log({slices, rewindedIndex, lastIndex, _initialIndex})
    }
    if (cmd === CMD.SAME) {
      const rewindedIndex = _initialIndex - cfg.paginationCnt
      _initialIndex = rewindedIndex <= 0 ? 0 : rewindedIndex;
      lastIndex = _initialIndex + cfg.paginationCnt;
      slices.splice(
        0,0,
        ...cfg.items.slice(
          rewindedIndex, lastIndex
        )
      );
    }


    const formatted_slices = slices.reduce((chain, slice, idx) => {
      const text = wikilinkToText(slice);
      const link = metadataCache.getFirstLinkpathDest(text,"")
      const isEvenRow = idx % 2 === 0;
      if (link === null) {
        // console.log({link,text})
        chain += `* <b style="filter: hue-rotate(180deg) saturate(0%) brightness(100%)"> NULL ${slice}</b>\n`
      }
      if (link && isEvenRow) {
        chain += `* ${slice}\n`;
      }
      if (link && !isEvenRow) {
        chain += `* <b style="filter: hue-rotate(77deg) saturate(40%) brightness(100%)"> ${slice}</b>\n`
      }

      return chain;
    }, "")
    
    // console.log({formatted_slices})

    const nextPageIndex = (lastIndex >= itemsLen) ? 0 : lastIndex;
    const updatedFm = Object.assign(fm, {[paginationFieldName]: nextPageIndex})

    const child = Array.from(this.container.children)
      .find((l) => {
        return l.nodeName === "UL"
      })

    if (child) this.container.removeChild(child)

    cfg.synchronizeYaml(updatedFm)

    // # Rendering ... slices
    manuEmbed(this, formatted_slices, this.container);

  }

}  

function wikilinkToText(wikilink) {
  const match = wikilink.match(/\[\[(.*?)\]\]/);
  return match ? match[1] : wikilink;
} // used with file link 
    

// # Rendering
function renderDetails(lteMarkdownLinks, propersCnt, LIMIT) {
   // render layer
 dv.paragraph(lteMarkdownLinks.length + " files that have less than or equal to " + LIMIT + " citums")
 dv.paragraph(propersCnt + " files that have greater than " + LIMIT + " citums")
}

function renderButton(cmd,clickHandler) {
  new obs.ButtonComponent(
    this.container
  )
  .setButtonText(cmd)
  .onClick(() => {
    clickHandler(cmd)
  })
}


function manuEmbed(
  ctx,
  markdown,
  container,
  sourcePath,
  isEmbed = false
) {
  // static render(app: App, markdown: string, el: HTMLElement, sourcePath: string, component: Component): Promise<void>;

  const render = isEmbed === true ? obs.MarkdownPreviewView.render : obs.MarkdownRenderer.render
  return render
    .call(
      ctx,
      ctx.app,
      markdown,
      container,
      sourcePath,
      ctx.component
    );
}


// # Extraction Code vX.X.X.

function extractParams(
  current_filepath,
  vf,
  strategies = {
    vault,
    getEmbedsFromVf,
    extractTargetEmbed,
    manuParams,
    parseStringToMap,
  }
) {
  const {
    vault,
    getEmbedsFromVf,
    extractTargetEmbed,
    manuParams,
    parseStringToMap,
  } = strategies;

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
}


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

```

# ---Transient DOC LOG

* v0.0.10 *2025-04-03*
  * Fix the misc/wip tag dependency.
* v0.0.1 *2025-01-30*
  * Used [[≈.~view-all-lcsh-headings]] as a base to paginate all links found in the litnotes folder.
    * add green to odd row links
    * add metadata pagination count by for more dynamic paging (might use transclusion instead, eh.)

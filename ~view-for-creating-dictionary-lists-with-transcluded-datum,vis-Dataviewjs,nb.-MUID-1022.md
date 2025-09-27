---
CREATION_DATE: "2023-06-06"
MUID: MUID-1022
PROJECT_PARENT: 
TEMPLATE_VERSION: 0.0.0
VERSION: v0.0.6
aliases: 
cssclasses:
  - table-max
  - table-nowrap
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


* [ ] Extract code into a partial dataview so that I can use it as a hoverable wikilink ➕ 2024-12-04 [[~view-to-auto-index-libraries-based-on-first-two-letters]]

[[partial-view-symbol,uti.-~view,bt.-Noteshippo-title-level-affix,]]
### 10÷About

- ~~This is a prototyping [[,aka-experiment-specced-note]] for developing [[Partial-dataview,vis-Noteshippo,]]s.~~

- The goal is to take a dictonary note like [[master-list-of-unfamiliar-vocabulary]] and group them by the first two letters of the indexed word

- This ~~experiment~~ is designed to be hovered over. When the wikilink is placed inside a [[,aka-library-specced-note]], the act of hovering will trigger the group by action. The experiment by itself will exit early since it is not a library note.
- [ ] Refactor code so that dependencies are lifted. Create factorized functions and figs ➕ 2025-05-08 #_todo/43-priority-low--/to-improve/upon-obsidian-codelet 

### 11÷Reference

# =

**file_bn--v0.0.7**: *`= this.file.name`* doc-`=this.DOC_VERSION` `= this.MUID`/`=this.heading`/`=this.PROJECT_PARENT`/


```dataviewjs
const {default: obs} = this?.app?.plugins?.plugins?.['templater-obsidian']?.templater?.current_functions_object?.obsidian;

const {workspace, vault, metadataCache, fileManager} = this.app;

// let file_path = "B_inbox/inbox-note,ad-finem-Fiction-Writing,et-alia/list-of-uncommon-vocabulary,ad-finem-Fiction-writing.md";
// let file_path = "_B_inbox/inbox,vis-Fiction-writing,etc/master-list-of-unfamiliar-vocabulary.md";

// for (let markdownFile of vault.getMarkdownFiles()) {
//   if (markdownFile.basename.startsWith("master-list-of") && markdownFile.basename.includes("vocabulary") )
//   file_path = markdownFile.path;
// }

// # KNOBS
const MIN_VOCAB_ENTRY_COUNT = 100;
const IS_PREVIEW_MODE = false;
const LIBRARY_SPEC_DENOTION_AFFIX = "list-of";
const STOPMARK_HEADINGLEVEL_TUPLE = ["---Transient", 1]
// # Utils
function wikilinkToText(wikilink) {
  const match = wikilink.match(/\[\[(.*?)\]\]/);
  return match ? match[1] : wikilink;
} // used with file link 

// # BOOTUP
workspace.onLayoutReady(bootstrap.bind(this))


function bootstrap() {
	if (!obs) {
		return;
	} // early escape.
	const boundedArchGenMain = archGenMain.bind(this);
	const archGenMainFig = { wikilinkToText };
	(boundedArchGenMain)(genMain.bind(this), this, archGenMainFig)
}

async function archGenMain(genMain, ctx, utils) {
		//consts.
    const vf = workspace.getActiveFile()
    
	  // # LOGIC
    const wls = await genMain.call(ctx);
    // console.log({wls})

		// # Middlewife
    const matrix = wls.slice(
          0,
          Math.min(MIN_VOCAB_ENTRY_COUNT, wls.length)
        )
        .map(
          ([h,v]) => ([
            IS_PREVIEW_MODE ? dv.header(2,h) : `${h} *${v.length}u*`,
            IS_PREVIEW_MODE ? v.map(previewPredicateUsingFileLink) : v 
            ])
        );
    // the native api doesnt transclude markdown
    function previewPredicateUsingFileLink(vi) { 
	    // filelink is as fast as just writing the link. Dv is slow
      // const [linktext,alias] = utils.wikilinkToText(vi).split('|'); 
      return dv.span("!" + vi)
    }
    
    // replaced dv.list so that it doesnt recurse and waste cpu cycles. 
    // ## Faster custom listing function.
    let res = ""
    for (let i = 0; i < matrix.length; i++) {
      const [h,items] = matrix[i]
      res += `* @ ${h}\n`
      const listedItemsInMarkdown = items.forEach((item) => {
        res += `  * ${item}\n`
      })
    }


		// # Rendering
    if (!IS_PREVIEW_MODE) {
      await genManuEmbed.call(ctx, ctx, res, ctx.container);
    }
    if (IS_PREVIEW_MODE) {
      await genManuEmbed.call(ctx, ctx, res, ctx.container, true);

      // dv.list(matrix) // <-- so slow> beta type
    }

  }

async function genManuEmbed(
  ctx,
  markdown,
  container,
  sourcePath,
  isEmbed = false
) {
  // static render(app: App, markdown: string, el: HTMLElement, sourcePath: string, component: Component): Promise<void>;

  const render = isEmbed === true ? obs.MarkdownPreviewView.render : obs.MarkdownRenderer.render
  return await render
    .call(
      ctx,
      ctx.app,
      markdown,
      container,
      sourcePath,
      ctx.component
    );
}

// <WORKHORSE-DICTIONARIFIER>
async function genMain() {
  const abstractFile = workspace.getActiveFile()
  // const folder_path = vf?.parent?.path

  const isLibraryType = abstractFile && abstractFile.basename.includes(LIBRARY_SPEC_DENOTION_AFFIX);

  if (isLibraryType === false) return;

  // const abstractFile = vault.getAbstractFileByPath(
  //   file_path
  // );
  const cache = metadataCache.getFileCache(abstractFile);
  const {headings} = cache;
  
  const startIdx = headings.findIndex(findPublicHeadingPred)
  function findPublicHeadingPred({heading, level}) {
     return heading === "=" && level == 1;
  }
  
  const _endIdx = headings.findIndex(findTransientHeading);
  function findTransientHeading({heading, level}) {
	  const [
		  target_heading, target_level
		] = STOPMARK_HEADINGLEVEL_TUPLE;
    return heading.startsWith(target_heading) && level === target_level;
  }

  const endIdx = _endIdx > -1
    ? _endIdx
    : headings.length - 1;

  const slicedHeadings = headings.slice(startIdx + 1,endIdx);

  const groupedHeadings = groupBy(slicedHeadings)
  // normalizedHeadings functionality that excludes numbers from headings is disabled until further notice.
  const headers = Object.keys(groupedHeadings);
  const matrix = Object.values(groupedHeadings);
  // console.log({headers, matrix});

  const wikilinks = matrix.map((headings) => {
    return headings.map((heading) => {
      const embed = getWikiLinkByVeeFile(
        abstractFile,
        {embed: heading.heading, alias: heading.heading}
      );

      return embed;
    })
  })
  // console.log({wikilinks})
  const result = [];
  for (let i = 0; i < wikilinks.length; i++) {
    const embed = wikilinks[i];
    result.push([ headers[i], embed]);
  }
  return result;
}
// < /WORKHORSE-DICT>

function groupBy(headings, normalizeHeading = (x) => [x]) {
  const sortedHeadings = headings.sort((a, b) => {
    const aHeading = normalizeHeading(a.heading)?.first()
    const bHeading = normalizeHeading(b.heading)
      ?.first() || 0;
    if (aHeading < bHeading) {
      return -1
    }
    if (aHeading > bHeading) {
      return 1;
    }
    return 0;
  })
  const dict = {};
  for (const sh of sortedHeadings) {
    const {heading} = sh
    const normalSh = normalizeHeading(heading)
      ?.first()?.slice(0,2);

    if (!dict[normalSh]) {
      dict[normalSh] = [sh];
    } else {
      dict[normalSh].push(sh)
    }
  }
  return dict;
}

function normalizeHeading(
  heading,
  regex_pattern = '[a-zA-Z][\sa-zA-Z]*'
) {
  const regexPattern = new RegExp(regex_pattern, 'g');
  return regexPattern.exec(heading);
}
function manuRenderConfig() {
  return {
    tag: "div",
    domInfo: {}
  }
}
async function render(
  text,
  config = manuRenderConfig()) {
  const {tag, domInfo} = Object.assign(
    manuRenderConfig(),
    config
  );
  let _text = text;
  if (typeof text === "object") {
    _text = JSON.stringify(text);
  }


  if (tag === "paragraph") {
    return await dv.paragraph(_text, domInfo)
  }
  return await dv.el("div", _text, domInfo)
}
async function genCreateDiv(
  ctx,
  config = { text: "" }) {
  const { text, parent } = config;
  return new Promise((resolve, reject) => {
    const domInfo = {
      text,
      parent: parent ?? ctx
    };
    window.createDiv(domInfo, callback);
    function callback($el) {
      ctx.app.workspace.onLayoutReady(() => {
        resolve({ $el });
      });
    }
  });
}

function getWikiLinkByVeeFile(
  vf,
  config = {embed:"", alias: ""}
) {
 const embed_config = config?.embed ?
     vf.path+"#"+config.embed :
     "";
 const embed_alias = config?.alias || ""

  //config.alias ?? vf.basename
  return fileManager
    .generateMarkdownLink(
      vf,
      vf.path,
      embed_config,
      embed_alias
    );
}

```

# ---Transient


## «--LR÷Use Request Animaiton Frame to solve rendering issue

* 📝 <https://discord.com/channels/686053708261228577/1014259487445622855/1075431577003237406>

```js
class DVTools {
  constructor() {
    this.dv = app.plugins.getPlugin('dataview')?.api;
  }

  async inject(dv, note) {
    const head = this.dv.page(note);
    const headload = await this.dv.io.load(head.file.path); //
    var container = document.createDocumentFragment();

    var $el = container.createEl('p');
    var $injectMark = createEl('inject-mark');
    const { path } = app.workspace.lastActiveFile;
    var result = await obsidian.MarkdownRenderer.renderMarkdown(
      headload,
      $el,
      path,
      dv.component
    );
    dv.container.replaceChildren($injectMark);
    /* replace the surrounding element a frame later */
    requestAnimationFrame(() => {
      $injectMark.parentElement.replaceWith(...$el.children);
    });
  }
}
```

## LA÷v1

* [ ] Generate dynamic transcluded links, made dynamic because the [[transclusion]] parameters are sourced from a distant [[master-list-of-unfamiliar-vocabulary#Extant|extant]] file, e.g. the [[anchor-heading,etc]]s (and thereby the goal would be to induce [[transclusion]] of their content). This codelet will execute within a dataview post processing block. In order to bypass the conflicts that two competing post processors may [[may-vs-might]] have upon each other, one solution is to shove the transformed markdown into a seperate container so that the native [[markdown-postprocessor,vis-ObsidianMD-app,]] can act upon it without interference.
* 🤔 Two awaits cause the rendering to go haywire.
* Embeds do not postprocess fully when dvjs renders
* Competing post processors?
* What happens if I use two different containers?
* What happens if I attach to dom as a shadow fragment?

```js
~~~dataviewjs
const {default: obs} = this.app.plugins.plugins['templater-obsidian'].templater.current_functions_object.obsidian;

const {workspace, vault, metadataCache, fileManager} = this.app;


const file_path = "uncommon-vocabulary-used-for-novel-writing.md";

(async function(ctx) {

  await genManuEmbed.call(ctx, ctx, "![[Untitled#=]]", ctx.container);
  // const div = await genCreateDiv(ctx, {text: "", parent: ctx.container});
  // ctx.container.append(div.$el);
  await genManuEmbed.call(ctx, ctx, "![[Untitled#=]]");

  // const wl = await main.call(ctx);
  // console.log({wl})

})(this)

async function genManuEmbed(ctx, existent,container) {
  return await obs.MarkdownRenderer.render.call(ctx,
    ctx.app,
    existent,
    container,
    "");
}

// <WORKHORSE-DICTIONARIFIER>
async function genMain() {
  const abstractFile = vault.getAbstractFileByPath(
    file_path
  );
  const cache = metadataCache.getFileCache(abstractFile);
  const {headings} = cache;
  const startIdx = headings.findIndex(findPublicHeadingPred)
  function findPublicHeadingPred({heading, level}) {
     return heading === "=" && level == 1;
  }
  const _endIdx = headings.findIndex(findTransientHeading);

  const endIdx = _endIdx > -1
    ? _endIdx
    : headings.length - 1;

  function findTransientHeading({heading, level}) {
    return heading === "---Transient" && level == 1;
  }
  const slicedHeadings = headings.slice(startIdx + 1,endIdx);

  const groupedHeadings = groupBy(slicedHeadings)
  // console.log({groupedHeadings})
  const headers = Object.keys(groupedHeadings);
  const matrix = Object.values(groupedHeadings);
  // console.log({headers, matrix});

  const wikilinks = matrix.map((headings) => {
    return headings.map((heading) => {
      const embed = getWikiLinkByVeeFile(
        abstractFile,
        {embed: heading.heading, alias: ""}
      );
      return `!${embed}`;
    })
  })

  return wikilinks
}
// < /WORKHORSE-DICT>

function groupBy(headings, normalizeHeading = (x) => [x]) {
  const sortedHeadings = headings.sort((a, b) => {
    const aHeading = normalizeHeading(a.heading)?.first()
    const bHeading = normalizeHeading(b.heading)
      ?.first() || 0;
    if (aHeading < bHeading) {
      return -1
    }
    if (aHeading > bHeading) {
      return 1;
    }
    return 0;
  })
  const dict = {};
  for (const sh of sortedHeadings) {
    const {heading} = sh
    const normalSh = normalizeHeading(heading)
      ?.first()?.slice(0,2);

    if (!dict[normalSh]) {
      dict[normalSh] = [sh];
    } else {
      dict[normalSh].push(sh)
    }
  }
  return dict;
}

function normalizeHeading(
  heading,
  regex_pattern = '[a-zA-Z][\sa-zA-Z]*'
) {
  const regexPattern = new RegExp(regex_pattern, 'g');
  return regexPattern.exec(heading);
}
function manuRenderConfig() {
  return {
    tag: "div",
    domInfo: {}
  }
}
async function render(text, config = manuRenderConfig()) {
  const {tag, domInfo} = Object.assign(
    manuRenderConfig(),
    config
  );
  let _text = text;obsidian://open?vault=sbrain&file=B_inbox%2Finbox%2Cvis-Fiction-writing%2Cetc%2Fmaster-list-of-unfamiliar-vocabulary
  if (typeof text === "object") {
    _text = JSON.stringify(text);
  }


  if (tag === "paragraph") {
    return await dv.paragraph(_text, domInfo)
  }
  return await dv.el("div", _text, domInfo)
}
async function genCreateDiv(
  ctx,
  config = { text: "" }) {
  const { text, parent } = config;
  return new Promise((resolve, reject) => {
    const domInfo = {
      text,
      parent: parent ?? ctx
    };
    window.createDiv(domInfo, callback);
    function callback($el) {
      ctx.app.workspace.onLayoutReady(() => {
        resolve({ $el });
      });
    }
  });
}

function getWikiLinkByVeeFile(
  vf,
  config = {embed:"", alias: ""}
) {
  const embed_config = config?.embed ?
     vf.path+"#"+config.embed :
     "";
  //config.alias ?? vf.basename
  return fileManager
    .generateMarkdownLink(
      vf,
      vf.path,
      embed_config,
      ""
    );
}


~~~
```

---
# ---Transient Local Archive


* v0.0.0 *No date.*
  * Created a list of embedded link in an array
    * ![[sandbox-view-for-creating-dictionary-lists-1695192439093.jpeg]]
    * `![[B_inbox/inbox-note,ad-finem-Fiction-Writing,et-alia/list-of-uncommon-vocabulary,ad-finem-Fiction-writing.md#Chaise]]` would be the typically data entry.
      * 💁 The goal is to see what bypasses the dataview js and allows Obsidian to process these embeds.
    * 🤔 The dv code incredibly laggy.

# ---Transient Doc Log

- [[transient-doc-log-endpoint,bt.-Noteshippo-heading-api,]]
- v0.0.7 *2025-05-08* 
	- Refactored 
	- Move old note information into archive.
- v0.0.6
	- Remove all --transient from dictionary scrape
	- add muid to title
* v0.0.5
  * prototype what exactly is faster. there seems to be no good way to transclude using native api. Not without sacrificing speed.
  * There's a lot of experiments here but the preview is too slow for more than 10. The markdown stuff i'm doing doesn't work. You need just in time rendering, adding a child and lifecycles.
* v0.0.4
  * replaced dv.list with a self markdown formatted text that runs through markdownrenderer so it renders more natively
    * ? why is dv.list so performance poor? Probably cuz of the dv.span calls made per line. (did you record you proof? 🔑 No. cuz i dont care enough to do so.)
* v0.0.3
  * Replaced the textual stuff that i was outputting with dv.list


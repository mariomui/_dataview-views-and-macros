---
CREATION_DATE: "2022-08-05"
DOC_VERSION: v0.0.3
MUID: MUID-154
PROJECT_PARENT: 
TEMPLATE_VERSION: 
aliases: 
tags:
  - _misc/_wip
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

- [ ] Remove buttons as a dependency from code.
- [ ] Does this deprecate [[ø--~button-for-ytranscript]]?

### 11-Reference

- https://www.youtube.com/example
- [[√-LEADERSHIP-LAB-Writing-Beyond-the-Academy-1-23-15-YouTube]] has some code to properly include the Youtube Transcripting Code without running into dataview issues.

## 20-Inlink

> [!abstract]- %%  %% Automated List of Reference Inlinks (v0.0.5)
> * ℹ Commit/design logs are located in this [[,aka-MUID-150|experiment note]]. 
> > `= join( map( sort( map( filter(this.file.inlinks, (link) => meta(link).path != this.file.path), (x) => [ split(meta(x).path, "/")[length(split(meta(x).path, "/")) - 1], x ] ) ), (b) => "• " + choice( length(b[0]) > 28, link( b[1], truncate( regexreplace(b[0], "(-of|of|the|-the|-for|-that|https-|ee)", ""), length( regexreplace(b[0], "(-of|of|the|-the|-for|-that|https-|ee)", "") ) * 0.75 ) ), link(b[1], regexreplace(b[0], "\.md$", "")) ) ), "<br>" )`


# =


## Transcript Control

```dataviewjs
// v1.0.4
const {default: obs} = this.app.plugins.plugins['templater-obsidian'].templater.current_functions_object.obsidian

const {workspace,metadataCache, vault} = this.app;
const {container} = this;
const priority_prefix = "√"

const getCurrentVersion = (ctx) => {
  const source_path = ctx.currentFilePath
  const sourcedAbstractFile = vault
    .getAbstractFileByPath(source_path);

  const {frontmatter} = metadataCache
    .getFileCache(sourcedAbstractFile);
  return parseFrontmatter(frontmatter, "VERSION");
}
const VERSION = getCurrentVersion(this);
let $buttonRef = null;

//https://youtu.be/pT2bSZlfQq0?si=VMhewAPAMuTK5TFy
const prefixes = ['https://www.youtube.com', 'https://youtu.be'];

const {plugins} = this.app.plugins;
const {createButton} = plugins["buttons"]
const yt = plugins['ytranscript'];

const makeOpenBtn = (container, url, name) => {
  console.log({url})
  return createButton({
    app,
    el: container,
    args: {
      name,
    },
    clickOverride: {
      click: openView.bind(this),
      params: [{
        url,
        getYtTabEl: getYtTabEl.bind(this),
        yt
      }],
    },
  })
}

//bootstrap
workspace.onLayoutReady(bootstrap.bind(this));

/**
 * Represents a URL context.
 * @typedef {Object} Link
 * @property {string} text
 */
/**
 * Represents a URL context.
 * @typedef {Object} UrlContext
 * @property {string} domain - The domain of the URL.
 * @property {Link} link - The domain of the URL.
 * @property {string} domain - The domain of the URL.
 */
function bootstrap() {
  makeOpenBtn(container, "", "Refresh " + VERSION);
  (async function(ctx) {
    const abstractFile = workspace.getActiveFile();
    const {listItems} = metadataCache
      .getFileCache(abstractFile);

    const cr = await vault.cachedRead(abstractFile)
    if (!listItems) return;

    const texts = getAllTextsFromListItems(
      listItems,
      cr
    );

    /**
    * @type {number}
    */
    const urlContext = findUrl(texts, prefixes);
    console.log({urlContext}, 'in bootstrap')
    if (urlContext !== null) {

      return main.call(ctx, urlContext);
    }
  })(this)
}



function main(urlContext) {
  (function(ctx, createUiEngine, dv) {
    ctx.container.style = 'border: 3px solid lightblue;margin: .5em 0;'
    console.log({urlContext}, 'in main')
    const row1 = [
      'YTranscript',
      '。。。。',
      makeOpenBtn.call(ctx, container,urlContext, 'Open')
    ];
    $buttonRef = row1[2];
    const md = dv.markdownTable('*,*,*'.split(","),[row1])
    const {link, domain, urlPath, link_desc} = urlContext
    
    const markdownLink = `[${link_desc}](${urlPath})`

    
    workspace.onLayoutReady(innerMain.bind(this))
    function innerMain() {
      (doUiMain.bind(this))()
      function doUiMain() {
        const ui = createUiEngine()
        ui.renderMarkdownLink(markdownLink)
        ui.renderParagraph(`\`\`\`timestamp-url\n${urlPath}\n\`\`\``)
      }
    }
  })(this, createUiEngine.bind(this), dv)
}

// ui

function createUiEngine() {

  return (function (ctx) {
    const pkg = {
        renderParagraph: renderParagraph.bind(this),
        genRenderMarkdownLink,
        renderMarkdownLink,
    };
    function renderParagraph(...args) {
        return dv.paragraph(...args)
    }
    async function genRenderMarkdownLink(...args) {
      return await obs.MarkdownRenderer.render.call(
        ctx, ctx.app, args.first() || "N/A", ctx.container, ""
      );
    };
    function renderMarkdownLink(markdownLink) {
      (async function(manuUiToolBoxInstance) {
        await manuUiToolBoxInstance
          .genRenderMarkdownLink(markdownLink)
      })(pkg)
    };

    return pkg
  }.bind(this))(this)
}



// logic
function parseMarkdownLink(markdown_link) {
  const href = markdown_link.replace(/.*\((.*)\).*/g,"$1")
  const text = markdown_link.replace(/.*\[(.*)\].*/g,"$1")
  
  const parsedContext = {
    text,
    href
  }
  return parsedContext
}
function getAllTextsFromListItems(listItems,cr) {
  const crlines = cr.split('\n');
  const listItemTextsMatrix = [];
  for (const listItem of listItems) {
    const startLineIdx = listItem?.position?.start?.line
    const endLineIdx = listItem?.position?.end?.line
    const listItemTexts = crlines
      .slice(startLineIdx, endLineIdx + 1);
    listItemTextsMatrix.push(
      {text: listItemTexts.join("")}
    );
  }
  return listItemTextsMatrix;
}

/**
* @return {UrlContext} 
**/
function findUrl(links, domains) {
  const tuples = [];
  for (let link of links) {
    for (const domain of domains) {
      if (link?.text?.contains(domain)) {
      
        tuples.push({link,domain})
      }
    }
  }
  console.log({tuples})
  const tuple = tuples.findLast(
    ({link,domain}) => {
      // #_todo/to_document/on-coding/regarding-dynamic-regex/regarding-using-escapes-to-interpolate-properly
      const dynamicRegex = new RegExp(`^(\s*[-*] ${priority_prefix} .*)$`, "g");
      const result = link?.text?.match(dynamicRegex);
      return result;
    }
  );
  if (!tuple) {
    return null
  }
  const {domain, link} = tuple;
  const urlPath = link?.text.replace(/.*\((.*?)\).*/g,"$1")
  const [link_desc] =  link?.text?.split(domain);
  const parsed_infos = [link, urlPath, domain]

  if (!parsed_infos.every(Boolean)) {
    return null;
  }
  const _link_desc = link_desc.replace(/.*?\[(.*?(?![\(]))\].*?$/g, "$1");
    console.log({parsed_infos,link_desc, _link_desc})
  return {
    link, 
    urlPath, 
    domain, 
    link_desc: _link_desc
  };
}


function openView({url, yt, getYtTabEl}) {
  new Notice( $buttonRef ? 'has' : 'nope')
  const ytTab = getYtTabEl()
  console.log({url, yt, getYtTabEl})
  if (!$buttonRef) {
    return;
  }
  if (ytTab?.tabHeaderCloseEl) {
    ytTab.tabHeaderCloseEl.click()
    $buttonRef.innerText = 'Open';
    return;
  }

  $buttonRef.innerText = 'Close';
  // const regex = /\(([^)]+)\)/;
  // const regexed_url = url.link.text.replace(regex, "$1");
  console.log({urlSuppliedToYTranscript: url})
  yt.openView(url.urlPath)

}

function getYtTabEl() {
  return workspace.rightSplit.children[0]
    ?.children.find(findViewPredicate)
}

function findViewPredicate({tabHeaderEl}) {
  return tabHeaderEl?.dataset?.type === 'transcript-view'
}

function parseFrontmatter(frontmatter, key) {
  if (!Object.keys(frontmatter)?.length) {
    return;
  }
  return frontmatter[key] ?? "";
}

function getMarkdownALinkByUrl(url) {
  if (!url) return "";
  return `[](${url})`
}
```

# ---Transient

# ---Transient Doc Log

**file_bn--v0.0.4**: *`= this.file.name`* doc-`=this.DOC_VERSION` `= this.MUID`/`=this.heading`/`=this.UMID`/

- v0.0.3
	- Add the MUID to the filename
	- Rename file to a partial data view meant to be transcluded.
- v0.0.2
  - Archive prototype list
- ## Versioned Prototypes
  - [[list-of-prototypes,nb.-Video-notetaking,vis-Noteshippo]]


* Bossfight
  * Boss counterattack all techniqu
---
alias:
CREATION_DATE: 2023-12-23
DOC_VERSION: v0.0.1
MUID: MUID-1934
tags: _wip 
TEMPLATE_VERSION: v1.0.4_blank-template
UMID: "[[UMID-aace1b9f-40fc-472e-b6b8-596280367ac3]]"
---

# -

## About




# =

**filename:** `=this.file.path`

> [!info]  This codelet searches for an existing **embedded query**and makes a **jobs line** to notify you of any links you may not have written about.
  - :~~See [[custom-transclusion-parameters,cf.-Kanzi,vis-ObisidianMD-app,]] for search term use.~~

```dataviewjs
const { default: obs } =
  this.app.plugins.plugins["templater-obsidian"].templater
    .current_functions_object.obsidian;

const {metadataCache,vault,workspace,fileManager} = this.app
const {adapter} = vault;

let MARKER = "---Transient Local Query";

const getCFP = () => this.currentFilePath;

workspace.onLayoutReady(main.bind(this))

function main(cmd) {
  (genMain)(this,renderUI.bind(this))
  async function genMain(ctx,renderUI) {
    // --
    const providing_path = getCFP()
    const argMap = extractParams(
      providing_path,
      workspace.getActiveFile()
    )

    MARKER = argMap.search_term || MARKER;

    // --

    const avf = workspace.getActiveFile()
    const mdc = metadataCache.getFileCache(avf)
    const avfv = workspace.getActiveFileView()

    const queryFileFigsPayload = await genProcessQueriedFileFigsByActiveFileView(avfv, MARKER)

    
    const {
      unaliasedQueryLinks,
      unaliasedEmbeddedLinks
    } = await genProcessDiffingArtifactFig(mdc,queryFileFigsPayload)
    
    const unusedLinks = processDiffJobLine({unaliasedQueryLinks, unaliasedEmbeddedLinks}).map((basename) => {
      const vf = metadataCache.getFirstLinkpathDest(basename,"")

      if (!vf) {
        console.log({vf,basename})
        return false;
      }
      const link = getMarkdownLink(vf,"")
      console.log({link}, 'inside of processDiffJobLine mapper')
      if (!link) return false;
      
      return link;
    }).filter(Boolean)

    
    //ui
    renderRefreshAndCopyButton.call(ctx, main,"copy")
    console.log({unusedLinks})
    if (unusedLinks.length >= 1) {
      
      return renderUI.call(ctx, unusedLinks, cmd)
    }  

    const { $el } = await genCreateDiv(
      ctx, { text: "" , cls: "deleteme"}
    );

    const callout_text = getTextForRender(argMap?.search_term || "Empty"); 
    const rendered_text = await obs.MarkdownRenderer.render(
      ctx.app,
      callout_text,
      $el,
      "",
      ctx.container.component
    );
    ctx.container.append($el)

  }
}

async function genProcessDiffingArtifactFig(mdc, queryFileFigsPayload) {
   
  const embeddedLinks = processAllEmbeddedLinks(mdc);
  const config = queryFileFigsPayload;
  if (!config?.data) return manuProcessDiffJobLineFig();
  
  const unaliasedEmbeddedLinks = await genProcessMarkdownLinksByWikiLink(embeddedLinks)
  const unaliasedQueryLinks = await genProcessMarkdownLinksByVf(config.data);

  return  {
    unaliasedEmbeddedLinks,
    unaliasedQueryLinks
  }
  // compare the links in the note will have less text than the ones in the query.
}

function manuProcessDiffJobLineFig() {
  return {
    unaliasedQueryLinks: [],
    unaliasedEmbeddedLinks: []
  }
}
function processDiffJobLine(
  fig = manuProcessDiffJobLineFig()
) {
  const config = Object.assign({}, manuProcessDiffJobLineFig(), fig);
  const { unaliasedQueryLinks, unaliasedEmbeddedLinks} = config
  console.log({unaliasedQueryLinks, unaliasedEmbeddedLinks})
  const unusedLinks = []
  for (const link of unaliasedQueryLinks) {
    const isUsed = unaliasedEmbeddedLinks.some(
      (embedLink) => {
        console.log({link, embedLink})
        return link.startsWith(embedLink)
      }
    )
    if (!isUsed) {
      unusedLinks.push(link)
    }
  }
  return unusedLinks;
}

async function genScrapeTextFromWikiLink(wikilink) {
  return new Promise((rs,rj) => { 
    wikilink.replace(/\[\[(.*?)\]\]/g, (match, data) => {
      if (data) {
        rs({data, err: null})
      } else {
        rj({data: null, err: `ERR: data is ${data||"N/A"}`})
      }
      return match
    }) 
  })
}
async function genProcessMarkdownLinksByWikiLink(wikilinks) {
  if (!Array.isArray(wikilinks)) return [];
  const unaliasedMarkdownLinks = []
  for (const wikilink of wikilinks) {
    const unaliasedMdLink = await genUnaliasedMarkdownLink(wikilink)
    unaliasedMarkdownLinks.push(unaliasedMdLink)
  }
  return unaliasedMarkdownLinks
}

async function genProcessMarkdownLinksByVf(vfs = []) {
  if (!Array.isArray(vfs)) return [];
  const unaliasedMarkdownLinks = []
  for (const datum of vfs) {
    const markdownLink = getMarkdownLink(datum, datum.basename)
    const unaliasedMdLink = await genUnaliasedMarkdownLink(markdownLink)
    unaliasedMarkdownLinks.push(unaliasedMdLink)
  }
  return unaliasedMarkdownLinks
}

// utils
function getMarkdownLink(avf,heading) {

  const _embed_text = !heading ? "" : `#${heading}`
  const markdownLink = fileManager
    .generateMarkdownLink(
      avf,
      avf.path,
      avf.basename + _embed_text,
      heading
    )
  return markdownLink;
}

async function genUnaliasedMarkdownLink(markdownLink) {
    const payload = await genScrapeTextFromWikiLink(
      markdownLink
    )
    if (payload?.err) {
      console.log({err})
      return "";
    }
    
    const parsedLink = obs.parseLinktext(payload.data);
    
    if (parsedLink?.path) {
      return parsedLink.path
    }
    throw new Error("Parsed path does not exist ")
}

// # processors
async function genProcessQueryFileFigToWikiLinks(fileFig) {
  
  const markdownLink = getMarkdownLink(avf,heading)
  try {
    const unaliasedMarkdownLink = await genUnaliasedMarkdownLink(markdownLink);
    return unaliasedMarkdownLink
  } catch(err) {
    const err_string = JSON.stringify(err);
    throw new Error(`genUnaliasedMarkdownLink error ${err_string}`)
  }
}

function processAllEmbeddedLinks(mdc) {


  const mdcLinks = mdc?.links?.map(({original}) => original) || [];
  const mdcEmbeds = mdc?.embeds?.map(({original}) => original) || [];
  const mdcs = [...mdcLinks, ...mdcEmbeds]
  return mdcs;
}


async function genProcessQueriedFileFigsByActiveFileView(avf, search_term) {
  // I couldnt find a callback to see when the queue finished populating but when i do i knwo this function will need it.
  return new Promise((rs,rj) => {
    {var result = { data: [], err: null};
    
      try {
        const data = avf?.
          _children?.at(0)?._children
            ?.find((child) => child.query && child.query.indexOf(search_term) > -1)
            ?.dom?.getFiles()
        result.data = data || []
        return rs(result)
      } catch (err) {
        result.err = err;
        return rj(result)
      }
    }
  })
}
// UI
function getTextForRender(search_term) {
      const warning =  `\> [!warning] Search_term is empty`
      const notification =  `\> [!note] ${search_term} has been used up`;
      if (!search_term) return warning;
}

function renderRefreshAndCopyButton(main,cmd) {
  const copy = cmd === "copy" ? "copy" : null
  const handleClick = () => {
    const $button = this.container.querySelector('button')
    $button.remove();
    this.container?.lastChild?.remove();
    main.call(this, copy);

  }
  new obs
    .ButtonComponent(this.container)
    .setButtonText('Refresh and Copy')
    .onClick(handleClick.bind(this))
}
function renderUI(embed_texts,cmd) {
    const uiMdText = embed_texts.reduce((chain,text) => {
      return chain + "* " + text + "\n"
    },"")
    const $el =dv.paragraph(
      uiMdText, {
        cls: "deleteme"
      }
    );
    if (cmd === "copy")  {
      const text = $el.innerText;
      navigator.clipboard.writeText(uiMdText);
    }
}

function genCreateFrag(ctx) {
  return new Promise((resolve, rej) => {
    ctx.container.win.createFragment(($el) => {
      ctx.app.workspace.onLayoutReady(() => {
        resolve({$el})
      })
    })
  })
}
async function genCreateDiv(ctx, config = {text: ''}) {
  const {text} = config;
  const $frag = await genCreateFrag(ctx);
  return new Promise((resolve, reject) => {
    const domInfo = {
      text,
      parent: $frag
    };
    ctx.container.win.createDiv(domInfo, callback)
    function callback($el) {
      ctx.app.workspace.onLayoutReady(() => {
        resolve({$el, $frag})
      })
    }
  })
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

# ---Transient Local Resources



[[differential-between-affix,experiment-vs-sandbox,vis-Noteshippo]]
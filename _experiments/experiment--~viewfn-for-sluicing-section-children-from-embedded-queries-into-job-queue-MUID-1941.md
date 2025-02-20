---
TEMPLATE_VERSION: v1.0.0
MUID: MUID-1941
CREATION_DATE: 2023-12-26
tags:
  - _meta
UMID: "[[UMID-aace1b9f-40fc-472e-b6b8-596280367ac3]]"
DOC_VERSION: v0.0.1
TEMPLATE_SOURCE: "[[20--default-meta-template]]"
---

# -
## About

This note is derived from [[interim--~viewfn-for-sluicing-out-embedded-query-into-a-job-queue,nb.-MUID-1934]]

> [!warning] this is an example of long filenames not workin because there is now to see the seperation between sentences fo the long function names.


# =

* ! DOESN"T GRAB THE TEXT INSIDE OF EMBED QUERIES

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

    const queryFileFigsPayload = await genProcessQueriedFileFigsByActiveFileViewV2(avfv, MARKER)

    const {
      unaliasedQueryLinks,
      unaliasedEmbeddedLinks
    } = await genProcessDiffingArtifactFig(mdc,queryFileFigsPayload)
    
    const unusedLinks = processDiffJobLine({unaliasedQueryLinks, unaliasedEmbeddedLinks}).map((link) => {

      const parsedLinkFig = obs.parseLinktext(link)

      //const link = getMarkdownLink(vf,"")
      return getMarkdownLinkFromParsedLinkText(parsedLinkFig)
      return `[[${link}]]`;
    })

    
    //ui
    renderRefreshAndCopyButton.call(ctx, main,"copy")

    if (unusedLinks.length >= 1) {
      
      return renderUI.call(ctx, unusedLinks, cmd)
    }  
    
    const { $el } = await genCreateDiv(
      ctx, { text: "" , cls: "deleteme"}
    );
    const callout_text = getTextForRender(argMap?.search_term || ""); 
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
  console.log({unaliasedEmbeddedLinks, embeddedLinks})
  const unaliasedQueryLinks = config.data || []

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
      (embedLink) => link.indexOf(embedLink) > -1
    )
    if (!isUsed) {
      unusedLinks.push(link)
    }
  }
  console.log({unusedLinks})
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

// unused
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


function getMarkdownLinkFromParsedLinkText(parsedLinkFig,alias ="") {
  const {path, subpath} = parsedLinkFig
  const vf = metadataCache.getFirstLinkpathDest(parsedLinkFig.path,"")
  const markdownLink = fileManager
    .generateMarkdownLink(
      vf,
      path,
      subpath,
      alias
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

    return parsedLink.path + parsedLink.subpath
}

// # processors
async function genProcessQueryFileFigToWikiLinks(fileFig) {
  
  const markdownLink = getMarkdownLink(avf,heading)
  const unaliasedMarkdownLink = await genUnaliasedMarkdownLink(markdownLink);
  return unaliasedMarkdownLink
}

function processAllEmbeddedLinks(mdc) {


  const mdcLinks = mdc?.links?.map(({original}) => original) || [];
  const mdcEmbeds = mdc?.embeds?.map(({original}) => original) || [];
  const mdcs = [...mdcLinks, ...mdcEmbeds]
  return mdcs;
}

async function genProcessQueriedFileFigsByActiveFileViewV2(activeViewFile, search_term) {
  return new Promise((rs,rj) => {
    {var result = { data: [], err: null};
    
      try {
        const findPredicate = (child) => {
          console.log({child, search_term})
          return child.query && (child.query.indexOf(search_term) > -1)
        }
        const found = activeViewFile?.
          _children?.at(0)?._children
            ?.find(findPredicate)

        const data = found?.dom?.getFiles() || []

        // each datum is a file
        const matches = [];
        for (const datum of data) {
        
          const result = activeViewFile?._children?.at(0)?._children
            ?.find((child) => child.query)
            ?.dom?.getResult(datum)
          
          if (!result?.result) break;
          console.log({filecontent: result.content})
          const isFilename = result.result.hasOwnProperty("filename")
          const isContent = result.result.hasOwnProperty("content")

          let matchesFig = {filenameMatches: [], contentMatches: []}
          if (isFilename) {
            matchesFig.filenameMatches = matchesFig.filenameMatches
              .concat(
                result.result.filename
              )
          } else if (isContent) {
            matchesFig.contentMatches = matchesFig.contentMatches
              .concat(
                result.result.content
              )
          }

          for (const match of matchesFig.contentMatches) {
            const [start,end] = match;
            
            if (!end) new obs.Notice("end idx missing")
            // const startIdx = findLastIndexFromIndex(start, result.content)
            console.log({datum})
            matches.push(
              datum.basename + result.content.slice(start, end)
            )
          }
        }
        console.log({matches})
        result.data = matches || []
        return rs(result)
      } catch (err) {
        result.err = err;
        return rj(result)
      }
    }
  })
}

async function genProcessQueriedFileFigsByActiveFileView(avf) {
  // I couldnt find a callback to see when the queue finished populating but when i do i knwo this function will need it.
  return new Promise((rs,rj) => {
    {var result = { data: [], err: null};
    
      try {
        const data = avf?.
          _children?.at(0)?._children
            ?.find((child) => child.query)
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

function findLastIndexFromIndex(idx, content, target = "\n") {
  let j = target.length - 1;
  for (let i = idx; i > 0; i-- ) {
    if (j < 0) {
      return i + 2 
    } 
    if (content[i] === target[j]) {
      j--
    } else {
      j = target.length - 1;
    }
  }
  return idx;
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

# ---Transient Bug Log

* ! Bugs
  * If there is another filename that has the text, it will also grab that.
  * What I really need is a way to grab these texts, validate it and put it into a uniqueMap.
---
# ---Transient Commit Log

- v0.0.1 add search for multiple query childs
- v.0.0.0 raw
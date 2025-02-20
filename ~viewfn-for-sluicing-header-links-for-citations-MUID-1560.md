---
aliases: 
CREATION_DATE: 2023-10-07
DOC_VERSION: v0.0.6
MUID: MUID-1560
tags:
  - _wip
TEMPLATE_VERSION: v1.0.4_blank-template
UMID:
---

# -

## About

This is a [[Partial-dataview,vis-Noteshippo,]]. It houses a [[,aka-codelet]] that uses [[Dataview-plugin,bt.-ObsidianMD-app,]] to create a view that scans the nearest H1 search term and collects all the H2s underneath it.

The result is displayed as a list of links.
There is a copy button. It is used to create a job queueing system for HEADERS under `---Transient Local Citations`

[[custom-transclusion-parameters,cf.-Kanzi,vis-ObisidianMD-app,]] allow this [[,aka-codelet]] to be dynamically controlled at runtime.

# =

**filename:** `=this.file.path`

- The following code scans the content under a specific H1 using the [[custom-transclusion-parameters,cf.-Kanzi,vis-ObisidianMD-app,]] syntax (?search_term=). Every H2 is transformed into a bulletpointed outline of markdown links.⤵

```dataviewjs
const { default: obs } =
  this.app?.plugins?.plugins?.["templater-obsidian"]?.templater?.current_functions_object.obsidian;

const {metadataCache,vault,workspace,fileManager} = this.app
const {adapter} = vault;

let MARKER = "---Transient Local Citations";

const getCFP = () => this.currentFilePath;

workspace.onLayoutReady(main.bind(this))
function main(cmd) {
	if (!obs) return;
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


    const [citations, embed_texts] = gatherEmbedTexts(
      avf, mdc
    );

    renderRefreshAndCopyButton
      .call(ctx, main,"copy")

    
    if (embed_texts.length >= 1) {
      renderUI.call(ctx, embed_texts, cmd)
    } else {
      const { $el } = await genCreateDiv(
        ctx, { text: "" , cls: "deleteme"}
      );


      const callout_text = getTextForRender(argMap?.search_term || ""); 
      const rendered_text = await obs
        .MarkdownRenderer
        .render(
          ctx.app,
          callout_text,
          $el,
          "",
          ctx.container.component
      );
      ctx.container.append($el)
    }
  }
}

// LOGIC layer

function getTextForRender(search_term) {
      const warning =  `\> [!warning] ${search_term} closest to analysis is empty`
      const notification =  `\> [!note] ${search_term} has been used up`;
      return warning;
}

function gatherEmbedTexts(avf = null, mdc = null) {
  if ([avf, mdc].every(Boolean) === false) return [];

  const embed_texts = [];
  const citations = [];
  let isReadFlag = false;
  
  if (!!mdc?.headings === false) return embed_texts;
  if (mdc.headings.length === 0) return embed_texts;
  
  // these are the used links
  const mdcLinks = mdc?.links?.map(({original}) => original) || [];
  const mdcEmbeds = mdc?.embeds?.map(({original}) => original) || [];
  const mdcs = [...mdcLinks, ...mdcEmbeds]

  for (let mdcHeading of mdc.headings) {
    const {level,heading} = mdcHeading;
    const markdownLink = getMarkdownLink(avf,heading)
    const unaliasedMarkdownLink = getUnaliasedMarkdownLink(
      markdownLink
    );
    // console.log({MARKER, heading, level, isReadFlag})

    if (heading.startsWith(MARKER) && level === 1) {
      isReadFlag = true;
      continue;
    }
    if (!heading.startsWith(MARKER) && level === 1) {
      isReadFlag = false;
    }
    if (isReadFlag === false) continue;
    if (isReadFlag && level === 2) {
      citations.push(markdownLink)
    }
  }
  
  for (const citation of citations) {
    const unaliasedMarkdownLink = getUnaliasedMarkdownLink(citation);
    const isUsed = mdcs.some((s) => {
	    const _s = s.toUpperCase()
	    const _u = unaliasedMarkdownLink.toUpperCase()
      const ret = _s.indexOf(_u) > -1
      // console.log({ret,s,unaliasedMarkdownLink})
      return ret;
    })
    if (!isUsed) {
      embed_texts.push(citation)
    }
  }

  return [citations, embed_texts];
}

function getMarkdownLink(avf,heading) {
  const _embed_text = `#${heading}`
  const markdownLink = fileManager
    .generateMarkdownLink(
      avf,
      avf.path,
      avf.basename + _embed_text,
      heading
    )
  return markdownLink;
}

function getUnaliasedMarkdownLink(markdownLink) {
    const parsedLink = obs.parseLinktext(markdownLink);
    return parsedLink.path + parsedLink.subpath.split('|').first()
}

// UI

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
    console.log({elInnerText: $el.innerText})
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



# ---Transient Commit Log

* v0.0.7 *2025-01-21*
	* Fix it so that a lint that upper cases the first letter wont register as a unused item by uppercasing all objects
* v0.0.6
  * Add MUID to note title 
  * Fix bug where the codelet did not exclude links inside of non markered headers
* v0.0.5 Remove weird flags and create a more straightforward implementation
  * Any time we go past the target heading, we create a fully featured markdownlink. These are our list of citation headers. The markdownlinks are in our backpocket so we scarpe the whole document for links, finidng that there isn't any...the list of links in the document. Because the headings are not links per se, that means, should the list of links every include a match from one of the list of citations
  * The log also helped against recidivistic bugs as i was about to forget to track embedded links as "consumed"
* v0.0.4 Fix bug with non existent array in mdc embedded texts
- v0.0.3 Fix bug where embedded links (used up) were not included as used up links.
- v0.0.2 Begin work to remove already linked texts from Sorted Citations
- v0.0.1 Add button for refreshing



---
alias:
CREATION_DATE: 2023-10-07
DOC_VERSION: v0.0.2
MUID: MUID-1560
tag: _wip 
TEMPLATE_VERSION: v1.0.4_blank-template
UMID: 
---

# -

## About

This [[,aka-reference-specced-note|aka-literature-specced-note]] is 


# =

**filename:** `=this.file.path`

* The following code scans the content under a specific H1 using the [[custom-transclusion-parameters,]] syntax (?search_term=).  Every H2 is transformed into a bulletpointed outline of markdown links.⤵

```dataviewjs
const { default: obs } =
  this.app.plugins.plugins["templater-obsidian"].templater
    .current_functions_object.obsidian;

const {metadataCache,vault,workspace,fileManager} = this.app
const {adapter} = vault;

let MARKER = "---Transient Citations";

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
    console.log({argMap})
    MARKER = argMap.search_term || MARKER;
    
    // --

    const avf = workspace.getActiveFile()
    const mdc = metadataCache.getFileCache(avf)


    const embed_texts = gatherEmbedTexts(
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
  
      const rendered_text = await obs
        .MarkdownRenderer
        .render(
          ctx.app,
          `\> [!warning] Transient Citations closest to analysis is empty`,
          $el,
          "",
          ctx.container.component
      );
      ctx.container.append($el)
    }
  }
}

// LOGIC layer

function gatherEmbedTexts(avf = null,mdc=null) {
  if ([avf, mdc].every(Boolean) === false) return [];
  let isReadFlag = false;
  const embed_texts = []
  let ct = 0;
  const mdcMarkdownLinks = mdc.links.map(({original}) => original) || [];
  for (let mdcHeading of mdc.headings) {
    const {level,heading} = mdcHeading;

    if (heading.startsWith(MARKER) && level === 1) {
      isReadFlag = !isReadFlag;
      ct++;
    }
    if (ct > 1 && isReadFlag === false) {
      break;
    }
    if (isReadFlag && level === 2) {
      const embed_text = `#${heading}`
      const mdlink = fileManager
        .generateMarkdownLink(
          avf,
          avf.path,
          avf.basename + embed_text,
          heading
        )
      const isMentioned = mdcMarkdownLinks.includes(mdlink)
      if (!isMentioned) {
        embed_texts.push(
          "`* " + mdlink + "`" + "\n"
        );
      }
    }    
  }
  console.log({embed_texts})
  return embed_texts;
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
    .setButtonText('Refresh')
    .onClick(handleClick.bind(this))
}
function renderUI(embed_texts,cmd) {
      const uiMdText = embed_texts.reduce((a,b) => a + b)
    const $el =dv.paragraph(
      uiMdText, {
        cls: "deleteme"
      }
    );
    if (cmd === "copy")  {
      const text = $el.innerText;
      navigator.clipboard.writeText(text);
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
  
  console.log({displayText})
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
        console.log({err})
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


# ---Transient Local Citations


# ---Transient CommitLog

* v0.0.2 Begin work to remove already linked texts from Sorted Citations
* v0.0.1 Add button for refreshing
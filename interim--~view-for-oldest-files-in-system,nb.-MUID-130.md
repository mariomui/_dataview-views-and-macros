---
tags:
  - _meta
  - _wip
CODELET_SHORTNAME: view-old-files
MUID: MUID-130
---


[[~view-for-unused-MUIDs]]
# -

* [ ] Can i hook into any data attributes?

This codelet is the prototype for "transclusion parameters". The ability to source code from an external markddown and provide it dynamic parameters means you can theoretically have dynamic views.

* `[[addAToB| -a 1 -b 2]]` for one note would show `3`
* `[[addAToB| -a 10 -b 20]]` for another note would show `30`
`	- ![[~view-for-oldest-files-in-system,nb.-TCODEID-3#=| -n 2 nlk]]`
(in use)
## 20-Inlink


> [!abstract]- %%  %% Automated List of Reference Inlinks (v0.0.4)
> * ℹ Commit/design logs are located in this [[π-lists-all-inlinks,nb.-MUID-128|experiment note]]. 
> > `= join( map( sort( map( filter(this.file.inlinks, (link) => meta(link).path != this.file.path), (x) => [ split(meta(x).path, "/")[length(split(meta(x).path, "/")) - 1], x ] ) ), (b) => "• " + choice( length(b[0]) > 28, link( b[1], truncate( regexreplace(b[0], "(-of|of|the|-the|-for|-that|https-|ee)", ""), length( regexreplace(b[0], "(-of|of|the|-the|-for|-that|https-|ee)", "") ) * 0.75 ) ), link(b[1], regexreplace(b[0], "\.md$", "")) ) ), "<br>" )`


# =

**base_filepath-v0.0.2**: *`= this.file.path`* doc-`= this.DOC_VERSION` / ids: `= this.MUID`,`= this.UMID` / lcsh: `= this.heading` / updated on: `= dateformat(this.file.mday, "yyyy-LL-dd")` / file-size: `= round(this.file.size/1024,2)` KB
```dataviewjs

const {default: obs} = this.app.plugins.plugins['templater-obsidian'].templater.current_functions_object.obsidian

const {vault, workspace, metadataCache} = this.app;
const vfs = vault.getFiles();
const currentFilePath = this.currentFilePath;

//knobs
const HISTORY_TO_KEEP_CNT = 5;

// entry
workspace.onLayoutReady(createMain.bind(this))
// bootstrap
function createMain() {
  return main.call(this,
    vfs, renderEpochTimeToDate, getCtime // arity 3
    // a middle ground of the unreadability of objects is to hold two parameters and have the 3rd one be a config.
  );
}


// workhorse
function main(
    vfs, 
    renderEpochTimeToDate,
    getCtime
  ) {
  // business domain


  const embeds = getEmbedsFromVf(
      workspace.getActiveFile()
    );
  const {name} = vault.adapter.
    path.parse(currentFilePath);
  if (!embeds || !name) {
      return;
  }


  const argsMap = convertEmbedDisplayTextToArgMap(
    extractTargetEmbed(name, embeds)
  )
  const historyToKeepCnt = Number(
    argsMap["-n"]
  );

  if (Number.isNaN(historyToKeepCnt)) return;
  
  const oldestVfs = queryOldestFilesInfo(
    vfs, {historyToKeep: historyToKeepCnt}
  );


  for (const oldestVf of oldestVfs) {
    const epochTime = getCtime(oldestVf)
    const dateString = convertEpochTimeToDateString(
      epochTime
    )

    // create a card;
    const $dateString = renderText(`
    ${dateString} \n
    ${oldestVf.path}\n
    `)
    const $div = window.createEl("div", "")
    $div.append($dateString)
    this.container.append($div)
  }
}
// # helper
// knobs for helper functions

// ## business helper
/**
* downstream user of the partial view will have a list
* of embeds, the upstream supplier(the codelet) will have
* the embed name, we target the embed on the downstream doc
* to obtain the transcluded alias which we will turn into
* makeshiftposition arguments.
**/
function extractTargetEmbed(
  embed_name, embeds
) {
  const embed = embeds.find(
  (embed) => {
    const parsed = obs.parseLinktext(
      embed.link
    );
    console.log(embed.link, parsed)
    return parsed.path === embed_name;
  });
  return embed;
}
function convertEmbedDisplayTextToArgMap(embed, config = {
  n: 5
}) {
  console.log({embrud: embed})
  const {displayText} = embed;
  if (!displayText) return { n: HISTORY_TO_KEEP_CNT };
  return parseStringToMap(displayText)
}

function parseStringToMap(str) { 
  const parts = str.trim().split(/\s+/); 
  const map = {}; 
  for (let i = 0; i < parts.length; i += 2) {
   const key = parts[i]; 
   const value = parts[i + 1]; 
   if (key && value) { 
     map[key] = value; 
   } 
  }
return map; 
}

function getEmbedsFromVf(vf) {
  const cache = metadataCache.getFileCache(vf)
console.log({cache})
  return cache?.embeds;
}

function getCtime(vf) {
  return vf ? vf.stat?.ctime : null
}
function queryOldestFilesInfo(vfs, config = {
  historyToKeep: HISTORY_TO_KEEP_CNT
}) {
  let carryCtime = Infinity;
  let carryVf = null;
  const oldestVfs = []
  for (const vf of vfs) {
    const ctime = getCtime(vf)
    if ( ctime < carryCtime) {
      carryCtime = ctime;
      carryVf = vf;
      oldestVfs.push(vf)
    }
    if (oldestVfs.length > config.historyToKeep) {
      oldestVfs.shift();
    }
  }
  return oldestVfs;
}
function convertEpochTimeToDateString(epochTime) {
  if (!epochTime) {
    return null;
  }
  const date = new Date(epochTime);
  return date.toLocaleString()
}
// ## ui helper
function renderEpochTimeToDate(epochTime) {
// timestamp representing the number of milliseconds since the Unix epoch (January 1, 1970).
  const dateString = convertEpochTimeToDateString(
    epochTime
  );
  return dv.paragraph(dateString)
}
function renderText(text) {
  return dv.paragraph(text);
}
```

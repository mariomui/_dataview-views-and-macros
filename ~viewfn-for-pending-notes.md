---
tag: _meta
VERSION: v1.0.0
nextPageIndex: 0
---
# -

This [[Partial-dataview,vis-Noteshippo,]] uses arguments as denoted by the special keyword inside of the note title( [[list-of-affixes,by-Endian,nb.-title-level,vis-Noteshippo#~viewfn]]). 

- [ ] What is the difference between an [[Z_akas/,aka-experiment-specced-note]] and a [[,aka-workboard-note]]? #_todo/to-muse/_upon-noteshippo/regarding-a-note-spec 

# =

> [!info] This partial uses args. 
> Usage: (-n 10 to grab 10 entries) 

```dataviewjs
const { plugins, workspace, vault, metadataCache, fileManager } =
  this.app;

const { default: obs } =
  plugins.plugins["templater-obsidian"].templater
    .current_functions_object.obsidian;

const opn = plugins.plugins["obsidian-pending-notes"];

//knobs
const RESULT_COUNT = 5;

// producer path
const getCurrentfilepath = () => this.currentFilePath;

workspace.onLayoutReady(bootstrap.bind(this));
function bootstrap() {
  (async function (ctx, container) {
    // business domain
    const providing_path = getCurrentfilepath();
    const argMap = extractParams(
      providing_path,
      workspace.getActiveFile(),
    );

    const pvf = vault.getAbstractFileByPath(providing_path);
    const { frontmatter: fm } = metadataCache.getFileCache(pvf);
    const VERSION = fm?.VERSION || "NA";
    const num = (argMap?.n ?? 4) + RESULT_COUNT;
    // ui
    manuButton(
      container,
      `Pull down pending \
      notes ${VERSION}`,
      () => {
        genLoadView(ctx, container, num);
      },
    );
    genMain.call(ctx);
  })(this, this.container);
}
function genMain() {
  const cnt = this.container;

  if (!isRootElExist(this.container)) {
    dv.el("div", "", { cls: "replaceme" });
  }
}

async function genLoadView(app, cnt, num) {
  await genReloadLogic(app, cnt, num, genHandlePnLeaf.bind(app));
}
async function genReloadLogic(app, cnt, num, cb) {
  const leaves = getLeaves();
  const leaf = leaves.first();
  if (!leaf) {
    return await opn.activateView();
  }
  workspace.onLayoutReady(close);

  function close() {
    cb(leaf, cnt, num).then(() => {
      leaf.tabHeaderCloseEl.click();
    });
  }
}

async function genHandlePnLeaf(leaf, cnt, num) {
  const html = leaf.view.contentEl.innerHTML;
  if (isRootElExist(cnt)) {
    cnt.lastChild.remove();
  }
  const md = await obs.htmlToMarkdown(html);
  const regex = new RegExp(`^((?:.*?\n){0,${num}})`, "g");

  const _md = regex.exec(md);
  const el = dv.el("p", _md[1], { cls: "replaceme" });
}

function isRootElExist(cnt) {
  return cnt.lastChild.classList.contains("replaceme");
}

function getLeaves() {
  const leaves = workspace.getLeavesOfType("pending-notes:main");
  return leaves;
}

// UI BOUNDARY
function manuTable(container, headers, matrix) {
  container.lastChild.remove();
  return dv.table(headers, matrix);
}
function manuButton(container, button_title, handleClick) {
  const el = new obs.ButtonComponent(container)
    .setButtonText(button_title)
    .onClick(handleClick);
  return el.buttonEl;
}

// params extraction
// extract params main
function manuParams() {
  return { n: 5 };
}
function extractParams(current_filepath, vf) {
  // v1.01-extractParams

  const embeds = getEmbedsFromVf(vf);
  const { name } = vault.adapter.path.parse(current_filepath);
  const embed = extractTargetEmbed(name, embeds);
  const displayText = embed?.displayText;

  console.log(displayText);
  if (!displayText) {
    return manuParams();
  }

  const argmap = parseStringToMap(displayText);
  const limit = argmap["-n"];
  // /end params extraction

  if (limit) {
    return {
      n: Number(limit),
    };
  }
  return manuParams();

  // helpers
  function getEmbedsFromVf(vf) {
    return metadataCache.getFileCache(vf)?.embeds || [];
  }

  function extractTargetEmbed(embed_name, embeds) {
    const embed = embeds?.find((embed) => {
      const parsed = obs.parseLinktext(embed.link);
      return parsed.path === embed_name;
    });
    return embed;
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
}

```
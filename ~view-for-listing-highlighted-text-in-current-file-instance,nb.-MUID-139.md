---
tags:
  - _misc/_wip
DOC_VERSION: v0.0.4
cssclasses:
  - cards
  - cards-cover
  - cards-2-3
  - table-max
MUID: MUID-139
---
# -

[[~view-for-unused-MUIDs,nb.-MUID-1196]]
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

This [[Partial-dataview,vis-Noteshippo,]] scrapes the entire document for highlighted text and displays it with a checkbox.

Its primary goal is to offer a controlled way to create outlines without depending on [[floating-toc-plugin,bt.-ObsidianMD-app,]]. 

I've added a little checkbox but any mutations are not permanent, id est, a marked checkbox does not retain its mark.


# =

**file_basename**: *`= this.file.name`* *`=this.DOC_VERSION`*

> [!info] List all highlighted material
> if highlighted material is `^like so` within the highlights then the list item will be checked.

```dataviewjs
const {workspace, vault, metadataCache } = this.app

const { default: obs } =
  this.app.plugins.plugins["templater-obsidian"].templater
    .current_functions_object.obsidian;

// button data
const button_title = "Please Wait For Document To Parse...";;
const time = 4000;

// higher order data dependencies
const createClickHandler = (cb) => {
  return (e) => {
    cb(e);
  };
}
const clickHandler = createClickHandler(() => {
    new obs.Notice(button_title, time)
    main.call(this);
    this.container?.lastChild?.remove();
});

// main

workspace.onLayoutReady(
  bootstrap.bind(this)
);
function bootstrap() {
  createButton.call(this, button_title, clickHandler);
  setTimeout(() => {
    main.call(this);
  })
}
function main() {
  
  const vf = workspace.getActiveFile();
  genRun(this)
  
  async function genRun(ctx) {
    // you have line, end ,start, after

    const parsedHighlights = await genParse(vf);
    const lines = parsedHighlights.map(
      (parsedHighlight) => {
        return [{
          line: parsedHighlight.line, 
          ender: parsedHighlight.ender
        }]
      }
    );

    const postFixes = parsedHighlights.map(
      (parsedHighlight) => parsedHighlight.postFix
    );
    const transposedMatrix = await transpose(ctx, lines, postFixes)

    const mdt = dv.markdownTable(
      createColumnHeaders(), 
      transposedMatrix
    );
    await genRenderParagraph.call(this,mdt, {
      attr: {
        style: "width: 100%;"
      }
    })
  }
}

async function genRenderParagraph(mdt,option = {}) {
    await dv.paragraph(mdt, option)
    console.log(this.container, "container");
}

// # helpers

// ## business helpers
async function transpose(ctx, _matrix, postFixes) {
  // postfixes aren't used
  const transposedItems = [];
  const matrix = _matrix.sort((a,b) => {
    return getCharacterValue(a.first().line) > getCharacterValue(b.first().line) ? 1 : -1
  })
  function getCharacterValue(letterOrWord) {
    return String.prototype.charCodeAt.call(letterOrWord,0)
  }
  let i = 0;
  let checkCnt = 0;
  
  for (const [rowIdx, row] of Object.entries(matrix)) {
    for (const item of row) {
      const {
        $el, 
        $frag
      } = await genCreateSpan(
        ctx, {text: item.line}
      );
      const container = await genCreateDiv(ctx);
      const isSuffixed = item.ender === "==!\n"
      const isChecked = item.line.startsWith("^") || isSuffixed
      const checkedAffix = isChecked ? {checked: true} : {};
		 if (isChecked) checkCnt++;
      const $box = window.createEl(
        'input', 
        {
          attr: {
            type: "checkbox",
            parent: $frag,
            ...checkedAffix,
            style: "\
            border-radius:0px;\
            border: 1px solid darkgoldenrod;\
            "
          }
        }
      );
      container.$el.append($box)
      container.$el.append($el)
      if (transposedItems.at(rowIdx) === undefined) {
        transposedItems[rowIdx] = [container.$el];
      } else {
        transposedItems[rowIdx].push(container.$el)
      }
    }
  }
  
  // console.log({matrix,transposedItems})
  if ( (checkCnt === transposedItems.length) && (checkCnt !== 0) ) transposedItems.length = 0;
  return transposedItems;
}
function createColumnHeaders(
  headerTuples = [],
  config = {placeholderHeaderValue: "•"}
) {
  const headerCnt = headerTuples.length
  if (headerCnt === 0) {
    return [config.placeholderHeaderValue]
  }
  const isTuples = headerTuples.some((tuple) => {
    return tuple.length < 2;
  }) === false;
  
  return headerTuples.map(() => {
    return config.placeholderHeaderValue;
  });
}
async function genParse(vf) {
  const input = await vault.cachedRead(vf)
  // const regex = /==([^=]+)==/g; 
  const regex = /==([^=].+)==/g
  let match = ""
  const matches = [];
  while ( (match = regex.exec(input) ) !== null) {
  // console.log({match})
    const start = match.index + match[0].indexOf(match[1]);
    const end = start + match[1].length;
    const ender = input.slice(end, end + 4);

    const postFix = (input.slice(end, end + 13))
    // console.log({lastOne: ender, postFix})
    matches.push({ line: match[1], start, end, ender, postFix});
  }
  return matches;
}

// ## render helpers
function genCreateFrag(ctx) {
  return new Promise((resolve, rej) => {
    ctx.container.win.createFragment(($el) => {
      ctx.app.workspace.onLayoutReady(() => {
        resolve({$el})
      })
    })
  })
}

async function genCreateSpan(ctx, config = {text: ''}) {
  const {text} = config;
  const $frag = await genCreateFrag(ctx);
  return new Promise((resolve, reject) => {
    const domInfo = {
      text,
      parent: $frag
    };
    window.createSpan(domInfo, callback)
    function callback($el) {
      ctx.app.workspace.onLayoutReady(() => {
        resolve({$el, $frag})
      })
    }
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

// BUTTONS

// ui render
function createButton(button_title, clickHandler) {
  new obs.ButtonComponent(
    this.container,
  )
  .setButtonText(button_title)
  .onClick(clickHandler.bind(this));
}


```

# ---Transient Doc Log

- v0.0.4 *2025-04-18*
	- Used [[kaleidoscope,bt.-Macos-app]] to sync with [[~view-for-listing-highlighted-text-in-current-file,nb.-MUID-113]]
	- Reuse and assign MUID to note from muid pool
* v0.0.3 *2024-05-28*
  * Add file base name to identify
  * Add a MUID
  * Rename file so that i can track it by MUID
* v0.0.2 Add Refresh button
  * The button fakes adds a delay so that it seems like the update is happening really fast. What it really does is that it wipes the last element and replaces with the updated content. The act of deleting an element causes a refresh cycle.





---
tags:
  - _wip
DOC_VERSION: v0.0.4
cssclasses:
  - cards
  - cards-cover
  - cards-2-3
  - table-max
---
# -

```dataview
task where file.name = this.file.name
```
## About

This [[Partial-dataview,vis-Noteshippo,]] scrapes the entire document for highlighted text and displays it with a checkbox.

Its primary goal is to offer a controlled way to create outlines without depending on [[floating-toc-plugin,ad-finem-ObsidianMD]]. 

I've added a little checkbox but any mutations are not permanent, id est, a marked checkbox does not retain its mark.
### Mocks

==Testing==

# =

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
    await genRenderParagraph(mdt, {
      attr: {
        style: "width: 100%;"
      }
    })
  }
}

async function genRenderParagraph(mdt,option = {}) {
    await dv.paragraph(mdt, option)
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
  console.log({matrix,transposedItems})
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
  console.log({match})
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


* v0.0.2 Add Refresh button
  * The button fakes adds a delay so that it seems like the update is happening really fast. What it really does is that it wipes the last element and replaces with the updated content. The act of deleting an element causes a refresh cycle.



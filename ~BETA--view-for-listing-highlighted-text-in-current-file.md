---
tag: _wip 
---
# -
```dataview
task where file.name = this.file.name
```
# =

**BETA**
```dataviewjs
const {workspace, vault, metadataCache } = this.app
const vf = workspace.getActiveFile()

this.app.workspace.onLayoutReady(main.bind(this));

function main() {

  genRun(this)
  
  async function genRun(ctx) {
    const parsedHighlights = await genParse(vf);
    const lines = parsedHighlights.map(
      (parsedHighlight) => [parsedHighlight.line]
    );
    const transposedMatrix = await transpose(ctx, lines)

    const mdt = dv.markdownTable(
      createColumnHeaders(), 
      transposedMatrix
    );
    await dv.paragraph(mdt)
  }
}

// # helpers

// ## business helpers
async function transpose(ctx, matrix) {

  const transposedItems = [];
  for (const [rowIdx, row] of Object.entries(matrix)) {
    for (const item of row) {
      const {
        $el, 
        $frag
      } = await genCreateSpan(
        ctx, {text: item}
      );
      const container = await genCreateDiv(ctx);
      const $box = window.createEl(
        'input', 
        {
          attr: {
            type: "checkbox",
            parent: $frag,
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
  const regex = /==([^=]+)==/g; 
  let match = ""
  const matches = [];
  while ( (match = regex.exec(input) ) !== null) {
    const start = match.index + match[0].indexOf(match[1]);
    const end = start + match[1].length;
    matches.push({ line: match[1], start, end });
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

```
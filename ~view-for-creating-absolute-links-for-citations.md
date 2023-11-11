---
TEMPLATE_VERSION: v1.0.7_note-refactor-template
MUID: MUID-1556
CREATION_DATE: 2023-10-06
tag: _wip 
UMID: 
---
# -

![[~view-for-local-tasks-using-a-progress-bar-MUID-698#=|nlk]]

```dataview
task where file.name = this.file.name and !completed
```

```dataview
task where file.name = this.file.name and completed
```

## About

### Reference

![[~view-for-referencing-current-jumpid#=|nlk]]

* †

# =

*`= this.file.name`*

* The following code scans For The Closest ---Transient Citations and transforms the [[anchor-heading,etc]]s into bulletpoints.⤵
```dataviewjs
const { default: obs } =
  this.app.plugins.plugins["templater-obsidian"].templater
    .current_functions_object.obsidian;
const {metadataCache,workspace,fileManager} = this.app

const MARKER = "---Transient Citations";

workspace.onLayoutReady(main.bind(this))
function main() {
  (genMain)(this)
  async function genMain(ctx) {

    const avf = workspace.getActiveFile()
    const mdc = metadataCache.getFileCache(avf)
    
    let isReadFlag = false;
  
    const embed_texts = []
    let ct = 0;
    for (let mdcHeading of mdc.headings) {
      const {level,heading} = mdcHeading;
  
      if (heading.startsWith(MARKER) && level === 1) {
        isReadFlag = !isReadFlag;
        ct++;
      }
      if (ct > 1) break;
      if (isReadFlag && level === 2) {
        const embed_text = `#${heading}`
        const mdlink = fileManager
          .generateMarkdownLink(
            avf,
            avf.path,
            avf.basename + embed_text,
            heading
          )
        embed_texts.push("`* " + mdlink + "`" + "\n");
      }    
    }
    if (embed_texts.length > 1) {
      dv.paragraph(embed_texts.reduce((a,b) => a + b))
    } else {
  
      const { $el } = await genCreateDiv(
        ctx, { text: "" }
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
```

---

# ---Transient
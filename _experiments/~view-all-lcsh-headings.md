---
TEMPLATE_VERSION: v1.0.7
TEMPLATE_SOURCE: "[[10--nascent-spec-template]]"
tags:
  - _wip
UMID: 
MUID: 
nextPageIndex: 15
---

# -

## 10-About

## 11-Reference

# =

**base_filepath**: *`= this.file.name`* doc-`=this.DOC_VERSION` `= this.MUID`/`=this.heading`/`=this.UMID`/

```dataviewjs


const { default: obs } = this.app.plugins.plugins["templater-obsidian"].templater.current_functions_object.obsidian;

const {workspace,fileManager} = this.app;
const lcshHeadings = this.app.metadataCache.getFrontmatterPropertyValuesForKey("heading");
const lcshHeadingsLength = (lcshHeadings.length - 1) || 0;

let initialIndex = 0;
let paginationCnt = 15;
let lastIndex = initialIndex + paginationCnt;

const paginationFieldName = "nextPageIndex"
workspace.onLayoutReady(() => {
  main.call(this)
});


function main(carryover = null) {
  
  const vf = workspace.getActiveFile()
  if (carryover === null) {
    genMain.call(this);
  } 

  if (carryover) {
    genMain.call(this, carryover)
  }

  async function genMain(carryover = null) {

    await fileManager.processFrontMatter(vf, async (fm) => {
      initialIndex = carryover < 0 ? 0 : carryover;
      lastIndex = (lastIndex + paginationCnt) > lcshHeadingsLength ? lcshHeadingsLength : (initialIndex + paginationCnt);
      
      const slices = lcshHeadings.slice(initialIndex, lastIndex)
      const formatted_slices = slices.reduce((chain, slice) => {
        chain += `* ${slice}\n`
        return chain;
      }, "")
      if (!fm.hasOwnProperty(paginationFieldName)) {
        fm[paginationFieldName] = 0;
        workspace.onLayoutReady(() => {

          manuEmbed(this, formatted_slices , this.container);
          renderButtons.call(this, fm);
        })
        return;
        // return genMain.call(this, 0);
      }
      

      if (carryover !== null) {

        fm[paginationFieldName] = carryover

        const child = Array.from(this.container.children)
          .find((l) => {
            return l.nodeName === "UL"
          })
        if (child) this.container.removeChild(child)

        // render
        return manuEmbed(this, formatted_slices , this.container);

      }
      renderButtons.call(this, fm);

      function renderButtons(fm) {
        if (Number.isNumber(fm[paginationFieldName])) {
          renderButton.call(this,"Prev", async () => {  
            
            const difference = fm[paginationFieldName] - paginationCnt;
            fm[paginationFieldName] = difference < 0 ? 0 : difference;  

            genMain.call(this, fm[paginationFieldName]);
            // return new obs.Notice(fm[paginationFieldName], 7000);
          });
          renderButton.call(this,"Next", async () => {  
            fm[paginationFieldName] = fm[paginationFieldName] + paginationCnt;

            genMain.call(this, fm[paginationFieldName]);
            // return new obs.Notice(fm[paginationFieldName], 7000);
          });
          renderButton.call(this, "Reset", async () => {
            fm[paginationFieldName] = 0;
            genMain.call(this, 0)
          })
        } 
      }
    })
  }
}

//dv.list(lcshHeadings)
// # Rendering

function renderButton(button_title, clickHandler) {
  new obs.ButtonComponent(
    this.container
  )
  .setButtonText(button_title)
  .onClick(clickHandler.bind(this))
}


function manuEmbed(
  ctx,
  markdown,
  container,
  sourcePath,
  isEmbed = false
) {
  // static render(app: App, markdown: string, el: HTMLElement, sourcePath: string, component: Component): Promise<void>;

  const render = isEmbed === true ? obs.MarkdownPreviewView.render : obs.MarkdownRenderer.render
  return render
    .call(
      ctx,
      ctx.app,
      markdown,
      container,
      sourcePath,
      ctx.component
    );
}
```

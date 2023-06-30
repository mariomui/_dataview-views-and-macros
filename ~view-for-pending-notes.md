---
tag: _meta 
---

# =
```dataviewjs
const {default: obs} = this.app.plugins.plugins['templater-obsidian'].templater.current_functions_object.obsidian

const opn = this.app.plugins.plugins["obsidian-pending-notes"]

if (!dv?.fig?.should) {
    new obs.Notice('should is here', 700)
    boostrap.call(this, main.bind(this))
}

function boostrap(main) {

    if (!dv?.fig) {
        main();
        this.fig = Object.assign({}, {
            should: true,
        })
    } else {
        // this.container.lastChild.remove()
        
        // main()
    }
}

function main() {
    manuButton(
        this.container, 
        'Pull down pending notes',
        loadView.bind(this)
    );
    
    if (!isRootElExist(this.container)) {
        dv.el("div", "", {cls: "replaceme"})
    }

    //loadView.call(this)
    
    
    function loadView() {
        (async function(app, cnt) {
            await reloadLogic(
                app,
                cnt, 
                handlePnLeaf.bind(this)
            )
        })(this.app, this.container)
    }
    function handlePnLeaf(leaf, cnt) {
        const html = leaf.view.contentEl.innerHTML;
        (async function(leaf, html) {
            if (isRootElExist(cnt)) {
                cnt.lastChild.remove()
            }
            const md = await obs.htmlToMarkdown(html)
 
            const regex = /^((?:.*?\n){0,20})/; 
            const _md = regex.exec(md);
            const el = dv.el(
                "p", _md[1], 
                {cls: "replaceme"}
            );
        
        })(leaf, html)
    }    

    async function reloadLogic(app, cnt, cb) {
        
        const leaves = getLeaves(app);
        const leaf = leaves.first()

        
        if (!leaf) {
            await opn.activateView()
        } else {
            setTimeout(() => {
                cb(leaf, cnt)
                leaf.tabHeaderCloseEl.click()
            }, 150)
        }
    }
}

function isRootElExist(cnt) {
    return cnt
            .lastChild
            .classList
            .contains("replaceme")
}
function getLeaves(app) {
    const leaves = app
        .workspace
        .getLeavesOfType("pending-notes:main")
    return leaves;
}

function manuTable(container, headers, matrix) {
    container.lastChild.remove();
    return dv.table(headers, matrix)
}
function manuButton(container,button_title,handleClick) {
    const el = new obs.ButtonComponent(container)
        .setButtonText(button_title)
        .onClick(handleClick)
    return el.buttonEl;
}

```
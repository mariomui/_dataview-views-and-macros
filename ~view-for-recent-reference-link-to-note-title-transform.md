---
tag: _meta
PARTIAL_VERSION: v1.0.1
COMMENTS: 
    - v1.0.1 Replaced settimeout with onlayout from obsmd core
    - v1.0.0 prototpye
---
# -
## About

This note contains a partial view that displays a converted reference.
The reference is automatically parsed from the active file, from the last item inside any markdown header named "Reference".

* Why use this partial view?
    * Because
        * 📁 *Aliases are troublesome dependencies*
            * Compare [[🔉-207-Adverbs-Why-All-the-Hate-Mythcreants-https-mythcreants-com-blog-podcasts-207-adverbs-why-all-the-hate#=]] to [[🔉-207-Adverbs-Why-All-the-Hate-Mythcreants-https-mythcreants-com-blog-podcasts-207-adverbs-why-all-the-hate|[207 – Adverbs, Why All the Hate? – Mythcreants](https://mythcreants.com/blog/podcasts/207-adverbs-why-all-the-hate/)]]
            * When the alias changes in this source note, obsidian does not rename the links whereas the note title propagates the changes.
        * 📁 *Overuse of [[Domain-specific-language#=|DSL]]  such as [[Dataview-plugin-for-ObsidianMD#=|DVJS]] should be avoided to avoid [[rigidity-ala-software-design]]*
            * 

*Details* 
This dvjs view converts the last item inside of a header named Reference into a standard obsidian title, removing:
* Brackets, quotes, slashes, parens, question marks, colons
    * `["“”[\]()?:,.\\/\s]+)`

With such a contract in place, downwind API consumers (mentions,backlinks) can refer to 🔉

### Reference
* † 🔉 [207 – Adverbs, Why All the Hate? – Mythcreants](https://mythcreants.com/blog/podcasts/207-adverbs-why-all-the-hate/)

# =

```dataviewjs
// PARTIAL_VERSION: v1.0.1
const {default: obs} = this.app.plugins.plugins['templater-obsidian'].templater.current_functions_object.obsidian

// data
const button_title = "♻",
    click_text = "♻",
    time = 700;

// higher order data dependencies
const createClickHandler = (data_text, doExtractLink) => {
    return () => {
        doExtractLink()
        // new obs.Notice(data_text,7000);
    }
}
const clickHandler = createClickHandler(click_text, doExtractLink.bind(this))

// main
main.call(
    this,
    button_title, 
    click_text, 
    clickHandler
);

// ui render
function main(
    button_title,
    click_text,
    clickHandler
) {
    if (!this.marioInjectionfig?.refreshRefTextBtn) {
        this.app.workspace.onLayoutReady(
            timeoutHandler.bind(this)
        );
    }
    
}
function timeoutHandler() {
    this.marioInjectionfig = {
        ...this.marioInjectionfig
    }
    
    this.marioInjectionfig
        .refreshRefTextBtn = true;
    new obs
        .ButtonComponent(this.container)
        .setButtonText(button_title)
        .onClick(clickHandler.bind(this))
    
    doExtractLink.call(this);
}

function doExtractLink() {
    const abf = this.app.workspace.getActiveFile();
    const {headings} = this.app
        .metadataCache
        .getCache(abf.path);
    
    
    const markers = [];
    let toggle = false;
    for (const heading of headings) {
    
        if (heading.display === "Reference" || toggle === true) {
            toggle = !toggle;
            markers.push(heading);
        }
        
    }
    const [start,end] = markers
    
    void (async function t(app) {
        const read = await app.vault.cachedRead(abf)
        const reads = read.split('\n')
        const title = 
            reads
                .slice(
                    start
                        .position
                        .start
                        .line,
                     end
                         .position
                         .start
                         .line
                 ).join("").split("*")
                 .slice(-1).first();
        const modifiedString = title
            .replace(/[|"“”[\]()?:,.\\/\s]+/g, "-")
            .replace(/-+/g, "-")
            .replace(/^-|-$/g, "")
            .replaceAll("–","-")
            .split("-").filter(Boolean).join("-")
            .replaceAll("-_-", "-")
            console.log({modifiedString})
            dv.paragraph(" " + modifiedString);
    })(this.app)

}


```


# ---Transient Sandbox

## Archived Versions

### v1.0.0

Setting buttons requires the document being ready before placing on the page. The [[engineering-style-differential-report-regarding-button-creation-in-obsidianmd]] using dataviewjs states that the document must be ready before dom insertion.  

* ℹ
    * v1.0.0 chose to use setTimeout to achieve documents.addEventListener("DOMContentReady") because the author(me) worried about cleanup. The nature of blackbox Obisdianmd programming means I cannot reliably ascertain the rules for memory clean up so I opted for setTimeout. (🤔 [[Leaky-abstraction]] rears its ugly head again!)

Eventually, this folder, containing all my dataview-powered partials,  shall be handed over to a [[version-control-system]], the more appropriate tool edifying change-log and code-related meta-tasks. To spur myself to quicken the migration, I've set the following rule:
* Previous versions of code will not be included in Archived Versions API as of *2023-06-28*


- [ ] Do all the cta button creation methods require a document onReady-like callback helper?
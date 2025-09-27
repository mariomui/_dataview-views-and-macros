---
tags:
  - _meta
DOC_VERSION: v1.0.5
MUID: MUID-115
---


# -

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

[[~view-for-unused-MUIDs,nb.-MUID-1196]]

> [!warning] Code breaks if you add new extra lines pre/post to the reference target
> ⚠ I dont like how shitty this code is. Overengineer the next version with codemirror stuff

This note contains a partial view that displays a converted reference.
The reference is automatically parsed from the active file, from the last item inside any markdown header named "Reference".

* ~~Why use this partial view?~~
  * Because
    * 📁 _Aliases are troublesome dependencies_
      * Compare
        * [[ß-207-Adverbs-Why-All-the-Hate-Mythcreants-https-mythcreants-com-blog-podcasts-207-adverbs-why-all-the-hate#=]]
          * to
        * [[ß-207-Adverbs-Why-All-the-Hate-Mythcreants-https-mythcreants-com-blog-podcasts-207-adverbs-why-all-the-hate|[207 – Adverbs, Why All the Hate? – Mythcreants](https://mythcreants.com/blog/podcasts/207-adverbs-why-all-the-hate/)]]
      * ~~When the alias changes in this source note, obsidian does not rename the links whereas the note title propagates the changes.~~
        * 🤔 I dont use aliases anymore, so the example is rendered moot.
    * 📁 _Overuse of [[Domain-specific-language,#=|DSL]] such as [[Dataview-plugin,bt.-Obsidianmd-app,#=|DVJS]] should be avoided to avoid [[rigidity,vis-System-design,etc]]__

_Details_
This dvjs view converts the last item inside of a header named Reference into a standard obsidian title, removing:

* Brackets, quotes, slashes, parens, question marks, colons
  * `["“”[\]()?:,.\\/\s]+)`

With such a contract in place, downwind API consumers (mentions,backlinks) can refer to 🔉

### 11-Reference

> [!info] [[~view-for-referencing-current-jumpid]]

* † ß [207 – Adverbs, Why All the Hate? – Mythcreants](https://mythcreants.com/blog/podcasts/207-adverbs-why-all-the-hate/)

# =

## Normalized Reference

```dataviewjs
const {
  plugins, 
  workspace, 
  vault, 
  metadataCache
} = this.app;

const {default: obs} = plugins.plugins['templater-obsidian'].templater.current_functions_object.obsidian;

// toggles
const MARKER = "Reference"

// data
const button_title = "♻",
      click_text = "♻",
      container = this.container;

// entry
workspace
  .onLayoutReady(bootstrap.bind(this));

// entry capsule

function bootstrap() {
  // ui
  new obs
    .ButtonComponent(container)
    .setButtonText(button_title)
    .onClick(doExtractLink.bind(this))

  // workhorse
  mainer.call(
    this,
    doExtractLink.bind(this)
  )
}

// workhorse
function mainer(
    doMain
) {
  doMain.call(this);
}

function doExtractLink() {
    const abf = workspace.getActiveFile();
    const {headings} = metadataCache
        .getFileCache(abf);
    const markers = [];
    
    if (!headings) return;
    
    let toggle = false;
    console.log({headings})
    for (const heading of headings) {

        if (
          heading.heading.contains(MARKER) || 
          toggle === true
        ) {
            toggle = !toggle;
            markers.push(heading);
        }

    }

    const [start,end] = markers;

    ((async function t(app) {
        const read = await vault.cachedRead(abf)
        const reads = read.split('\n')
        // console.log({reads, start, end})
        let title =
            reads
                .slice(
                    start
                        .position
                        .start
                        .line + 2,
                     end
                         .position
                         .start
                         .line
                 ).join("").split("*")
                 .slice(-1).first();
        if (title.length) {
          title = searchStringAndReplace(title, /[a-zA-Z]/, (match) => {
            const m = match.toLowerCase();
            return m;
          })
        } else {
          return "x";
        }
        const modifiedString = title
            .replace(/[|"“”[\]()?:,.\\/\s]+/g, "-")
            .replace(/#+/,"-")
            .replace(/-+/g, "-")
            .replace(/^-|-$/g, "")
            .replaceAll("–","-")
            .split("-").filter(Boolean).join("-")
            .replaceAll("-_-", "-")
            dv.paragraph(" " + modifiedString);
    }).bind(this))(this.app)

}

/**
 * @param input string
**/
function searchStringAndReplace(input, pattern, replacePredicate) {
  let result = input.replace(pattern, replacePredicate);
  return result;
}


```

---

# ---Transient

# ---Transient Local Archives

## LA--v1.0.0

Setting buttons requires the document being ready before placing on the page. The [[engineering-style-differential-report-regarding-button-creation-in-obsidianmd]] using dataviewjs states that the document must be ready before dom insertion.

* ℹ
  * v1.0.0 chose to use setTimeout to achieve documents.addEventListener("DOMContentReady") because the author(me) worried about cleanup. The nature of blackbox Obisdianmd programming means I cannot reliably ascertain the rules for memory clean up so I opted for setTimeout. (🤔 [[Law-of-leaky-abstraction,vis-Coding,]] rears its ugly head again!)

Eventually, this folder, containing all my dataview-powered partials, shall be handed over to a [[version-control-system]], the more appropriate tool edifying change-log and code-related meta-tasks. To spur myself to quicken the migration, I've set the following rule:

* Previous versions of code will not be included in Archived Versions API as of _2023-06-28_

* [ ] Do all the cta button creation methods require a document onReady-like callback helper?

testing is auto fixing but not arto fix stuff you dont manually have a dictionary for.

# ---Transient Doc Log

[[transient-doc-log-endpoint,bt.-Noteshippo-heading-api,]]

- v1.0.7 *2025-04-23*
	- Remove the idea of a commit version. Conflating it to a doc version. 
		- Prefer subtracting before adding. Prefer waiting before merging. Prefer merging before adding. [[prefer-procrastination-over-complexity,vis-System-design,prima-facie]] 
			- The COMMIT VERSION inside of the yaml is a prime example of premature feature addition. If I had waited, I would have seen that the doc version would serve as an adequate substitute for the doc version. If I had considered the other yaml fields I would have also not made a system design error. Deleting however, loses track of the history of misakes, so it's important to document the mistakes in an easily retrievable format.
			- ≠--commit-version,bt.-Noteshippo-frontmatter-field-key 
			- #_noteshippo/bt/noteshippo-frontmatter-property-name 
 #_noteshippo/vis/system-design 
* v1.0.6 _2025-03-12_
  * lower case the captured hovertext to indicate that these files need to be manually written as mature.
* v1.0.5 _2024-07-17_
  * make it so that it takes the first REFERENCE so that i can do 11-Reference rather htan hardcode it.
    * in the future consider creating a custom transclusion version of this partial dataview
* v1.0.4 _2024-06-26_
  * Add MUID to title and to yaml
* v1.0.3 Graduate codelet from interim to a Noteshippo mechanism.
  * 🤔 When the linked mentions grow to > 30, that's prolly when the interim tag can be removed.

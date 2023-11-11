---
tag: _meta
DOC_VERSION: v1.0.0
CODELET_SHORT_NAME: cta-button-to-open-ytranscript-text
---

# -

![[~view-for-local-tasks-using-a-progress-bar-MUID-698#=|nlk]]

```dataview
task where file.name = this.file.name and !completed
```

```dataview
task where file.name = this.file.name and completed
```

- [ ] Prettify unformatted code.
- [ ] Conform function names to mario standard.
- [ ] Consider deleting
  - 🤔 No notes use this partial.

### Reference

* https://youtube.com/dd

# =

// ⤵ this thing below is pretty much broken. I forgot the expected behavior.


- [ ] This entire doc needs to be archived because i deleted Buttons plugins ⤵
```dataviewjs
let $buttonRef = null;
const links = dv.current().file.lists.values;

const prefix = "https://www.youtube.com";

const url = findUrl(links, prefix);
function findUrl(links, sep) {
  const found = links?.findLast((link) => {
    return link?.text?.contains(sep);
  });
  return found?.text?.split(sep)[1];
}

const { plugins } = this.app.plugins;
const { createButton } = plugins["buttons"];
const yt = plugins["ytranscript"];

const makeOpenBtn = (container, url, name) =>
  createButton({
    app,
    el: container,
    args: {
      name,
    },
    clickOverride: {
      click: openView.bind(this),
      params: [
        {
          url,
          getYtTabEl: getYtTabEl.bind(this),
          yt,
        },
      ],
    },
  });
makeOpenBtn(this.container, "", "♻");
console.log({url})
if (url?.length) {
  console.log({ url });
  ubermain(main.bind(this), prefix + url);
}

function ubermain(main, url) {
  console.log({ url });
  main(url);
}

function main(url) {
  this.container.style = "border: 3px solid lightblue;margin: .5em 0;";
  const row1 = [
    "YTranscript",
    "。。。。",
    makeOpenBtn(this.container, url, "Open"),
  ];
  console.log({row1})
  $buttonRef = row1[2];
  dv.table("", [row1]);
}
function openView({ url, yt, getYtTabEl }) {
  const ytTab = getYtTabEl();
  console.log(this.container, "button");
  if (ytTab?.tabHeaderCloseEl) {
    ytTab.tabHeaderCloseEl.click();
    $buttonRef.innerText = "Open";
    console.log($buttonRef.previousSibling);
  } else {
    $buttonRef.innerText = "Close";
    yt.openView(url);
  }
}
function getYtTabEl() {
  return this.app.workspace.rightSplit.children[0]?.children.find(
    findViewPredicate,
  );
}
function findViewPredicate({ tabHeaderEl }) {
  return tabHeaderEl?.dataset?.type === "transcript-view";
}

```

---

```dataviewjs
const {workspace} = this.app;
const {plugins} = this.app.plugins

let $buttonRef = null;



const {createButton} = plugins["buttons"]
const yt = plugins['ytranscript'];

const makeOpenBtn = (
  container, url, name
) => createButton({
    app,
    el: container,
    args: {
        name,
    },
    clickOverride: {
        click: openView.bind(this),
        params: [{
            url,
            getYtTabEl: getYtTabEl.bind(this),
            yt
        }],
    },
})



workspace.onLayoutReady(ubermain.bind(this))



function ubermain() {
  const links = dv.current().file.lists.values;

  const prefix = 'https://www.youtube.com';

  const url = findUrl(links, prefix)
  function findUrl(links, sep) {
      const found = links?.findLast((link) => {
          return link?.text?.contains(sep)
      })
      return found?.text?.split(sep)[1];
  }
  return;
  makeOpenBtn(this.container, "", "♻")
  console.log({url})
  if (url?.length) {
      main.call(this, prefix + url)
  }
}

function main(url) {
    this.container.style = 'border: 3px solid lightblue;margin: .5em 0;'
    const row1 = [
        'YTranscript',
        '。。。。',
        makeOpenBtn(this.container,url, 'Open')
    ];
    $buttonRef = row1[2]
    dv.table(
        '',
        [
            row1,
        ],
    )



}
function openView({url, yt, getYtTabEl}) {

    const ytTab = getYtTabEl()
    console.log(this.container, 'button')
    if (ytTab?.tabHeaderCloseEl) {
        ytTab.tabHeaderCloseEl.click()
        $buttonRef.innerText = 'Open';
console.log($buttonRef.previousSibling);
    } else {
        $buttonRef.innerText = 'Close';
        yt.openView(url)
    }
}
function getYtTabEl() {
    return this.app.workspace.rightSplit
        .children[0]
        ?.children
        .find(findViewPredicate)
}
function findViewPredicate({tabHeaderEl}) {
    return tabHeaderEl
            ?.dataset?.type
                === 'transcript-view'
}
```

# ---Transient

🤔 The code is problematic with spaces and Reference and hard to read code with slicing.

- The code below is a copy of the [[~view-for-recent-reference-link-to-note-title-transform]]

```js
~~~dataviewjs
const {workspace} = this.app;
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
    const read = await app.vault.read(abf)
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
    const ui = title
      .slice(-1).first()
      .replaceAll("/", " ")
      .replace(","," ")
      .replace("-", "")
      .split(" ")
      .filter(f => f.length > 1).join("-")

    dv.paragraph(ui, {cls: "note"})
})(this.app)
~~~
```

# ---Transient

```js
~~~dataviewjs
let $buttonRef = null;
const links = dv.current().file.lists.values;

const prefix = "https://www.youtube.com";

const url = findUrl(links, prefix);
function findUrl(links, sep) {
  const found = links?.findLast((link) => {
    return link?.text?.contains(sep);
  });
  return found?.text?.split(sep)[1];
}

const { plugins } = this.app.plugins;
const { createButton } = plugins["buttons"];
const yt = plugins["ytranscript"];

const makeOpenBtn = (container, url, name) =>
  createButton({
    app,
    el: container,
    args: {
      name,
    },
    clickOverride: {
      click: openView.bind(this),
      params: [
        {
          url,
          getYtTabEl: getYtTabEl.bind(this),
          yt,
        },
      ],
    },
  });
makeOpenBtn(this.container, "", "♻");

if (url?.length) {
  console.log({ url });
  ubermain(main.bind(this), prefix + url);
}

function ubermain(main, url) {

  main(url);
}

function main(url) {
  this.container.style = "border: 3px solid lightblue;margin: .5em 0;";
  const row1 = [
    "YTranscript",
    "。。。。",
    makeOpenBtn(this.container, url, "Open"),
  ];
  $buttonRef = row1[2];
  dv.table("", [row1]);
}
function openView({ url, yt, getYtTabEl }) {
  const ytTab = getYtTabEl();
  console.log(this.container, "button");
  if (ytTab?.tabHeaderCloseEl) {
    ytTab.tabHeaderCloseEl.click();
    $buttonRef.innerText = "Open";
    console.log($buttonRef.previousSibling);
  } else {
    $buttonRef.innerText = "Close";
    yt.openView(url);
  }
}
function getYtTabEl() {
  return this.app.workspace.rightSplit.children[0]?.children.find(
    findViewPredicate,
  );
}
function findViewPredicate({ tabHeaderEl }) {
  return tabHeaderEl?.dataset?.type === "transcript-view";
}

~~~
```

---
TEMPLATE_VERSION: v1.0.4
TEMPLATE_SOURCE: "[[10--nascent-spec-template]]"
tags:
  - _wip
UMID: 
MUID: 
cssclasses: []
---

# -

## About

# =

**file_basename**: *`= this.file.name`* doc-`=this.DOC_VERSION`

* 📃✨ 
  * The codelet below randomly generates 5 links.
* ⚙
  * The implementation is that is RNGs a number between 0 and the number of files in vault, and inserts it inside an array of 5. 
* 🔎  
  * 133, 2334, 53, 23, 1 <--- example
    * ⚙ 
      * This integer list is then used to map the array to the file
      * The api gives a list of files that are indexed so its only a matter of supplying a random index number within bounds.
* 🤔 I really don't like the fact that i use dv.paragraph to do my rendering.
* ![[wip--≈-view-to-randomly-show-five-links#LR--code--show five files|LR--code--show five files]]


# ---Transient Jobs

![[~viewfn-for-sluicing-header-links-for-citations-MUID-1560#=|?search_term=---Transient Local&t=nlknoui-scroll]]

# ---Transient Local Citations

# ---Transient Local Resources



## LR--code--show five files

~~~dataviewjs 
const {vault, fileManager} = this.app;
const abstractFiles = vault.getFiles()

const unique = {}

//knobs
const cnt = 5;

// business logic
const randomNums = Array(cnt).fill(null).map((x,idx) => {
  let num = randomNum(0,abstractFiles.length)
  while (unique[num]) {
    num = randomNum(0,abstractFiles.length)
  }
  unique[num] = true;
  return num; 
})

const randomAbstractFiles = randomNums.map((num) => abstractFiles[num]);

const randomPreLinkPacks = randomAbstractFiles.map((file) => {
  return ({
    file, 
    file_path: file.path
  })
});

const randomLinks = randomPreLinkPacks.map(prepareLinkPredicate)
function prepareLinkPredicate({file, file_path}) {
  return fileManager.generateMarkdownLink(file, file_path);
} 

const transposedItems = transpose(this, [randomLinks]);
const md = dv.markdownTable(
  ["*"], [transposedItems]
);

// ui boundary
dv.paragraph(md) 

// util helpers
function randomNum(min, max) {
    return Math.floor(Math.random() * (max - min)) + min
}

function transpose(ctx, matrix) {
  const transposedItems = []
  for (const [rowIdx, row] of Object.entries(matrix)) {
    for (const item of row) {
      if (transposedItems.at(rowIdx) === undefined) {
        transposedItems[rowIdx] = [item];
      } else {
        transposedItems[rowIdx].push(item)
      }
    }
  }
  return transposedItems;
}
~~~

# ---Transient


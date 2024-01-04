---
---
# -

This [[Partial-dataview,vis-Noteshippo,]] uses a lot of [[22--inbox-list-of-api,by-node-primitive,b.t.-ObsidianMD-app#generateMarkdownLink]]
# =

## TABLE OF CONTENTS

~~~dataviewjs

const {fileManager,workspace,metadataCache} = this.app

workspace.onLayoutReady(main.bind(this))
function main() {
  const vf = workspace.getActiveFile()
  const mdc = metadataCache.getFileCache(vf)
  let stack = []
  let flag = true
  mdc.headings.map(mapByHeading)
  
  function mapByHeading({heading,level}, idx) {
      const link = fileManager.generateMarkdownLink(vf, vf.path, vf.path+"#"+heading,heading);
    const hheading = "#" + heading;
    const content = `* ${link}`;
    if (level === 1 && flag) {
      stack.push(`* # ${link}`);
    } 
    if (level > 1 && flag) {
      const _content = createNestedHeadingContent(level - 1, content);
      stack.push(_content)
    }
    
    return heading
    
    function createNestedHeadingContent(sandwichCnt, content) {
      for (let i = 0; i < sandwichCnt; i++) {
        content = Array(content);
      }
      return content;
    }
  }
  console.log(stack)
  
  dv.list(stack)
}
~~~

# ---Transient Fork Using Floating TOC as a scraper

```dataviewjs
const {plugins,fileManager,workspace} = this.app

workspace.onLayoutReady(main.bind(this))

function main() {
  const hds = plugins.plugins['floating-toc'].headingdata
  const vf = workspace.getActiveFile();

  
  const _hds = hds.map(mapToListLink)
  dv.list(_hds)
  
  function mapToListLink(hd) {
    const heading = generateLinkFromHeading(vf,hd.heading,hd.level);
    if (hd.level === 1) {
      return `* # ${hd.heading}`
    }
    return createNestedHeadingContent(hd.level - 1, heading)
  }
}
function createNestedHeadingContent(
  sandwichCnt,
  content) {
  for (let i = 0; i < sandwichCnt; i++) {
    content = Array(content);
  }
  return content;
}
function generateLinkFromHeading(vf, heading, level) {
  const link = fileManager.generateMarkdownLink(
    vf, 
    vf.path, 
    vf.path+"#"+heading,
    heading
  );
  return "* " + "#".repeat(level) +" " + link
}
```

# ---Transient Local Resources

## LR--Demo Image For Github
![[Screen Shot 2023-09-17 at 12.08.54 PM.png]]
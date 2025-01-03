---
tags:
  - _meta
DOC_VERSION: v0.0.1
---

# -

~~Uses the randomizing alogirthim found in  [Randomly order rows in a Dataview query - Share & showcase - Obsidian Forum](https://forum.obsidian.md/t/randomly-order-rows-in-a-dataview-query/46989?u=oirammui) to~~  create a 100 sample set, where each file's calculated [[IO-composite-key,vis-Noteshippo-dashboard,]] represents a file's inactivity with respects to input and output link ratio.

- ! The current implementation filters all system notes using dvjs. This results in performance issues.
  - 🔑 *caching*
    - Cache previous results using the inputs and the current date as a key.
    - 📉 Involves,
      - external dependencies. 
      - state management
  - & 🔑 *O(Limit)* 
    - Rathan loop through each item in the set, create random integer offsets, and use those as the items to calculate the status from.
    - 📈 Easy to implement.
- [ ] Give the dataview the ability to use  [[custom-transclusion-parameters,cf.-Kanzi,vis-ObisidianMD-app,]]

# =

```dataviewjs
const limit = 50;
const current_filepath = dv.currentFilePath;
const { default: obs } =
  this.app.plugins.plugins["templater-obsidian"].templater
    .current_functions_object.obsidian;
const { workspace, metadataCache, vault, fileManager} = this.app;

workspace.onLayoutReady(main.bind(this));
function main() {
  (function (genWorkhorse) {
    genWorkhorse();
  })(genWorkhorse.bind(this));
}
function genWorkhorse() {

  const statusMap = {
    "00": "Orphan",
    10: "Unused",
    "01": "Orphan+",
  };
  const wip_pages = dv.pages("#_wip").values;


  const futureDvPages = Array(limit).fill(null).map((l) => {
    const idx = calcRandomNum(0, wip_pages.length - 1)
    
    return wip_pages[idx]
  })

  const shuffled = dv.array(futureDvPages).file
    .map((file) => {
      const outlinksCount = file.outlinks.length;
      const inlinksCount = file.inlinks.length;
      const compositeIOKey = createCompositeKey({
        outlinksCount,
        inlinksCount,
      });
      const link = generateMarkdownLinkByFileName(file.name);
      console.log({link})
      if (outlinksCount < 8 && inlinksCount < 8) {
        return {
          name: link || file,
          status: statusMap[compositeIOKey] || null,
        };
      }
      return null;
    })
    .filter((x) => {
      if (!x) return false;
      return !!x.status;

    })
    .sort(file => file.status)

  const vf = vault.getAbstractFileByPath(
    current_filepath
  );
  const {
    frontmatter: {
      VERSION 
    }
  } = metadataCache.getFileCache(vf)
  const headers = [`FILE (${VERSION})`, "STATUS"]
  renderTable.call(this, headers, shuffled.values);
}

function generateMarkdownLinkByFileName(file_basename) {
  const vf = metadataCache
    .getFirstLinkpathDest(file_basename, "");

  return fileManager.generateMarkdownLink(vf, "")
}
function renderTable(headers, data) {
  const transes = data.map(datum => {
    return Object.values(datum);
  })

  dv.table(headers, transes)
}

function manuCompositeKey() {
  return {
    inlinksCount: 0,
    outlinksCount: 0,
  };
}
function createCompositeKey(config) {
  const { inlinksCount, outlinksCount } = config;
  return leftPad(`${inlinksCount}${outlinksCount}`, 2);
}

function leftPad(text, places, filler = "0") {
  const leftPlaces = Math.abs(text.length - places);
  return Array(leftPlaces).fill(filler).concat(text).join("");
}
function calcRandomNum(min,max) {
  return Math.ceil((max * Math.random()) - min) + min;
  // 0 - .9
  // 1    6
  //    4.9999
}

```

# ---Transient

```sql
TABLE 
 io, 
 file.frontmatter.MUID as "MUID-####", 
 status
FROM #_wip
LIMIT 200
FLATTEN date(now) as Now
FLATTEN (file.mtime.year + file.mtime.hour + file.mtime.day + file.mtime.hour + file.mtime.minute + file.mtime.second + file.size + Now.hour + Now.minute + Now.second) * 15485863 as Hash
FLATTEN ((Hash * Hash * Hash) % 2038074743) / 2038074743 as Rand 
SORT Rand
LIMIT 80
FLATTEN (date(now) - file.mtime) as tilltoday
WHERE tilltoday.minutes > 10000
FLATTEN length(file.inlinks) as inlink_cnt
FLATTEN length(file.outlinks) as outlink_cnt
WHERE inlink_cnt < 8
AND outlink_cnt < 8
FLATTEN (
 join(
  list(
   inlink_cnt, outlink_cnt
  ) , [""] 
 ) 
) as "io"
FLATTEN (
 {"00": "Orphan", "10": "Unused", "01": "Orphan+"}
) as ioToStatusMapping
FLATTEN string(
 extract(ioToStatusMapping, io)
) as p_raw_status
FLATTEN (
 regexreplace(
  p_raw_status,"[^a-zA-Z\+]+", 
  ""
 )
) as status
FLATTEN (
 join(
  map(
   list(
    econtains(file.etags, "#_meta"), 
    econtains(file.etags,"#_wip"),
    file.name = "default"
   ),
   ((item) => choice(item, "1", "0"))
  ), [""]
 )
) as p_tags_flag
FLATTEN (
 econtains(
  list("010"), p_tags_flag
 )
) as isARealNote
FLATTEN (
 econtains(
  list("00", "01", "10"), io
 )
) as isLowActivity
WHERE isARealNote AND isLowActivity
SORT file.mtime desc, io asc, file.name asc
LIMIT 40
```

~~~sql
```dataview
TABLE 
 io, 
 file.frontmatter.MUID as "MUID-####", 
 status
FROM #_wip
FLATTEN length(file.inlinks) as inlink_cnt
FLATTEN length(file.outlinks) as outlink_cnt
WHERE inlink_cnt < 8
AND outlink_cnt < 8
FLATTEN (
 join(
  list(
   inlink_cnt, outlink_cnt
  ) , [""] 
 ) 
) as "io"
FLATTEN (
 {"00": "Orphan", "10": "Unused", "01": "Orphan+"}
) as ioToStatusMapping
FLATTEN string(
 extract(ioToStatusMapping, io)
) as p_raw_status
FLATTEN (
 regexreplace(
  p_raw_status,"[^a-zA-Z\+]+", 
  ""
 )
) as status
FLATTEN (
 join(
  map(
   list(
    econtains(file.etags, "#_meta"), 
    econtains(file.etags,"#_wip"),
    file.name = "default"
   ),
   ((item) => choice(item, "1", "0"))
  ), [""]
 )
) as p_tags_flag
FLATTEN (
 econtains(
  list("010"), p_tags_flag
 )
) as isARealNote
FLATTEN (
 econtains(
  list("00", "01", "10"), io
 )
) as isLowActivity
WHERE isARealNote AND isLowActivity
SORT file.mtime desc, io asc, file.name asc
LIMIT 20
```
~~~

# ---Transient Commit Log

* v1.0.0
  * improved algorithm by limiting the loops and removing the sort. O(50) instead of O(N)
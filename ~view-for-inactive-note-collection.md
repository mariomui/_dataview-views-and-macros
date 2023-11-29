---
tag: _meta
VERSION: v1.0.1
---

# -

~~Uses the randomizing alogirthim found in  [Randomly order rows in a Dataview query - Share & showcase - Obsidian Forum](https://forum.obsidian.md/t/randomly-order-rows-in-a-dataview-query/46989?u=oirammui) to~~  create a 100 sample set, where each file's calculated [[IO-composite-key,vis-Noteshippo-dashboard,]] represents a file's inactivity with resepcts to input and output link ratio.

Unfortunately, i have no caching process to draw upon, and the filter on all notes in the system using dvjs has performance issues. 
- [x] Make a javascript version of this so I can use caches.
- [ ] Give the dataview the ability to use  [[custom-transclusion-parameters,]]

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

  const shuffled = dv.pages("#_wip").file
    .sort(() => 0.5 - Math.random())
    .limit(limit)
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
    .filter(Boolean)
    .filter(({status}) => status)
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
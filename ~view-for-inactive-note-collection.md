---
tag: _meta
---

# -

Uses the randomizing alogirthim found in  [Randomly order rows in a Dataview query - Share & showcase - Obsidian Forum](https://forum.obsidian.md/t/randomly-order-rows-in-a-dataview-query/46989?u=oirammui) to create a 100 sample set, where each file's calculated [[IO-composite-key]] represents a file's inactivity with resepcts to input and output link ratio.

Unfortunately, i have no caching process to draw upon, and the filter on all notes in the system using dvjs has performance issues. 
- [ ] Make a javascript version of this so I can use caches.

# =

```dataview
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

# ---Transient Sandbox


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
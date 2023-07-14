
```dataview
TABLE length(rows)
FLATTEN string(join(list(length(file.inlinks), length(file.outlinks)) , [""])) as "inlinks_outlinks"
WHERE econtains(list("00", "01", "10"),inlinks_outlinks) = true
AND !meta
GROUP BY inlinks_outlinks

```


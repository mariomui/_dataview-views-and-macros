# =
```dataviewjs

/*
unresolvedLinks: Record<
    SourceFilename extends string, 
    Record<
        brokenLinkName extends string, 
        brokenLinkCount extends number
        >
    > as BrokenLinkNameToCount

const exampleUnresolvedLinksValue: BrokenLinknameToCount = {
 "idontexist.md" : 1
}
A) currently: dv.app.metadataCache.unresolvedLinks  = {
    "-": NonEmptyObject, 
    "otherstuff": EmptyObject
} 
B) i-want-a-dataview-proper-format:
    unresolvedLinks : [ [NonEmptyObject.key, nonEmptyObject.values], ... ]   
    // otherstuff and its EmptyObject is removed. 
*/

// type unresolvedTuples: UnResolvedTuple[]
// type UnresolvedTuple = [soureFileName, BrokenLinkNameToCount]
// purpose: exclude empty objects from our unresolved tuples list.

// DATA BOUNDARY
const sourceFileNameAndBrokenLinkToCountTuples = Object.entries(
  dv?.app?.metadataCache?.unresolvedLinks || {}
)
// BLOGIC BOUNDARY
const unresolvedTuples = sourceFileNameAndBrokenLinkToCountTuples
  .filter(([k, v]) => Object.keys(v).length);
console.log({unresolvedTuples})

// UI BOUNDARY
const souceFileLinkAndLinkNamesTuples = unresolvedTuples.map(
  ([sourceFileName, linkNameToCountContext]) => {
    const linkNames = Object.entries(linkNameToCountContext).map(
      keepOnlyKeyFromTupleMapPredicate
    );

    const sourceFileLink = dv.fileLink(sourceFileName);
    return [sourceFileLink, linkNames];
  }
);

const headers = ["source-link", "non-existent-link-name"];
dv.table(headers, dv.array(souceFileLinkAndLinkNamesTuples));

function keepOnlyKeyFromTupleMapPredicate([key]) {
  return key;
}


```


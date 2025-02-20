---
alias: 
CREATION_DATE: 2023-10-05
DOC_VERSION: v0.0.0
MUID: MUID-1552
tag: _wip
TEMPLATE_VERSION: v1.0.4_blank-template
UMID: 
cssClasses: cards, cards-cover, cards-2-3, table-max
---

# -

## About

# =

**base_filepath-v0.0.2**: *`= this.file.path`* doc-`= this.DOC_VERSION` / ids: `= this.MUID`,`= this.UMID` / lcsh: `= this.heading` / updated on: `= dateformat(this.file.mday, "yyyy-LL-dd")` / file-size: `= round(this.file.size/1024,2)` KB

```dataviewjs

const {default: obs} = this.app.plugins.plugins['templater-obsidian'].templater.current_functions_object.obsidian

const {workspace, vault, metadataCache, fileManager} = this.app;
//*2023-07-22*

workspace.onLayoutReady(bootstrap.bind(this));

// entry file
function bootstrap() {
  (genMain)()
}
// workhorse
async function genMain() {
  const veeFile = getActiveVeeFile();
  const veeFolder = getClosestVeeFolder(
    veeFile
  ); 

  const {
    files, 
    folders
  } = await getFolderContentsByVeeFolder(
    veeFolder
  );

  const {data} = await processTableFromFiles(
    files.slice(0, -1), 
    veeFolder
  );

  console.log({files,folders, data})

  if (data) {
    renderMarkdownTable(...data)
  }
}


async function processTableFromFiles(
  files, 
  sourceVeeFolder
) {
  // error validataion goes here.
  try {
    return {
      data: await genTableTupleFromFilePaths(
          files, sourceVeeFolder
        ),
      error: null
    }
  } catch(err) {
    // obs.Notice(JSON.stringify(err));
    return {
      data: null,
      err: "genTableTuple error"
    }
  }
} 

const NOT_AVAILABLE = "N/A";

async function genTableTupleFromFilePaths(
  file_paths, 
  sourceVeeFolder
) {
    const basenames = file_paths
      .map(parseBasename)
      .map((basename) => {
        return `!(${basename})[]`;
      })
    
    const vfs = file_paths
      .map((file_path) => {
        return getVeeFileByRelativePath(file_path);
      });

 
    const headers = [
      {field: 'year'}, 
      {field: 'director'}, 
      {field: 'title'}, 
      {
        field: 'BANNER',
        cb: (v) => getMarkdownLinkByFilePath(v)
      }
    ];
    const header_titles = headers.map(
      (header) => header.field
    );
    
    const fms = file_paths.map(getFrontMatterFromFilePath);
    const fmDatums = fms.map((fm) => {
      const column_values = headers.map(({field,cb}) => {
        const value = fm[field];
        return cb ? cb(value) : value;
      })
      return column_values;
    });
    const matrixDatums = [];
    for (let i = 0; i < basenames.length; i++) {
      const basename = basenames[i];    
      const fmDatum = fmDatums[i];
      matrixDatums.push(fmDatum)
    }

    return [header_titles, matrixDatums];
}

function getClosestVeeFolder(
  vf, 
  config = {recurseLimit: 20}
) {
  if (vf?.isRoot && vf.isRoot()) return vf;
  
  const closestVeeFolderConfig = {
    ...config,
    recurseLimit: config.recurseLimit - 1
  } // no forever things.
  const isFolder = vf.hasOwnProperty("children")
  if (!isFolder) {
    return getClosestVeeFolder(
      vf.parent, 
      closestVeeFolderConfig
    );
  }
  return vf;
}
async function getFolderContentsByVeeFolder(vf) {
  // all utility function shouldn't be validataed. 
  // validate higher among the stack.
  return await vault.adapter.list(vf.path + "/");
  // TODO use normalizepath before adding a /
}

function getActiveVeeFile() {
  return workspace.getActiveFile();
}

function getMarkdownLinkByFilePath(file_path) {
  if (!file_path) return "";
  return `![](${file_path})`
}
function getWikiLinkByVeeFile(
  vf,
  config = {embed:""}
) {
  const embed_config = config?.embed ?
     vf.path+"#"+config.embed : 
     "";

  return fileManager
    .generateMarkdownLink(
      vf, 
      vf.path,
      embed_config,
      vf.basename
    );
}

function renderMarkdownTable(headers, matrix) {
  dv.table(headers, matrix)
}
function getFrontMatterFromFilePath(file_path) {
  const vf = getVeeFileByRelativePath(file_path);
  return metadataCache.getFileCache(vf)?.frontmatter || {}
}
function getVeeFileByRelativePath(
  file_path, 
  relative_path = ""
) {

  const result = metadataCache.getFirstLinkpathDest(
    file_path, relative_path
  );
  console.log("getVeeFileByRelativePath",{result})
  return result;
}
function parseBasename(file_path) {
  const path = vault.adapter.path.parse(file_path)
  return path.base;
}
```

# ---Transient Local Resources

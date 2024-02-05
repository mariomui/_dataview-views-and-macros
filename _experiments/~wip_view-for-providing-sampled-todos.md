---
TEMPLATE_VERSION: v1.0.5_default-template
VERSION: v0.0.1
MUID: MUID-1378
CREATION_DATE: 2023-07-27
tag: _wip
UMID: 
---
# -

```dataview
TASK 
WHERE file.name = this.file.name
AND !completed
```

## About


This [[Partial-dataview,vis-Noteshippo]] is a prototype. The goal is to provide a sample set of todos for dashboard use.

### Log
* v0.0.2
* v0.0.1
  * Set up a basic test environment housed in [[demo-of-transclusion-parameters-and-sampled-todos]] 
  * There are large probelems with [[inbox-note,vis-ObsidianMD-app,etc]]'s Automatic View Refresh where any vault changes refreshes the view causing the random sampling function to retrigger. This happens everytime a checkbox is marked.
  * ![[Screen Shot 2023-08-01 at 5.05.17 PM.png]]
  * Attempts:
    * global window.fig 
      * unsuccessful
### Reference
![[~view-for-referencing-current-jumpid#=|nlk]]
* † 

# =



```dataviewjs
const {default: obs} = this.app.plugins.plugins['templater-obsidian'].templater.current_functions_object.obsidian
const {workspace, vault, metadataCache } = this.app;

const getCurrentfilepath = () => this.currentFilePath

// entry point



workspace.onLayoutReady(bootstrap.bind(this))


// starter gun.
function bootstrap() {

  main.call(
    this, 
    getCurrentfilepath(),
    {
      getEmbedsFromVf: getEmbedsFromVf.bind(this),
      convertEmbedDisplayTextToArgMap: convertEmbedDisplayTextToArgMap.bind(this),
      extractTargetEmbed: extractTargetEmbed.bind(this)
    },
  );
}

// workhorse.
function main(current_filepath, depConfig) {

  const {
    getEmbedsFromVf,
    convertEmbedDisplayTextToArgMap,
    extractTargetEmbed,

  } = depConfig;

  // extract params main
  const vf = workspace.getActiveFile();
  new obs.Notice(vf.path + "::ACTIVE", 10000);
  const embeds = getEmbedsFromVf(vf)
  const {name} = vault.adapter.path
    .parse(current_filepath);
  if (!embeds) {
    new obs.Notice('no embeds');
  }
  const embed = extractTargetEmbed(name, embeds);
  if (!embed) return;

  const argmap =  convertEmbedDisplayTextToArgMap(embed);
  const limit = argmap["-n"]
  // /end params extraction 
  
  const {data} = processTasks({limit});

  const taskDatums = data.values;

  if (!taskDatums) {
    return new obs.Notice("No task data available.")
  };

  console.log(window.fig, 'fig')
  if (window?.fig) {
    const param = Number(JSON.parse(window.fig.params));
    if (limit === param) {
      return renderTasks(
        JSON.parse(window.fig.term)
      );
    }
  } 
  renderTasks(taskDatums);
  useMemo(window,taskDatums, limit)
}

// utils
function useMemo(ctx, term, params) {
  if (!ctx.fig) {
    ctx.fig = {
      term: JSON.stringify(term),
      params: JSON.stringify(params)
    }
    return "begin"
  }
  if (JSON.stringify(term) === ctx.fig.term ) {
    return "stop"
  }
  ctx.fig.term = JSON.stringify(term);
  ctx.fig.params = JSON.stringify(params);
  return "begin";
}


// ui
function renderTasks(taskDatums) {
  dv.taskList(taskDatums)
}

// helpers
function manuTasksConfig(
  config = {}
) {
  return ({
    limit: 5,
    wherePred: (t) => !t.completed,
    sortPred: (t) => 0.5 - Math.random(),
    ...config
  });
}
function queryTasks(
  config = manuTasksConfig()
) {
  const {
    wherePred, 
    sortPred,
    limit
  } = manuTasksConfig(
    config
  );
  const {file} = dv.pages("");
  if (!file) {
    throw new Error('files not found');
    return;
  }
  const {tasks} = file;
  if (!tasks) {
    throw new Error('tasks not found');
    return;
  }
  
  return tasks
    .where(wherePred)
    .sort(sortPred)
    .limit(limit);
}
function processTasks(
  config = manuTasksConfig()
) {
  const _config = manuTasksConfig(
    config
  );
  try {
    const datums = queryTasks(_config);
    return ({
      err: null,
      data: datums
    });
  } catch(err) {
    return ({
      err,
      data: null
    })
  }
}

// transclusion parameters

function extractTargetEmbed(
  embed_name, embeds
) {
  const embed = embeds?.find(
  (embed) => {
    const parsed = obs.parseLinktext(
      embed.link
    );
    return parsed.path === embed_name;
  });
  return embed;
}

function convertEmbedDisplayTextToArgMap(embed, config = {
  n: 5
}) {
  const {displayText} = embed;
  if (!displayText) return { n: HISTORY_TO_KEEP_CNT };
  return parseStringToMap(displayText)
}

function parseStringToMap(str) { 
  const parts = str.trim().split(/\s+/); 
  const map = {}; 
  for (let i = 0; i < parts.length; i += 2) {
   const key = parts[i]; 
   const value = parts[i + 1]; 
   if (key && value) { 
     map[key] = value; 
   } 
  }
return map; 
}

function getEmbedsFromVf(vf) {
  const cache = metadataCache.getFileCache(vf)
  return cache?.embeds;
}
```



# ---Transient

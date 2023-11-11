---
MUID: MUID-698
DOC_VERSION: v1.0.3
PARTIAL_PARAM_CONFIG:
  IS_LOGGING_SILENT: true
CODELET_SHORTNAME: see-progress-of-local-tasks-via-ui-bar
---

# -

- [ ] Make a common js tools i use like this logger thing.
  - The PARTIAL_PARAM_CONFIG allows me to control the logger very well.
```dataview
TASK WHERE file.name = this.file.name AND !completed
```

```dataview
TASK WHERE file.name = this.file.name AND completed
```

## About
- [ ] #_todo/priority-high/to-design/on-todos/regarding-the-prioritization-of-frequency
  - Some tasks are low priority but require steady chipping.
  - Such as switching from MUID to ID.
- [ ] Document all the Incremental IDs. What is Plateid? #_todo/to-process/on-Noteshippo/regarding-todo-naming 
- [ ] Move logger out into one ring plugin
- [ ] Devise a more contained method for logging silent.
  - 🔑 The code in [[~view-for-oldest-files-in-system-TCODEID-3]] includes a design and codelet showcasing [[custom-transclusion-parameters,]]. This allows the author to [[Lower-the-scope-of-entities-makes-coding-more-robust]] 
- [ ] What is an example of a [[practice-note]]?
  - An example [[practice-note]] in the [[how-does-andy-matuschaks-note-taking-system-work?]] has: "Effective system design requires insights drawn from serious contexts of use". This is usually a [[claim-note,et-alia]] so it may be that 
- [ ] Make the main function a module so the [[arity,vis-Coding,#=]] is more apparent.
- [x] Create centralized parameters in metadata called PARTIAL_PARAM_CONFIG
- [x] Prototype a logging system that only logs inside the partial and not on the sourcing note.
* ℹ I re-used a deleted MUID from [[~view-for-unused-MUIDs#=]]
* [ ] DEPRECATE PARTIAL_VERSION in current file's dvjs code.

This partial view is transcluded when one needs to see a progress bar over all the tasks of the current file.

* Note:
  * The progress will show,
    * the count for all completed tasks as the first number.
    * the count for all tasks as the second number.
  * The tasks will only be for the parent task node.

* 💣 Logic  
  * v1.0.2 has a a bug identifying root subtasks.
  * Two tasks can have the same parent id, thereby negating the second task which should qualify as a root sub task.
  * 🤔
    * I rather do a BFS/DFS on the trees structure and grab the top root leaf that isn't a task.
  * [ ] Create DFS
  * [ ] ignoreme: This second task in ==`= this.file.name`== does not register because it has the same parent id as "Create DFS"

# =

~~~dataviewjs
// instance
const {workspace, metadataCache, plugins} = this.app;
const {default: obs} = plugins.plugins['templater-obsidian'].templater.current_functions_object.obsidian;



const PARTIAL_VERSION = "v1.0.3";
// v1.0.3 see if i can get the loading problem padding to stop being anal. shoving more stuff into main.


// knobs

const OBSIDIAN_COLD_START_TIME = 5000; //5s startup

workspace.onLayoutReady(main.bind(this));

function main() {
  let isLogSilent = false;

  if (!obs) return;
  const vf = workspace.getActiveFile();
  const page_path = vf.path;
  const cmData = metadataCache.getFileCache(vf);
  
  const PARTIAL_PARAM_CONFIG = "PARTIAL_PARAM_CONFIG";
  const config = cmData?.frontmatter?.[PARTIAL_PARAM_CONFIG];
  
  if (config && config?.IS_LOGGING_SILENT) {
    isLogSilent = config.IS_LOGGING_SILENT;
  } else if (!config) {
    isLogSilent = true;
  }

  const logg = createLogg.call(
      this,
      {isSilent: isLogSilent}
  );
  

  const currentTasks = dv.page(page_path)?.file?.tasks?.values || [];
  const progressionInfo = calculateProgressionInfo.call(this, currentTasks);

  renderProgressionInfo.call(this, progressionInfo);

  function createLogg(config = {isSilent: false}) {
    const self = this;
    const LogStyle = {
      background: "var(--material-color-red-700)",
      padding: "1em 1em",
    };
    const style = serializeStyle(LogStyle);
    return function logger(text, time = 5000, isBypass = false) {
      const isSilent = self?.fig?.silent === true || config.isSilent;
      if (isSilent && !isBypass) {
        return;
      }
      const _text = typeof text === "string" ? text : JSON.stringify(text);
      const $span = self.container.createDiv({
        text: `logger:${_text}`,
        attr: {
          style,
        },
      });
      new obs.Notice($span, time);
    };
  }
}

function getTaskCounts(tasks) {
  let rootTasksCount = 0;
  let completedRootTasks = 0;

  tasks.forEach(task => {
    if (!task.parent) {
      rootTasksCount++;
      if (task.fullyCompleted) {
        completedRootTasks++;
      }
    }
  });

  return [
    rootTasksCount,
    completedRootTasks,
  ]
}


function calculateProgressionInfo(currentTasks = []) {
  const uniqueListIdCheckingContext = {};

  const uniqueTasks = [];

  const [totalTaskCnt, completedTaskCnt] = getTaskCounts(currentTasks)

  // business domaind derived
  
  const raw_percentage = (completedTaskCnt / totalTaskCnt).toFixed(2) * 100;
  const percentage = Math.round(raw_percentage) || 0;

  const unfinishedTaskCnt = totalTaskCnt - completedTaskCnt;

  return manuProgressionInfo({
    completedTaskCnt,
    percentage,
    totalTaskCnt,
    unfinishedTaskCnt,
  });
}

function renderProgressionInfo(progressionInfo) {

  const { percentage, totalTaskCnt, unfinishedTaskCnt, completedTaskCnt } =
    progressionInfo;

  const attributeConfig = {
    value: percentage,
    max: 100,
    appearance: "none",
  };

  this.container.createEl(
    "progress",
    {
      text: "hi",
      attr: attributeConfig,
      cls: "mario-progress",
    },
    ($p) => {
      const style = `font-size: smaller`;
      const span = this.container.createSpan({
        text: `${percentage} % | ${completedTaskCnt} of ${totalTaskCnt}`,
        attr: {
          style,
        },
      });
      span.style.color = "var(--accent)";
    }
  );
}

function serializeStyle(style) {
  return Object.entries(style).reduce((chain, [k, v]) => {
    return `${k}:${v};` + chain;
  }, "");
}



function manuProgressionInfo(progressionInfo = {}) {
  return {
    completedTaskCnt: 0,
    unfinishedTaskCnt: 0,
    totalTaskCnt: 0,
    percentage: 0,
    ...progressionInfo,
  };
}
~~~

# ---Transient Local Archive


## LA--archive--old version of MUID-698

* Passing context into the logger to only allow logging in the 2nd pbar
  * 🤔doesn't work, this is shared.
    * [[archived_~view-for-local-tasks-using-a-progress-bar-MUID-698]]
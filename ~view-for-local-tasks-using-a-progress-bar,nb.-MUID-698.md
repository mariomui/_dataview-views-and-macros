---
MUID: MUID-698
DOC_VERSION: v1.0.4
PARTIAL_PARAM_CONFIG:
  IS_LOGGING_SILENT: true
CODELET_SHORTNAME: see-progress-of-local-tasks-via-ui-bar
---

# -

[[~view-for-local-tasks-using-a-progress-bar,nb.-MUID-698#=|olk]]
* [ ] Make a common js tools i use like this logger thing.
  * The PARTIAL_PARAM_CONFIG allows me to control the logger very well.
```dataview
TASK WHERE file.name = this.file.name AND !completed
```

```dataview
TASK WHERE file.name = this.file.name AND completed
```

## About

* [x]  #_todo/50-backlog--/to-design/on-todos/regarding-the-prioritization-of-frequency ✅ 2023-12-22
  * Some tasks are low priority but require steady chipping.
  * Such as switching from MUID to ID.
* [ ] Document all the Incremental IDs. #_todo/to-process/upon-noteshippo/regarding-todo-naming
  * 🔑 [[sandbox,bt.-Noteshippo-title-level-flag,]]
* [x] What is Plateid? ✅ 2024-03-20
  * 🔑 [[interim--internal-guide-to-using-plateids-to-track-plates,vis-Writing,]] A plate id is way to make sure that plates are recorded an identified using incremental id but it doesn't work well since it doesn't flag like html. It lacks the ability to alert me when a [[plate,vis-Writing,]] hasn't been closed yet. #_todo/70-done--/to-muse/on-writing/regarding-plate-tracking
* [ ] Move logger into one ring plugin #_todo/52-priority-low--/to-code
* [ ] Devise a more contained method for logging silent.
  * 🔑 The code in [[interim--~view-for-oldest-files-in-system,nb.-MUID-130]] includes a design and codelet showcasing [[custom-transclusion-parameters,cf.-Kanzi,vis-ObisidianMD-app,]]. This allows the author to [[Lower-the-scope-of-entities-makes-coding-more-robust]]

  * ~~🔑An example [[,aka-practice-note]]~~ in ~~the [[how-does-andy-matuschaks-note-taking-system-work?]] has: "Effective system design requires insights drawn from serious contexts of use". This is usually a [[claim-note,etc]] so it may be that ➕ 2024-06-05~~
    * What is an example of a [[,aka-practice-note]]?
* [ ] Make the main function a module so the [[arity,vis-Coding,]] is more apparent.
* ℹ I re-used a deleted MUID from [[~view-for-unused-MUIDs]]
* [ ] DEPRECATE PARTIAL_VERSION in current file's dvjs code.

This partial view is [[,aka-transclude]]d when one needs to see a progress bar over all the tasks of the current file.

* Note:
  * The progress will show,
    * the count for all completed tasks as the first number.
    * the count for all tasks as the second number.
  * The tasks will only be for the parent task node.




# =

~~~dataviewjs
// instance
const {workspace, metadataCache, plugins} = this.app;
const obs = plugins?.plugins?.['templater-obsidian']?.templater?.current_functions_object?.obsidian?.default;

const PARTIAL_VERSION = "v1.0.5";
// v1.0.5 add more try catches
// v1.04 refactor silent / simplify
// v1.0.3 see if i can get the loading problem padding to stop being anal. shoving more stuff into main.

// knobs

const OBSIDIAN_COLD_START_TIME = 5000; //5s startup
const PARTIAL_PARAM_CONFIG = "PARTIAL_PARAM_CONFIG";

workspace.onLayoutReady(bootstrap.bind(this));

function bootstrap() {
	if(!obs) return; // on boot up templater obsidian doesn't load the obsidian apis I need fast enough. so early return.
	
  (function(self,genMain) {
    genMain()
  })(this,genMain.bind(this))
}

function manuSetupLoggerFig() {
  return {
    IS_LOGGING_SILENT: false
  }
}
function processSetupLogger(fig = manuSetupLoggerFig()) {
  const config = Object.assign({}, manuSetupLoggerFig(), fig)
  
  const logg = createLogg.call(
      this,
      {isSilent: config.IS_LOGGING_SILENT}
  );
  return logg  
}

function getObs() {
  return obs;
} 


async function genMain(obs = getObs()) {
  
  if (!obs) return;
  
  const vf = workspace.getActiveFile();
  const page_path = vf.path;
  
  const cmData = metadataCache.getFileCache(vf);
  const config = cmData?.frontmatter?.[PARTIAL_PARAM_CONFIG] ?? manuSetupLoggerFig();

  this.logg = processSetupLogger.call(this, config).bind(this);


  const currentTasks = dv.page(page_path)?.file?.tasks?.values || [];
  const currentLiDatums = dv.page(page_path)?.file?.lists?.values || [];

  const _currentLiDatums = currentLiDatums.filter(({parent, list}) => parent === list)
  // console.log({currentTasks})
  if (currentLiDatums.length === 0 || currentTasks.length === 0) {
    return renderProgressionInfo.call(this, manuProgressionInfo());
  }
  try {
   const progressionInfo = await genCalculateProgressionInfo
    .call(
      this, currentTasks, currentLiDatums, {logg:this.logg}
   );
  return renderProgressionInfo.call(this, progressionInfo);
 } catch(e) {
  this.logg({e,text: "genMainfucked"})
  return manuProgressionInfo.call(this);
 }
}

function createLogg(config) {

  const LogStyle = {
    background: "var(--material-color-red-700)",
    padding: "1em 1em",
  };
  const style = serializeStyle(LogStyle);
  return logger.bind(this);
  function logger(text, time = 5000, isBypass = false) {
    if (config.isSilent && !isBypass) {
      return;
    }
    const _text = typeof text === "string" ? text : JSON.stringify(text);
    const $span = this.container.createDiv({
      text: `logger:${_text}`,
      attr: {
        style,
      },
    });
    new obs.Notice($span, time);
  };
}

function createLiDatumsStrategy() {
  const payload = {
    completedTasksCnt: 0,
    rootTasksCnt: 0,
  }
  return function strategy(liDatums = []) {
    let parentMap = {}

    walk(liDatums);
    return payload;
    
    function walk(liDatums, flag = false) {
     
      for (let i = 0; i < liDatums.length; i++) {
        const li = liDatums[i]

        if (li.task && flag && !parentMap[li.parent]) {
        
          if (li.fullyCompleted) {
            payload.completedTasksCnt++
          }
          payload.rootTasksCnt++;
          parentMap[li.parent] = true;
        }
        if (!li.task && li.children.length) {
          walk(li.children, true)
        }
      }
      
    }
  }
}
function createDefaultGenStrategy() {
  return function genStrategy(tasks, liDatums) {
  
    const payload = createLiDatumsStrategy()(liDatums)
    
    return new Promise((rs,rj) => {
      
      let rootTasksCount = 0 + payload.rootTasksCnt;
      let completedTasksCnt = 0 + payload.completedTasksCnt;
      tasks.forEach((task,idx) => {
        if (!task.parent) {
          rootTasksCount++;
          if (task.fullyCompleted) {
            completedTasksCnt++;
          }
        }
        if( idx === (tasks.length - 1)) {
          return rs([
            rootTasksCount, completedTasksCnt
          ])
        }
      });
    })
  }
}

async function genTaskCounts(
    tasks, 
    liDatums, 
    strat = createDefaultGenStrategy()
) {

  
  const taskCountTuple = await strat(tasks, liDatums)

  return taskCountTuple;

}


async function genCalculateProgressionInfo(
  currentTasks = [], 
  currentLiDatums = [],
  config
) {

  const {logg} = config;
  const uniqueListIdCheckingContext = {};
  const uniqueTasks = [];
  

  try {
   const [totalTaskCnt, completedTaskCnt] = await genTaskCounts(
     currentTasks, currentLiDatums
   )
 
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
  } catch(e) {
    logg("we got problems in genCalculateProgressionInfo");
    return manuProgressionInfo()

  }
}

function renderProgressionInfo(progressionInfo = manuProgressionInfo()) {
  const _progressionInfo = {...manuProgressionInfo(), ...progressionInfo};
  const { percentage, totalTaskCnt, unfinishedTaskCnt, completedTaskCnt } = _progressionInfo;

  const attributeConfig = {
    value: percentage || 0,
    max: 100,
    appearance: "none",
  };

  this.container.createEl(
    "progress",
    {
      text: "hi",
      attr: attributeConfig,
      cls: "mario-progress", // is mario-progress in my css? if it is please take it out and inline it.
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

* [ ] Use the new algorithm and update progress ➕ 2023-12-22 #_todo/42-priority-high--/to-fix/on-a-codelet/regarding-a-parsing-bug
  * the new code in [[~viewfn-for-listed-items-that-contain-specific-targetted-text,nb.-MUID-1925]] uses a recursive mechanism that records the levels. A [[breath-first-search,vis-Coding,]] that does work as it pops off the stack should be able to check whehter or not a root node is done.
  * ⏺ *history*
    * 💣 Logic ➕ 2023-07-11
      * v1.0.2 has a a bug identifying root subtasks.
      * Two tasks can have the same parent id, thereby negating the second task which should qualify as a root sub task.
      * 🤔
        * I rather do a BFS/DFS on the trees structure and grab the top root leaf that isn't a task.
      * [ ] Create DFS
      * [ ] ignoreme: This second task in ==`= this.file.name`== does not register because it has the same parent id as "Create DFS"
## LA--archive--old version of MUID-698

* Passing context into the logger to only allow logging in the 2nd pbar
  * 🤔doesn't work, this is shared.
    * [[archived--~view-for-local-tasks-using-a-progress-bar-MUID-698]]

# ---Transient Commit Log

* v1.0.3
  * Introduced new algorithm that only records the root node for the task.
    * 🐛 Four tasks, only 3 are collected

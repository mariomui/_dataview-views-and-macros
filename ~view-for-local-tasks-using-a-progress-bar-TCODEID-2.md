---
MUID: MUID-698
ID: TCODEID-2
VERSION: v1.0.0
PARTIAL_PARAM_CONFIG: 
  IS_LOGGING_SILENT: false
SHORT_NAME: see-progress-of-local-tasks-via-ui-bar
---

# -

```dataview
TASK WHERE file.name = this.file.name AND !completed
```

```dataview
TASK WHERE file.name = this.file.name AND completed
```

## About
- [ ] #_todo/high_priority/to-design-a-dashboard/on-todos/regarding-the-prioritization-of-frequency
  - Some tasks are low priority but require steady chipping.
  - Such as switching from MUID to ID.
- [ ] Document all the Incremental IDs. What is Plateid? #_todo/meta 
- [ ] Move logger out into one ring plugin
- [ ] 
- [ ] Devise a more contained method for logging silent.
  - 🔑 The code in [[~view-for-oldest-files-in-system-TCODEID-3]] includes a design and codelet showcasing [[custom-transclusion-parameters]]. This allows the author to [[Lower-the-scope-of-entities-makes-coding-more-robust]] 
- [ ] What is an example of a [[practice-note]]?
  - An example [[practice-note]] in the [[note-taking-system-designed-by-andy-matuschak]] has: "Effective system design requires insights drawn from serious contexts of use". This is usually a [[claim-note]] so it may be that 
- [ ] Make the main function a module so the [[arity#=]] is more apparent.
- [x] Create centralized parameters in metadata called PARTIAL_PARAM_CONFIG
- [x] Prototype a logging system that only logs inside the partial and not on the sourcing note.
* ℹ I re-used a deleted MUID from [[~view-for-unused-MUIDs#=]]

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
  * [ ] ignoreme: This second task does not register because it has the same parent id as "Create DFS"

# =

~~~dataviewjs
// instance

const PARTIAL_VERSION = "v1.0.3";
// v1.0.3 see if i can get the loading problem padding to stop being anal. shoving more stuff into main.
const {workspace, metadataCache, plugins} = this.app
let obsModule = plugins?.plugins['templater-obsidian']?.templater?.current_functions_object?.obsidian
// knobs
let isLogSilent = false;

const OBSIDIAN_COLD_START_TIME = 20000; //20s startup
let obs = null;
if (obsModule) {
  obs = obsModule.default;
  workspace.onLayoutReady(main.bind(this));
} else {
  setTimeout(main.bind(this), 40000)
}

function main() {
  if (!obs) {
    obs = plugins?.plugins['templater-obsidian']?.templater?.current_functions_object?.obsidian
  }
  if (!obs) {
    window.alert('dang')
    return;
  }

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


  const listItems = cmData?.listItems?.filter((task) => task?.task);
  if (!listItems?.length) return;
  const uniques = unique(listItems);

  logg({
      logName: "findFirstsInSet",
      data: findFirstsInSet(uniques, cmData?.listItems)
    },
    20000
  );

  this.fig = {    
    silent: isLogSilent,
  };

  if (!this.fig) {
    this.fig = {
      ...this.fig,
    };
    this.fig.silent = false;
  }
  
  function findFirstsInSet(uniques, listItems) {
    let k = 0;
    let remember = uniques[k];
    let set = [];

    for (const [idx, li] of cmData.listItems.entries()) {

      if (!remember) {
        return set;
      }
      const isParentsMatch = li.parent === remember.parent;

      if (isParentsMatch && li?.task) {
        set.push(li);
        remember = uniques[++k];
      }
    }
    return set;
  }

  function unique(listItems) {
    const parents = [];
    let rememberParent = null;
    for (const listItem of listItems) {
      const parent = listItem.parent;
      if (rememberParent !== parent) {
        parents.push(parent);
        rememberParent = parent;
      }
    }
    return parents;
  }



  const currentTasks = dv.page(page_path).file.tasks;
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
function calculateProgressionInfo(currentTasks) {
  const uniqueListIdCheckingContext = {};

  const uniqueTasks = [];
  let totalTaskCnt = 0;
  let completedTaskCnt = 0;

  // get first of the ids.
  for (const currentTask of currentTasks) {
    const { list: listId, text } = currentTask;

    if (!uniqueListIdCheckingContext[listId]) {
      uniqueListIdCheckingContext[listId] = true;
      uniqueTasks.push(currentTask);
      totalTaskCnt++;
    }
  }

  for (const uniqueTask of uniqueTasks) {
    const { completed } = uniqueTask;
    if (completed) {
      completedTaskCnt++;
    }
  }

  // business domaind derived
  const raw_percentage = (completedTaskCnt / totalTaskCnt).toFixed(2) * 100;
  const percentage = Math.round(raw_percentage);

  const unfinishedTaskCnt = totalTaskCnt - completedTaskCnt;

  return manuProgressionInfo({
    completedTaskCnt,
    percentage,
    totalTaskCnt,
    unfinishedTaskCnt,
  });

  function checkIsTaskCompleted(task) {
    return task.completed;
  }
  function checkIsRootTask(task) {
    const isParentATask = checkIsParentATask(listIds, task);

    if (task.parent && !isParentATask) {
      return true;
    }
    if (!task.parent) {
      return true;
    }
    return false;

    function checkIsParentATask(listIds, task) {
      return listIds.includes(task.parent);
    }
  }
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

# ---Transient Sandbox

* Passing context into the logger to only allow logging in the 2nd pbar
  * 🤔doesn't work, this is shared.

## v0.0.0

```js
~~~dataviewjs
// instance
const {default: obs} = this.app.plugins.plugins['templater-obsidian'].templater.current_functions_object.obsidian

//virtual file
const vf = this.app.workspace.getActiveFile();

const page_path = vf.path;

const manuProgressionInfo = (progressionInfo = {}) => {
    return ({
        completedTaskCnt:0, 
        unfinishedTaskCnt:0,
        totalTaskCnt:0,
        percentage: 0,
        ...progressionInfo
    });    
}

this.container.fig = {
    silent: true
}
const logg = createLogg.call(
    this, 
    this.container.fig,
    {
        silent: false
    }
)


this.app.workspace.onLayoutReady(main.bind(this))

function main() {
    if (!this.container.fig) {
        this.container.fig = {
            ...this.containerfig
        };
        this.container.fig.silent = false;
    }
        
    const currentTasks = dv.page(page_path).file.tasks;
    const progressionInfo = calculateProgressionInfo
        .call(this, currentTasks);
    
    renderProgressionInfo
        .call(this, progressionInfo);
    
}
function calculateProgressionInfo(currentTasks) {
    const uniqueListIdCheckingContext = {};

    const uniqueTasks = []
    let totalTaskCnt = 0;
    let completedTaskCnt = 0;

    // get first of the ids.
    for (const currentTask of currentTasks) {
         const {list: listId, text} = currentTask;
    
        if (!uniqueListIdCheckingContext[listId]) {
            uniqueListIdCheckingContext[listId] = true;
            uniqueTasks.push(currentTask)
            totalTaskCnt++;
        }
    }

    for (const uniqueTask of uniqueTasks) {
         const {completed} = uniqueTask;
         if (completed) {
             completedTaskCnt++;
         }
    }

  
    // business domaind derived
    const raw_percentage = (completedTaskCnt / totalTaskCnt)
        .toFixed(2)  * 100;
    const percentage = Math.round(raw_percentage)

    const unfinishedTaskCnt = totalTaskCnt - completedTaskCnt;

    return manuProgressionInfo({
        completedTaskCnt,
        percentage,
        totalTaskCnt,
        unfinishedTaskCnt
    });

    function checkIsTaskCompleted(task) {
        return task.completed
    }
    function checkIsRootTask(task) {
        const isParentATask = checkIsParentATask(listIds, task);
        
        if (task.parent && !isParentATask) {
            return true;
        }
        if (!task.parent) {
            return true;
        }
        return false;
        
        function checkIsParentATask(listIds, task) {
            return listIds.includes(task.parent);
        }
    }
}

function renderProgressionInfo(progressionInfo) {
 

    const {
        percentage,
        totalTaskCnt, 
        unfinishedTaskCnt,
        completedTaskCnt
    } = progressionInfo;

    const attributeConfig = {
        value: percentage,
        max: 100,
        appearance: "none"
    }

    this.container.createEl(
        "progress",
        {
            text: "hi",
            attr: attributeConfig,
            cls: "mario-progress"
        }, 
        ($p) => {
            const style = (
                `font-size: smaller`
            );
            const span = this.container.createSpan(
                {
                    text: `${percentage} % | ${completedTaskCnt} of ${totalTaskCnt}`,
                    attr: {
                        style
                    }
                }
            )
            span.style.color = "var(--accent)"
        }
    )
}

const manuCreateLoggConfig = () => ({ silent: false })
function createLogg(fig, config = manuCreateLoggConfig()) {
    const _config {
        ...manuCreateLoggConfig(),
        ...config;
    }
    return function logger(
        text, 
        time = 5000, 
        config = {..._config}
    ) {
        const __config = {
            ...config,
            ..._config
        }
        const isSilent = __config.silent === true;
        if (isSilent && !__config.silent) {
           return;
        }
        const _text = typeof text === "string" 
            ? text 
            : JSON.stringify(text);
        new obs.Notice(_text, time)
    }
}
~~~
```

## Non JS code

```
const progress_html = `<progress value="${String(percentage)}" max="100"></progress><span style="font-size:smaller;color:white">${percentage} % | ${unfinishedTaskCnt} left of ${totalTaskCnt} </span>`

logg(progress_html, 3000)
```

### v0.0.0-dovos

* 🐦📝 [Discord](https://discord.com/channels/686053708261228577/710585052769157141/1124438998245453847)
  * 💁
    * This particular code is from dovos.
    * I translated it into js.
```js
~~~dataviewjs

const {default: obs} = this.app.plugins.plugins['templater-obsidian'].templater.current_functions_object.obsidian
const {path} = this.app.workspace.getActiveFile()
const page_path = path;

const filter_term = "Status Tasks" ; 

const createCheckTaskForFilterTerm = (
    filter_term, task_property_key
) => {
    return (task) => {
        const context = task[task_property_key];
        return String(context).includes(filter_term)
    }
}
const checkTaskForFilterTerm = createCheckTaskForFilterTerm(
    filter_term, "section"
);
const checkForCompletedTask = (task) => task.completed;
const currentTasks = dv.page(page_path).file.tasks;

const completedTasksCnt = currentTasks
    .where(checkForCompletedTask)
    //.where(checkTaskForFilterTerm)
    .length
const totalTasksCnt = currentTasks
    //.where(checkTaskForFilterTerm)
    .length
const value = Math.round(
    completedTasksCnt / totalTasksCnt
) * 100


const unfinishedTasksCnt = totalTasksCnt - completedTasksCnt
const progress_html = `<progress value="${value}" max="100"></progress><span style="font-size:smaller;color:var(--text-muted)">${value} % | ${unfinishedTasksCnt} left</span>`
// new obs.Notice(progress_html, 5000)

this.app.workspace.onLayoutReady( main.bind(this) )
function main() {
    renderEl.call(this,progress_html);
}
function renderEl(html) {
    const $p = this.container.createEl("p")
    $p.innerHTML = progress_html;
}
~~~
```
- [ ] Keep this task for test purposes.
  - [ ] This sub task exemplifies  bug in the `v0.0.0-dovos` progress bar implementation.
    - A sub root tag appears as a task to be completed.

This particular code also has bugs but in different situations. It doesn't register nested code i believe. It also record non subrooted tasks.
* Visually it looks like this⤵
~~~dataviewjs

const {default: obs} = this.app.plugins.plugins['templater-obsidian'].templater.current_functions_object.obsidian
const {path} = this.app.workspace.getActiveFile()
const page_path = path;

const filter_term = "Status Tasks" ; 

const createCheckTaskForFilterTerm = (
    filter_term, task_property_key
) => {
    return (task) => {
        const context = task[task_property_key];
        return String(context).includes(filter_term)
    }
}
const checkTaskForFilterTerm = createCheckTaskForFilterTerm(
    filter_term, "section"
);
const checkForCompletedTask = (task) => task.completed;





this.app.workspace.onLayoutReady( main.bind(this) )
function main() {
    const currentTasks = dv.page(page_path).file.tasks;
    
    const completedTasksCnt = currentTasks
        .where(checkForCompletedTask)
        .length
        //.where(checkTaskForFilterTerm)
    const totalTasksCnt = currentTasks
        .length
        
    const value = (
        (completedTasksCnt / totalTasksCnt).toFixed(2)
    ) * 100;
    const _value = Math.round(value)
    const unfinishedTasksCnt = totalTasksCnt - completedTasksCnt
    const progress_html = `<progress value="${_value}" max="100"></progress><span style="font-size:smaller;color:var(--text-muted)">${_value} % | ${unfinishedTasksCnt} left</span>`
    
        //.where(checkTaskForFilterTerm)
    renderEl.call(this,progress_html);
}
function renderEl(progress_html) {
    const $p = this.container.createEl("p")
    $p.innerHTML = progress_html;
}
~~~

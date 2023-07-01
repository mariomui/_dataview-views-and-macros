# -

This partial view is transcluded when one needs to see a progress bar over all the tasks of the current file. 

* Note:
    * The progress will show,
        * the count for all completed tasks as the first number.
        * the count for all tasks as the second number.
    * The tasks will only be for the parent task node.

# =

~~~dataviewjs
// instance
const {default: obs} = this.app.plugins.plugins['templater-obsidian'].templater.current_functions_object.obsidian

// knobs
const isLogSilent = true;

//virtual file
const vf = this.app.workspace.getActiveFile()
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
const logg = createLogg.call(this)
this.fig = {
    silent: isLogSilent
}

this.app.workspace.onLayoutReady(main.bind(this))

function main() {
    if (!this.fig) {
        this.fig = {
            ...this.fig
        };
        this.fig.silent = false;
    }
        
    const currentTasks = dv.page(page_path).file.tasks;
    const progressionInfo = calculateProgressionInfo
        .call(this, currentTasks);
    logg(progressionInfo, 8000);
    
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
    
    const progress_html = `<progress value="${String(percentage)}" max="100"></progress><span style="font-size:smaller;color:white">${percentage} % | ${unfinishedTaskCnt} left of ${totalTaskCnt} </span>`
    
    logg(progress_html, 3000)


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

function createLogg() {
    const self = this;
    return function logger(text, time=5000, isBypass=false) {
        const isSilent = self?.fig?.silent === true;
        if (isSilent && !isBypass) {
           return;
        }
        const _text = typeof text === "string" 
            ? text 
            : JSON.stringify(text);
        new obs.Notice(_text, time)
    }
}
~~~


# ---Transient Sandbox
* Passing context into the logger to only allow logging in the 2nd pbar
    * 🤔doesn't work, this is shared.
~~~dataviewjs
// instance
const {default: obs} = this.app.plugins.plugins['templater-obsidian'].templater.current_functions_object.obsidian

//virtual file
const vf = this.app.workspace.getActiveFile()
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
    logg(progressionInfo, 8000);
    
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
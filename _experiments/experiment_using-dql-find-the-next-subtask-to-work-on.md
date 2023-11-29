---
TEMPLATE_VERSION: v1.0.5_default-template
MUID: MUID-1208
CREATION_DATE: 2023-06-25 
alias: 
tags: _wip
UMID: 
---
# -

```dataview
TASK 
WHERE file.name = this.file.name
AND !completed
```

## About

### Reference
![[~view-for-referencing-current-jumpid#=|nlk]]
* † 

# =

## = TITLE
*`= this.file.name`*
![[~view-for-calculating-reading-time#=|nlk]]



# ---Transient

The following is example codelet on how to do subtasks.
* [ ] project 1
    * [x] 1
    * [ ] 2
    * [ ] 3
    * [ ] 4
* [ ] project 2
    * [ ] a
    * [x] b
    * [x] c [completion:: 2023-07-29]


```dataview
TABLE WITHOUT ID FileLists.text,
Subtasks.text 
WHERE file.name = this.file.name
FLATTEN file.lists as FileLists
WHERE FileLists.subtasks != [] 
FLATTEN FileLists.subtasks as Subtasks
WHERE Subtasks.line = min(
    filter(
        FileLists.subtasks, (x) => !x.checked
    ).line
)

```

Store below into memory.
~~~js
```js

```dataview
TABLE WITHOUT id FileLists.text,
Subtasks.text 
where file.name = this.file.name
FLATTEN file.lists as FileLists
where FileLists.subtasks != [] 
FLATTEN FileLists.subtasks as Subtasks
where Subtasks.line = min(
    filter(
        FileLists.subtasks, 
        (x) => !x.checked
    ).line
)
```
```
~~~

The problem is: how does file.name actually work?
Does it iterate per file, grab the file.name and compare?
It looks like it does.
But head toward the Subtasks.line section.
If File.lists.subtasks (Subtasks) is `[...]`
then each line is is [].line 
that's obviously not the case sine an ARRAY can't have a property.

Now assume that there is a line 
```
{
  line: 100
}
```
That means Subtasks.line is checking for a Subtask item.
[{}, {}, {}]
So the act of flattening Filelist.subtasks as Subtasks is....

```js
FLATTEN FileLists.subtasks as Subtasks
where Subtasks.line = min(....

const Subtasks = (file.lists.subtasks)
Subtasks.where(subtask => (
    return subtask.line === ...min()
))
```

this means the WHERE clause fundamnetally changes the data shape of the single array into its itemized component. It works more like a dot FIND on subtasks.

```js
where Subtasks.line = min(
    filter(
        FileLists.subtasks, 
        (x) => !x.checked
    ).line
)
```
The filter function behaves like the js we know.
Filelists.subtask is an array, and we iterate over the x and compare.
```ts
type iSubTasks = { line: number, text: number, checked: boolean, ...I}[]
/* 
{1},{2},{3}{4} -> [{2},{3},
{ text: 4, checked: true}] (filtered for not check)
*/
```

Once filter is evaluated, it turns back into a Psuedo entity again.
`.line` is applied to every item in the evaluated collection

Just like .line was applied to every item in the Flattened collection earlier (Subtasks) 

When a where acts upon a file, we are assuming the data set is on a set of files.

It's not SQL. it buidl sboutn itself.
DataviewAPI.page("daily-notes-plugin-for-obsidianmd").file.lists.where(a => a.subtasks.length > 0)

so that's up to where `FileLists.subtasks != []``;

we have an ARRAY of 2
looks like this.

FileLists' Abridged Properties version: ⤵
Why is it fileLists?  because the where clause acts as predicate function.
When a where acts upon the flattened Subtasks, we are assuming the data set is on a set of files.
```json
[
    {
        "symbol": "*",
        "link": {
            ...
        },
        "section": {
            ...
        },
        "text": "project 1",
        "line": 33,

        "children": [
            {
                "text": "1",
                "line": 34,
                "children": [],
                "task": true,
                "subtasks": [],
                "parent": 33,
                "checked": true,
            },
            ...
            {
                "text": "4",
                "line": 37,
                "list": 33,
                "children": [],
                "task": true,
                "parent": 33,
                "status": " ",
                "checked": false,
                "completed": false,
                "fullyCompleted": false
            }
        ],
        "subtasks": [
            {
                "symbol": "*",
                "text": "1",
                "line": 34,
                "list": 33,
                "path": "daily-notes-plugin-for-obsidianmd.md",
                "children": [],
                "task": true,
                "subtasks": [],
                "real": true,
                "parent": 33,
                "status": "x",
                "checked": true,
                "completed": true,
                "fullyCompleted": true
            },
            ...
            {
                "symbol": "*",
                "link": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "section": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "text": "4",
                "tags": [],
                "line": 37,
                "lineCount": 1,
                "list": 33,
                "outlinks": [],
                "path": "daily-notes-plugin-for-obsidianmd.md",
                "children": [],
                "task": true,
                "annotated": false,
                "position": {
                    "start": {
                        "line": 37,
                        "col": 2,
                        "offset": 455
                    },
                    "end": {
                        "line": 37,
                        "col": 11,
                        "offset": 464
                    }
                },
                "subtasks": [],
                "real": true,
                "header": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "parent": 33,
                "status": " ",
                "checked": false,
                "completed": false,
                "fullyCompleted": false
            }
        ],
        "checked": false,
        "completed": false,
        "fullyCompleted": false
    },
    // project 2
    {
        "text": "project 2",
        "tags": [],
        "line": 38,
        "lineCount": 1,
        "list": 33,
        "outlinks": [],
        "path": "daily-notes-plugin-for-obsidianmd.md",
        "children": [
            {
                "symbol": "*",
                "link": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "section": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "text": "a",
                "tags": [],
                "line": 39,
                "lineCount": 1,
                "list": 33,
                "outlinks": [],
                "path": "daily-notes-plugin-for-obsidianmd.md",
                "children": [],
                "task": true,
                "annotated": false,
                "position": {
                    "start": {
                        "line": 39,
                        "col": 2,
                        "offset": 483
                    },
                    "end": {
                        "line": 39,
                        "col": 11,
                        "offset": 492
                    }
                },
                "subtasks": [],
                "real": true,
                "header": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "parent": 38,
                "status": " ",
                "checked": false,
                "completed": false,
                "fullyCompleted": false
            },
            {
                "symbol": "*",
                "link": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "section": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "text": "b",
                "tags": [],
                "line": 40,
                "lineCount": 1,
                "list": 33,
                "outlinks": [],
                "path": "daily-notes-plugin-for-obsidianmd.md",
                "children": [],
                "task": true,
                "annotated": false,
                "position": {
                    "start": {
                        "line": 40,
                        "col": 2,
                        "offset": 495
                    },
                    "end": {
                        "line": 40,
                        "col": 11,
                        "offset": 504
                    }
                },
                "subtasks": [],
                "real": true,
                "header": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "parent": 38,
                "status": "x",
                "checked": true,
                "completed": true,
                "fullyCompleted": true
            },
            {
                "symbol": "*",
                "link": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "section": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "text": "c",
                "tags": [],
                "line": 41,
                "lineCount": 1,
                "list": 33,
                "outlinks": [],
                "path": "daily-notes-plugin-for-obsidianmd.md",
                "children": [],
                "task": true,
                "annotated": false,
                "position": {
                    "start": {
                        "line": 41,
                        "col": 2,
                        "offset": 507
                    },
                    "end": {
                        "line": 41,
                        "col": 11,
                        "offset": 516
                    }
                },
                "subtasks": [],
                "real": true,
                "header": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "parent": 38,
                "status": " ",
                "checked": false,
                "completed": false,
                "fullyCompleted": false
            }
        ],
        "task": true,
        "annotated": false,
        "position": {
            "start": {
                "line": 38,
                "col": 0,
                "offset": 465
            },
            "end": {
                "line": 38,
                "col": 15,
                "offset": 480
            }
        },
        "subtasks": [
            {
                "symbol": "*",
                "link": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "section": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "text": "a",
                "tags": [],
                "line": 39,
                "lineCount": 1,
                "list": 33,
                "outlinks": [],
                "path": "daily-notes-plugin-for-obsidianmd.md",
                "children": [],
                "task": true,
                "annotated": false,
                "position": {
                    "start": {
                        "line": 39,
                        "col": 2,
                        "offset": 483
                    },
                    "end": {
                        "line": 39,
                        "col": 11,
                        "offset": 492
                    }
                },
                "subtasks": [],
                "real": true,
                "header": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "parent": 38,
                "status": " ",
                "checked": false,
                "completed": false,
                "fullyCompleted": false
            },
            {
                "symbol": "*",
                "link": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "section": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "text": "b",
                "tags": [],
                "line": 40,
                "lineCount": 1,
                "list": 33,
                "outlinks": [],
                "path": "daily-notes-plugin-for-obsidianmd.md",
                "children": [],
                "task": true,
                "annotated": false,
                "position": {
                    "start": {
                        "line": 40,
                        "col": 2,
                        "offset": 495
                    },
                    "end": {
                        "line": 40,
                        "col": 11,
                        "offset": 504
                    }
                },
                "subtasks": [],
                "real": true,
                "header": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "parent": 38,
                "status": "x",
                "checked": true,
                "completed": true,
                "fullyCompleted": true
            },
            {
                "symbol": "*",
                "link": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "section": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "text": "c",
                "tags": [],
                "line": 41,
                "lineCount": 1,
                "list": 33,
                "outlinks": [],
                "path": "daily-notes-plugin-for-obsidianmd.md",
                "children": [],
                "task": true,
                "annotated": false,
                "position": {
                    "start": {
                        "line": 41,
                        "col": 2,
                        "offset": 507
                    },
                    "end": {
                        "line": 41,
                        "col": 11,
                        "offset": 516
                    }
                },
                "subtasks": [],
                "real": true,
                "header": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "parent": 38,
                "status": " ",
                "checked": false,
                "completed": false,
                "fullyCompleted": false
            }
        ],
        "real": true,
        "header": {
            "path": "daily-notes-plugin-for-obsidianmd.md",
            "type": "header",
            "subpath": "Testing task list"
        },
        "status": " ",
        "checked": false,
        "completed": false,
        "fullyCompleted": false
    }
]
```
For full json click here: [[#Full json]]




---

## Full json
```json
[
    {
        "symbol": "*",
        "link": {
            "path": "daily-notes-plugin-for-obsidianmd.md",
            "type": "header",
            "subpath": "Testing task list"
        },
        "section": {
            "path": "daily-notes-plugin-for-obsidianmd.md",
            "type": "header",
            "subpath": "Testing task list"
        },
        "text": "project 1",
        "tags": [],
        "line": 33,
        "lineCount": 1,
        "list": 33,
        "outlinks": [],
        "path": "daily-notes-plugin-for-obsidianmd.md",
        "children": [
            {
                "symbol": "*",
                "link": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "section": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "text": "1",
                "tags": [],
                "line": 34,
                "lineCount": 1,
                "list": 33,
                "outlinks": [],
                "path": "daily-notes-plugin-for-obsidianmd.md",
                "children": [],
                "task": true,
                "annotated": false,
                "position": {
                    "start": {
                        "line": 34,
                        "col": 2,
                        "offset": 419
                    },
                    "end": {
                        "line": 34,
                        "col": 11,
                        "offset": 428
                    }
                },
                "subtasks": [],
                "real": true,
                "header": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "parent": 33,
                "status": "x",
                "checked": true,
                "completed": true,
                "fullyCompleted": true
            },
            {
                "symbol": "*",
                "link": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "section": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "text": "2",
                "tags": [],
                "line": 35,
                "lineCount": 1,
                "list": 33,
                "outlinks": [],
                "path": "daily-notes-plugin-for-obsidianmd.md",
                "children": [],
                "task": true,
                "annotated": false,
                "position": {
                    "start": {
                        "line": 35,
                        "col": 2,
                        "offset": 431
                    },
                    "end": {
                        "line": 35,
                        "col": 11,
                        "offset": 440
                    }
                },
                "subtasks": [],
                "real": true,
                "header": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "parent": 33,
                "status": "x",
                "checked": true,
                "completed": true,
                "fullyCompleted": true
            },
            {
                "symbol": "*",
                "link": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "section": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "text": "3",
                "tags": [],
                "line": 36,
                "lineCount": 1,
                "list": 33,
                "outlinks": [],
                "path": "daily-notes-plugin-for-obsidianmd.md",
                "children": [],
                "task": true,
                "annotated": false,
                "position": {
                    "start": {
                        "line": 36,
                        "col": 2,
                        "offset": 443
                    },
                    "end": {
                        "line": 36,
                        "col": 11,
                        "offset": 452
                    }
                },
                "subtasks": [],
                "real": true,
                "header": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "parent": 33,
                "status": " ",
                "checked": false,
                "completed": false,
                "fullyCompleted": false
            },
            {
                "symbol": "*",
                "link": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "section": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "text": "4",
                "tags": [],
                "line": 37,
                "lineCount": 1,
                "list": 33,
                "outlinks": [],
                "path": "daily-notes-plugin-for-obsidianmd.md",
                "children": [],
                "task": true,
                "annotated": false,
                "position": {
                    "start": {
                        "line": 37,
                        "col": 2,
                        "offset": 455
                    },
                    "end": {
                        "line": 37,
                        "col": 11,
                        "offset": 464
                    }
                },
                "subtasks": [],
                "real": true,
                "header": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "parent": 33,
                "status": " ",
                "checked": false,
                "completed": false,
                "fullyCompleted": false
            }
        ],
        "task": true,
        "annotated": false,
        "position": {
            "start": {
                "line": 33,
                "col": 0,
                "offset": 401
            },
            "end": {
                "line": 33,
                "col": 15,
                "offset": 416
            }
        },
        "subtasks": [
            {
                "symbol": "*",
                "link": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "section": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "text": "1",
                "tags": [],
                "line": 34,
                "lineCount": 1,
                "list": 33,
                "outlinks": [],
                "path": "daily-notes-plugin-for-obsidianmd.md",
                "children": [],
                "task": true,
                "annotated": false,
                "position": {
                    "start": {
                        "line": 34,
                        "col": 2,
                        "offset": 419
                    },
                    "end": {
                        "line": 34,
                        "col": 11,
                        "offset": 428
                    }
                },
                "subtasks": [],
                "real": true,
                "header": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "parent": 33,
                "status": "x",
                "checked": true,
                "completed": true,
                "fullyCompleted": true
            },
            {
                "symbol": "*",
                "link": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "section": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "text": "2",
                "tags": [],
                "line": 35,
                "lineCount": 1,
                "list": 33,
                "outlinks": [],
                "path": "daily-notes-plugin-for-obsidianmd.md",
                "children": [],
                "task": true,
                "annotated": false,
                "position": {
                    "start": {
                        "line": 35,
                        "col": 2,
                        "offset": 431
                    },
                    "end": {
                        "line": 35,
                        "col": 11,
                        "offset": 440
                    }
                },
                "subtasks": [],
                "real": true,
                "header": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "parent": 33,
                "status": "x",
                "checked": true,
                "completed": true,
                "fullyCompleted": true
            },
            {
                "symbol": "*",
                "link": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "section": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "text": "3",
                "tags": [],
                "line": 36,
                "lineCount": 1,
                "list": 33,
                "outlinks": [],
                "path": "daily-notes-plugin-for-obsidianmd.md",
                "children": [],
                "task": true,
                "annotated": false,
                "position": {
                    "start": {
                        "line": 36,
                        "col": 2,
                        "offset": 443
                    },
                    "end": {
                        "line": 36,
                        "col": 11,
                        "offset": 452
                    }
                },
                "subtasks": [],
                "real": true,
                "header": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "parent": 33,
                "status": " ",
                "checked": false,
                "completed": false,
                "fullyCompleted": false
            },
            {
                "symbol": "*",
                "link": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "section": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "text": "4",
                "tags": [],
                "line": 37,
                "lineCount": 1,
                "list": 33,
                "outlinks": [],
                "path": "daily-notes-plugin-for-obsidianmd.md",
                "children": [],
                "task": true,
                "annotated": false,
                "position": {
                    "start": {
                        "line": 37,
                        "col": 2,
                        "offset": 455
                    },
                    "end": {
                        "line": 37,
                        "col": 11,
                        "offset": 464
                    }
                },
                "subtasks": [],
                "real": true,
                "header": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "parent": 33,
                "status": " ",
                "checked": false,
                "completed": false,
                "fullyCompleted": false
            }
        ],
        "real": true,
        "header": {
            "path": "daily-notes-plugin-for-obsidianmd.md",
            "type": "header",
            "subpath": "Testing task list"
        },
        "status": " ",
        "checked": false,
        "completed": false,
        "fullyCompleted": false
    },
    {
        "symbol": "*",
        "link": {
            "path": "daily-notes-plugin-for-obsidianmd.md",
            "type": "header",
            "subpath": "Testing task list"
        },
        "section": {
            "path": "daily-notes-plugin-for-obsidianmd.md",
            "type": "header",
            "subpath": "Testing task list"
        },
        "text": "project 2",
        "tags": [],
        "line": 38,
        "lineCount": 1,
        "list": 33,
        "outlinks": [],
        "path": "daily-notes-plugin-for-obsidianmd.md",
        "children": [
            {
                "symbol": "*",
                "link": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "section": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "text": "a",
                "tags": [],
                "line": 39,
                "lineCount": 1,
                "list": 33,
                "outlinks": [],
                "path": "daily-notes-plugin-for-obsidianmd.md",
                "children": [],
                "task": true,
                "annotated": false,
                "position": {
                    "start": {
                        "line": 39,
                        "col": 2,
                        "offset": 483
                    },
                    "end": {
                        "line": 39,
                        "col": 11,
                        "offset": 492
                    }
                },
                "subtasks": [],
                "real": true,
                "header": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "parent": 38,
                "status": " ",
                "checked": false,
                "completed": false,
                "fullyCompleted": false
            },
            {
                "symbol": "*",
                "link": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "section": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "text": "b",
                "tags": [],
                "line": 40,
                "lineCount": 1,
                "list": 33,
                "outlinks": [],
                "path": "daily-notes-plugin-for-obsidianmd.md",
                "children": [],
                "task": true,
                "annotated": false,
                "position": {
                    "start": {
                        "line": 40,
                        "col": 2,
                        "offset": 495
                    },
                    "end": {
                        "line": 40,
                        "col": 11,
                        "offset": 504
                    }
                },
                "subtasks": [],
                "real": true,
                "header": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "parent": 38,
                "status": "x",
                "checked": true,
                "completed": true,
                "fullyCompleted": true
            },
            {
                "symbol": "*",
                "link": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "section": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "text": "c",
                "tags": [],
                "line": 41,
                "lineCount": 1,
                "list": 33,
                "outlinks": [],
                "path": "daily-notes-plugin-for-obsidianmd.md",
                "children": [],
                "task": true,
                "annotated": false,
                "position": {
                    "start": {
                        "line": 41,
                        "col": 2,
                        "offset": 507
                    },
                    "end": {
                        "line": 41,
                        "col": 11,
                        "offset": 516
                    }
                },
                "subtasks": [],
                "real": true,
                "header": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "parent": 38,
                "status": " ",
                "checked": false,
                "completed": false,
                "fullyCompleted": false
            }
        ],
        "task": true,
        "annotated": false,
        "position": {
            "start": {
                "line": 38,
                "col": 0,
                "offset": 465
            },
            "end": {
                "line": 38,
                "col": 15,
                "offset": 480
            }
        },
        "subtasks": [
            {
                "symbol": "*",
                "link": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "section": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "text": "a",
                "tags": [],
                "line": 39,
                "lineCount": 1,
                "list": 33,
                "outlinks": [],
                "path": "daily-notes-plugin-for-obsidianmd.md",
                "children": [],
                "task": true,
                "annotated": false,
                "position": {
                    "start": {
                        "line": 39,
                        "col": 2,
                        "offset": 483
                    },
                    "end": {
                        "line": 39,
                        "col": 11,
                        "offset": 492
                    }
                },
                "subtasks": [],
                "real": true,
                "header": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "parent": 38,
                "status": " ",
                "checked": false,
                "completed": false,
                "fullyCompleted": false
            },
            {
                "symbol": "*",
                "link": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "section": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "text": "b",
                "tags": [],
                "line": 40,
                "lineCount": 1,
                "list": 33,
                "outlinks": [],
                "path": "daily-notes-plugin-for-obsidianmd.md",
                "children": [],
                "task": true,
                "annotated": false,
                "position": {
                    "start": {
                        "line": 40,
                        "col": 2,
                        "offset": 495
                    },
                    "end": {
                        "line": 40,
                        "col": 11,
                        "offset": 504
                    }
                },
                "subtasks": [],
                "real": true,
                "header": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "parent": 38,
                "status": "x",
                "checked": true,
                "completed": true,
                "fullyCompleted": true
            },
            {
                "symbol": "*",
                "link": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "section": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "text": "c",
                "tags": [],
                "line": 41,
                "lineCount": 1,
                "list": 33,
                "outlinks": [],
                "path": "daily-notes-plugin-for-obsidianmd.md",
                "children": [],
                "task": true,
                "annotated": false,
                "position": {
                    "start": {
                        "line": 41,
                        "col": 2,
                        "offset": 507
                    },
                    "end": {
                        "line": 41,
                        "col": 11,
                        "offset": 516
                    }
                },
                "subtasks": [],
                "real": true,
                "header": {
                    "path": "daily-notes-plugin-for-obsidianmd.md",
                    "type": "header",
                    "subpath": "Testing task list"
                },
                "parent": 38,
                "status": " ",
                "checked": false,
                "completed": false,
                "fullyCompleted": false
            }
        ],
        "real": true,
        "header": {
            "path": "daily-notes-plugin-for-obsidianmd.md",
            "type": "header",
            "subpath": "Testing task list"
        },
        "status": " ",
        "checked": false,
        "completed": false,
        "fullyCompleted": false
    }
]
```

## On dataview random number generation and intracacies of flatten

- 📃✨ its so unperformant. might as well use javascript.
- ⚙creates random number using dql lang
~~~sql
```dataview
TABLE RNG, RNG2,file.size, (this.seed - file.size) as me, this.seed as seed
FLATTEN (file.size + this.seed) %50 as RNG 
FLATTEN (this.seed - file.size) %101 as RNG2 
sort RNG desc, RNG2 desc limit 10
```

## Ω

---
~~~
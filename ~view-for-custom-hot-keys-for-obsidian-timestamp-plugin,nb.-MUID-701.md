---
CREATION_DATE: 2023-06-08
DOC_VERSION: v0.0.0
MUID: MUID-701
tag: _meta
TEMPLATE_VERSION: v1.0.1-blank-template
---

# - 

- & This partial shows the hot keys for the timestamper plugin
* ⚖
  * Dependencies
    * Requires knowledge of the plugin name 

* ℹ To create similar hotkey views for another plugin, use [[∑--~viewfn-to-recall-hot-key]] on a new note.

# =

`=this.file.name`
```dataviewjs
const keyToSymbolMap = {
  'Mod': '⌘',
  'Shift': '⇧',
  'Alt': '⌥',
  'Ctrl': '⌃',
  'ArrowRight': '→',
  'ArrowLeft': '←',
  'ArrowUp': '↑',
  'ArrowDown': '↓',
  'Enter': '⏎',
  'Tab': '⇥',
  ' ': 'space'
};
const {customKeys} = this.app.hotkeyManager;
const {commands} = this.app.commands;




const pluginName = `obsidian-timestamp`

const keyToModsMaps = [];
for (let key in customKeys) {
    if (key.startsWith(pluginName)) {
        keyToModsMaps.push({
            ck: customKeys[key],
            co: commands[key]
        })
    }
}

dv.paragraph(`${keyToModsMaps.length} \
     ==${pluginName}== plugin commands with assigned \
     hotkeys.<br><br>\
 `);

 
console.log({keyToModsMaps})
const fnt = () => keyToModsMaps
    .map(
        ({co: {name, id}, ck}) => {
            const {key, modifiers } = ck.first();
            console.log(typeof(key),key, 'dkdj')
            const chars = [ key, modifiers[0], modifiers[1] ]
                .reverse();
            const icons = chars
                .map(
                    (char) => keyToSymbolMap[char] 
                        || char).join("");
            return [name.split(':')[1],id.split(':')[1],icons]
        }
    )

dv.table(
    ["Command ID", "Name in current locale", "Hotkeys"],
    fnt()
);

```

# ---Transient Local Archives

## LA--archive--2023-10-13


````

```dataviewjs
const keyToSymbolMap = {
  'Mod': '⌘',
  'Shift': '⇧',
  'Alt': '⌥',
  'Ctrl': '⌃',
  'ArrowRight': '→',
  'ArrowLeft': '←',
  'ArrowUp': '↑',
  'ArrowDown': '↓',
  'Enter': '⏎',
  'Tab': '⇥',
  ' ': 'space'
};
const {customKeys} = this.app.hotkeyManager;
const {commands} = this.app.commands;




const pluginName = `obsidian-timestamp`

const keyToModsMaps = [];
for (let key in customKeys) {
    if (key.startsWith(pluginName)) {
        keyToModsMaps.push({
            ck: customKeys[key],
            co: commands[key]
        })
    }
}

dv.paragraph(keyToModsMaps.length + " commands with assigned hotkeys.<br><br>");

 
console.log({keyToModsMaps})
const fnt = () => keyToModsMaps
    .map(
        ({co: {name, id}, ck}) => {
            const {key, modifiers } = ck.first();
            console.log(typeof(key),key, 'dkdj')
            const chars = [ key, modifiers[0], modifiers[1] ]
                .reverse();
            const icons = chars
                .map(
                    (char) => keyToSymbolMap[char] 
                        || char).join("");
            return [name.split(':')[1],id.split(':')[1],icons]
        }
    )

dv.table(
    ["Command ID", "Name in current locale", "Hotkeys"],
    fnt()
);

```
````

##  LA--archive--2023-06-06 


````
```dataviewjs
const keyToSymbolMap = {
  'Mod': '⌘',
  'Shift': '⇧',
  'Alt': '⌥',
  'Ctrl': '⌃',
  'ArrowRight': '→',
  'ArrowLeft': '←',
  'ArrowUp': '↑',
  'ArrowDown': '↓',
  'Enter': '⏎',
  'Tab': '⇥',
  ' ': 'space'
};
const {customKeys} = this.app.hotkeyManager;
const {commands} = this.app.commands;




const pluginName = `obsidian-timestamp`

const keyToModsMaps = [];
for (let key in customKeys) {
    if (key.startsWith(pluginName)) {
        keyToModsMaps.push({
            ck: customKeys[key],
            co: commands[key]
        })
    }
}

dv.paragraph(keyToModsMaps.length + " commands with assigned hotkeys.<br><br>");

 
console.log({keyToModsMaps})
const fnt = () => keyToModsMaps
    .map(
        ({co: {name, id}, ck}) => {
            const {key, modifiers } = ck.first();
            console.log(typeof(key),key, 'dkdj')
            const chars = [ key, modifiers[0], modifiers[1] ]
                .reverse();
            const icons = chars
                .map(
                    (char) => keyToSymbolMap[char] 
                        || char).join("");
            return [name.split(':')[1],id.split(':')[1],icons]
        }
    )

dv.table(
    ["Command ID", "Name in current locale", "Hotkeys"],
    fnt()
);

```
````
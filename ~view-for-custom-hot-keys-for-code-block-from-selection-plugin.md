---
CREATION_DATE: 2023-09-29 
DOC_VERSION: v0.0.1
MUID: MUID-1539
tag: _wip 
TEMPLATE_VERSION: v1.0.1-blank-template
UMID: 
---



# - 
This partial shows the hot keys for the timestamper plugin
* ⚖
    * Dependencies
        * Requires knowledge of the plugin name 

* ℹ To create similar hotkey views for another plugin, use [[wip_viewfn-to-recall-hot-key]] on a new note.

# =

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




const pluginName = `code-block-from-selection`

const keyToModsMaps = [];
for (let key in customKeys) {
    if (key.startsWith(pluginName)) {
        if (customKeys[key] && commands[key]) {
          keyToModsMaps.push({
              ck: customKeys[key],
              co: commands[key]
          })
        }
    }
}

dv.paragraph(`${keyToModsMaps.length} \
     ==${pluginName}== plugin commands with \nassigned \
     hotkeys.<br>`);

console.log({keyToModsMaps})
const fnt = () => keyToModsMaps
    .map(
        (obj) => {
            console.log({obj})
            if (!obj?.co) {
              return null
            }
            const {co: {name, id}, ck} = obj;
            const {key, modifiers } = ck.first();
            console.log(typeof(key),key, 'dkdj')
            const chars = [ key, modifiers[0], modifiers[1] ]
                .reverse();
            const icons = chars
                .map(
                    (char) => keyToSymbolMap[char] 
                        || char).join("");
            return [
              name,
              icons
            ];
        }
    )
const fnts = fnt();
dv.table(
    ["Name", "Hotkeys"],
    fnts
);

```

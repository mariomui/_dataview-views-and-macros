---
tags: _misc/_wip
---


# -

## About

This [[Partial-dataview,vis-Noteshippo,]] is designed to provide a list ot hotkey/shortcuts of used by  [[C10,master-list-of-plugins,bt.-ObsidianMD-app,etc|_list-of-obsidianMD-plugins]].

* [ ] Refactor code to use the improved  transclusion parameter design rather than by user prompt. #_todo/to-code/upon-codelet

* ! Even if the coe is broken it's still good!

# =

* ! Transclusion has not been implemented
* 🤔 so far this is a macro.

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




const pluginName = `null`

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

<%*/*
HC-VERSION: v1.0.2-RHK
DESC: User prompted hotkey lister
RHK-HC-COMMITS: [
    "v1.0.2: Add runtime prompt for plugin name to obtain dynamic output"
    "v1.0.1: Better prompt",
   " v1.0.0: Pump out hotkeys for timestamp note plugin"
] */%>

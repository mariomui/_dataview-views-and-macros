---
tag: _meta 
---

# -

## About

- [ ]  How is this code different from [[~view-for-custom-hot-keys-for-obsidian-timestamp-plugin]]?


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



# ---Transient

## v2023-06-06 
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

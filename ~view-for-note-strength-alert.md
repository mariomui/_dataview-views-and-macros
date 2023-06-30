---
tag: _meta
---
# -

# =
```dataviewjs
const {default:obs} = this.app.plugins.plugins['templater-obsidian'].templater.current_functions_object.obsidian;

const {frontmatter} = dv.current().file;
const checkIfStructuralNote =() => {
    if (!frontmatter.has(NTS_TYPE)) {
        return true;
    }
}
const isStructuralNote = frontmatter.tag?.includes('_nts-v1/structural-note') || true;

const sharedStyleConfig = {
    padding: ".5em .5em",
    ["border-radius"]: "0 1em 0 1em"
}
const structuralConfig = {
    message: "I am a structural note",
    style: {
        background: "var(--material-color-red-900)",
        ...sharedStyleConfig
    }
};
const foundationalConfig = {
    message: "I am a foundational note",
    style: {
        background: "var(--material-color-blue-900)",
        ...sharedStyleConfig
    }
};
const style = isStructuralNote 
    ? structuralConfig.style
    : foundationalConfig.style
const message = isStructuralNote 
    ? structuralConfig.message
    : foundationalConfig.message

const el = dv.paragraph(message);

for (let key in style) {
    el.style[key] = style[key];
}

if (!this.marioinjectionConfig) {
    this.marioinjectionConfig = {
        hasPoppedAlert: false
    };
}
if (!this.marioinjectionConfig.hasPoppedAert) {

    new obs.Notice(el.cloneNode(true), 6000);
    this.marioinjectionConfig.hasPoppedAert = true;
} 
```
# ---Transient Sandbox
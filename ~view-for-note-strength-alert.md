---
tag: _meta
---
# -
- [ ] Reduce as much dataview as possible
# =
```dataviewjs
const { default: obs } =
  this.app.plugins.plugins["templater-obsidian"].templater
    .current_functions_object.obsidian;
const { workspace, metadataCache } = this.app;

const vf = workspace.getActiveFile();
const {frontmatter} = metadataCache.getFileCache(vf)

//knobs 
// any helper function localled into main should also bring its knobs inside. Which breaks the data boundary. 
// hrm.

const STRUCTURAL_NOTE = "_nts-v1/structural-note";


const sharedStyleConfig = {
  padding: ".5em .5em",
  ["border-radius"]: "0 1em 0 1em",
};
const structuralConfig = {
  message: "I am a structural note",
  style: {
    background: "var(--material-color-red-900)",
    ...sharedStyleConfig,
  },
};

const nonstructuralConfig = {
  message: "I am a yet undefined",
  style: {
    background: "var(--material-color-blue-900)",
    ...sharedStyleConfig,
  },
};

// entrypoint
workspace.onLayoutReady(bootstrap.bind(this));

// if you put everything into modules, the complexity will always go up. Some function belong inside the bootstrap to lower complexity even if composability is decreased.
function bootstrap() {
  (function bootstrap(
  // easy documentation but makes code hard to read.
    ctx,
    checkTagsForTagName, // normal function stuff in
    manuMainConfig,
    obtainMessageConfigByNoteType, // these should be in
    once, // this should be out
    renderMessage, // ui methods should be out
  ) {
    const helperFunctions = {
      checkTagsForTagName,
      obtainMessageConfigByNoteType,
      once,
      renderMessage,
    };
    const mainConfig = manuMainConfig({ helperFunctions });
    main.call(ctx, frontmatter, mainConfig);
  })(
    this,
    checkTagsForTagName.bind(this),
    manuMainConfig.bind(this),
    obtainMessageConfigByNoteType.bind(this),
    once.bind(this),
    renderMessage.bind(this),
  );
}
// arity control
function manuMainConfig(config = {}) {
  const helperFunctions = {
    checkTagsForTagName,
    obtainMessageConfigByNoteType,
    once,
    renderMessage,
    ...(config?.helperFunctions || {}),
  };
  return {
    helperFunctions,
  };
}
// workhorse
function main(frontmatter, config = manuMainConfig()) {
  const { helperFunctions } = config;
  const { checkTagsForTagName, obtainMessageConfigByNoteType, once } =
    helperFunctions;

  // BUSINESS BOUNDARY
  const isStructuralNote = checkTagsForTagName(
    frontmatter?.tag, STRUCTURAL_NOTE
  );
  

  const { message, style } = obtainMessageConfigByNoteType(
    isStructuralNote
  );

  // UI Boundary
  const el = renderMessage(message);
  for (let key in style) {
    el.style[key] = style[key];
  }
  //once(this.fig, el)
  new obs.Notice(el.cloneNode(true), 6000);
}
// ## ui helpers
function renderMessage(message) {
  return dv.paragraph(message);
}
// ## business helper

function manuObtainConfigByNoteTypeConfig() {
  return {
    nonstructuralConfig,
    structuralConfig,
  };
}
function obtainMessageConfigByNoteType(
  isStructuralNote = false,
  config = manuObtainConfigByNoteTypeConfig(),
) {
  const { structuralConfig, nonstructuralConfig } = config;
  if (isStructuralNote) {
    return structuralConfig;
  }
  // think of a strategy pattern here.
  return nonstructuralConfig;
}


function checkTagsForTagName(tags, tagName) {
  if (tags === undefined) return true;
  if (tags.length === 0) return true;
  return tags.includes(tagName) 
}

function once(fig, el) {
  if (!fig) {
    fig = {
      hasPoppedAlert: false,
    };
    return;
  }
  if (!fig.hasPoppedAert) {
    new obs.Notice(el.cloneNode(true), 6000);
    fig.hasPoppedAert = true;
  }
}
/**

**/
```
# ---Transient
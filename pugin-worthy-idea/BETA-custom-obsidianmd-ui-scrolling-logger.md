## When to use getlinkpastotfirstdest 

### breadcrumbs

* this.container.win.createFragment
  * the dataview api allows access to windows via a property inside it.
  * the dataview createParagraph stuff automatically does some sort of markdown rendering on the text html you place into the the parameter.
  * 
```
window.createFragment = function(t) {
    var e = document.createDocumentFragment();
    return t && t(e),
    e
}
```

~~~dataviewjs
const file_path = "A_fleeting_notes/2023-07-05.md";

const {default: obs} = this.app.plugins.plugins['templater-obsidian'].templater.current_functions_object.obsidian

const logg = createLogg.call(this);
this.app.workspace.onLayoutReady(main.bind(this))

function main() {
  const _vfile = this.app.workspace.getActiveFile()
  const vfile = this.app.vault.getAbstractFileByPath(file_path);
  const cache = this.app.metadataCache.getFileCache(vfile)

  const serializerConfig = {
    serialize
  }
  const woot = this.app.metadataCache.getFirstLinkpathDest(vfile.basename, vfile.path);
  
  logg.call(this,
    woot, 
    { 
      time: 50000,
      serializer: serializerConfig
    }
  );
  function serialize(text) {
    const newline = "<br/>"
    const tab = "&nbsp";
    const checkIsObject = (v) => {
      return typeof v === 'object' && !Array.isArray(v) 
    };
    const getType = (v) => {
      if (checkIsObject(v)) return 'object';
      if (Array.isArray(v)) return 'array';
      return typeof v;
    }
  
    if (typeof text === 'function') return text?.name;
    // return things that arent an object
    if (!checkIsObject(text)) return `${String(text)}`;
    let res = "";
    if (!text) return "undefined"
    
    // top level
    for (const key in text) {
      const val = `<hr/>${key}: ${JSON.stringify(text[key], customReplacer)} ${newline}`;
      res += val;
    }
    return res.replaceAll("\"", "");
 
    function customReplacer(k,v) {
      if (checkIsObject(v)) {
        return `${recurse(v, 1)}`
      }
      if (typeof v === "function") return '() => eh';
      return `${newline}  ${String(v)} :: ${typeof v}`;
    }
    function recurse(v, counter) {
      // how many levels deep do you want to string.
      const indent = tab.repeat(counter*5)
      if (counter > 5) {
        if (['number','boolean', 'string'].includes(typeof v)) {
          return String(v) + "last";
        }
        return `${getType(v)}end`;
      }
      if (!v) return typeof v;
      if (checkIsObject(v)) {
        let res = ``;
        let keyCnt = 0;
        for (const [key, value] of Object.entries(v)) {
          if (keyCnt < 5) {
        
            res += `${newline}${indent}${key}:${recurse(value, counter + 1)}`
          } else {
            break;
          }
          keyCnt++;
        }
        return `${res}`;
      }
      if (typeof v === "function") return `${indent}() => eh`;
      //return `${tab.repeat(counter)}`;
    }
  }
}

function manuCreateLogg(config = {}) {
  return {
    time:5000, 
    isBypass:false,
    serializer: {
       serialize: JSON.stringify
    },
    ...config
  }
}
function createLogg(config = manuCreateLogg()) {
  const _config = {
    ...manuCreateLogg(),
    ...config
  }
  return function logger(text, __config = _config) {
    const config = {
      ..._config,
      ...__config
    }
    const {
      isBypass, 
      serializer,
      time
    } = config;
     if (!serializer?.serialize) {
      return new obs.Notice('serialize not found')
    }
    const { serialize } = serializer;
    const isSilent = this?.fig?.silent === true;
    if (isSilent && !isBypass) {
       return;
    }

    (async function(ctx) {
      const serialized_text = serialize(text);
      const {$el} = await genCreateDiv(ctx, {text: ''})

      const rendered_text = await obs.MarkdownRenderer.render(
        ctx.app,
        serialized_text,
        $el,
        "",
        ctx.container.component
      )
      //const $el = await dv.el("div", {text: serialize(text)})
      const _text = typeof text === "string" 
          ? text 
          : $el;
      $el.style.background = "var(--material-color-red-800)";
      const $notice = await new obs.Notice(_text, time)
      $notice.noticeEl.style['max-width'] = "500px"
      $notice.noticeEl.style['max-height'] = "500px";
      $notice.noticeEl.style['overflow'] = "scroll";
      console.log({$notice})
    })(this)
    
    function genCreateFrag(ctx) {
      return new Promise((resolve, rej) => {
        ctx.container.win.createFragment(($el) => {
          ctx.app.workspace.onLayoutReady(() => {
            resolve({$el})
          })
        })
      })
    }
    async function genCreateDiv(ctx, config = {text: ''}) {
      const {text} = config;
      const $frag = await genCreateFrag(ctx);
      return new Promise((resolve, reject) => {
        const domInfo = {
          text,
          parent: $frag
        };
        ctx.container.win.createDiv(domInfo, callback)
        function callback($el) {
          ctx.app.workspace.onLayoutReady(() => {
            resolve({$el, $frag})
          })
        }
      })
    }
  }
}
~~~
some logging.
# --- Transient Sandbox

v1.0.0
```js
~~~dataviewjs
const file_path = "A_fleeting_notes/2023-07-05.md";

const {default: obs} = this.app.plugins.plugins['templater-obsidian'].templater.current_functions_object.obsidian

const logg = createLogg.call(this);
this.app.workspace.onLayoutReady(main.bind(this))

function main() {
  const _vfile = this.app.workspace.getActiveFile()
  const vfile = this.app.vault.getAbstractFileByPath(file_path);
  const cache = this.app.metadataCache.getFileCache(vfile)

  const serializerConfig = {
    serialize
  }
  const woot = this.app.metadataCache.getFirstLinkpathDest(vfile.basename, vfile.path);
  
  logg.call(this,
    woot, 
    { 
      time: 50000,
      serializer: serializerConfig
    }
  );
  function serialize(text) {
    const newline = "<br/>"
    const tab = "&nbsp";
    const checkIsObject = (v) => {
      return typeof v === 'object' && !Array.isArray(v) 
    };
    const getType = (v) => {
      if (checkIsObject(v)) return 'object';
      if (Array.isArray(v)) return 'array';
      return typeof v;
    }
  
    if (typeof text === 'function') return text?.name;
    // return things that arent an object
    if (!checkIsObject(text)) return `${String(text)}`;
    let res = "";
    if (!text) return "undefined"
    
    // top level
    for (const key in text) {
      const val = `<hr/>${key}: ${JSON.stringify(text[key], customReplacer)} ${newline}`;
      res += val;
    }
    return res.replaceAll("\"", "");
 
    function customReplacer(k,v) {
      if (checkIsObject(v)) {
        return `${recurse(v, 1)}`
      }
      if (typeof v === "function") return '() => eh';
      return `${newline}  ${String(v)} :: ${typeof v}`;
    }
    function recurse(v, counter) {
      // how many levels deep do you want to string.
      const indent = tab.repeat(counter*5)
      if (counter > 5) {
        if (['number','boolean', 'string'].includes(typeof v)) {
          return String(v) + "last";
        }
        return `${getType(v)}end`;
      }
      if (!v) return typeof v;
      if (checkIsObject(v)) {
        let res = ``;
        let keyCnt = 0;
        for (const [key, value] of Object.entries(v)) {
          if (keyCnt < 5) {
        
            res += `${newline}${indent}${key}:${recurse(value, counter + 1)}`
          } else {
            break;
          }
          keyCnt++;
        }
        return `${res}`;
      }
      if (typeof v === "function") return `${indent}() => eh`;
      //return `${tab.repeat(counter)}`;
    }
  }
}

function manuCreateLogg(config = {}) {
  return {
    time:5000, 
    isBypass:false,
    serializer: {
       serialize: JSON.stringify
    },
    ...config
  }
}
function createLogg(config = manuCreateLogg()) {
  const _config = {
    ...manuCreateLogg(),
    ...config
  }
  return function logger(text, __config = _config) {
    const config = {
      ..._config,
      ...__config
    }
    const {
      isBypass, 
      serializer,
      time
    } = config;
     if (!serializer?.serialize) {
      return new obs.Notice('serialize not found')
    }
    const { serialize } = serializer;
    const isSilent = this?.fig?.silent === true;
    if (isSilent && !isBypass) {
       return;
    }

    (async function(ctx) {
      const serialized_text = serialize(text);
      const {$el} = await genCreateDiv(ctx, {text: ''})

      const rendered_text = await obs.MarkdownRenderer.render(
        ctx.app,
        serialized_text,
        $el,
        "",
        ctx.container.component
      )
      //const $el = await dv.el("div", {text: serialize(text)})
      const _text = typeof text === "string" 
          ? text 
          : $el;
      $el.style.background = "var(--material-color-red-800)";
      const $notice = await new obs.Notice(_text, time)
      $notice.noticeEl.style['max-width'] = "500px"
      $notice.noticeEl.style['max-height'] = "500px";
      $notice.noticeEl.style['overflow'] = "scroll";
      console.log({$notice})
    })(this)
    
    function genCreateFrag(ctx) {
      return new Promise((resolve, rej) => {
        ctx.container.win.createFragment(($el) => {
          ctx.app.workspace.onLayoutReady(() => {
            resolve({$el})
          })
        })
      })
    }
    async function genCreateDiv(ctx, config = {text: ''}) {
      const {text} = config;
      const $frag = await genCreateFrag(ctx);
      return new Promise((resolve, reject) => {
        const domInfo = {
          text,
          parent: $frag
        };
        ctx.container.win.createDiv(domInfo, callback)
        function callback($el) {
          ctx.app.workspace.onLayoutReady(() => {
            resolve({$el, $frag})
          })
        }
      })
    }
  }
}
~~~
```

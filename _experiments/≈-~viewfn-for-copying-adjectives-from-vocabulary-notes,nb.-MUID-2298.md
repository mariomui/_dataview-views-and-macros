---
CREATION_DATE: "2024-05-12"
MUID: MUID-2298
PROJECT_PARENT: 
TEMPLATE_SOURCE: "[[∑--experiment-template]]"
TEMPLATE_VERSION: v1.0.1
tags:
  - _meta
---

# -
## 00-Meta

> [!info]- Progress Bar v0.0.3
> > ![[~view-for-local-tasks-using-a-progress-bar,nb.-MUID-698#=|olk]]
> ```dataview
> task where file.name = this.file.name and !completed
> ```
> >
> ```dataview
> task where file.name = this.file.name and completed
> ```

### 10÷About

* ? Should [[,aka-experiment-specced-note]]s merge with [[,aka-question_specced_note]]?
	* no, [[,aka-question-note-template]] isn't suitable for experiment notes. Qualitative templates and quantitative templates have different purposes.
- The goal of this [[Partial-dataview,vis-Noteshippo,]] is to collect the [[interim--class-type-category-symbol,uti.-classical_building-emoji,bt.-Noteshippo-CLA,]] tagged notes into a queue. 
	- @ Inputs,
		- & MUID of the target file. 
		- & target_text 
			- ~ 🔎  ⚛🏛 [[adjective-phrase,vis-Grammar,]]
				- ! it should be robust to handle brackets in the query parameters
				- 🎲 [[sometimes-or-typically-symbol,uti.-game_die-emoji,bt.-Noteshippo-content-level-affix,]]
	- @ Output 
		- none 
	- @ Side Effect 
		- A rendered display of the list of Headers whose contents contain the *target_text*

## 20-Inlink

> [!abstract]- %%  %% Automated List of Reference Inlinks (v0.0.5)
> * ℹ Commit/design logs are located in this [[,aka-MUID-150|experiment note]].
> > `= join( map( sort( map( filter(this.file.inlinks, (link) => meta(link).path != this.file.path), (x) => [ split(meta(x).path, "/")[length(split(meta(x).path, "/")) - 1], x ] ) ), (b) => "• " + choice( length(b[0]) > 28, link( b[1], truncate( regexreplace(b[0], "(-of|of|the|-the|-for|-that|https-|ee)", ""), length( regexreplace(b[0], "(-of|of|the|-the|-for|-that|https-|ee)", "") ) * 0.75 ) ), link(b[1], regexreplace(b[0], "\.md$", "")) ) ), "<br>" )`

# =

```dataviewjs
// # PLACEHOLDER
const renderUI = {};

	// # PRE-INSTANCED TOOLS
const { default: obs } = this.app.plugins?.plugins?.["templater-obsidian"]?.templater
		?.current_functions_object?.obsidian;
const {metadataCache,vault,workspace,fileManager} = this.app

const ACTIVE_DEBUG = true; // todo , extract this prop from the consuming file.
const debug = (...args) => {
	if (ACTIVE_DEBUG) {
		console.log.call(console, ...args)
	}
}

const providing_path = getCFP.call(this)

workspace.onLayoutReady(bootup.bind(this))

function bootup() {
	const extractManager = new ExtractManager(obs, metadataCache, vault);
	const boundedGenMain = genMain.bind(this);
	const ctx = {};
	(boundedGenMain)(ctx, {
		extractManager,
		renderUI,
		providing_path
	})
}

async function genMain(ctx,
	{
		extractManager, 
		renderUI,
		providing_path
	}
) {
	// # CONSTANTS & TOOLS
	const _SEARCH_TERM = "---Transient Local Query";

	// # LOGIC 

	const argMap = extractManager.extractParams(
		providing_path,
		workspace.getActiveFile.call(workspace)
	)
	debug({providing_path,argMap})


	const search_term = argMap.search_term || _SEARCH_TERM;

}

// ## Small Utilities

// Gets the templater file path
function getCFP() {
	return this.currentFilePath
}

// ## Large Helper Utilites
function ExtractManager(obs, metadataCache, vault) {


	this.vaultAdapterParse = vault.adapter.path.parse.bind(vault.adapter);
	this.vaultAdapterUrlParse = vault.adapter?.url?.parse.bind(vault.adapter);
	this.getMetadataFileCache = metadataCache.getFileCache.bind(metadataCache);
	this.obsParseLinktext = obs.parseLinktext.bind(obs);

	this.extractTargetEmbed = extractTargetEmbed.bind(this);
	this.manuParams = manuParams.bind(this)
	this.parseStringToMap = parseStringToMap.bind(this);
	this.getEmbedsFromVf = getEmbedsFromVf.bind(this);
	this.extractParams = extractParams.bind(this);

	function manuParams() {
		return {
			search_term: "",
			regex_flag: "g"
		}
	}

	function extractParams(
		current_filepath,
		vf
	) {

		const embeds = this.getEmbedsFromVf(vf)
		const {name} = this.vaultAdapterParse(current_filepath);
		const embed = this.extractTargetEmbed(
			name,
			embeds
		);
		debug({embeds,name,embed})
		const displayText = embed?.displayText;


		if (!displayText) {
			return this.manuParams()
		};

		const argMap = this.parseStringToMap(
			displayText
		);
		debug({argMap})

		return {
			...(this.manuParams()),
			...argMap
		};
		// /end params extraction

		// helpers
	}

	function getEmbedsFromVf(vf) {
		return this.getMetadataFileCache(vf)?.embeds || [];
	}


	function extractTargetEmbed(
		embed_name,
		embeds
	) {
		const embed = embeds?.find(
		(embed) => {
			const parsed = this.obsParseLinktext(
				embed.link
			);
			return parsed.path === embed_name;
		});
		return embed;
	}


	function parseStringToMap(str) {
		{ var parsed = {};
			try {
				parsed = this.vaultAdapterUrlParse(str, true);
				debug({parsed, str})
			} catch (err) {
				console.log({err})
				return {
					err
				}
			}
			return parsed?.query || {};
		}
		return {};
	}
}
```

# ---Transient

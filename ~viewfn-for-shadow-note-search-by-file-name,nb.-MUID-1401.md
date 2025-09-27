---
CREATION_DATE: "2023-08-04"
DOC_VERSION: v1.0.6
MUID: MUID-1401
PROJECT_PARENT: 
VERSION: "[[~viewfn-for-shadow-note-search-by-file-name,nb.-MUID-1401]]"
tags:
  - _misc/_wip
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

* [ ] make a dashboard that views pending notes. Or adapt pending notes so it ignores specific folders. ➕ 2025-03-14 #_todo/50-backlog--/to-code
- [[dashboard,view-all-pending-notes]]
### 10÷About

* This is a [[Partial-dataview,vis-Noteshippo,]] designed to search [[,aka-shadow-note]]s for a specific term using regex.

[[≠--tcode,bt.-Noteshippo-title-level-affix,]]
* Rough structure is ⤵
```ts
interface TCODEID4 {
		search_term: "",
		regex_flag: "g"
}
TCODEID4: (queryString: TCODEID4) => void
```

![[~viewfn-for-shadow-note-search-by-file-name,nb.-MUID-1401#LV--Usage Guide|olk]]

* The code used here might be more suitable for a plugin.

* After trying [[Obsidian-text-expander-plugin,b.t.-ObsidianMD]], the ability to create a query without resulting to pulling it from the search bar is missing.
	* @ Cons
		* Hard codes the file names exposing vulnerableness to stale data.
			* Mutates the file note (displeases me greatly)

### 11÷Reference

* † [[,aka-shadow-note]]

## 20-Inlink

> [!abstract]- %%  %% Automated List of Reference Inlinks (v0.0.5)
> * ℹ Commit/design logs are located in this [[,aka-MUID-150|experiment note]].
> > `= join( map( sort( map( filter(this.file.inlinks, (link) => meta(link).path != this.file.path), (x) => [ split(meta(x).path, "/")[length(split(meta(x).path, "/")) - 1], x ] ) ), (b) => "• " + choice( length(b[0]) > 28, link( b[1], truncate( regexreplace(b[0], "(-of|of|the|-the|-for|-that|https-|ee)", ""), length( regexreplace(b[0], "(-of|of|the|-the|-for|-that|https-|ee)", "") ) * 0.75 ) ), link(b[1], regexreplace(b[0], "\.md$", "")) ) ), "<br>" )`

# =

**base_filepath-v0.0.4**: `= choice( contains(this.file.folder, this.file.name), link(this.file.name), join(["*",this.file.name,"*"], ""))` doc-`= this.DOC_VERSION` / ids: `= this.MUID`,`= this.UMID` / lcsh: `= this.heading`

![[~viewfn-for-shadow-note-search-by-file-name,nb.-MUID-1401#LV--Usage Guide|olk]]

```dataviewjs
const {default: obs} = this.app.plugins.plugins['templater-obsidian'].templater.current_functions_object.obsidian;
const {
	workspace, metadataCache, vault
} = this.app;
const {adapter} = vault;

const getCFP = () => this.currentFilePath;

workspace.onLayoutReady(bootup.bind(this))

function bootup() {
	main.call(this)();
}
function main() {
	return function () {
		const providing_path = getCFP()
		const argMap = extractParams(
			providing_path,
			workspace.getActiveFile()
		)
	
		const pvf = vault
			.getAbstractFileByPath(
				providing_path
			);
		const {
			frontmatter: fm
		} = metadataCache
			.getFileCache(pvf);
		const VERSION = fm?.VERSION || "NA";
		
		try {

			// logic scrape
			const data = transformData(argMap);
			console.log({data})

			renderList(
				`==${VERSION}==`,
				data.map((datum) => `[[${datum}]]`)
			);
		} catch(err) {
			console.log({err})
		}
	}
	function transformData(paramMap) {
		const {
			search_term,
			regex_flag
		} = paramMap
		
		const uls = metadataCache
			.unresolvedLinks;
		
		const isInvalid = ["", null, undefined].includes(search_term);
		
		if (isInvalid) return [];
		
		try {
			const regex = new RegExp(search_term, regex_flag);
			return queryShadowNotes(
				uls,
				regex
			)
		} catch(err) {
			return new Error(JSON.stringify(err))
		}

	}
	function queryShadowNotes(
		uls = [],
		regex
	) {
		const shadowNotes = [];
		const uniq = {}

		for (const file_name in uls) {
			for (const shadow_note in uls[file_name]) {
				
				if ( regex.exec(shadow_note) && !uniq?.[shadow_note] ) {
					uniq[shadow_note] = true;
					shadowNotes.push(shadow_note);
				}
			}
		}
		return shadowNotes;
	}
	function renderList(
		header,
		datums
	) {
		const md = dv
			.markdownList(
				[header,...datums]
			);
		const _md = md
			.split('\n').map((m,i) => {
				if (i > 0) {
					return "  " + m;
				}
				return m
			})
		dv.paragraph(_md.join('\n'))
	}
}

// # Extraction Code vX.X.X. 
// impossible to version when inside another codelet.
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

	const embeds = getEmbedsFromVf(vf)
	const {name} = vault.adapter.path
		.parse(current_filepath);
	const embed = extractTargetEmbed(
		name, 
		embeds
	);
	const displayText = embed?.displayText;
	
	console.log({displayText})
	if (!displayText) {
		return manuParams()
	};
	
	const argMap = parseStringToMap(
		displayText
	);
	return {
		...manuParams(),
		...argMap
	};
	// /end params extraction 

	// helpers
	function getEmbedsFromVf(vf) {
		return metadataCache
			.getFileCache(vf)?.embeds || [];
	}
	
	function extractTargetEmbed(
		embed_name, 
		embeds
	) {
		const embed = embeds?.find(
		(embed) => {
			const parsed = obs.parseLinktext(
				embed.link
			);
			return parsed.path === embed_name;
		});
		return embed;
	}
	
	
	function parseStringToMap(str) {
		{ var parsed = {};
			try {
				parsed = adapter
					?.url
					?.parse(str, true);
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

---
# ---Transient Local Resources

## LV--Usage Guide

* API is ⤵
	* `[[` `= this.file.name` `|?search_term=`**note**`&regex_flag=gm`]]` (example search term: `.\*consum.\*`)

# ---Transient Sandbox

* [ ] Is there a way to export the function extractParams as dataview module using viewState? #_todo/to-muse/on-obsidianmd-app/regarding-a-codelet/regarding-global-code-sharing

# ---Transient Commit log

[[transient-commit-log-endpoint,bt.-Noteshippo-heading-api,]]

- v1.0.6 *2025-06-07*
	- Make displayed text into unresolved [[wikilink,]]s
* v1.0.1 
	* is somewhere floating hard coded out there in the [[notesphere,bt.-Noteshippo-lingo,]]
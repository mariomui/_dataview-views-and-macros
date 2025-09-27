---
CREATION_DATE: "2023-12-23"
DOC_VERSION: v0.0.7
MUID: MUID-1934
PROJECT_PARENT:
  - "[[π-create-code-to-scrape-links,nb.-targetting-embedded-queries]]"
TEMPLATE_VERSION: v1.0.4_blank-template
aliases: 
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


### 10÷About

* [[~viewfn-for-sluicing-out-embedded-query-into-a-job-queue,nb.-MUID-1934A]] sluices All queries, even `[[z--]] and [[z--b]]`
* For queries that don't care if you start with `[[a]` will not catch `[ab]`  
* ? Do updating the titles break everything? 
	* [ ] Check the dependencies before changing the title to include version and also document it. ➕ 2025-08-06 #_todo/to-document 

## 20-Inlink

> [!info]- Get Macros that Consume v0.0.1 `=this.MUID`
`= join( map( filter( this.file.inlinks, (link) => icontains(meta(link).path, "macro") ) , (link) => "• " + link ), "<br>")`

# =

**base_filepath-v0.0.9**: `= choice( contains(this.file.folder, this.file.name), link(this.file.path), join(["*",this.file.path,"*"], ""))` doc-`= this.DOC_VERSION` / ids: `= this.MUID`,PP:`= this.PROJECT_PARENT`,alias: *`= this.aliases`*,nb: *`=this.NOTA_BENE`* , authors: *`= this.authors`* / lcsh: `= link(this.heading)`

> [!info]  This codelet searches for an existing **embedded query**and makes a **jobs line** to notify you of any links you may not have written about.

~~~dataviewjs

const { default: obs } =
	this.app.plugins.plugins["templater-obsidian"].templater
		.current_functions_object.obsidian;

const {metadataCache,vault,workspace,fileManager} = this.app
const {adapter} = vault;

// HARD CODED (fix this in the future)
let MARKER = "---Transient Local Query";

const getCFP = () => this.currentFilePath;

// # BOOTSTRAP
workspace.onLayoutReady(main.bind(this))

function main(cmd) {
	try {
		(genMain)(this,renderUI.bind(this))
	} catch(err) {
		console.log({err})
	}

	// # WORKHORSE
	async function genMain(ctx,renderUI) {
		// --
		const providing_path = getCFP()
		const argMap = extractParams(
			providing_path,
			workspace.getActiveFile()
		)
		if (argMap.search_term.contains("%")) {
			throw new Error("contains percentage in search term")
		}
		console.log({providing_path,argMap})

		MARKER = argMap.search_term || MARKER;

		// --

		const avf = workspace.getActiveFile()
		const mdc = metadataCache.getFileCache(avf)
		const avfv = workspace.getActiveFileView()

		const queryFileFigsPayload = await genProcessQueriedFileFigsByActiveFileView(avfv, MARKER)

		
		const {
			unaliasedQueryLinks,
			unaliasedEmbeddedLinks
		} = await genProcessDiffingArtifactFig(mdc,queryFileFigsPayload)
		
		const unusedLinks = processDiffJobLine({unaliasedQueryLinks, unaliasedEmbeddedLinks}).map((basename) => {
			const vf = metadataCache.getFirstLinkpathDest(basename,"")

			if (!vf) {
				// console.log({vf,basename})
				return false;
			}
			const link = getMarkdownLink(vf,"")
			// console.log({link}, 'inside of processDiffJobLine mapper')
			if (!link) return false;
			
			return link;
		}).filter(Boolean)

		
		//ui
		console.log({unusedLinks})
		renderRefreshAndCopyButton.call(ctx, main,"copy")
		if (unusedLinks.length >= 1) {
			
			return renderUI.call(ctx, unusedLinks, cmd)
		}  

		const { $el } = await genCreateDiv(
			ctx, { text: "" , cls: "deleteme"}
		);
		// "[authors: /emma.*darwin/" <-- this must be turned markdown safe for MarkdownRenderer to render properly.
		const callout_text = getTextForRender( `*searchterm*: ${argMap?.search_term || "Empty"}`);

		const rendered_text = await obs.MarkdownRenderer.render(
			ctx.app,
			callout_text,
			$el,
			"",
			ctx.container.component
		);
		ctx.container.append($el)

	}
}

async function genProcessDiffingArtifactFig(mdc, queryFileFigsPayload) {
	 
	const embeddedLinks = processAllEmbeddedLinks(mdc);
	const config = queryFileFigsPayload;
	if (!config?.data) return manuProcessDiffJobLineFig();
	
	const unaliasedEmbeddedLinks = await genProcessMarkdownLinksByWikiLink(embeddedLinks)
	const unaliasedQueryLinks = await genProcessMarkdownLinksByVf(config.data);

	return  {
		unaliasedEmbeddedLinks,
		unaliasedQueryLinks
	}
	// compare the links in the note will have less text than the ones in the query.
}

function manuProcessDiffJobLineFig() {
	return {
		unaliasedQueryLinks: [],
		unaliasedEmbeddedLinks: []
	}
}
function processDiffJobLine(
	fig = manuProcessDiffJobLineFig()
) {
	const config = Object.assign({}, manuProcessDiffJobLineFig(), fig);
	const { unaliasedQueryLinks, unaliasedEmbeddedLinks} = config
	console.log({unaliasedQueryLinks, unaliasedEmbeddedLinks}, "inside processdiffjobline")
	const unusedLinks = []
	for (const link of unaliasedQueryLinks) {
		const isUsed = unaliasedEmbeddedLinks.some(
			(embedLink) => {
				
			  // (links in query) (links being used) 
			  // i wna tthe diff of (x ()() x) : I want x;
				return link === embedLink
			}
		)
		if (!isUsed) {
			unusedLinks.push(link)
		}
	}
	return unusedLinks;
}

async function genScrapeTextFromWikiLink(wikilink) {
	return new Promise((rs,rj) => { 
		wikilink.replace(/\[\[(.*?)\]\]/g, (match, data) => {
			if (data) {
				rs({data, err: null})
			} else {
				rj({data: null, err: `ERR: data is ${data||"N/A"}`})
			}
			return match
		}) 
	})
}
async function genProcessMarkdownLinksByWikiLink(wikilinks) {
	if (!Array.isArray(wikilinks)) return [];
	const unaliasedMarkdownLinks = []
	for (const wikilink of wikilinks) {
		const unaliasedMdLink = await genUnaliasedMarkdownLink(wikilink)
		if (unaliasedMdLink) {
			unaliasedMarkdownLinks.push(unaliasedMdLink)
		}
	}
	return unaliasedMarkdownLinks
}

async function genProcessMarkdownLinksByVf(vfs = []) {
	if (!Array.isArray(vfs)) return [];
	const unaliasedMarkdownLinks = []
	for (const datum of vfs) {
		const markdownLink = getMarkdownLink(datum, datum.basename)
		const unaliasedMdLink = await genUnaliasedMarkdownLink(markdownLink)
		unaliasedMarkdownLinks.push(unaliasedMdLink)
	}
	return unaliasedMarkdownLinks
}

// utils
function getMarkdownLink(avf,heading) {

	const _embed_text = !heading ? "" : `#${heading}`
	const markdownLink = fileManager
		.generateMarkdownLink(
			avf,
			avf.path,
			avf.basename + _embed_text,
			heading
		)
	return markdownLink;
}

async function genUnaliasedMarkdownLink(markdownLink) {
		const fnName = genUnaliasedMarkdownLink?.name
		const payload = await genScrapeTextFromWikiLink(
			markdownLink
		)
		if (payload?.err) {
			const errPkg = {
				err: payload.err, 
				errDesc: fnName + " fucked up"
			}
			
			// TODO (dependency to dataview, fix in future to use outside ui only package)
			dv.paragraph("fucking error")
			throw new Error(JSON.stringify(errPkg))
			return "";
		}
		
		const parsedLink = obs.parseLinktext(payload.data);
		
		if (parsedLink?.path === "" && parsedLink?.subpath?.startsWith("#")) {
			return "";
		}
		if (parsedLink?.path) {
			// if parsed link path doesn't exist suchas in [[#link]]s then there is an error. It should really return itself.
			return parsedLink.path
		}
		throw new Error(`Parsed path does not exist in ${markdownLink}`)
}

// # processors
async function genProcessQueryFileFigToWikiLinks(fileFig) {
	
	const markdownLink = getMarkdownLink(avf,heading)
	try {
		const unaliasedMarkdownLink = await genUnaliasedMarkdownLink(markdownLink);
		return unaliasedMarkdownLink
	} catch(err) {
		const err_string = JSON.stringify(err);
		throw new Error(`genUnaliasedMarkdownLink error ${err_string}`)
	}
}

function processAllEmbeddedLinks(mdc) {


	const mdcLinks = mdc?.links?.map(({original}) => original) || [];
	const mdcEmbeds = mdc?.embeds?.map(({original}) => original) || [];
	const mdcs = [...mdcLinks, ...mdcEmbeds]
	return mdcs;
}


async function genProcessQueriedFileFigsByActiveFileView(avf, search_term) {
	// I couldnt find a callback to see when the queue finished populating but when i do i knwo this function will need it.
	return new Promise((rs,rj) => {
		{var result = { data: [], err: null};
		
			try {
				const data = avf?.
					_children?.at(0)?._children
						?.find((child) => child.query && child.query.indexOf(search_term) > -1)
						?.dom?.getFiles()
				result.data = data || []
				return rs(result)
			} catch (err) {
				result.err = err;
				return rj(result)
			}
		}
	})
}
// UI
function getTextForRender(text) {
	const warning =  `\> [!warning] Search_term is empty`;
	const notification =  `\> [!note] ${text} has been used up`;
	
	if (!text) return warning;
	return text
}

function renderRefreshAndCopyButton(main,cmd) {
	const copy = cmd === "copy" ? "copy" : null
	const handleClick = () => {
		const $button = this.container.querySelector('button')
		$button.remove();
		this.container?.lastChild?.remove();
		main.call(this, copy);

	}
	new obs
		.ButtonComponent(this.container)
		.setButtonText('Refresh and Copy')
		.onClick(handleClick.bind(this))
}
function renderUI(embed_texts,cmd) {
		const uiMdText = embed_texts.reduce((chain,text) => {
			return chain + "* " + text + "\n"
		},"")
		const $el =dv.paragraph(
			uiMdText, {
				cls: "deleteme"
			}
		);
		if (cmd === "copy")  {
			const text = $el.innerText;
			navigator.clipboard.writeText(uiMdText);
		}
}

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

// # Extraction Code vX.X.X.
// impossible to version when inside another codelet.
function manuParams() {
	return {
		search_term: "",
		regex_flag: "g"
	}
}
/**
@output: argMap {search_term: ..., t: ...} etc
**/
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

	if (!displayText) {
		return manuParams()
	};

	const argMap = parseStringToMap(
		displayText
	);
	// console.log({embed,displayText,argMap})
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
	/**
	@returns {err} | {} | queryParamsObject
	**/
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

	// this is the actual function that turns ?x=t to {x:t}
	function parseStringToMap(query_str) {
		{ var parsed = {};
			try {
				console.log({query_str})
				const fixed_query = query_str.replace(/\+/g, '%2B'); // + signs are no bueno.
				// const fixed_query = query_str;

				parsed = adapter
					?.url
					?.parse(fixed_query, true);
				// url parse is deprecated but its still useful and fast.
			} catch (err) {

				return {
					err
				}
			}

			return parsed?.query || {};
		}
		return {};
	}
}
~~~

# ---Transient Commit Log

[[transient-commit-log-endpoint,bt.-Noteshippo-heading-api,]]

- v0.0.7 *2025-08-23*
	- update the [[base_filepath,bt.-Noteshippo-lingo,]] display to v0.0.9
- v0.0.6 *2025-08-06*
	- Include the query string in the display of the partial
- v0.0.5 
	- Update the diff comparator. Startswith doesn't seem to work.
		- the link is the query but the embedlink is the one on the page. So only does it match should it be used. when a link isn't on the page such as [[Stylometry--Jargon]] , it registers it as true that it startswith ... oh, yeah, why did i not want the additional childrens? 
* v0.0.4 *2025-05-17*
	* Update so that t=olk or ui=olk triggers the css to turn off pointer events.
		* Eventually, it would be good to just inline css the style whenever the t or ui argument is olk and nlk.
* v0.0.3 *2025-05-16*
	* Update the parsing function to allow non safe url characters like + which url parse treats like a space.
* v0.0.2 *2025-03-07*
	* Working on bug where it seems to be terrible on one note but not the other. it works in [[~viewfn-for-sluicing-out-embedded-query-into-a-job-queue,nb.-MUID-1934A]]
* v0.0.
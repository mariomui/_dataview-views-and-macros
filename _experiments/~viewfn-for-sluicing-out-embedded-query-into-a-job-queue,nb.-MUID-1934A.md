---
aliases: 
CREATION_DATE: 2023-12-23
DOC_VERSION: v0.0.2
MUID: MUID-1934A
tags:
  - _misc/_wip
TEMPLATE_VERSION: 0.0.0
UMID: "[[π-create-code-to-scrape-links,nb.-targetting-embedded-queries]]"
---

# -

## 10-About

* 🐛 lcsh subheadings are not diffed or scraped from the unaliased query list.
## 20-Inlink

> [!abstract]- %%  %% Automated List of Reference Inlinks (v0.0.5)
> * ℹ Commit/design logs are located in this [[,aka-MUID-150|experiment note]].
> > `= join( map( sort( map( filter(this.file.inlinks, (link) => meta(link).path != this.file.path), (x) => [ split(meta(x).path, "/")[length(split(meta(x).path, "/")) - 1], x ] ) ), (b) => "• " + choice( length(b[0]) > 28, link( b[1], truncate( regexreplace(b[0], "(-of|of|the|-the|-for|-that|https-|ee)", ""), length( regexreplace(b[0], "(-of|of|the|-the|-for|-that|https-|ee)", "") ) * 0.75 ) ), link(b[1], regexreplace(b[0], "\.md$", "")) ) ), "<br>" )`

# =

**filename:** `=this.file.path`

> [!info]  This codelet searches for an existing **embedded query**and makes a **jobs line** to notify you of any links you may not have written about.

* :~~See [[custom-transclusion-parameters,bt.-Noteshippo-terminology,]] for search term use.~~

~~~dataviewjs
const ACTIVE_DEBUG = false; // todo , extract this prop from the consuming file.
const { default: obs } =
	this.app.plugins?.plugins?.["templater-obsidian"]?.templater
		?.current_functions_object?.obsidian;

const {metadataCache,vault,workspace,fileManager} = this.app
const {adapter} = vault;

let SEARCH_TERM = "---Transient Local Query";

const getCFP = () => this.currentFilePath;

const debug = (...args) => {
	if (ACTIVE_DEBUG) {
		console.log.call(console, ...args)
	}
}
workspace.onLayoutReady(main.bind(this))
function main(cmd) {
 if (!obs) return;

 (genMain)(this,renderUI.bind(this))

	async function genMain(ctx,renderUI) {
		// --
		const providing_path = getCFP()
		const argMap = extractParams(
			providing_path,
			workspace.getActiveFile()
		)
		debug({providing_path,argMap})

		SEARCH_TERM = argMap.search_term || SEARCH_TERM;

		// --

		const avf = workspace.getActiveFile()
		const mdc = metadataCache.getFileCache(avf)
		const avfv = workspace.getActiveFileView()

		// scrape links from the obsidian query
		const queryFileFigsPayload = await genProcessQueriedFileFigsByActiveFileView(avfv, SEARCH_TERM)
		debug({queryFileFigsPayload})

		
		const {
			unaliasedQueryLinks,
			unaliasedEmbeddedLinks
		} = await genProcessDiffingArtifactFig(mdc,queryFileFigsPayload)
		debug({unaliasedQueryLinks, unaliasedEmbeddedLinks})

		
		const unusedLinks = processDiffJobLine({unaliasedQueryLinks, unaliasedEmbeddedLinks}).map((basename) => {
			const vf = metadataCache.getFirstLinkpathDest(basename,"")

			if (!vf) {
				debug({vf,basename})
				return false;
			}
			const link = getMarkdownLink(vf,"")
			debug({link}, 'inside of processDiffJobLine mapper')
			if (!link) return false;
			
			return link;
		}).filter(Boolean)

		
		//ui
		renderRefreshAndCopyButton.call(ctx, main,"copy")
		debug({unusedLinks})
		if (unusedLinks.length >= 1) {
			
			return renderUI.call(ctx, unusedLinks, cmd)
		}  

		const { $el } = await genCreateDiv(
			ctx, { text: "" , cls: "deleteme"}
		);

		const callout_text = getTextForRender(argMap?.search_term || "Empty"); 
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

//take the links from the query and process it.
/**
@param queryFileFigsPayload {data: VFs, err: string | null}
**/
async function genProcessDiffingArtifactFig(mdc, queryFileFigsPayload) {
	 
	const embeddedLinks = processAllEmbeddedLinks(mdc);
	const config = queryFileFigsPayload;
	if (!config?.data) return manuProcessDiffJobLineFig();
	debug({embeddedLinks, config})
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
	debug({unaliasedQueryLinks, unaliasedEmbeddedLinks})
	const unusedLinks = []
	for (const link of unaliasedQueryLinks) {
		const isUsed = unaliasedEmbeddedLinks.some(
			(embedLink) => {
				//debug({link, embedLink})
				const isAccepted = (link.length - embedLink.length) * 1 < 2;
				return link.startsWith(embedLink) && isAccepted
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
		} else {
		 debug({unaliasedMdLink}, wikilinks)
		}
	}
	return unaliasedMarkdownLinks
}

async function genProcessMarkdownLinksByVf(vfs = []) {
	if (!Array.isArray(vfs)) return [];
	const unaliasedMarkdownLinks = []
	debug({vfs})
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
		const payload = await genScrapeTextFromWikiLink(
			markdownLink
		)
		if (payload?.err) {
			debug({err})
			dv.paragraph("fucking error")
			return "";
		}
		const parsedLink = obs.parseLinktext(payload.data);
 
		// if (parsedLink?.path) {
		return parsedLink.path
		// }
	// debug({parsedLink, payload})
		// throw new Error("Parsed path does not exist ")
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
function getTextForRender(search_term) {
			const warning =  `\> [!warning] Search_term is empty`
			const notification =  `\> [!note] ${search_term} has been used up`;
			if (!search_term) return warning;
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

# ---Transient

# ---Transient Commit Log

* v0.0.2
	* Fix bug where the parsing empty text gives early error.

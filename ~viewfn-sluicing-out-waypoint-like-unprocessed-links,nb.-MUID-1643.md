---
TEMPLATE_VERSION: v1.0.7_note-refactor-template
MUID: MUID-1643
CREATION_DATE: 2023-10-29
tags:
  - _misc/_wip
DOC_VERSION: v0.0.3
UMID:
---

# -

## About

Much of this code was sourced from [[~viewfn-for-sluicing-header-links-for-citations,nb.-MUID-1560]].

* It uses,
	* [[custom-transclusion-parameters,bt.-Noteshippo-terminology,]]

The purpose of this view is to dynamically source all the links contained within a specified H1 (usually named ---Transient Waypoints). The view will automatically remove any links that have been mentioned AGAIN, creating an almost queue like behavior where the view keeps tracks of how many links you've written about. The goal is to see zero waypoints.

# =

* filename: *`= this.file.name`*
* usage: `[[~viewfn-for-creating-absolute-links-for-waypoints?search_term=---Transient Waypoints]]`

```dataviewjs
const { default: obs } =
	this.app.plugins.plugins["templater-obsidian"].templater
		.current_functions_object.obsidian;

const {metadataCache,vault,workspace,fileManager} = this.app
const {adapter} = vault;

let MARKER = "---Transient Waypoints";

const getCFP = () => this.currentFilePath;

workspace.onLayoutReady(main.bind(this))
function main(cmd) {
	(genMain)(this,renderUI.bind(this))
	async function genMain(ctx,renderUI) {
		// --
		const providing_path = getCFP()
		const argMap = extractParams(
			providing_path,
			workspace.getActiveFile()
		)
		MARKER = argMap.search_term || MARKER;

		// console.log({argMap})

		const avf = workspace.getActiveFile()
		const mdc = metadataCache.getFileCache(avf)
		
		const {headings} = mdc;
		const mdclinks = mdc.links;
		
		
		const filterHeadingByLevel = ({level}) => level === 1;
		
		const markedHeadings = headings
			.filter(filterHeadingByLevel)
			
		const markedHeadingIdx = markedHeadings
			.findIndex((heading) => {
				return heading.heading === MARKER
			})
			
		const targetHeadings = markedHeadings.slice(
			markedHeadingIdx, 
			markedHeadingIdx + 2
		);
	
		const betweenSpec = getBetweenSpec(targetHeadings)
		// console.log({targetHeadings, markedHeadings, headings, mdclinks, betweenSpec})
		// error handling
		if (!betweenSpec) {
			const $el = await genManuUiFailure(ctx, { 
				text: "" , cls: "deleteme"
				}
			);
			return ctx.container.append($el)
		};
		 
		// get heading positions for H1
		// if heading target found
		// get +1 offset, get that position, find betweens.

		// if not, then anything greater is fair game.
		const filterListItemByBetweenSpec = (listItem) => {
			const listStart = listItem.position.start;
			const isBetween = checkIsBetween(listStart.line, betweenSpec)
			return isBetween
		}

		// these only give positions
		const targetListItems = mdc.listItems.filter(filterListItemByBetweenSpec)
		// filter out list items that are between my spec 

		// use this positional list to get the lists that match in the links.
		// these give links and textdata.
		const filterLinksByMatchedStart = (link) => {
			const {position: {start: linkStart} } = link
			const linkStartLineNo = linkStart.line;
			const isBetween = checkIsBetween(linkStartLineNo, betweenSpec)
			return isBetween;
		}
		const matchedLinks = mdclinks.filter(filterLinksByMatchedStart);

		// get matched wikilinks
		const wikilinks = matchedLinks.map((matchedLink) => {
			return matchedLink.original
		});
		console.log({wikilinks,matchedLinks})
		//use wikilinks to get a list of links that don't isn't between.
		const analyzedLinks = wikilinks.filter((wikilink) => {
			return mdclinks.find((mdclink) => {
				return mdclink.original === wikilink && !checkIsBetween(mdclink.position.start.line, betweenSpec)
			})
		})
		console.log({analyzedLinks})

		// only output wikilinks that are not in analyzed links

		const nonAnalyzedWikilinks = wikilinks.filter((wikilink) => {
			return !analyzedLinks.includes(wikilink)
		})
		const markdownListedWikilinks = markdownListify(
			nonAnalyzedWikilinks
		);

		renderRefreshAndCopyButton
			.call(ctx, main,"copy")

		if (markdownListedWikilinks.length >= 1) {
			renderUI.call(ctx, markdownListedWikilinks, cmd)
		} else {
			const $el = await genManuUiFailure(ctx, { text: "" , cls: "deleteme"})
			ctx.container.append($el)
		}
	}
}



// # utils
function checkIsBetween(number, spec, mode="[a,b)") {
	if (!spec?.from?.line) {
		// console.log({spec})
		return false;
	}
	if (mode === "[a,b)") {
		return number >= spec.from.line && number < spec.to.line
	}
	return false;
}

function getBetweenSpec(headings) {
	if (headings.length === 0) return null;
	if (headings.length === 1) {
		const heading = headings.first()
		return {
			from: heading.position.start,
			to: {
				line: Infinity
			}
		}
	}
	const positions = headings.reduce((chain, {position}) => {
		return chain.concat(position.start)
	}, []);
	return {
		from: positions.at(0),
		to: positions.at(1)
	};
}



// # ui 
// ## ui-pre-logic

function markdownListify(texts) {
	return texts.map((text) => {
		return "`* " + text + "`" + "\n"
	})
}
function generateWikiLink(avf, heading) {
		const _embed_text = `#${heading}`
		const mdlink = fileManager
			.generateMarkdownLink(
				avf,
				avf.path,
				avf.basename + _embed_text,
				heading
			)
		const parsedLink = obs.parseLinktext(mdlink);
		const wikilink = parsedLink.path + parsedLink.subpath.split('|').first()
		return wikilink
}
// ## UI Components

async function genManuUiFailure(ctx, domInfo) {
		const { $el } = await genCreateDiv(
			ctx, domInfo
		);
		const rendered_text = await obs
			.MarkdownRenderer
			.render(
				ctx.app,
				`\> [!warning] Zero Waypoint links detected`,
				$el,
				"",
				ctx.container.component
		);
		return $el
}
// ## ui-renderre

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
			const uiMdText = embed_texts.reduce((a,b) => a + b)
		const $el =dv.paragraph(
			uiMdText, {
				cls: "deleteme"
			}
		);
		if (cmd === "copy")  {
			const text = $el.innerText;
			navigator.clipboard.writeText(text);
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

// # Extraction Code See MUID-1634 for versioning 
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
```

---

# ---Transient Commit Log

* v0.0.3 *2025-02-19*
	* Normalize title to use nb with the MUID affix
* v0.0.2 Update ui failure text
* v0.0.1 bug-fix/on-escape-handler-not-rendering-ui-failure-notification
	* The early escape when betweenSpec could not be found returned null instead of triggering the Error UI handler.

# ---Transient

# ---Transient Local Resources

# LR--demo--queueing waypoint links

![[demo-of-waypont-queue-MUID-742.gif]]

## Ω

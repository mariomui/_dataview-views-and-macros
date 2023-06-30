# =

```dataviewjs
renderContent(getDomText.call(this))

// UI business Logic
function getDomText() {
    const nodes = this.app
        .statusBar
        ?.containerEl
        ?.childNodes
    for (const node of nodes) {
        const isTargetNode = node.classList
            .contains(
                'plugin-obsidian-reading-time'
            )
        if (isTargetNode) {
            return node?.textContent
        }
    }
}
// UI
function renderContent(
    minutesRead,
    option = {thresh: 1}
) {
    const isBelowThresh = Number(
            minutesRead.split('').first()
        ) <= option.thresh;
    dv.paragraph(
        isBelowThresh ? '0 min' : minutesRead
    );
}

```
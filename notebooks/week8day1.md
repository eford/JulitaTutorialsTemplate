~~~
<!-- PlutoStaticHTML.Begin -->
<!--
    # This information is used for caching.
    [PlutoStaticHTML.State]
    input_sha = "b263f8bec78967c4d2af17fe8ce7cb2ef36149aef4d6da0dd27c7efb957656a3"
    julia_version = "1.8.2"
-->

<div class="markdown"><h2>Queries &amp; Data Wrangling</h2>
<p><strong>Astro 497, Week 8, Monday</strong></p>
</div>

<pre class='language-julia'><code class='language-julia'>TableOfContents()</code></pre>
<script>
	
const indent = true
const aside = true
const title_text = "Table of Contents"
const include_definitions = false


const tocNode = html`<nav class="plutoui-toc">
	<header>
	 <span class="toc-toggle open-toc"></span>
	 <span class="toc-toggle closed-toc"></span>
	 ${title_text}
	</header>
	<section></section>
</nav>`

tocNode.classList.toggle("aside", aside)
tocNode.classList.toggle("indent", indent)


const getParentCell = el => el.closest("pluto-cell")

const getHeaders = () => {
	const depth = Math.max(1, Math.min(6, 3)) // should be in range 1:6
	const range = Array.from({length: depth}, (x, i) => i+1) // [1, ..., depth]
	
	const selector = [
		...(include_definitions ? [
			`pluto-notebook pluto-cell .pluto-docs-binding`, 
			`pluto-notebook pluto-cell assignee:not(:empty)`, 
		] : []),
		...range.map(i => `pluto-notebook pluto-cell h${i}`)
	].join(",")
	return Array.from(document.querySelectorAll(selector)).filter(el => 
		// exclude headers inside of a pluto-docs-binding block
		!(el.nodeName.startsWith("H") && el.closest(".pluto-docs-binding"))
	)
}


const document_click_handler = (event) => {
	const path = (event.path || event.composedPath())
	const toc = path.find(elem => elem?.classList?.contains?.("toc-toggle"))
	if (toc) {
		event.stopImmediatePropagation()
		toc.closest(".plutoui-toc").classList.toggle("hide")
	}
}

document.addEventListener("click", document_click_handler)


const header_to_index_entry_map = new Map()
const currently_highlighted_set = new Set()

const last_toc_element_click_time = { current: 0 }

const intersection_callback = (ixs) => {
	let on_top = ixs.filter(ix => ix.intersectionRatio > 0 && ix.intersectionRect.y < ix.rootBounds.height / 2)
	if(on_top.length > 0){
		currently_highlighted_set.forEach(a => a.classList.remove("in-view"))
		currently_highlighted_set.clear()
		on_top.slice(0,1).forEach(i => {
			let div = header_to_index_entry_map.get(i.target)
			div.classList.add("in-view")
			currently_highlighted_set.add(div)
			
			/// scroll into view
			/*
			const toc_height = tocNode.offsetHeight
			const div_pos = div.offsetTop
			const div_height = div.offsetHeight
			const current_scroll = tocNode.scrollTop
			const header_height = tocNode.querySelector("header").offsetHeight
			
			const scroll_to_top = div_pos - header_height
			const scroll_to_bottom = div_pos + div_height - toc_height
			
			// if we set a scrollTop, then the browser will stop any currently ongoing smoothscroll animation. So let's only do this if you are not currently in a smoothscroll.
			if(Date.now() - last_toc_element_click_time.current >= 2000)
				if(current_scroll < scroll_to_bottom){
					tocNode.scrollTop = scroll_to_bottom
				} else if(current_scroll > scroll_to_top){
					tocNode.scrollTop = scroll_to_top
				}
			*/
		})
	}
}
let intersection_observer_1 = new IntersectionObserver(intersection_callback, {
	root: null, // i.e. the viewport
  	threshold: 1,
	rootMargin: "-15px", // slightly smaller than the viewport
	// delay: 100,
})
let intersection_observer_2 = new IntersectionObserver(intersection_callback, {
	root: null, // i.e. the viewport
  	threshold: 1,
	rootMargin: "15px", // slightly larger than the viewport
	// delay: 100,
})

const render = (elements) => {
	header_to_index_entry_map.clear()
	currently_highlighted_set.clear()
	intersection_observer_1.disconnect()
	intersection_observer_2.disconnect()

		let last_level = `H1`
	return html`${elements.map(h => {
	const parent_cell = getParentCell(h)

		let [className, title_el] = h.matches(`.pluto-docs-binding`) ? ["pluto-docs-binding-el", h.firstElementChild] : [h.nodeName, h]

	const a = html`<a 
		class="${className}" 
		title="${title_el.innerText}"
		href="#${parent_cell.id}"
	>${title_el.innerHTML}</a>`
	/* a.onmouseover=()=>{
		parent_cell.firstElementChild.classList.add(
			'highlight-pluto-cell-shoulder'
		)
	}
	a.onmouseout=() => {
		parent_cell.firstElementChild.classList.remove(
			'highlight-pluto-cell-shoulder'
		)
	} */
		
		
	a.onclick=(e) => {
		e.preventDefault();
		last_toc_element_click_time.current = Date.now()
		h.scrollIntoView({
			behavior: 'smooth', 
			block: 'start'
		})
	}

	const row =  html`<div class="toc-row ${className} after-${last_level}">${a}</div>`
		intersection_observer_1.observe(title_el)
		intersection_observer_2.observe(title_el)
		header_to_index_entry_map.set(title_el, row)

	if(className.startsWith("H"))
		last_level = className
		
	return row
})}`
}

const invalidated = { current: false }

const updateCallback = () => {
	if (!invalidated.current) {
		tocNode.querySelector("section").replaceWith(
			html`<section>${render(getHeaders())}</section>`
		)
	}
}
updateCallback()
setTimeout(updateCallback, 100)
setTimeout(updateCallback, 1000)
setTimeout(updateCallback, 5000)

const notebook = document.querySelector("pluto-notebook")


// We have a mutationobserver for each cell:
const mut_observers = {
	current: [],
}

const createCellObservers = () => {
	mut_observers.current.forEach((o) => o.disconnect())
	mut_observers.current = Array.from(notebook.querySelectorAll("pluto-cell")).map(el => {
		const o = new MutationObserver(updateCallback)
		o.observe(el, {attributeFilter: ["class"]})
		return o
	})
}
createCellObservers()

// And one for the notebook's child list, which updates our cell observers:
const notebookObserver = new MutationObserver(() => {
	updateCallback()
	createCellObservers()
})
notebookObserver.observe(notebook, {childList: true})

// And finally, an observer for the document.body classList, to make sure that the toc also works when it is loaded during notebook initialization
const bodyClassObserver = new MutationObserver(updateCallback)
bodyClassObserver.observe(document.body, {attributeFilter: ["class"]})

// Hide/show the ToC when the screen gets small
let m = matchMedia("(max-width: 1000px)")
let match_listener = () => 
	tocNode.classList.toggle("hide", m.matches)
match_listener()
m.addListener(match_listener)

invalidation.then(() => {
	invalidated.current = true
	intersection_observer_1.disconnect()
	intersection_observer_2.disconnect()
	notebookObserver.disconnect()
	bodyClassObserver.disconnect()
	mut_observers.current.forEach((o) => o.disconnect())
	document.removeEventListener("click", document_click_handler)
	m.removeListener(match_listener)
})

return tocNode
</script>
<style>
@media not print {

.plutoui-toc {
	font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen-Sans, Cantarell, Helvetica, Arial, "Apple Color Emoji",
		"Segoe UI Emoji", "Segoe UI Symbol", system-ui, sans-serif;
	--main-bg-color: #fafafa;
	--pluto-output-color: hsl(0, 0%, 36%);
	--pluto-output-h-color: hsl(0, 0%, 21%);
	--sidebar-li-active-bg: rgb(235, 235, 235);
	--icon-filter: unset;
}

@media (prefers-color-scheme: dark) {
	.plutoui-toc {
		--main-bg-color: #303030;
		--pluto-output-color: hsl(0, 0%, 90%);
		--pluto-output-h-color: hsl(0, 0%, 97%);
		--sidebar-li-active-bg: rgb(82, 82, 82);
		--icon-filter: invert(1);
	}
}

.plutoui-toc.aside {
	color: var(--pluto-output-color);
	position: fixed;
	right: 1rem;
	top: 5rem;
	width: min(80vw, 300px);
	padding: 0.5rem;
	padding-top: 0em;
	/* border: 3px solid rgba(0, 0, 0, 0.15); */
	border-radius: 10px;
	/* box-shadow: 0 0 11px 0px #00000010; */
	max-height: calc(100vh - 5rem - 90px);
	overflow: auto;
	z-index: 40;
	background-color: var(--main-bg-color);
	transition: transform 300ms cubic-bezier(0.18, 0.89, 0.45, 1.12);
}

.plutoui-toc.aside.hide {
	transform: translateX(calc(100% - 28px));
}
.plutoui-toc.aside.hide section {
	display: none;
}
.plutoui-toc.aside.hide header {
	margin-bottom: 0em;
	padding-bottom: 0em;
	border-bottom: none;
}
}  /* End of Media print query */
.plutoui-toc.aside.hide .open-toc,
.plutoui-toc.aside:not(.hide) .closed-toc,
.plutoui-toc:not(.aside) .closed-toc {
	display: none;
}

@media (prefers-reduced-motion) {
  .plutoui-toc.aside {
	transition-duration: 0s;
  }
}

.toc-toggle {
	cursor: pointer;
    padding: 1em;
    margin: -1em;
    margin-right: -0.7em;
    line-height: 1em;
    display: flex;
}

.toc-toggle::before {
    content: "";
    display: inline-block;
    height: 1em;
    width: 1em;
    background-image: url("https://cdn.jsdelivr.net/gh/ionic-team/ionicons@5.5.1/src/svg/list-outline.svg");
	/* generated using https://dopiaza.org/tools/datauri/index.php */
    background-image: url("data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSI1MTIiIGhlaWdodD0iNTEyIiB2aWV3Qm94PSIwIDAgNTEyIDUxMiI+PHRpdGxlPmlvbmljb25zLXY1LW88L3RpdGxlPjxsaW5lIHgxPSIxNjAiIHkxPSIxNDQiIHgyPSI0NDgiIHkyPSIxNDQiIHN0eWxlPSJmaWxsOm5vbmU7c3Ryb2tlOiMwMDA7c3Ryb2tlLWxpbmVjYXA6cm91bmQ7c3Ryb2tlLWxpbmVqb2luOnJvdW5kO3N0cm9rZS13aWR0aDozMnB4Ii8+PGxpbmUgeDE9IjE2MCIgeTE9IjI1NiIgeDI9IjQ0OCIgeTI9IjI1NiIgc3R5bGU9ImZpbGw6bm9uZTtzdHJva2U6IzAwMDtzdHJva2UtbGluZWNhcDpyb3VuZDtzdHJva2UtbGluZWpvaW46cm91bmQ7c3Ryb2tlLXdpZHRoOjMycHgiLz48bGluZSB4MT0iMTYwIiB5MT0iMzY4IiB4Mj0iNDQ4IiB5Mj0iMzY4IiBzdHlsZT0iZmlsbDpub25lO3N0cm9rZTojMDAwO3N0cm9rZS1saW5lY2FwOnJvdW5kO3N0cm9rZS1saW5lam9pbjpyb3VuZDtzdHJva2Utd2lkdGg6MzJweCIvPjxjaXJjbGUgY3g9IjgwIiBjeT0iMTQ0IiByPSIxNiIgc3R5bGU9ImZpbGw6bm9uZTtzdHJva2U6IzAwMDtzdHJva2UtbGluZWNhcDpyb3VuZDtzdHJva2UtbGluZWpvaW46cm91bmQ7c3Ryb2tlLXdpZHRoOjMycHgiLz48Y2lyY2xlIGN4PSI4MCIgY3k9IjI1NiIgcj0iMTYiIHN0eWxlPSJmaWxsOm5vbmU7c3Ryb2tlOiMwMDA7c3Ryb2tlLWxpbmVjYXA6cm91bmQ7c3Ryb2tlLWxpbmVqb2luOnJvdW5kO3N0cm9rZS13aWR0aDozMnB4Ii8+PGNpcmNsZSBjeD0iODAiIGN5PSIzNjgiIHI9IjE2IiBzdHlsZT0iZmlsbDpub25lO3N0cm9rZTojMDAwO3N0cm9rZS1saW5lY2FwOnJvdW5kO3N0cm9rZS1saW5lam9pbjpyb3VuZDtzdHJva2Utd2lkdGg6MzJweCIvPjwvc3ZnPg==");
    background-size: 1em;
	filter: var(--icon-filter);
}

.aside .toc-toggle.open-toc:hover::before {
    background-image: url("https://cdn.jsdelivr.net/gh/ionic-team/ionicons@5.5.1/src/svg/arrow-forward-outline.svg");
	/* generated using https://dopiaza.org/tools/datauri/index.php */
    background-image: url("data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSI1MTIiIGhlaWdodD0iNTEyIiB2aWV3Qm94PSIwIDAgNTEyIDUxMiI+PHRpdGxlPmlvbmljb25zLXY1LWE8L3RpdGxlPjxwb2x5bGluZSBwb2ludHM9IjI2OCAxMTIgNDEyIDI1NiAyNjggNDAwIiBzdHlsZT0iZmlsbDpub25lO3N0cm9rZTojMDAwO3N0cm9rZS1saW5lY2FwOnJvdW5kO3N0cm9rZS1saW5lam9pbjpyb3VuZDtzdHJva2Utd2lkdGg6NDhweCIvPjxsaW5lIHgxPSIzOTIiIHkxPSIyNTYiIHgyPSIxMDAiIHkyPSIyNTYiIHN0eWxlPSJmaWxsOm5vbmU7c3Ryb2tlOiMwMDA7c3Ryb2tlLWxpbmVjYXA6cm91bmQ7c3Ryb2tlLWxpbmVqb2luOnJvdW5kO3N0cm9rZS13aWR0aDo0OHB4Ii8+PC9zdmc+");
}
.aside .toc-toggle.closed-toc:hover::before {
    background-image: url("https://cdn.jsdelivr.net/gh/ionic-team/ionicons@5.5.1/src/svg/arrow-back-outline.svg");
	/* generated using https://dopiaza.org/tools/datauri/index.php */
    background-image: url("data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSI1MTIiIGhlaWdodD0iNTEyIiB2aWV3Qm94PSIwIDAgNTEyIDUxMiI+PHRpdGxlPmlvbmljb25zLXY1LWE8L3RpdGxlPjxwb2x5bGluZSBwb2ludHM9IjI0NCA0MDAgMTAwIDI1NiAyNDQgMTEyIiBzdHlsZT0iZmlsbDpub25lO3N0cm9rZTojMDAwO3N0cm9rZS1saW5lY2FwOnJvdW5kO3N0cm9rZS1saW5lam9pbjpyb3VuZDtzdHJva2Utd2lkdGg6NDhweCIvPjxsaW5lIHgxPSIxMjAiIHkxPSIyNTYiIHgyPSI0MTIiIHkyPSIyNTYiIHN0eWxlPSJmaWxsOm5vbmU7c3Ryb2tlOiMwMDA7c3Ryb2tlLWxpbmVjYXA6cm91bmQ7c3Ryb2tlLWxpbmVqb2luOnJvdW5kO3N0cm9rZS13aWR0aDo0OHB4Ii8+PC9zdmc+");
}



.plutoui-toc header {
	display: flex;
	align-items: center;
	gap: .3em;
	font-size: 1.5em;
	/* margin-top: -0.1em; */
	margin-bottom: 0.4em;
	padding: 0.5rem;
	margin-left: 0;
	margin-right: 0;
	font-weight: bold;
	/* border-bottom: 2px solid rgba(0, 0, 0, 0.15); */
	position: sticky;
	top: 0px;
	background: var(--main-bg-color);
	z-index: 41;
}
.plutoui-toc.aside header {
	padding-left: 0;
	padding-right: 0;
}

.plutoui-toc section .toc-row {
	white-space: nowrap;
	overflow: hidden;
	text-overflow: ellipsis;
	padding: .1em;
	border-radius: .2em;
}

.plutoui-toc section .toc-row.H1 {
	margin-top: 1em;
}


.plutoui-toc.aside section .toc-row.in-view {
	background: var(--sidebar-li-active-bg);
}


	
.highlight-pluto-cell-shoulder {
	background: rgba(0, 0, 0, 0.05);
	background-clip: padding-box;
}

.plutoui-toc section a {
	text-decoration: none;
	font-weight: normal;
	color: var(--pluto-output-color);
}
.plutoui-toc section a:hover {
	color: var(--pluto-output-h-color);
}

.plutoui-toc.indent section a.H1 {
	font-weight: 700;
	line-height: 1em;
}

.plutoui-toc.indent section a.H6,
.plutoui-toc.indent section .after-H6 a  {
	padding-left: 50px;
}
.plutoui-toc.indent section a.H5,
.plutoui-toc.indent section .after-H5 a  {
	padding-left: 40px;
}
.plutoui-toc.indent section a.H4,
.plutoui-toc.indent section .after-H4 a  {
	padding-left: 30px;
}
.plutoui-toc.indent section a.H3,
.plutoui-toc.indent section .after-H3 a  {
	padding-left: 20px;
}
.plutoui-toc.indent section a.H2,
.plutoui-toc.indent section .after-H2 a {
	padding-left: 10px;
}
.plutoui-toc.indent section a.H1 {
	padding-left: 0px;
}

.plutoui-toc.indent section a.pluto-docs-binding-el,
.plutoui-toc.indent section a.ASSIGNEE
	{
	font-family: JuliaMono, monospace;
	font-size: .8em;
	/* background: black; */
	font-weight: 700;
    font-style: italic;
	color: var(--cm-var-color); /* this is stealing a variable from Pluto, but it's fine if that doesnt work */
}
.plutoui-toc.indent section a.pluto-docs-binding-el::before,
.plutoui-toc.indent section a.ASSIGNEE::before
	{
	content: "> ";
	opacity: .3;
}
</style>



<div class="markdown"><h1>Databases</h1>
</div>


<div class="markdown"><h3>Simplest form</h3>
<ul>
<li><p>Store data </p>
</li>
<li><p>Retrieve data</p>
</li>
<li><p>Commonly implemented as a set of <strong>tables</strong></p>
<ul>
<li><p><strong>Columns</strong> contain different fields &#40;e.g., ID, magnitude, RA, Dec&#41;</p>
</li>
<li><p><strong>Rows</strong> contain entries &#40;e.g., 51 Pegasi, Kepler-10, HAT-P-13,... &#41;</p>
</li>
</ul>
</li>
</ul>
<h3>Value-added features</h3>
<ul>
<li><p>Return subset of data <em>efficiently</em></p>
</li>
<li><p>Many strategies for how to filter data &#40;e.g., order of operations&#41;</p>
</li>
<li><p>Database server can use heuristics to pick good strategy</p>
</li>
<li><p>Allow for transactions to update database</p>
</li>
</ul>
</div>


<div class="markdown"><h3>Fundamental properties of databases</h3>
<ul>
<li><p>Atomicity:  All part of a transaction succeed, or the database is rolled back to its previous state</p>
</li>
<li><p>Consistency:  Data in database always satisfies its validation rules</p>
</li>
<li><p>Isolation: Even if multiple transactions are made concurrently, there is no interference between transactions</p>
</li>
<li><p>Durability:  Once a transaction is committed, it will remain committed</p>
</li>
</ul>
</div>


<div class="markdown"><h3>SQL-based Database Servers</h3>
<p>Open-source:</p>
<ul>
<li><p>MySQL</p>
</li>
<li><p>PostgreSQL &amp; Greenplum</p>
</li>
</ul>
<p>Commercial:</p>
<ul>
<li><p>Microsoft SQL Server: </p>
</li>
<li><p>IBM DB2</p>
</li>
<li><p>Oracle Database</p>
</li>
<li><p>...</p>
</li>
</ul>
</div>


<div class="markdown"><h4>Continuing innovation in database systems</h4>
<ul>
<li><p>SciDB &#40;array &amp; disk based database&#41;</p>
</li>
<li><p>MonetDB &#40;column store&#41;</p>
</li>
<li><p>JuliaDB &#40;pure Julia, for persistent data&#41;</p>
</li>
</ul>
<p>When selecting a database for a project, consider:</p>
<ul>
<li><p>How much data is to be stored?</p>
</li>
<li><p>How frequent/large will transactions be?</p>
</li>
<li><p>Are there specific hardware or OS requirements?</p>
</li>
<li><p>Does the team have someone dedicated to supporting database?</p>
</li>
</ul>
</div>


<div class="markdown"><h3>Database Clients</h3>
<ul>
<li><p>One database server many clients simultaneously</p>
</li>
<li><p>Different clients can use different interfaces</p>
<ul>
<li><p>Command line</p>
</li>
<li><p>Webpage</p>
</li>
<li><p>URL-based</p>
</li>
<li><p>Custom Graphical user interface &#40;GUI&#41;</p>
<ul>
<li><p>TopCat</p>
</li>
</ul>
</li>
</ul>
</li>
</ul>
</div>


<div class="markdown"><h1>Queries</h1>
</div>


<div class="markdown"><p><strong>Query</strong>:  An expression that requests database to return a specific subset of data.  </p>
<h4>Query languages:</h4>
<ul>
<li><p>Structured Query Language &#40;SQL&#41;:  Dated, but by far the most common</p>
</li>
<li><p>Astronomical Data Query Language &#40;ADQL&#41;:  Astronomy-specific</p>
</li>
<li><p>Language Integrated Query &#40;LINQ&#41;:  Microsoft-supported</p>
</li>
<li><p>Many more</p>
</li>
</ul>
</div>


<div class="markdown"><h2>SQL essentials</h2>
<ul>
<li><p>Selecting &#40;columns&#41;</p>
</li>
<li><p>Filtering &#40;for rows&#41;</p>
</li>
<li><p>Joining &#40;multiple tables&#41;</p>
</li>
<li><p>Aggregating &#40;rows within a table&#41;</p>
</li>
</ul>
<h3>SQL programming</h3>
<ul>
<li><p>Variables</p>
</li>
<li><p>Functions</p>
</li>
<li><p>Procedures</p>
</li>
<li><p>Data management</p>
</li>
<li><p>Transactions</p>
</li>
</ul>
</div>


<div class="markdown"><h2>Virtual Observatory &#40;VO&#41;</h2>
<p>Defines standards that help astronomers to collaborate effectively, emphasizing working with multiple data sources.  </p>
<ul>
<li><p>Astronomical Data Query Language &#40;ADQL&#41; </p>
</li>
<li><p>Table Access Protocol &#40;TAP&#41;</p>
</li>
</ul>
</div>


<div class="markdown"><h2>Astronomy-specific functions in ADQL</h2>
<ul>
<li><p>AREA</p>
</li>
<li><p>BOX</p>
</li>
<li><p>CENTROID</p>
</li>
<li><p>CIRCLE</p>
</li>
<li><p>CONTAINS</p>
</li>
<li><p>COORD1</p>
</li>
<li><p>COORD2</p>
</li>
<li><p>COORDSYS</p>
</li>
<li><p>DISTANCE</p>
</li>
<li><p>INTERSECTS</p>
</li>
<li><p>POINT</p>
</li>
<li><p>POLYGON</p>
</li>
<li><p>REGION</p>
</li>
</ul>
</div>


<div class="markdown"><h2>Example Bad SQL Query</h2>
<h4>Do not send</h4>
<pre><code class="language-sql">select * from SomeTable</code></pre>
<p>Why?</p>
</div>


<div class="markdown"><h2>Example SQL Queries</h2>
</div>


<div class="markdown"><h3>Take a quick peak two columns data for first few entries</h3>
<pre><code class="language-sql">select top 10 X, Y from SomeTable </code></pre>
</div>


<div class="markdown"><h2>Find extreme values of X</h2>
<pre><code class="language-sql">select top 10 X, Y 
from SomeTable 
order by X</code></pre>
</div>


<div class="markdown"><h2>Filter which rows are returned using expression</h2>
<pre><code class="language-sql">select top 10 x,y
from SomeTable 
where x*x&#43;y*y between 0 and 1
order by x </code></pre>
</div>


<div class="markdown"><h2>Check how many rows are in a table</h2>
<pre><code class="language-sql">select COUNT&#40;designation&#41; as N from gaiadr2.gaia_source</code></pre>
</div>


<div class="markdown"><h4><a href="https://gea.esac.esa.int/archive/">Gaia Archive</a></h4>
</div>


<div class="markdown"><h2>Check how many rows satisfy a filter</h2>
<pre><code class="language-sql">select COUNT&#40;designation&#41; as n, AVG&#40;astrometric_n_good_obs_al&#41; as astrometric_n_good_obs_al_ave
from gaiadr2.gaia_source
where phot_g_mean_mag &lt; 14</code></pre>
</div>


<div class="markdown"><h2>Grouping data to make a histogram</h2>
<pre><code class="language-sql">select COUNT&#40;designation&#41; as N, 
       AVG&#40;astrometric_n_good_obs_al&#41; as astrometric_n_good_obs_al_ave,  
       AVG&#40;phot_g_mean_mag&#41; as phot_g_mean_mag_ave,
       ROUND&#40;phot_g_mean_mag,1&#41; as bin
from gaiadr2.gaia_source
where phot_g_mean_mag &lt; 14
group by bin 
order by bin</code></pre>
</div>


<div class="markdown"><h1>Table Access Protocol &#40;TAP&#41;</h1>
</div>


<div class="markdown"><p>Start with ADQL </p>
<pre><code class="language-sql">SELECT &lt;column list&gt; FROM &lt;table&gt; WHERE &lt;constraints&gt;</code></pre>
<p>but transform it into a url, by</p>
<ol>
<li><p>Prepend a base service url</p>
</li>
<li><p>convert spaces to <code>&#43;</code>&#39;s</p>
</li>
<li><p>Deal with other special characters &#40;e.g., &#43;, quotes&#41;</p>
</li>
<li><p>Optionally, specify format for results</p>
</li>
</ol>
<p>e.g., </p>
<pre><code class="language-url">https://exoplanetarchive.ipac.caltech.edu/TAP/sync?query&#61;select&#43;pl_name,pl_masse,ra,dec&#43;from&#43;ps</code></pre>
</div>

<pre class='language-julia'><code class='language-julia'>url_ex1 = make_tap_query_url(nexsci_query_base_url, "ps", select_cols="pl_name,gaia_id,sy_kepmag,ra,dec", where="default_flag=1")</code></pre>
<pre id='var-url_ex1' class='code-output documenter-example-output'>"https://exoplanetarchive.ipac.caltech.edu/TAP/sync?query=select+pl_name,gaia_id,sy_kepmag,ra,dec+from+ps+where+default_flag=1&format=tsv"</pre>

<pre class='language-julia'><code class='language-julia'>df_ex1 = query_to_df(url_ex1)</code></pre>
<table>
<tr>
<th></th>
<th>pl_name</th>
<th>gaia_id</th>
<th>sy_kepmag</th>
<th>ra</th>
<th>dec</th>
</tr>
<tr>
<td>1</td>
<td>"OGLE-TR-10 b"</td>
<td>"Gaia DR2 4056443366649948160"</td>
<td>missing</td>
<td>267.868</td>
<td>-29.8765</td>
</tr>
<tr>
<td>2</td>
<td>"BD-08 2823 c"</td>
<td>"Gaia DR2 3770419611540574080"</td>
<td>missing</td>
<td>150.197</td>
<td>-9.51657</td>
</tr>
<tr>
<td>3</td>
<td>"HR 8799 c"</td>
<td>"Gaia DR2 2832463659640297472"</td>
<td>missing</td>
<td>346.87</td>
<td>21.134</td>
</tr>
<tr>
<td>4</td>
<td>"HD 110014 b"</td>
<td>"Gaia DR2 3676091134604409728"</td>
<td>missing</td>
<td>189.811</td>
<td>-7.99567</td>
</tr>
<tr>
<td>5</td>
<td>"HIP 5158 b"</td>
<td>"Gaia DR2 2351405057377686272"</td>
<td>missing</td>
<td>16.5095</td>
<td>-22.4536</td>
</tr>
<tr>
<td>6</td>
<td>"HD 44219 b"</td>
<td>"Gaia DR2 3001428566419966592"</td>
<td>missing</td>
<td>95.06</td>
<td>-10.7251</td>
</tr>
<tr>
<td>7</td>
<td>"HD 132563 b"</td>
<td>"Gaia DR2 1585765117538284800"</td>
<td>missing</td>
<td>224.589</td>
<td>44.0429</td>
</tr>
<tr>
<td>8</td>
<td>"Kepler-24 c"</td>
<td>"Gaia DR2 2052823535171095296"</td>
<td>14.925</td>
<td>290.413</td>
<td>38.3437</td>
</tr>
<tr>
<td>9</td>
<td>"CHXR 73 b"</td>
<td>"Gaia DR2 5201175987817179136"</td>
<td>missing</td>
<td>166.619</td>
<td>-77.6259</td>
</tr>
<tr>
<td>10</td>
<td>"alf Ari b"</td>
<td>missing</td>
<td>missing</td>
<td>31.7933</td>
<td>23.4624</td>
</tr>
<tr>
<td>...</td>
</tr>
<tr>
<td>5197</td>
<td>"TOI-1749 d"</td>
<td>"Gaia DR2 2253774094189458432"</td>
<td>missing</td>
<td>282.737</td>
<td>64.4195</td>
</tr>
</table>


<pre class='language-julia'><code class='language-julia'>desig = replace_spaces_for_tap(df_ex1.gaia_id[8])</code></pre>
<pre id='var-desig' class='code-output documenter-example-output'>"Gaia+DR2+2052823535171095296"</pre>

<pre class='language-julia'><code class='language-julia'>url_ex2 = make_tap_query_url(gaia_query_base_url, "gaiadr2.gaia_source", where="designation='$(desig)'",select_cols="*",max_rows=5)</code></pre>
<pre id='var-url_ex2' class='code-output documenter-example-output'>"https://gea.esac.esa.int/tap-server/tap/sync?REQUEST=doQuery&LANG=ADQL&FORMAT=csv&QUERY=select+top+5+*+from+gaiadr2.gaia_source+where+designation='Gaia+DR2+2052823535171095296'&format=tsv"</pre>

<pre class='language-julia'><code class='language-julia'>df_ex2 = query_to_df(url_ex2)</code></pre>
<table>
<tr>
<th></th>
<th>solution_id</th>
<th>designation</th>
<th>source_id</th>
<th>random_index</th>
<th>ref_epoch</th>
<th>ra</th>
<th>ra_error</th>
<th>dec</th>
<th>...</th>
</tr>
<tr>
<td>1</td>
<td>1635721458409799680</td>
<td>"Gaia DR2 2052823535171095296"</td>
<td>2052823535171095296</td>
<td>197836529</td>
<td>2015.5</td>
<td>290.413</td>
<td>0.020076</td>
<td>38.3437</td>
<td></td>
</tr>
</table>



<div class="markdown"><h1>Joins</h1>
</div>


<div class="markdown"><p>Joining tables is a fundamental concept that can be applied either to DataFrames stored locally or as part of SQL/ADQL queries.</p>
</div>


<div class="markdown"><ul>
<li><p><code>innerjoin</code> &amp; <code>semijoin</code>:  Return rows for values of the key that exist <strong>in both tables</strong></p>
</li>
<li><p><code>outerjoin</code>: Return rows for values of the key that exist <strong>in either table</strong></p>
</li>
<li><p><code>leftjoin</code>: Return rows for values of the key that exist <strong>in first table</strong></p>
</li>
<li><p><code>rightjoin</code>: Return rows for values of the key that exist <strong>in second table</strong></p>
</li>
</ul>
<ul>
<li><p><code>antijoin</code>: Return rows Return rows for values of the key that exist <strong>in first table but not the second table</strong></p>
</li>
</ul>
<ul>
<li><p><code>crossjoin</code>: Return table with every row from first table as rows and every row from second table as columns</p>
</li>
</ul>
</div>


<div class="markdown"><h3>Examples</h3>
</div>

<pre class='language-julia'><code class='language-julia'>df_ex3 = innerjoin(df_ex1,df_ex2, on=:gaia_id=&gt;:designation, matchmissing=:notequal, makeunique=true )</code></pre>
<table>
<tr>
<th></th>
<th>pl_name</th>
<th>gaia_id</th>
<th>sy_kepmag</th>
<th>ra</th>
<th>dec</th>
<th>solution_id</th>
<th>source_id</th>
<th>random_index</th>
<th>...</th>
</tr>
<tr>
<td>1</td>
<td>"Kepler-24 c"</td>
<td>"Gaia DR2 2052823535171095296"</td>
<td>14.925</td>
<td>290.413</td>
<td>38.3437</td>
<td>1635721458409799680</td>
<td>2052823535171095296</td>
<td>197836529</td>
<td></td>
</tr>
<tr>
<td>2</td>
<td>"Kepler-24 d"</td>
<td>"Gaia DR2 2052823535171095296"</td>
<td>14.925</td>
<td>290.413</td>
<td>38.3437</td>
<td>1635721458409799680</td>
<td>2052823535171095296</td>
<td>197836529</td>
<td></td>
</tr>
<tr>
<td>3</td>
<td>"Kepler-24 b"</td>
<td>"Gaia DR2 2052823535171095296"</td>
<td>14.925</td>
<td>290.413</td>
<td>38.3437</td>
<td>1635721458409799680</td>
<td>2052823535171095296</td>
<td>197836529</td>
<td></td>
</tr>
<tr>
<td>4</td>
<td>"Kepler-24 e"</td>
<td>"Gaia DR2 2052823535171095296"</td>
<td>14.925</td>
<td>290.413</td>
<td>38.3437</td>
<td>1635721458409799680</td>
<td>2052823535171095296</td>
<td>197836529</td>
<td></td>
</tr>
</table>


<pre class='language-julia'><code class='language-julia'>names(df_ex3)</code></pre>
<pre id='var-hash178506' class='code-output documenter-example-output'>99-element Vector{String}:
 "pl_name"
 "gaia_id"
 "sy_kepmag"
 "ra"
 "dec"
 "solution_id"
 "source_id"
 ⋮
 "radius_percentile_lower"
 "radius_percentile_upper"
 "lum_val"
 "lum_percentile_lower"
 "lum_percentile_upper"
 "datalink_url"</pre>

<pre class='language-julia'><code class='language-julia'>tip(md"Originally, both tables contained columns named `ra` and `dec`.  The joined table contains columns `ra` and `ra_1` (and `dec` and `dec_1`) because we set `makeunique`.")</code></pre>
<div class="markdown"><div class="admonition is-tip">
  <header class="admonition-header">
    Tip
  </header>
  <div class="admonition-body">
    <p>Originally, both tables contained columns named <code>ra</code> and <code>dec</code>.  The joined table contains columns <code>ra</code> and <code>ra_1</code> &#40;and <code>dec</code> and <code>dec_1</code>&#41; because we set <code>makeunique</code>.</p>
  </div>

</div>
</div>


<div class="markdown"><h4>What if we didn&#39;t know the Gaia designation?</h4>
</div>

<pre class='language-julia'><code class='language-julia'>targetpos = (; ra = df_ex1.ra[1], dec = df_ex1.dec[1] )</code></pre>
<pre id='var-targetpos' class='code-output documenter-example-output'>(ra = 267.8677483, dec = -29.8764758)</pre>

<pre class='language-julia'><code class='language-julia'>url_ex4 = make_tap_query_url(gaia_query_base_url, "gaiadr3.gaia_source", where="1=contains(POINT($(targetpos.ra),$(targetpos.dec)),CIRCLE(ra,dec,30./3600.))", select_cols="*,DISTANCE(POINT($(targetpos.ra),$(targetpos.dec)),POINT(ra,dec))+AS+ang_sep",order_by_cols="ang_sep",max_rows=1000)</code></pre>
<pre id='var-url_ex4' class='code-output documenter-example-output'>"https://gea.esac.esa.int/tap-server/tap/sync?REQUEST=doQuery&LANG=ADQL&FORMAT=csv&QUERY=select+top+1000+*,DISTANCE(POINT(267.8677483,-29.8764758),POINT(ra,dec))+AS+ang_sep+from+gaiadr3.gaia_source+where+1=contains(POINT(267.8677483,-29.8764758),CIRCLE(ra,dec,30./3600.))+order+by+ang_sep&format=tsv"</pre>

<pre class='language-julia'><code class='language-julia'>df_ex4 = query_to_df(url_ex4)</code></pre>
<table>
<tr>
<th></th>
<th>solution_id</th>
<th>designation</th>
<th>source_id</th>
<th>random_index</th>
<th>ref_epoch</th>
<th>ra</th>
<th>ra_error</th>
<th>dec</th>
<th>...</th>
</tr>
<tr>
<td>1</td>
<td>1636148068921376768</td>
<td>"Gaia DR3 4056443366649948160"</td>
<td>4056443366649948160</td>
<td>628955980</td>
<td>2016.0</td>
<td>267.868</td>
<td>0.0484435</td>
<td>-29.8765</td>
<td></td>
</tr>
<tr>
<td>2</td>
<td>1636148068921376768</td>
<td>"Gaia DR3 4056443366695974528"</td>
<td>4056443366695974528</td>
<td>1783248586</td>
<td>2016.0</td>
<td>267.867</td>
<td>0.301148</td>
<td>-29.8763</td>
<td></td>
</tr>
<tr>
<td>3</td>
<td>1636148068921376768</td>
<td>"Gaia DR3 4056443366694853888"</td>
<td>4056443366694853888</td>
<td>82166964</td>
<td>2016.0</td>
<td>267.868</td>
<td>0.597072</td>
<td>-29.8772</td>
<td></td>
</tr>
<tr>
<td>4</td>
<td>1636148068921376768</td>
<td>"Gaia DR3 4056443366696002944"</td>
<td>4056443366696002944</td>
<td>586843531</td>
<td>2016.0</td>
<td>267.869</td>
<td>0.238754</td>
<td>-29.8765</td>
<td></td>
</tr>
<tr>
<td>5</td>
<td>1636148068921376768</td>
<td>"Gaia DR3 4056443370989465856"</td>
<td>4056443370989465856</td>
<td>1370584052</td>
<td>2016.0</td>
<td>267.869</td>
<td>9.60017</td>
<td>-29.8771</td>
<td></td>
</tr>
<tr>
<td>6</td>
<td>1636148068921376768</td>
<td>"Gaia DR3 4056443366695975424"</td>
<td>4056443366695975424</td>
<td>1032450251</td>
<td>2016.0</td>
<td>267.869</td>
<td>0.207094</td>
<td>-29.8754</td>
<td></td>
</tr>
<tr>
<td>7</td>
<td>1636148068921376768</td>
<td>"Gaia DR3 4056443366694961152"</td>
<td>4056443366694961152</td>
<td>1119708763</td>
<td>2016.0</td>
<td>267.867</td>
<td>0.596232</td>
<td>-29.8779</td>
<td></td>
</tr>
<tr>
<td>8</td>
<td>1636148068921376768</td>
<td>"Gaia DR3 4056443370989465728"</td>
<td>4056443370989465728</td>
<td>589362746</td>
<td>2016.0</td>
<td>267.869</td>
<td>5.98055</td>
<td>-29.8774</td>
<td></td>
</tr>
<tr>
<td>9</td>
<td>1636148068921376768</td>
<td>"Gaia DR3 4056443366694892160"</td>
<td>4056443366694892160</td>
<td>781324219</td>
<td>2016.0</td>
<td>267.869</td>
<td>0.40115</td>
<td>-29.8777</td>
<td></td>
</tr>
<tr>
<td>10</td>
<td>1636148068921376768</td>
<td>"Gaia DR3 4056443370989472384"</td>
<td>4056443370989472384</td>
<td>1372304020</td>
<td>2016.0</td>
<td>267.866</td>
<td>0.953113</td>
<td>-29.8758</td>
<td></td>
</tr>
<tr>
<td>...</td>
</tr>
<tr>
<td>276</td>
<td>1636148068921376768</td>
<td>"Gaia DR3 4056443370989452672"</td>
<td>4056443370989452672</td>
<td>1608917468</td>
<td>2016.0</td>
<td>267.871</td>
<td>5.46342</td>
<td>-29.8843</td>
<td></td>
</tr>
</table>



<div class="markdown"><h4>Wait, which row is the best match?</h4>
</div>

<pre class='language-julia'><code class='language-julia'>sort(df_ex4[!,[:designation,:ang_sep,:phot_g_mean_mag] ], :ang_sep)</code></pre>
<table>
<tr>
<th></th>
<th>designation</th>
<th>ang_sep</th>
<th>phot_g_mean_mag</th>
</tr>
<tr>
<td>1</td>
<td>"Gaia DR3 4056443366649948160"</td>
<td>7.49048e-7</td>
<td>15.669</td>
</tr>
<tr>
<td>2</td>
<td>"Gaia DR3 4056443366695974528"</td>
<td>0.000588803</td>
<td>18.363</td>
</tr>
<tr>
<td>3</td>
<td>"Gaia DR3 4056443366694853888"</td>
<td>0.000686453</td>
<td>18.3986</td>
</tr>
<tr>
<td>4</td>
<td>"Gaia DR3 4056443366696002944"</td>
<td>0.00116672</td>
<td>18.6382</td>
</tr>
<tr>
<td>5</td>
<td>"Gaia DR3 4056443370989465856"</td>
<td>0.00118059</td>
<td>20.2377</td>
</tr>
<tr>
<td>6</td>
<td>"Gaia DR3 4056443366695975424"</td>
<td>0.00138877</td>
<td>18.5433</td>
</tr>
<tr>
<td>7</td>
<td>"Gaia DR3 4056443366694961152"</td>
<td>0.00148488</td>
<td>19.7284</td>
</tr>
<tr>
<td>8</td>
<td>"Gaia DR3 4056443370989465728"</td>
<td>0.00152604</td>
<td>20.2813</td>
</tr>
<tr>
<td>9</td>
<td>"Gaia DR3 4056443366694892160"</td>
<td>0.00153313</td>
<td>19.1686</td>
</tr>
<tr>
<td>10</td>
<td>"Gaia DR3 4056443370989472384"</td>
<td>0.00170832</td>
<td>18.3691</td>
</tr>
<tr>
<td>...</td>
</tr>
<tr>
<td>276</td>
<td>"Gaia DR3 4056443370989452672"</td>
<td>0.00831395</td>
<td>19.7214</td>
</tr>
</table>


<pre class='language-julia'><code class='language-julia'>df_ex1.sy_kepmag[8]</code></pre>
<pre id='var-hash104717' class='code-output documenter-example-output'>14.925</pre>


<div class="markdown"><h2>Questions?</h2>
</div>


<div class="markdown"><div class="admonition is-admonition">
  <header class="admonition-header">
    Question
  </header>
  <div class="admonition-body">
    <p>Would it be possible to use available data sets to discover planets that have not been found yet by anyone else?</p>
  </div>
</div>
</div>


<div class="markdown"><h1>Setup &amp; Helper Code</h1>
</div>

<pre class='language-julia'><code class='language-julia'>ChooseDisplayMode()</code></pre>
<!-- https://github.com/fonsp/Pluto.jl/issues/400#issuecomment-695040745 -->
<input
        type="checkbox"
        id="width-over-livedocs"
        name="width-over-livedocs"
    onclick="window.plutoOptIns.toggle_width(this)"
        >
<label for="width-over-livedocs">
        Full Width Mode
</label>
<style>
        body.width-over-docs #helpbox-wrapper {
        display: none !important;
        }
        body.width-over-docs main {
               max-width: none !important;
               margin: 0 !important;
                #max-width: 1100px;
                #max-width: calc(100% - 4rem);
                #align-self: flex-star;
                #margin-left: 50px;
                #margin-right: 2rem;
        }
</style>
<script>
        const toggle_width = function(t) {
                t.checked
                ? document.body.classList.add("width-over-docs")
                : document.body.classList.remove("width-over-docs") }
        window.plutoOptIns = window.plutoOptIns || {}
        window.plutoOptIns.toggle_width = toggle_width
        
</script>
&nbsp; &nbsp; &nbsp;
<input
        type="checkbox"
        id="present-mode"
        name="present-mode"
        onclick="present()"
        >
<label for="present_mode">
        Present Mode
</label>



<pre class='language-julia'><code class='language-julia'>begin
    using CSV, DataFrames, Query
    using HTTP
    using PlutoUI, PlutoTeachingTools
end</code></pre>


<pre class='language-julia'><code class='language-julia'>begin
    nexsci_query_base_url = "https://exoplanetarchive.ipac.caltech.edu/TAP/sync?query="
    gaia_query_base_url = 
    "https://gea.esac.esa.int/tap-server/tap/sync?REQUEST=doQuery&LANG=ADQL&FORMAT=csv&QUERY="
end;</code></pre>


<pre class='language-julia'><code class='language-julia'>begin
    make_tap_query_url_url = "#" * (PlutoRunner.currently_running_cell_id |&gt; string)
"""
`make_tap_query_url(base_url, table_name; ...)`

Returns url for a Table Access Protocol (TAP) query.
Inputs:
- base url 
- table name
Optional arguments (and default):
- `max_rows` (all)
- `select_cols` (all)
- `where` (no requirements)
- `order_by_cols` (not sorted)
- `format` (tsv)
See [NExScI](https://exoplanetarchive.ipac.caltech.edu/docs/TAP/usingTAP.html#sync) or [Virtual Observatory](https://www.ivoa.net/documents/TAP/) for more info.
"""
function make_tap_query_url(query_base_url::String, query_table::String; max_rows::Integer = 0, select_cols::String = "", where::String = "", order_by_cols::String = "", format::String="tsv" )
    
    query_select = "select"
    if max_rows &gt; 0 
        query_select *= "+top+" * string(max_rows)
    end
    if length(select_cols) &gt;0
        query_select *= "+" * select_cols 
    else
        query_select *= "+*"
    end
    query_from = "+from+" * query_table
    query_where = length(where)&gt;0 ? "+where+" * where : ""
    query_order_by = length(order_by_cols) &gt; 0 ? "+order+by+" * order_by_cols : ""
    query_format = "&format=" * format
    url = query_base_url * query_select * query_from * query_where * query_order_by * query_format
end
end</code></pre>


<pre class='language-julia'><code class='language-julia'>"""
`query_to_df(url)` downloads data from a URL and attempts to place it into a DataFrame
"""
query_to_df(url) = CSV.read(HTTP.get(url).body,DataFrame)</code></pre>


<pre class='language-julia'><code class='language-julia'>"""`replace_spaces_for_tap(str)`

Replace spaces with +'s as expected for TAP queries.
"""
replace_spaces_for_tap(s::AbstractString) = replace(s," "=&gt;"+")</code></pre>


<pre class='language-julia'><code class='language-julia'>begin
    """ 
    `select_cols_for_tap(cols)`

    Returns a string of comma-separated columns names from a vector of columns names (as either strings or symbols), for using in a TAP query.
    """
    function select_cols_for_tap end
    select_cols_for_tap(cols_to_keep::AbstractVector{Symbol}) = select_cols_for_tap(string.(cols_to_keep)) #string(map(s-&gt;string(s) * "+", cols_to_keep)...)[1:end-1]
    select_cols_for_tap(cols_to_keep::AbstractVector{AS}) where {AS&lt;:AbstractString} = string(map(s-&gt;s * ",", cols_to_keep)...)[1:end-1]
    select_cols_for_tap(col_to_keep::Symbol) = string(col_to_keep)
    select_cols_for_tap(col_to_keep::AbstractString) = col_to_keep
end
</code></pre>
<pre id='var-select_cols_for_tap' class='code-output documenter-example-output'>select_cols_for_tap (generic function with 4 methods)</pre>



<div class='manifest-versions'>
<p>Built with Julia 1.8.2 and</p>
CSV 0.10.4<br>
DataFrames 1.3.6<br>
HTTP 1.4.0<br>
PlutoTeachingTools 0.2.3<br>
PlutoUI 0.7.43<br>
Query 1.0.0
</div>

<!-- PlutoStaticHTML.End -->
~~~

_To run this tutorial locally, download [this file](/notebooks/week8day1.jl) and open it with
[Pluto.jl](https://plutojl.org)._


_To run this tutorial locally, download [this file](/notebooks/week8day1.jl) and open it with
[Pluto.jl](https://plutojl.org)._


_To run this tutorial locally, download [this file](/notebooks/week8day1.jl) and open it with
[Pluto.jl](https://plutojl.org)._

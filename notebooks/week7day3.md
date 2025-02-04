~~~
<!-- PlutoStaticHTML.Begin -->
<!--
    # This information is used for caching.
    [PlutoStaticHTML.State]
    input_sha = "425fb1d603f16ecb142a2aeed44db47dfb2d0e048abf76e88de79dd392375ce2"
    julia_version = "1.8.2"
-->

<div class="markdown"><h1>Class Projects</h1>
<p><strong>Astro 497, Week 7, Day 3</strong></p>
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



<div class="markdown"><h2>Logistics</h2>
<ul>
<li><p>Exam Results</p>
</li>
<li><p>Lessons Learned</p>
</li>
<li><p>Mid-Semester Survey</p>
</li>
</ul>
</div>


<div class="markdown"><h3>Project Overview</h3>
<p>Students will synthesize lessons learned in the class by building an exoplanet <em>dashboard</em> that ingests data related to detecting and/or characterizing exoplanets, performs basic data manipulations, fits a model to the data, assesses the quality of the model for the given observations, and effectively visualizes the results.</p>
</div>


<div class="markdown"><h1>What is a Dashboard?</h1>
</div>


<div class="markdown"><h2>Purpose</h2>
<ul>
<li><p>Efficiently communicate what can be learned from data</p>
</li>
</ul>
<h2>How</h2>
<ul>
<li><p>Automating common tasks</p>
<ul>
<li><p>Incorporating &#40;new&#41; data into decision making process</p>
</li>
<li><p>Data wrangling &#40;e.g., cleaning, transforming, analyzing&#41;</p>
</li>
<li><p>Applying simple models</p>
</li>
<li><p>Evaluating models</p>
</li>
<li><p>Providing common visualizations</p>
</li>
</ul>
</li>
<li><p>Facilitate communications &amp; learning</p>
<ul>
<li><p>Visualizing data</p>
</li>
<li><p>Visualizing model predictions</p>
</li>
<li><p>Providing common model assessment metrics</p>
</li>
<li><p>Automate easy decisions</p>
</li>
<li><p>Ease finding information to make hard decisions</p>
</li>
</ul>
</li>
</ul>
</div>


<div class="markdown"><h2>Dashboard <a href="https://psuastro497.github.io/Fall2022/project/#elements">Elements</a></h2>
<ul>
<li><p>Ingest data</p>
</li>
<li><p>Data Wrangling</p>
</li>
<li><p>Model Fitting</p>
</li>
<li><p>Model Assessment</p>
</li>
<li><p>Visualization</p>
</li>
<li><p>Warning Messages</p>
</li>
</ul>
</div>


<div class="markdown"><h1>Project <a href="https://psuastro497.github.io/Fall2022/project/#timeline">Timeline</a></h1>
<ul>
<li><p><a href="https://psuastro497.github.io/Fall2022/project/plan">Project Plan</a> &#40;due Oct 19&#41;</p>
</li>
<li><p><a href="https://psuastro497.github.io/Fall2022/project/checkpoint">Project Checkpoint/Progress Report 1</a> &#40;due Oct 31&#41;</p>
</li>
<li><p><a href="https://psuastro497.github.io/Fall2022/project/checkpoint">Project Checkpoint/Progress Report 2</a> &#40;due Nov 14&#41;</p>
</li>
<li><p><a href="https://psuastro497.github.io/Fall2022/project/dashboard/">Project Dashboard</a> &#40;due Nov 28&#41;</p>
</li>
<li><p><a href="https://psuastro497.github.io/Fall2022/project/presentation/">Project Presentations</a> &#40;due Dec 2 - 9&#41;</p>
</li>
<li><p><a href="https://psuastro497.github.io/Fall2022/project/report/">Individual Report &amp; Reflection</a> &#40;due Dec 9&#41;</p>
</li>
</ul>
</div>


<div class="markdown"><h1>Project Plan</h1>
<h3>Purpose</h3>
<p>What will be the purpose of the dashboard? </p>
<h3>Obtaining Data</h3>
<p>What data set&#40;s&#41; will your dashboard use for its analysis?</p>
<ul>
<li><p>What observatories and instruments could provide the data to be analyzed?</p>
</li>
<li><p>How many different objects &#40;or time periods&#41; are publicly available?</p>
</li>
<li><p>Where will you/your dashboard download the data from?</p>
</li>
<li><p>Is the data small enough that we will download the entire dataset once? Or is the dataset large enough that the dashboard will query a database to retrieve the data for each object &#40;or time period&#41; separate?</p>
</li>
<li><p>What format will the data be in?</p>
</li>
</ul>
<h3>Data Wrangling</h3>
<ul>
<li><p>What data wrangling tasks &#40;e.g., cleaning, transforming&#41; do you anticipate needing to perform?</p>
</li>
<li><p>Will the data for each object &#40;or time period&#41; arrive in a single table? Or will you need to perform joins across multiple tables?</p>
</li>
</ul>
<h3>Modeling</h3>
<ul>
<li><p>What models will your dashboard fit to the data?</p>
</li>
<li><p>What will serve as the robust baseline model?</p>
</li>
<li><p>What will serve as the more sophisticated model?</p>
</li>
<li><p>What will the models predict?</p>
</li>
<li><p>How will you assess your models?</p>
</li>
</ul>
<h3>Visualize/Communicate Results</h3>
<p>Describe the plots that will be displayed on your dashboard. For each plot:</p>
<ul>
<li><p>What data will be shown?</p>
</li>
<li><p>Will it be plotted with a curve, points, contours, histogram, etc.?  </p>
</li>
<li><p>What will be the axes?</p>
</li>
<li><p>Is there additional information that could be conveyed through other attributes like size or color of points?</p>
</li>
<li><p>Would it be helpful to include multiple panels &#40;e.g., to show data on different x or y scales, or to show predictions of different models&#41;?</p>
</li>
<li><p>Will the figures that you have already described be sufficient for the dashboard to achieve its purpose? Or do you anticipate needing additional experimentation to convey the results of the analysis effectively? If you have some early ideas, then provide enough information that you can get constructive feedback on them.</p>
</li>
</ul>
<h3>Project schedule</h3>
<ul>
<li><p>What tasks do you &#40;or each member of your team&#41; plan to accomplish each week? Make sure to account for scheduling constraints such as exams or big assignments in other classes, holidays, and travel. Be sure to allow some contingency in the schedule for tasks that take longer than expected or other unexpected delays.</p>
</li>
<li><p>If you&#39;re working as part of a team, then make a plan for how your team will work. Will you work together on each task simultaneously? Will each person be responsible for writing code to do specific tasks separately? It&#39;s particularly important to make a plan that doesn&#39;t create problematic dependencies &#40;e.g., one person needs to wait for working code from someone else and the team can only meet the deadline if everything goes perfectly&#41;.</p>
</li>
<li><p>If you or your team have any hard scheduling constraints that would prevent them from presenting during class on Dec 2, 5, 7 or 9. You may also indicate any additional scheduling preferences.</p>
</li>
</ul>
</div>


<div class="markdown"><h2>Teamwork</h2>
</div>


<div class="markdown"><h1>Questions</h1>
</div>

<pre class='language-julia'><code class='language-julia'>question(md"""
Is it necessary to do the final project in Julia? 
Can we do it in a language like R or Python instead?
""")</code></pre>
<div class="markdown"><div class="admonition is-tip">
  <header class="admonition-header">
    Question
  </header>
  <div class="admonition-body">
    <p>Is it necessary to do the final project in Julia?  Can we do it in a language like R or Python instead?</p>
  </div>

</div>
</div>


<div class="markdown"><h3>By far the easiest way to meet class requirements:</h3>
<ul>
<li><p>Pluto notebook &amp; Julia</p>
</li>
<li><p>For nice UI can use <a href="https://github.com/fonsp/Pluto.jl">PlutoUI.jl</a></p>
</li>
</ul>
<h3>A little extra hassle, but very possibly worth it</h3>
<ul>
<li><p>Pluto notebook with Julia, plus calls to Python or R</p>
<ul>
<li><p><a href="https://github.com/JuliaPy/PyCall.jl">PyCall.jl</a>:  Justin &amp; I have tested on Roar for you.</p>
</li>
<li><p><a href="https://github.com/cjdoris/PythonCall.jl">PythonCall.jl</a>:  Probably nicer in long term, but I&#39;m not sure it&#39;s ready yet.</p>
</li>
<li><p><a href="https://juliainterop.github.io/RCall.jl/stable/">RCall.jl</a>:  For R users</p>
</li>
</ul>
</li>
<li><p>Examples of when this would make sense:  </p>
<ul>
<li><p>Reading data in obscure file formats using <a href="https://docs.astropy.org/en/stable/io/unified.html">astropy.io</a></p>
</li>
<li><p>Downloading data using <a href="https://astroquery.readthedocs.io/en/latest/">astroquery</a> or archive specific package &#40;e.g., lightkurve or pyneid&#41;</p>
</li>
</ul>
</li>
</ul>
<h3>In theory, there are environments that could work</h3>
<h4>But it will probably take significantly more time.</h4>
<ul>
<li><p><a href="https://dash.gallery/Portal/">Dash</a> &#40;Python-specific&#41;</p>
</li>
<li><p><a href="https://github.com/herbps10/Reactor/">Reactor</a> &#40;R-specific&#41;</p>
</li>
<li><p><a href="https://shiny.rstudio.com/">Shiny</a> &#40;R-specific&#41;</p>
</li>
<li><p>Potentially more, but I&#39;m worried that they may be less mature, reliable, polished, well documented, etc.:</p>
<ul>
<li><p><a href="https://dataflownb.github.io/">Dataflow notebooks</a>? &#40;Python-specific, on top of Jupyter&#41;</p>
</li>
<li><p><a href="https://observablehq.com/">Observable</a>? &#40;Javascript&#43;Something else&#41; </p>
</li>
</ul>
</li>
</ul>
</div>

<pre class='language-julia'><code class='language-julia'>warning_box(md"""
If you try something other than Pluto, be prepared to spend significant ammount of time: figuring it out yourself, rewriting code to do tasks that I've already provided examples for, making the dashboard work reliably, and automating the setup process.
""")</code></pre>
<div class="markdown"><div class="admonition is-warning">
  <header class="admonition-header">
    Warning:
  </header>
  <div class="admonition-body">
    <p>If you try something other than Pluto, be prepared to spend significant ammount of time: figuring it out yourself, rewriting code to do tasks that I&#39;ve already provided examples for, making the dashboard work reliably, and automating the setup process.</p>
  </div>

</div>
</div>

<pre class='language-julia'><code class='language-julia'>warning_box(md"""
It is to be a dashboard, not a notebook or a project report:
- It should work on "new" data that you won't have been able to test it on
- Can not require users to rerun cells in a specific order after selecting dataset (e.g., target or date range) or changing a parameter.
- It should be *extremely easy* for users to use.
""")</code></pre>
<div class="markdown"><div class="admonition is-warning">
  <header class="admonition-header">
    Warning:
  </header>
  <div class="admonition-body">
    <p>It is to be a dashboard, not a notebook or a project report:</p>
  </div>
<ul>
<li><p>It should work on &quot;new&quot; data that you won&#39;t have been able to test it on</p>
</li>
<li><p>Can not require users to rerun cells in a specific order after selecting dataset &#40;e.g., target or date range&#41; or changing a parameter.</p>
</li>
<li><p>It should be <em>extremely easy</em> for users to use.</p>
</li>
</ul>

</div>
</div>

<pre class='language-julia'><code class='language-julia'>warning_box(md"""
If using another language make absolutely sure that your dashboard works reliably for other users and on other systems.  
- Exactly reproduces all package versions
- Any dependencies need to be automatically installed (likely in user space)
- Works on Linux (ideally also MacOS, Windows, etc., but I won't test that) 
- Automatically deals with file paths, system libraries, etc.
- These details are often annoying, but: (1) the Julia & Pluto developers have taken care of the first two, and (2) Justin and I have already setup Roar to solve the remaining details, including using PyCall with astropy, pyquery and lightkurve.
""")</code></pre>
<div class="markdown"><div class="admonition is-warning">
  <header class="admonition-header">
    Warning:
  </header>
  <div class="admonition-body">
    <p>If using another language make absolutely sure that your dashboard works reliably for other users and on other systems.  </p>
  </div>
<ul>
<li><p>Exactly reproduces all package versions</p>
</li>
<li><p>Any dependencies need to be automatically installed &#40;likely in user space&#41;</p>
</li>
<li><p>Works on Linux &#40;ideally also MacOS, Windows, etc., but I won&#39;t test that&#41; </p>
</li>
<li><p>Automatically deals with file paths, system libraries, etc.</p>
</li>
<li><p>These details are often annoying, but: &#40;1&#41; the Julia &amp; Pluto developers have taken care of the first two, and &#40;2&#41; Justin and I have already setup Roar to solve the remaining details, including using PyCall with astropy, pyquery and lightkurve.</p>
</li>
</ul>

</div>
</div>

<pre class='language-julia'><code class='language-julia'>warning_box(md"""
- I recommend that students who want to engage in original research become fluent in at least one high-level language (e.g., julia, python, R, IDL, matlab, Mathematica,...) and one compiled and strongly-typed language (e.g., julia, C/C++, Fortran,...).  
- If you are only fluent in high-level language(s), then there will come a time when you are severely limited in what you can do.  This is particularly a concern for people likely to work with large datasets, large models and/or computationally expensive models.  

→ If you are only fluent in high-level language(s), then I suggest using this opportunity to expand your skillset.  
""")</code></pre>
<div class="markdown"><div class="admonition is-warning">
  <header class="admonition-header">
    Warning:
  </header>
<ul>
<li>  <div class="admonition-body">
    <p>I recommend that students who want to engage in original research become fluent in at least one high-level language &#40;e.g., julia, python, R, IDL, matlab, Mathematica,...&#41; and one compiled and strongly-typed language &#40;e.g., julia, C/C&#43;&#43;, Fortran,...&#41;.  </p>
  </div>
</li>
<li><p>If you are only fluent in high-level language&#40;s&#41;, then there will come a time when you are severely limited in what you can do.  This is particularly a concern for people likely to work with large datasets, large models and/or computationally expensive models.  </p>
</li>
</ul>
<p>→ If you are only fluent in high-level language&#40;s&#41;, then I suggest using this opportunity to expand your skillset.  </p>

</div>
</div>

<pre class='language-julia'><code class='language-julia'>question(md"""Will we have access to any public databases or specific ones chosen for us?""")</code></pre>
<div class="markdown"><div class="admonition is-tip">
  <header class="admonition-header">
    Question
  </header>
  <div class="admonition-body">
    <p>Will we have access to any public databases or specific ones chosen for us?</p>
  </div>

</div>
</div>


<div class="markdown"><p>Your choice.   &#40;If you have a datasource in mind and would like suggestions for how to access it, let me know.&#41;</p>
<p>Potential Data Sources</p>
<ul>
<li><p>Transit light curves:</p>
<ul>
<li><p>Kepler/K2</p>
</li>
<li><p>TESS</p>
</li>
</ul>
</li>
<li><p>Transit Timing Variations:</p>
<ul>
<li><p>Table of transit times from Holczer et al. &#40;2016&#41;</p>
</li>
</ul>
</li>
<li><p>Radial Velocities:</p>
<ul>
<li><p>California Legacy Survey RVs</p>
</li>
<li><p>NEID standard star observations</p>
</li>
<li><p>NEID solar observations</p>
</li>
</ul>
</li>
<li><p>Host star properties</p>
<ul>
<li><p>California Legacy Survey spectra</p>
</li>
<li><p>NEID standard star spectra</p>
</li>
<li><p>Gaia</p>
</li>
</ul>
</li>
</ul>
</div>

<pre class='language-julia'><code class='language-julia'>question(md"What are some good external resources that provide in-depth explanations on methods for exoplanet data analysis?")</code></pre>
<div class="markdown"><div class="admonition is-tip">
  <header class="admonition-header">
    Question
  </header>
  <div class="admonition-body">
    <p>What are some good external resources that provide in-depth explanations on methods for exoplanet data analysis?</p>
  </div>

</div>
</div>


<div class="markdown"><p>It depends on the method.  For most problems, details are sufficiently technical that you need to go to original journal articles.  Usually, the state-of-the-art requires reading a set of papers each of which describes one or two steps in detail, but cite other papers for the details of the other steps.  &#40;I can help find those for a particular method.&#41;</p>
</div>

<pre class='language-julia'><code class='language-julia'>question(md"Would it be possible to use available data sets to discover planets that have not been found yet by anyone else?")</code></pre>
<div class="markdown"><div class="admonition is-tip">
  <header class="admonition-header">
    Question
  </header>
  <div class="admonition-body">
    <p>Would it be possible to use available data sets to discover planets that have not been found yet by anyone else?</p>
  </div>

</div>
</div>


<div class="markdown"><p>Yes.  It&#39;s possible.  That said, the easiest-to-find planets are the least likely to have been overlooked.  I&#39;d encourage you to set goals that don&#39;t depend on what others have done in the past &#40;e.g., build the capability to detect a planet, but call it a success even if all your detections have been discovered previously&#41;.</p>
</div>


<div class="markdown"><h2>Dashboard Checklist</h2>
<ol>
<li><p>Dashboard successfully reads in data for user selected objects and/or time periods. &#40;1 point&#41;</p>
</li>
<li><p>Dashboard performs whatever data wrangling is necessary to provide high-quality results in subsequent analysis. &#40;1 point&#41;</p>
</li>
<li><p>Dashboard provides effective visualizations of input data &#40;with relatively little preprocessing and, if applicable, after any potentially significant preprocessing&#41;. &#40;1 point&#41;</p>
</li>
<li><p>Dashboard successfully fits baseline model to user-selected data. &#40;1 point&#41;</p>
</li>
<li><p>Dashboard effectively visualizes the predictions of the baseline model and the deviations of the predictions from observations. &#40;1 point&#41;</p>
</li>
<li><p>Dashboard provides accurate and useful assessment of quality of results from baseline model. &#40;1 point&#41;</p>
</li>
<li><p>Dashboard successfully fits at least one more sophisticated model to user-selected data. &#40;2 point&#41;</p>
</li>
<li><p>Dashboard effectively visualizes the predictions of at least one more sophisticated model and the deviations of the predictions from observations. &#40;1 point&#41;</p>
</li>
<li><p>Dashboard provides accurate and useful assessment of quality of results from at least one more sophisticated model. &#40;2 point&#41;</p>
</li>
<li><p>Dashboard provides additional visualizations and prominent warning messages that communicate the results of the analysis effectively and clearly. &#40;3 point&#41;</p>
</li>
<li><p>Dashboard successfully runs to completion &#40;and any error messages are in plain English&#41; on Roar without manual setup steps. &#40;2 points&#41;</p>
</li>
<li><p>Does the dashboard provide a simple and effective user interface for selecting data to be analyzed, setting any user-specified parameters and/or interacting with visualizations. &#40;2 points&#41;</p>
</li>
<li><p>Does the dashboard effectively achieve its stated goals? &#40;2 point&#41;</p>
</li>
</ol>
</div>


<div class="markdown"><h1>Setup</h1>
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



<pre class='language-julia'><code class='language-julia'>using PlutoUI, PlutoTeachingTools</code></pre>


<pre class='language-julia'><code class='language-julia'>question(str; invite="Question") = Markdown.MD(Markdown.Admonition("tip", invite, [str]))</code></pre>
<pre id='var-question' class='code-output documenter-example-output'>question (generic function with 1 method)</pre>
<div class='manifest-versions'>
<p>Built with Julia 1.8.2 and</p>
PlutoTeachingTools 0.2.3<br>
PlutoUI 0.7.43
</div>

<!-- PlutoStaticHTML.End -->
~~~

_To run this tutorial locally, download [this file](/notebooks/week7day3.jl) and open it with
[Pluto.jl](https://plutojl.org)._


_To run this tutorial locally, download [this file](/notebooks/week7day3.jl) and open it with
[Pluto.jl](https://plutojl.org)._


_To run this tutorial locally, download [this file](/notebooks/week7day3.jl) and open it with
[Pluto.jl](https://plutojl.org)._

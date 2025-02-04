~~~
<!-- PlutoStaticHTML.Begin -->
<!--
    # This information is used for caching.
    [PlutoStaticHTML.State]
    input_sha = "7beefbe1d2588ce7a0531ea81d0d3dc65272cb3373f5399019dad122782b1cad"
    julia_version = "1.8.2"
-->

<div class="markdown"><h1>Data Science Lifecycle</h1>
<p><strong>Astro 497, Week 9, Day 1</strong></p>
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

.plutoui-toc.indent section .after-H2 a { padding-left: 10px; }
.plutoui-toc.indent section .after-H3 a { padding-left: 20px; }
.plutoui-toc.indent section .after-H4 a { padding-left: 30px; }
.plutoui-toc.indent section .after-H5 a { padding-left: 40px; }
.plutoui-toc.indent section .after-H6 a { padding-left: 50px; }

.plutoui-toc.indent section a.H1 { padding-left: 0px; }
.plutoui-toc.indent section a.H2 { padding-left: 10px; }
.plutoui-toc.indent section a.H3 { padding-left: 20px; }
.plutoui-toc.indent section a.H4 { padding-left: 30px; }
.plutoui-toc.indent section a.H5 { padding-left: 40px; }
.plutoui-toc.indent section a.H6 { padding-left: 50px; }


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



<div class="markdown"><h2>Reading questions</h2>
</div>

<pre class='language-julia'><code class='language-julia'>question(md"""
Is there one type of star that is more frequently found having exoplanets orbiting it, and if so, could that be due to selection effects as well?
""")</code></pre>
<div class="markdown"><div class="admonition is-tip">
  <header class="admonition-header">
    Question
  </header>
  <div class="admonition-body">
    <p>Is there one type of star that is more frequently found having exoplanets orbiting it, and if so, could that be due to selection effects as well?</p>
  </div>

</div>
</div>


<div class="markdown"><p>By number of known planets:</p>
<ul>
<li><p>Most planets discovered by RVs around G &amp; K type stars &#40;sweat spot for RVs&#41;</p>
</li>
<li><p>Most planets discovered by transits around G &amp; F type stars &#40;brighter&#41;</p>
</li>
</ul>
<p>By occurrence rate of planets:</p>
<ul>
<li><p>Cool stars</p>
</li>
<li><p>Metal-rich stars</p>
</li>
</ul>
<p>What biases could contribute to these apparent trends?</p>
</div>

<pre class='language-julia'><code class='language-julia'>hint(md"""
- Cooler main sequence stars have smaller masses and radii →
  - RV amplitude is larger for given planet mass
  - Transit depth is larger for given planet size
- Metal-rich stars have larger opacity in photosphere →
  - Brighter for given mass and age

- Early indication of a preference for giant planets around metal-rich stars led some RV surveys to intentionally select metal-rich stars.
  
""")</code></pre>
<div class="markdown"><div class="admonition is-hint">
  <header class="admonition-header">
    Hint
  </header>
<ul>
<li>  <div class="admonition-body">
    <p>Cooler main sequence stars have smaller masses and radii →</p>
  </div>
<ul>
<li><p>RV amplitude is larger for given planet mass</p>
</li>
<li><p>Transit depth is larger for given planet size</p>
</li>
</ul>
</li>
<li><p>Metal-rich stars have larger opacity in photosphere →</p>
<ul>
<li><p>Brighter for given mass and age</p>
</li>
</ul>
</li>
<li><p>Early indication of a preference for giant planets around metal-rich stars led some RV surveys to intentionally select metal-rich stars.</p>
</li>
</ul>

</div>
</div>

<pre class='language-julia'><code class='language-julia'>question(md"What are the selection effects for other methods of exoplanet detection?")</code></pre>
<div class="markdown"><div class="admonition is-tip">
  <header class="admonition-header">
    Question
  </header>
  <div class="admonition-body">
    <p>What are the selection effects for other methods of exoplanet detection?</p>
  </div>

</div>
</div>


<div class="markdown"><ul>
<li><p>Transit Timing Variations</p>
<ul>
<li><p>Systems near mean-motion resonances</p>
</li>
<li><p>Very closely spaced planets</p>
</li>
<li><p>TTV period short enough to see TTVs during Kepler mission </p>
</li>
<li><p>Orbital periods long enough that TTV amplitude could be detected</p>
</li>
</ul>
</li>
<li><p>Imaging</p>
<ul>
<li><p>Large orbital separations</p>
</li>
<li><p>Prefer nearly face-on orbits</p>
</li>
<li><p>Planets bright in IR → </p>
<ul>
<li><p>Nearby</p>
</li>
<li><p>Hot → Massive &amp; Young</p>
</li>
</ul>
</li>
</ul>
</li>
<li><p>Microlensing</p>
<ul>
<li><p>Sweet spot in projected angular distance</p>
</li>
<li><p>More massive planets have longer microlensing signatures</p>
</li>
</ul>
</li>
</ul>
</div>

<pre class='language-julia'><code class='language-julia'>question(md"Do rogue planets impact exoplanet selection effects or number distributions at all? ")</code></pre>
<div class="markdown"><div class="admonition is-tip">
  <header class="admonition-header">
    Question
  </header>
  <div class="admonition-body">
    <p>Do rogue planets impact exoplanet selection effects or number distributions at all? </p>
  </div>

</div>
</div>


<div class="markdown"><ul>
<li><p>Microlensing</p>
</li>
</ul>
</div>

<pre class='language-julia'><code class='language-julia'>question(md"""
Will multi-observatory transit surveys detection ability limited by "red noise"? """)</code></pre>
<div class="markdown"><div class="admonition is-tip">
  <header class="admonition-header">
    Question
  </header>
  <div class="admonition-body">
    <p>Will multi-observatory transit surveys detection ability limited by &quot;red noise&quot;? </p>
  </div>

</div>
</div>


<div class="markdown"><p>When using different telescopes from different locations, better coverage in time-domain is achieved.  However, correlated noise due to <em>atmospheric effects</em> or <em>stellar variability</em> will still affect transit survey sensitivity.</p>
</div>


<div class="markdown"><h2>Periodograms</h2>
</div>

<pre class='language-julia'><code class='language-julia'>question(md"""
Can you explain the LS periodogram?
""")</code></pre>
<div class="markdown"><div class="admonition is-tip">
  <header class="admonition-header">
    Question
  </header>
  <div class="admonition-body">
    <p>Can you explain the LS periodogram?</p>
  </div>

</div>
</div>


<div class="markdown"><p>See <a href="http://localhost:1234/edit?id&#61;47d17f80-4e22-11ed-3bed-83c8eef24851&amp;isolated_cell_id&#61;5a73b1fc-99bc-4530-ae4a-49ce25df99dc&amp;isolated_cell_id&#61;27119a64-236d-4b8f-b48a-0f4690f2a2f1">periodograms</a></p>
</div>

<pre class='language-julia'><code class='language-julia'>question(md"""
Is it possible to measure multiple periodogram power peaks that are all similar in magnitude and above the detection threshold, and what should be done in these cases?
""")</code></pre>
<div class="markdown"><div class="admonition is-tip">
  <header class="admonition-header">
    Question
  </header>
  <div class="admonition-body">
    <p>Is it possible to measure multiple periodogram power peaks that are all similar in magnitude and above the detection threshold, and what should be done in these cases?</p>
  </div>

</div>
</div>


<div class="markdown"><h2>Labs</h2>
</div>

<pre class='language-julia'><code class='language-julia'>question(md"""
Crossjoin:  What does the cartesian product of rows exactly mean?
""")</code></pre>
<div class="markdown"><div class="admonition is-tip">
  <header class="admonition-header">
    Question
  </header>
  <div class="admonition-body">
    <p>Crossjoin:  What does the cartesian product of rows exactly mean?</p>
  </div>

</div>
</div>

<pre class='language-julia'><code class='language-julia'>df1 = DataFrame(:x=&gt;1:3, :a=&gt;["a","b","c"] )</code></pre>
<table>
<tr>
<th></th>
<th>x</th>
<th>a</th>
</tr>
<tr>
<td>1</td>
<td>1</td>
<td>"a"</td>
</tr>
<tr>
<td>2</td>
<td>2</td>
<td>"b"</td>
</tr>
<tr>
<td>3</td>
<td>3</td>
<td>"c"</td>
</tr>
</table>


<pre class='language-julia'><code class='language-julia'>df2 = DataFrame(:y=&gt;10:10:30, :b=&gt;rand(3) )</code></pre>
<table>
<tr>
<th></th>
<th>y</th>
<th>b</th>
</tr>
<tr>
<td>1</td>
<td>10</td>
<td>0.0673925</td>
</tr>
<tr>
<td>2</td>
<td>20</td>
<td>0.85735</td>
</tr>
<tr>
<td>3</td>
<td>30</td>
<td>0.774101</td>
</tr>
</table>


<pre class='language-julia'><code class='language-julia'>crossjoin(df1,df2)</code></pre>
<table>
<tr>
<th></th>
<th>x</th>
<th>a</th>
<th>y</th>
<th>b</th>
</tr>
<tr>
<td>1</td>
<td>1</td>
<td>"a"</td>
<td>10</td>
<td>0.0673925</td>
</tr>
<tr>
<td>2</td>
<td>1</td>
<td>"a"</td>
<td>20</td>
<td>0.85735</td>
</tr>
<tr>
<td>3</td>
<td>1</td>
<td>"a"</td>
<td>30</td>
<td>0.774101</td>
</tr>
<tr>
<td>4</td>
<td>2</td>
<td>"b"</td>
<td>10</td>
<td>0.0673925</td>
</tr>
<tr>
<td>5</td>
<td>2</td>
<td>"b"</td>
<td>20</td>
<td>0.85735</td>
</tr>
<tr>
<td>6</td>
<td>2</td>
<td>"b"</td>
<td>30</td>
<td>0.774101</td>
</tr>
<tr>
<td>7</td>
<td>3</td>
<td>"c"</td>
<td>10</td>
<td>0.0673925</td>
</tr>
<tr>
<td>8</td>
<td>3</td>
<td>"c"</td>
<td>20</td>
<td>0.85735</td>
</tr>
<tr>
<td>9</td>
<td>3</td>
<td>"c"</td>
<td>30</td>
<td>0.774101</td>
</tr>
</table>



<div class="markdown"><h2>File Formats</h2>
<h3>What type of data does it store?</h3>
<ul>
<li><p>Text</p>
</li>
<li><p>Documents</p>
</li>
<li><p>Numerical values</p>
</li>
<li><p>Time-series</p>
</li>
<li><p>Images</p>
</li>
<li><p>Data cubes</p>
</li>
</ul>
<h3>Very common file formats</h3>
<ul>
<li><p>Text &#40;ASCII or Unicode&#41;</p>
<ul>
<li><p>Delimited &#40;e.g., CSV, TSV&#41;</p>
</li>
<li><p>Fixed-width &#40;e.g, AAS machine-readable tables&#41;</p>
</li>
<li><p>Markup languages &#40;e.g., html, xml, toml, yaml,...&#41;</p>
</li>
</ul>
</li>
<li><p>Binary</p>
<ul>
<li><p>FITS: Standard for astronomical observations</p>
</li>
<li><p>HDF5: Standard for numerical simulations</p>
</li>
</ul>
</li>
</ul>
<h3>Key questions to ask when choosing a file format</h3>
<ul>
<li><p>How big is the dataset?</p>
</li>
<li><p>Will users want to read all data at once or small pieces of data?</p>
</li>
<li><p>Is dataset highly structured &#40;e.g., large tables or images&#41;?</p>
</li>
<li><p>Is it important to include machine-readable metadata?</p>
</li>
<li><p>Does it make sense to compress data?</p>
</li>
</ul>
<h3>What not to use</h3>
<ul>
<li><p>Your own custom binary file format</p>
</li>
<li><p>File formats that depend on versions of your softare &#40;e.g., pickle&#41;</p>
</li>
<li><p>Markup languages for highly structured data </p>
</li>
<li><p>Text formats for large datasets</p>
</li>
</ul>
</div>


<div class="markdown"><h1>Data Science Lifecycle</h1>
</div>


<div class="markdown"><h2>Example of a Data Science Lifecycle</h2>
<p>&#40;This is just one of many.&#41;</p>
<ol>
<li><p>Ask an interesting question</p>
<ul>
<li><p>What is the scientific goal?</p>
</li>
<li><p>What would you do if you had all the data?</p>
</li>
<li><p>What do you want to predict or estimate?</p>
</li>
</ul>
</li>
<li><p>Get the data</p>
<ul>
<li><p>How were the data sampled?</p>
</li>
<li><p>Which data are relevant?</p>
</li>
<li><p>Are there privacy issues?</p>
</li>
</ul>
</li>
<li><p>Explore the data</p>
<ul>
<li><p>Plot the data.</p>
</li>
<li><p>Are there anomalies?</p>
</li>
<li><p>Are there patterns?</p>
</li>
</ul>
</li>
<li><p>Model the data</p>
<ul>
<li><p>Build a model.</p>
</li>
<li><p>Fit the model.</p>
</li>
<li><p>Validate the model.</p>
</li>
</ul>
</li>
<li><p>Communicate and visualize the results</p>
<ul>
<li><p>What did we learn?</p>
</li>
<li><p>Do the results make sense?</p>
</li>
<li><p>Can we tell a story?</p>
</li>
</ul>
</li>
</ol>
<p>–- Blitzstein &amp; Pfister for <a href="https://cs109.github.io/2015/">Harvard CS109</a></p>
</div>


<div class="markdown"><h4>What&#39;s missing?</h4>
</div>

<pre class='language-julia'><code class='language-julia'>hint(md"""
- Making iterative process/loops explicit
- Interpreting results for oneself
- Deploying model to work for future data
""")</code></pre>
<div class="markdown"><div class="admonition is-hint">
  <header class="admonition-header">
    Hint
  </header>
<ul>
<li>  <div class="admonition-body">
    <p>Making iterative process/loops explicit</p>
  </div>
</li>
<li><p>Interpreting results for oneself</p>
</li>
<li><p>Deploying model to work for future data</p>
</li>
</ul>

</div>
</div>


<div class="markdown"><h3>Some workflows common in industry</h3>
<h4>OSEMN</h4>
<ul>
<li><p>Obtain</p>
</li>
<li><p>Scrub</p>
</li>
<li><p>Explore</p>
</li>
<li><p>Model</p>
</li>
<li><p>iNterpret</p>
</li>
</ul>
<h4>CRISP-DM</h4>
<ul>
<li><p>Business Understanding</p>
</li>
<li><p>Data Understanding</p>
</li>
<li><p>Data Preparation</p>
</li>
<li><p>Modeling</p>
</li>
<li><p>Evaluation</p>
</li>
<li><p>Deployment</p>
</li>
</ul>
<p>Emphasizes loops and deployment</p>
<h4>Team Data Science Process &#40;TDSP&#41;</h4>
<p>Combines a workflow with project templates and recommendations for infrastructure and tools.  Favors MS products.</p>
</div>


<div class="markdown"><h4>Domino’s data science life cycle is founded on three guiding principles:</h4>
<ol>
<li><p>Ideation</p>
</li>
<li><p>Data Acquisition and Exploration</p>
</li>
<li><p>Research &amp; Development</p>
</li>
<li><p>Validation</p>
</li>
<li><p>Delivery</p>
</li>
<li><p>Monitoring</p>
</li>
</ol>
<p>Emphasizes frequest iteration, collaboration and reproducibility.</p>
</div>


<div class="markdown"><h2>Adapting Data Science Workflows from Industry to Scientific Setting</h2>
<ul>
<li><p>Reinterpret terms like &quot;business case&quot; and &quot;customer&quot;</p>
</li>
<li><p>Often don&#39;t know to quantify success when we start a project</p>
</li>
<li><p>Generally, place more value on interpretability </p>
</li>
<li><p>Can accommodate projects requiring longer timescales</p>
</li>
<li><p>Increasingly, plan to make data &amp; codes with public</p>
</li>
<li><p>Often communication is primarily with other scientists</p>
</li>
</ul>
</div>


<div class="markdown"><h1>ICDS Fall 2022 Symposium</h1>
<h4>Data Science, AI, and a Sustainable, Resilient, and Equitable Future</h4>
<h2>Keynote speaker: danah boyd</h2>
<h4>Partner Researcher at Microsoft Research, the founder of Data &amp; Society and a Distinguished Visiting Professor at Georgetown University</h4>
<p>Quotes that stood out to me.</p>
<ul>
<li><p>&quot;I found that the people who ascribe the most power to statistics and data are not people who do statistics and data science.  They are executives who give the vision talks about the power of data...&quot; - Jeff Hammerbacher &#40;2016&#41;, former lead of Data Science at Facebook</p>
</li>
<li><p>&quot;Performing math is different than doing math.&quot;  &#40;in context of redistrictors making it look like yore being objective&#41;</p>
</li>
<li><p>&quot;You&#39;re not making true claims.  You&#39;re making invitations for inquiry.&quot; &#40;in context of redistricting&#41;</p>
</li>
<li><p>&quot;It&#39;s easier for me to agree with the model&quot; &#40;in context of career risk&#41;</p>
</li>
</ul>
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



<pre class='language-julia'><code class='language-julia'>begin
using PlutoUI, PlutoTeachingTools, HypertextLiteral
using DataFrames
end</code></pre>


<pre class='language-julia'><code class='language-julia'>question(str; invite="Question") = Markdown.MD(Markdown.Admonition("tip", invite, [str]))</code></pre>
<pre id='var-question' class='code-output documenter-example-output'>question (generic function with 1 method)</pre>
<div class='manifest-versions'>
<p>Built with Julia 1.8.2 and</p>
DataFrames 1.4.1<br>
HypertextLiteral 0.9.4<br>
PlutoTeachingTools 0.2.3<br>
PlutoUI 0.7.44
</div>

<!-- PlutoStaticHTML.End -->
~~~

_To run this tutorial locally, download [this file](/notebooks/week9day1.jl) and open it with
[Pluto.jl](https://plutojl.org)._


_To run this tutorial locally, download [this file](/notebooks/week9day1.jl) and open it with
[Pluto.jl](https://plutojl.org)._


_To run this tutorial locally, download [this file](/notebooks/week9day1.jl) and open it with
[Pluto.jl](https://plutojl.org)._

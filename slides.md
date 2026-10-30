---
theme: default
title: Building the Norwegian Railway Knowledge Graph
info: |
  ## Building the Norwegian Railway Knowledge Graph
  Where the Semantic Web stack runs off the track.
  Connected Data London 2026 · 12 November 2026.
base: /
routerMode: hash
colorSchema: dark
layout: cover
class: cdl-cover
background: /assets/dim-background.png
transition: slide-left
mdc: true
highlighter: shiki
lineNumbers: false
seoMeta:
  ogTitle: Building the Norwegian Railway Knowledge Graph
  ogDescription: A production railway knowledge graph at Bane NOR, and where the Semantic Web stack runs off the track.
drawings:
  persist: false
---

<div class="cdl-cover-inner">

<div class="flex items-center gap-3 mb-6">
  <img src="/assets/matdata.svg" class="h-9 w-9" alt="Matdata logo" />
  <span class="text-lg font-bold">Matdata &</span>
  <img src="/assets/banenor-logo-inverted.svg" class="h-9" alt="Bane NOR logo" />
</div>

<div class="kicker">Connected Data London 2026</div>

# Building the Norwegian Railway Knowledge Graph

<div class="cdl-sub">where the Semantic Web stack runs off the track</div>

<div class="mt-8 opacity-90 text-sm">
Mathias Vanden Auweele · Matdata<br>
with the Bane NOR Digital Infrastructure Model
</div>

</div>

<CoverTrack />

<!--
[0:15] Hi, I'm Mathias. For the next half hour I'll take you on a train ride through a railway knowledge graph that runs in production in Norway: what it does for the business, what we got wrong building it, and how we fixed it.
-->

---
layout: two-cols-header
station: Start
---

# Who is behind this?

::left::

<div class="pr-6 text-sm">

<div style="display:flex;align-items:center;gap:0.6rem;margin-bottom:0.4rem">
  <img src="/assets/matdata.svg" style="height:2.2rem;width:2.2rem" alt="Matdata logo" />
  <span style="font-size:1.4rem;font-weight:700">Matdata</span>
</div>

A **one-person data consultancy** for semantic mapping and knowledge graphs in rail.

- Working with **ERA**, **Bane NOR** and **Certifer**
- railML, RTM / RSM, OpenTNF and vendor topology models
- <b>Role here:</b> semantic modelling and pipeline partner

<div class="logos">
  <img src="/assets/era.svg" style="height:1.5rem" alt="ERA" />
  <img src="/assets/certifer.jpg" style="height:1.1rem" alt="Certifer" />
  <img src="/assets/Infrabel_logo.svg" style="height:1.1rem" alt="Infrabel" />
  <span style="display:inline-flex;align-items:center;gap:.3rem;color:#1d3b2a;font-weight:700;font-size:.8rem"><img src="/assets/datatreehouse.svg" style="height:1.1rem" alt="" />Datatreehouse</span>
  <img src="/assets/Kapernikov_logo_horiz.png" style="height:1.1rem" alt="Kapernikov" />
  <img src="/assets/cropped-cognizone_logo-2048x529.png" style="height:1.1rem" alt="Cognizone" />
  <img src="/assets/liw-member-5081.png" style="height:1.1rem" alt="Transurb" />
</div>

</div>

::right::

<div class="pl-6 border-l border-gray-400/40 text-sm">

<img src="/assets/banenor-logo-inverted.svg" style="height:3.4rem;margin:-0.6rem 0 0.1rem" alt="Bane NOR logo" />

Norway's **national railway infrastructure manager**.

- **5,000+ km** of track, signals, switches, bridges, tunnels, catenary
- Builds, maintains, renews and **plans decades ahead**
- Owner of the **Digital Infrastructure Model (DIM)**, the graph in this talk

</div>

<style>
.logos { display: flex; flex-wrap: wrap; align-items: center; gap: .6rem 1.2rem; margin-top: .9rem; padding: .6rem .8rem; background: #fff; border-radius: 10px; }
</style>

<!--
[0:40] Two organisations. Matdata brings the semantic web know-how; I'm a one-person consultancy working for ERA, Bane NOR and others. Bane NOR owns the railway, the data and the regulatory obligations. Everything you'll see is their production system, the DIM.
-->

---

# Open-source tools that grew out of this work

<div class="grid grid-cols-3 gap-3 mt-2">
<div class="oss glass"><b>MatGUI + Yasgui plugins</b><span>matgui.matdata.eu</span><p>SPARQL workbench with table, graph, map, chart and explain plugins.</p></div>
<div class="oss glass"><b>SHACL Report</b><span>shacl-report.matdata.eu</span><p>Readable validation reports, and the coverage of your shapes.</p></div>
<div class="oss glass"><b>RDF Rules Playground</b><span>rdf-rules-playground.matdata.eu</span><p>Run SHACL-AF rules in the browser; see which rule inferred each triple.</p></div>
<div class="oss glass"><b>RINF Atlas</b><span>rinf-atlas.matdata.eu</span><p>Map and faceted search over the EU infrastructure register.</p></div>
<div class="oss glass"><b>SPARQL Guard</b><span>sparql-guard</span><p>Access control for any SPARQL endpoint, policies in SHACL.</p></div>
<div class="oss glass"><b>RDF Compare · Explorers</b><span>rdf-compare · ontology-explorer</span><p>Diff two RDF files; ontologies and shapes as diagrams.</p></div>
</div>

<div class="mt-4 text-sm muted">All on <b>github.com/Matdata-eu</b>, most of them running in the browser.</div>

<style>
.oss { padding: 10px 12px; border-top: 3px solid var(--c-sky); }
.oss b { display: block; font-size: 14px; color: #fff; }
.oss span { font-family: ui-monospace, monospace; font-size: 10.5px; color: var(--c-sky); }
.oss p { font-size: 12px; line-height: 1.35; margin: 4px 0 0; color: var(--c-ink-mod); }
</style>

<!--
[0:25] A quick word on tools: many open-source tools came out of this project, a SPARQL workbench, a SHACL report viewer, a rules playground, RINF Atlas, and more. All on GitHub. Now, the railway.
-->

---

<ShatteredRailway :step="$clicks" />

<div v-click="3" />

<!--
[1:30] I believe the railway is the most fantastic environment there is. I don't know any other industry that covers this many distinct concepts and niche specialities in one system: track, switches, signalling, ETCS, catenary and substations, bridges and tunnels, timetables, rolling stock, possessions, work orders. And they are all connected.
(click) Then computers came along. And we took this beautiful, complete model, dropped it on the floor, and it broke into a million pieces.
(click) We picked up the pieces that suited us, glued them together in our own vision, and called them use cases. Maintenance, timetabling, ERTMS, energy, projects. Look at how many of them carry their own copy of the track, each a little different. Sometimes so different you'd think they describe different railways. And a change in one never reaches the others.
(click) The DIM project is about restoring the complete picture: one model of the railway, as a knowledge graph, where every use case is a view on the same reality.
-->

---
layout: section
station: Production
---

<div class="kicker">Part 1</div>

# It runs in production

<!--
[0:05] First: is this real?
-->

---

# Proof of concept, pilot, or production?

<MaturityLadder />

<!--
[1:15] When someone at a conference shows you a knowledge graph, ask: is it a proof of concept, a pilot, or production? It makes a big difference.
Railway is a slow business; processes inside an infrastructure manager don't change overnight. But DIM is in production: others depend on it.
The ERTMS programme, the European train control system roll-out, uses DIM as its starting point to engineer ETCS on the lines.
DIM is the chosen successor for supplying infrastructure data to traffic management, traffic planning and timetabling. That makes it a foundation.
And next, DIM stops being only a copy built from the sources: it becomes the master of the topology itself.
-->

---

# RINF: Europe's register of railway infrastructure

<div class="grid grid-cols-5 gap-6 mt-1">

<div class="col-span-2 text-sm leading-relaxed">

- An **EU obligation**: every infrastructure manager publishes its network to the **Register of Infrastructure**, run by the EU Agency for Railways (ERA).
- Railway undertakings use it to check **whether a train fits a route** before it runs: gauge, electrification, platform length, train protection, …
- ERA also publishes it as **linked data**, with the ERA ontology.

<div v-click class="glass p-3 mt-4 glow-sky">
Norway sent the <b class="hl-sky">first micro-level submission</b>, generated straight from the knowledge graph.
</div>

</div>

<div class="col-span-3">
  <div class="flex gap-2 mb-1 text-xs">
    <span class="lvl" :class="{ on: $clicks < 1 }">Macro: operational points + sections of line</span>
    <span class="lvl" :class="{ on: $clicks >= 1 }">Micro: the topology</span>
  </div>
  <RinfMacroMicro :step="$clicks" />
  <div class="text-xs muted text-right">schematic</div>
</div>

</div>

<style>
.lvl { padding: 2px 10px; border-radius: 99px; border: 1px solid var(--c-stroke); color: var(--c-ink-mod); transition: all .4s; }
.lvl.on { border-color: var(--c-sky); color: var(--c-sky); background: rgba(100, 200, 250, .1); }
</style>

<!--
[1:15] A word on RINF, because it matters for the business case. It's the EU Register of Infrastructure. Every infrastructure manager in Europe must publish its network there, and railway undertakings use it to check whether a train can run on a route: the gauge, the electrification, platform lengths, the train protection system.
Most countries report at macro level: operational points, so stations, connected by sections of line.
(click) Norway went further. Because DIM is a topology, we could send the first ever submission at micro level: real tracks, switches, signals and platforms, positioned on the network. And it's not a parallel dataset that drifts: it is generated from the graph.
-->

---

# Who relies on it today

<ConsumersHub />

<!--
[1:00] So who depends on it? ERTMS, for ETCS engineering. Strategic traffic planners: run-time calculations and timetable simulation on the network as it is modelled in DIM. RINF, the EU register. Railway undertakings: platform lengths, so they know which trains fit and can tell passengers where to board.
And the big one coming: traffic management and timetabling will get their infrastructure data from DIM.
-->

---

# What is inside

<div class="grid grid-cols-4 gap-4 mt-6">
  <div class="stat glass"><div class="n hl-sky"><CountUp :to="13" suffix="M" /></div><div class="l">triples</div></div>
  <div class="stat glass"><div class="n hl-green"><CountUp :to="17" :delay="150" /></div><div class="l">ontologies</div></div>
  <div class="stat glass"><div class="n hl-amber"><CountUp :to="5000" :delay="300" suffix="+" /></div><div class="l">km of track</div></div>
  <div class="stat glass"><div class="n hl-purple"><CountUp :to="40" :delay="450" /></div><div class="l">asset types</div></div>
</div>

<div class="mt-10 text-center">
  <div class="big-quote" style="font-size:1.5rem"><span class="hl-sky">Topology-first</span>: every asset sits on the network graph, not just on a map.</div>
</div>

<div class="mt-5 text-sm text-center muted">
Outputs: railML 3.2 / 3.3 · SPARQL endpoint · JSON-LD REST API · RINF · datasets in RDF, JSON-LD and CSV
</div>

<style>
.stat { padding: 18px 10px; text-align: center; }
.stat .n { font-family: 'Poppins', sans-serif; font-size: 2.6rem; font-weight: 700; line-height: 1; text-shadow: 0 0 22px currentColor; }
.stat .l { font-size: 14px; color: var(--c-ink-mod); margin-top: 6px; }
</style>

<!--
[0:45] The numbers: about 13 million triples, 17 ontologies, more than five thousand kilometres of track, about 40 asset types. And one property that will matter later: it is topology-first. Every signal, every balise is positioned on the network graph, not just as a point on a map.
-->

---
station: Value
---

# The value tour

<ValueTour />

<!--
[0:20] So what does a knowledge graph give the business? Eight stops on this line, all running on the same graph: the portal, key figures, the RINF status, the search builder, the data catalog, the TPS mapping, the validation portal and the REST API.
-->

---
layout: full
---

<Clip full src="/assets/01-website-navigation.gif" caption="DIM portal · railML files per subnetwork · datasets in RDF, JSON-LD and CSV" />

<!--
[0:25] Let me show you the value. This is the DIM portal; everything on it comes from the knowledge graph: railML files per subnetwork, and the datasets in RDF, JSON-LD and CSV.
-->

---
layout: full
---

<Clip full src="/assets/02-key-figures.gif" caption="Key figures, computed from the graph" />

<!--
[0:15] Key figures, computed straight from the graph. No spreadsheet, no manual count.
-->

---
layout: full
---

<Clip full src="/assets/03-rinf-status-report.gif" caption="RINF status report · which RINF parameters we fill, and which are still missing" />

<!--
[0:20] The RINF status report: for every RINF parameter, whether we already fill it from the graph, or it's still missing.
-->

---
layout: full
---

<Clip full src="/assets/04-sparnatural-search.gif" caption="Search builder on Sparnatural: point-and-click SPARQL" />

<!--
[0:20] The search builder: people build SPARQL queries by clicking, without knowing SPARQL.
-->

---
layout: full
---

<Clip full src="/assets/05-data-catalog.gif" caption="Data catalog of what each dataset contains" />

<!--
[0:20] The data catalog documents what each dataset contains and where it comes from.
-->

---
layout: full
---

<Clip full src="/assets/06-tps-mapping.gif" caption="TPS mapping · DIM netelements linked to the timetable-planning topology, both maps in sync" />

<!--
[0:25] Linking to other systems: the topology of the timetable-planning system mapped onto ours. Hover a net element on our map and its counterpart lights up in their schematic.
-->

---
layout: full
---

<Clip full src="/assets/07-data-validation-portal.gif" caption="Data validation portal · SHACL results per run, quality score and progress over time" />

<!--
[0:20] And the data validation portal: SHACL results on every run, a quality score, and how the findings evolve. Data quality becomes visible, and the source owners can act on it.
-->

---

# A REST API that is secretly a knowledge graph

<div class="grid grid-cols-2 gap-6 mt-2">

<img src="/assets/dim-api.png" class="rounded-lg border border-gray-400/30 max-h-80 object-contain bg-white" alt="DIM REST API documentation" />

<div class="text-base">

<v-clicks>

- A **conventional REST API**, documented with **Swagger / OpenAPI**
- Responses are **JSON-LD**
- Built **from the knowledge graph**, with some indexing in the backend
- Developers get the JSON they expect; **the data stays linked**

</v-clicks>

</div>

</div>

<!--
[0:50] Most developers don't want SPARQL. So we offer a conventional REST API with Swagger. (click) The responses are JSON-LD, (click) built from the knowledge graph with a bit of indexing so it is fast. (click) People work with the graph the way they're used to, and the data stays linked.
-->

---
layout: section
station: Topology
---

<div class="kicker">Part 2</div>

# Get the model right first

<!--
[0:05] Before any pipeline: the model.
-->

---

# A hierarchy is not a network

<HierarchyVsTopology :step="$clicks" />

<div v-click="1" />

<!--
[1:30] Many asset databases, built on SAP or Maximo, but also RDS and the old railML 2 versions, model the railway as a hierarchy: a line has stations, a station has tracks, a track has switches and signals. That's great for maintenance work orders.
(click) But ask the questions an operator or a planner asks. Which station does this switch belong to? The tree knows. Can a train get from track 1 to track 3? The tree has no idea; and notice that switch 12 hangs under one track, while in reality it connects three. Which signal protects this switch? How far is it from the signal to the platform? Only the topology answers those.
And geometry doesn't save you: a signal can physically stand closer to the wrong track. Geometry lies; topology doesn't.
So your initial data model must be solid: topology first, then hang everything on it.
-->

---

# A minute of topology

<TopoShortcut :step="$clicks" />

<div v-click="3" />

<!--
[1:00] One minute of railway topology, in RailTopoModel terms.
The network is a graph of net elements, pieces of track with a direction from 0 to 1, connected by net relations that say where a train can go.
(click) A station, an operational point, is an area location: it lists the net elements it covers.
(click) A signal is a spot location: one net element and an intrinsic coordinate, e2 at 0.8. No coordinates needed.
(click) And the question every user asks: in which station is this signal? In RDF that is five hops.
-->

---

# Enrich for the user: topology shortcuts

<div class="grid grid-cols-5 gap-6">

<div class="col-span-3">

```turtle
bnd:_inference_netentity_inOperationalPoint
  a sh:SPARQLRule ;
  sh:order 3 ;
  sh:construct """
    CONSTRUCT { $this bno:inOperationalPoint ?op . }
    WHERE {
      ?op a railml3:OperationalPoint ;
        rtm:hasAreaLocation/rtm:hasAssociatedNetElement
          /rtm:onNetElement ?netElement .
      { $this rtm:hasSpotLocation/rtm:onNetElement ?netElement }
      UNION
      { $this rtm:hasLinearLocation/rtm:hasAssociatedNetElement
          /rtm:onNetElement ?netElement }
    }""" .
```

</div>

<div class="col-span-2 text-sm">

**In which station is this signal?** In RailTopoModel that is a **5–6 hop** path.

<div class="glass p-3 mt-3" style="border-top:3px solid var(--c-purple)">
SHACL rules turn the topology into <b class="hl-purple">one-hop shortcuts</b>:<br>
<code>bno:inOperationalPoint</code> · <code>bno:onRailwayLine</code> · <code>bno:onRailwayTrack</code> · <code>bno:onRailwayLineSection</code>
</div>

<div class="mt-3 text-xs muted">
Computed once per build, used by the datasets, the search and the explorer. Users query one hop; the topology stays the truth.
</div>

</div>

</div>

<!--
[0:50] Nobody should have to write that five-hop query. So a SHACL rule computes it once per build and adds a one-hop shortcut: in operational point, on railway line, on railway track. The datasets, the search and the explorer all read those shortcuts. That's the payoff of a solid topology: you can infer what the users need, and the topology stays the source of truth.
-->

---
layout: section
station: Mapping
---

<div class="kicker">Part 3</div>

# How we built it,<br>and what went wrong

<!--
[0:05] Now the honest part: how we built it.
-->

---

# Why SPARQL Anything

<div class="big-quote mt-8 max-w-3xl">
<span class="hl-sky">One language.</span> The team was new to knowledge graphs and had to learn SPARQL anyway.
</div>

<div class="grid grid-cols-3 gap-4 mt-10 text-sm">
<div class="glass p-4">No R2RML, RML or custom code: a mapping <b>is</b> a SPARQL <code>CONSTRUCT</code>.</div>
<div class="glass p-4">Façade-X gives one lens over CSV, XML, JSON, spreadsheets and more.</div>
<div class="glass p-4">Written and tested in the same query editor the team already uses.</div>
</div>

<!--
[0:40] We chose SPARQL Anything because the team was new to knowledge graphs. They had to learn SPARQL to query the graph anyway, so why learn R2RML or RML on top? One language for mapping and querying. The tool was never the problem. How we used it was.
-->

---

# What we did first: straight into the target

<div class="grid grid-cols-5 gap-6">

<div class="col-span-3">

```sparql
CONSTRUCT {
  ?balise a railml3:Balise, rtm:NetEntity ;
    rdfs:label ?label ;
    rtm:hasSpotLocation ?loc .
  ?loc a rtm:SpotLocation ;
    rtm:onNetElement ?netelement ;
    rtm:hasIntrinsicCoordinate ?coord .
}
WHERE {
  SERVICE <x-sparql-anything:location=
      balises.csv,csv.headers=true> {
    ?s xyz:atb_banedata_id ?id ;
       xyz:name ?label ;
       xyz:measure_on_link ?coord .
  }
  BIND(IRI(CONCAT(STR(bnd:),
      "_balise_", STR(?id))) AS ?balise)
}
```

</div>

<div class="col-span-2 text-sm">

- One `CONSTRUCT` per source file
- Reads the CSV through Façade-X…
- …and writes **railML / RTM** directly: source and target in **one step**

<div v-click class="glass p-3 mt-6" style="border-left:4px solid var(--c-amber)">
Fine for one file. But real mappings span files, need geometry, and need fixes to the source data.
</div>

</div>

</div>

<!--
[0:40] This is how we started: one CONSTRUCT per source file. Read the CSV, write railML and RTM triples. Source to target in one step.
(click) Fine for one file. But real mappings span files, need geometry, and need fixes in the source data.
-->

---

# One mapping file per source file

<CopyPasteMaps />

<div class="text-xs muted mt-1">The same happens with R2RML or RML: one mapping per table or file, and the shared modelling copied into each.</div>

<!--
[1:00] And this is what "one CONSTRUCT per source file" turns into. Each bar is a mapping file; the coloured part are lines that also appear in another mapping file. Validity periods, designators, labels, the network level, geometry and positioning: the same blocks, copy-pasted.
Over all 71 mapping files, 63 percent of the lines exist somewhere else too. Fix a bug in how we model validity, and you fix it in 45 files.
And this is not a SPARQL Anything problem: with R2RML or RML you get exactly the same, a mapping per table, with the shared modelling copied into each one.
-->

---
overpass: true
---

# The mess that followed

<MessPipeline :step="$clicks" />

<div v-click="3" />

<!--
[1:30] Here's what grew around that simple CONSTRUCT. The source is a GeoPackage, plus XML and JSON. SPARQL Anything doesn't read databases, so a Source Ingestor prepares the files.
(click) Data quality issues take months to fix at the source, so we patch them in SQL: about a hundred patches, and they must run before anything else reads the data.
(click) Then geometry. GeoSPARQL is geometry-centric: no linear referencing, no routing, not even a function to get the X coordinate. And engines don't implement the full spec. So spatial logic went into SQL views and six Python steps working on the graph.
(click) And because mappings span files, 61 post-processing UPDATEs run in a fixed order, deleting and rewriting each other. That is an imperative program written in SPARQL. Nobody can tell you which rule produced which triple.
-->

---

# What it costs

<div class="grid grid-cols-5 gap-8 mt-2">

<div class="col-span-3">

<TimeBars :rows="[
  { label: 'Source ingestion', min: 3.5, tone: 'gray', note: 'SpatiaLite → CSV' },
  { label: 'SPARQL Anything mapping', min: 10, tone: 'sky' },
  { label: 'Post-processing', min: 45, tone: 'amber', note: '61 UPDATEs' },
  { label: 'Splitting', min: 90, tone: 'red', note: '15 sequential queries' },
  { label: 'railML generation', min: 15, tone: 'purple' },
  { label: 'RINF, SHACL rules in maplib', min: 10, tone: 'green' },
  { label: 'RINF, same rules in TopBraid', min: 120, tone: 'red' },
]" :stagger="300" />

<div class="text-xs muted mt-2">Full production run, one worker (1–4 vCPU, 28–32 GiB). The two RINF rows are alternatives.</div>

</div>

<div class="col-span-2 flex flex-col justify-center gap-3 text-sm">
<div class="glass p-3" style="border-left:4px solid var(--c-sky)"><b class="hl-sky">The mapping is not the bottleneck.</b> Triplifying everything takes 10 minutes.</div>
<div class="glass p-3" style="border-left:4px solid var(--c-red)"><b class="hl-red">The fix-up layers are.</b> Post-processing and splitting take over two hours.</div>
<div class="glass p-3" style="border-left:4px solid var(--c-green)"><b class="hl-green">The engine matters.</b> The same SHACL rules: 10 minutes or 2 hours.</div>
</div>

</div>

<!--
[0:50] What does it cost? The mapping itself takes ten minutes. It's not the problem. The fix-up layers are: post-processing and splitting take over two hours. And the engine matters: the same SHACL rule set runs in 10 minutes in maplib versus 2 hours in TopBraid. An order of magnitude, just by choosing the engine.
-->

---
layout: section
station: Pipeline
---

<div class="kicker">Part 4</div>

# Before and after

<!--
[0:05] So we rebuilt it. Two refactors, before and after.
-->

---

# Lift first, in the language of the source

<LiftUplift :step="$clicks" />

<div v-click="1" />

<!--
[1:40] Before: one CONSTRUCT per file went straight from the source into railML and RTM, 63 percent copy-paste, and then 61 ordered updates to fix it up.
(click) After: two steps. First we lift every source mechanically into RDF, in its own namespace: trase for the Trasé network database, dimdb for the DIM database. One generic query per table: every column becomes a property. No interpretation, nothing lost, nothing guessed. The graph speaks the source's language first, with the source's concepts.
Then the uplift: SHACL shapes that target a source class. They validate the source data, and they carry the rule that builds the DIM entity. Shared logic like validity and designators sits in one common shape instead of in 45 files. And every DIM entity says which source version it was derived from.
That's the message I want you to take home: describe the source first, then transform with rules.
-->

---

# Lift every table, uplift every concept

<div class="grid grid-cols-2 gap-5 small-code">

<div>
<div class="kicker mb-1">Lift · mechanical</div>

```sparql
CONSTRUCT {
  ?version a trase:Plattformkant ;
    ?property ?value .
} WHERE {
  SERVICE <x-sparql-anything:location=
      plattformkant.csv,csv.headers=true> {
    ?row xyz:property_oid ?oid ;
      ?column ?value .
  }
  BIND(fx:entity(trase:, ?oid) AS ?version)
  BIND(fx:entity(trase:, STRAFTER(
      STR(?column), STR(xyz:))) AS ?property)
}
```

</div>

<div>
<div class="kicker mb-1">Uplift · a SHACL shape with a rule</div>

```turtle {all|1-4|6-10|12-25}
bnd:_upliftShape_trase_Plattformkant a sh:NodeShape ;
  sh:targetClass trase:Plattformkant ;
  sh:property bnd:_upliftShape_trase_Plattformkant_lengde ;
  sh:rule bnd:_uplift_trase_Plattformkant_lengde .

bnd:_upliftShape_trase_Plattformkant_lengde
  a sh:PropertyShape ;
  sh:path trase:lengde ;
  sh:maxCount 1 ;
  sh:pattern "^[0-9]+(\\.[0-9]+)?$" .

bnd:_uplift_trase_Plattformkant_lengde a sh:SPARQLRule ;
  sh:construct """CONSTRUCT {
      ?edge railml3:hasLength ?length ;
        prov:wasDerivedFrom $this .
      ?length qudt:numericValue ?value ;
        qudt:hasUnit unit:M .
    } WHERE {
      $this trase:lengde ?lengde .
      BIND(STRAFTER(STR($this), STR(trase:)) AS ?oid)
      BIND(IRI(CONCAT(STR(bnd:), "_platform_edge_", ?oid)) AS ?edge)
      BIND(IRI(CONCAT(STR(bnd:), "_length_", ?oid)) AS ?length)
      BIND(STRDT(?lengde, xsd:decimal) AS ?value)
    }""" .
```

</div>

</div>


<!--
[1:00] What it looks like. On the left the lift: the same query shape for every table. Every row becomes a trase entity, every column a trase property. No railML anywhere.
On the right the uplift. (click) A node shape targets the source class, a Trasé platform edge. (click) A property shape validates the source: at most one length, and it must be a number. (click) And the rule turns that length into the railML length of the platform edge, with prov:wasDerivedFrom pointing back to the source entity. The body is still SPARQL, so the team didn't have to learn a new language.
-->

---

# railML generation: stop splitting, start scoping

<RailmlBeforeAfter :step="$clicks" />

<div v-click="1" />

<!--
[1:30] The second refactor is about looking at the whole pipeline instead of optimising one query.
Before: we split the complete graph into one graph per subnetwork with 15 sequential queries, an hour and a half, and then generated railML per subnetwork with rdflib in memory, which ran out of memory on the big ones. And you could only get the subnetworks we had pre-defined.
(click) After: no splitting. We load the complete knowledge graph into Oxigraph, an efficient SPARQL engine, in memory, in 36 seconds. An area location defines the scope: a predefined subnetwork, or any area. Both railML versions for the complete network take under five minutes. We save about an hour and a half on every run.
-->

---

# A library, not a pipeline step

<div class="grid grid-cols-5 gap-6 mt-1">

<div class="col-span-3">
  <RailmlMap />
</div>

<div class="col-span-2 text-sm">

<v-clicks>

- The generator is now a **standalone Python library**: <code>rdf2railml</code>
- No Airflow, no file-system assumptions: give it **a graph and an area**
- Airflow is just one caller. A map where you **draw the area you need** is another
- Checked line by line against the old generator before the switch

</v-clicks>

</div>

</div>

<!--
[1:00] And there's a bonus. Because the generator no longer depends on pre-processing, it became an independent library. Give it a knowledge graph and an area location, and you get railML.
(click) Airflow is just one caller. You can draw on the map the area you need, and get exactly that area as a railML file. A project team can get the railML for their construction site without waiting for anyone.
(click) We checked the output line by line against the old generator before we switched.
-->

---
layout: center
class: text-center
station: Q&A
---

# Takeaways

<div class="mt-4 max-w-3xl mx-auto text-left text-lg leading-relaxed space-y-3">
<div v-click class="tkw glass"><i class="num sky">1</i>A knowledge graph <b class="hl-sky">can run production systems</b>: DIM feeds ETCS, traffic planning and RINF.</div>
<div v-click class="tkw glass"><i class="num amber">2</i><b class="hl-amber">Topology first.</b> A hierarchy of assets can't tell you where a train can go.</div>
<div v-click class="tkw glass"><i class="num green">3</i><b class="hl-green">Lift the source first, then uplift with rules.</b> Don't map straight into your target.</div>
<div v-click class="tkw glass"><i class="num purple">4</i><b class="hl-purple">Optimise the system, not the query.</b> And GeoSPARQL still needs linear referencing.</div>
</div>

<div class="mt-5 flex justify-center gap-10">
  <div class="text-center">
    <img src="https://api.qrserver.com/v1/create-qr-code/?size=180x180&data=https%3A%2F%2Fcdl2026.matdata.eu%2F" alt="QR code for these slides" class="w-20 h-20 rounded bg-white p-1" />
    <div class="mt-1 text-xs opacity-80">Slides</div>
  </div>
  <div class="text-center">
    <img src="https://api.qrserver.com/v1/create-qr-code/?size=180x180&data=https%3A%2F%2Fgithub.com%2FMatdata-eu%2Fpaper-sem4tra2026" alt="QR code for the Sem4Tra paper" class="w-20 h-20 rounded bg-white p-1" />
    <div class="mt-1 text-xs opacity-80">Paper</div>
  </div>
  <div class="text-center">
    <img src="https://api.qrserver.com/v1/create-qr-code/?size=180x180&data=https%3A%2F%2Fdim.apps.banenor.no%2F" alt="QR code for the DIM portal" class="w-20 h-20 rounded bg-white p-1" />
    <div class="mt-1 text-xs opacity-80">DIM portal</div>
  </div>
  <div class="text-center">
    <img src="https://api.qrserver.com/v1/create-qr-code/?size=180x180&data=https%3A%2F%2Frinf-atlas.matdata.eu%2F" alt="QR code for RINF Atlas" class="w-20 h-20 rounded bg-white p-1" />
    <div class="mt-1 text-xs opacity-80">RINF Atlas</div>
  </div>
</div>

<div class="mt-4 text-sm opacity-80">Mathias Vanden Auweele · mathias@matdata.eu</div>

<style>
.tkw { padding: 7px 14px; font-size: 1.02rem; line-height: 1.35; }
.num { margin-right: 10px; vertical-align: middle; width: 26px; height: 26px; border-radius: 50%; display: inline-flex; align-items: center; justify-content: center; font-style: normal; font-weight: 700; font-family: 'Poppins', sans-serif; color: #0B1226; font-size: 14px; }
.num.sky { background: var(--c-sky); } .num.amber { background: var(--c-amber); } .num.green { background: var(--c-green); } .num.purple { background: var(--c-purple); }
</style>

<!--
[1:15] Four things to take home.
(click) A knowledge graph can run production systems: DIM feeds ETCS engineering, traffic planning and RINF.
(click) Topology first: get the model right; a hierarchy can't tell you where a train can go.
(click) Lift the source first, in its own language, then uplift with rules.
(click) And optimise the system, not the query: removing the splitting saved more than any query tuning ever did. And one wish for the community: linear referencing in GeoSPARQL.
Slides, paper, the portal and RINF Atlas are behind these codes. Thank you; I'm happy to take questions.
-->

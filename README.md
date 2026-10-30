# Building the Norwegian Railway Knowledge Graph

Slides for the Connected Data London 2026 talk *Building the Norwegian Railway Knowledge Graph, where the
Semantic Web stack runs off the track* (12 November 2026, 13:40, 30 minutes). Built on the
[ISWC 2026 deck](https://github.com/Matdata-eu/slides-iswc2026) with [Slidev](https://sli.dev).

```bash
npm ci
npm run dev      # http://localhost:3030
npm run export   # PDF
```

- `global-bottom.vue`: the train footer. Slides with a `station:` frontmatter become stations, `overpass: true` adds a bridge.
- `components/`: the animated slides (maturity ladder with confetti, consumers hub, RINF macro/micro, hierarchy vs topology,
  the mess, time bars, lift/uplift and railML before/after, the railML area map).

Pushing to `main` deploys to GitHub Pages on cdl2026.matdata.eu; a `v*` tag attaches a PDF to the release.

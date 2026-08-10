# CLAUDE.md — hvadkanclaude

Arbejdshukommelse for projektet. Læs dette inden du starter på opgaver.

---

## Projektet i én sætning

`hvadkanclaude.html` er et selvstændigt HTML-referenceværktøj der gør Claudes muligheder synlige og konkrete — med det overordnede mål at inspirere til nye idéer om hvornår og hvordan AI kan bruges.

---

## Deployment

| | |
|---|---|
| **Live URL** | `nulpunkt.net/modaliteter` |
| **Server** | Hetzner · Nginx |
| **Repo** | `github.com/Tlindskow/hvadkanclaude` |
| **Primær fil** | `hvadkanclaude.html` (enkeltfil, alt inline) |

Deploy-workflow i dag: download → gem lokalt → git push → manuelt på server.
**Ønsket workflow:** GitHub Actions auto-deploy ved push (ikke sat op endnu).

---

## Arkitektur

Én selvstændig HTML-fil. Alt CSS, JS og indhold er inline — ingen dependencies, ingen build-trin, ingen eksterne filer. Kan åbnes direkte i browser.

### Fane-struktur (12 paneler)

| Tab-id | Titel | Farve |
|---|---|---|
| `chat` | Chat | `#2563EB` |
| `cowork` | Cowork | `#0F766E` |
| `code` | Code | `#C2410C` |
| `design` | Design | `#6D28D9` |
| `iot` | IoT Sensorer | `#0369A1` |
| `agent` | Agentisk | `#7C3AED` |
| `sim` | Simulation | `#B45309` |
| `musik` | Musik & lyd | `#0E7490` |
| `maker` | Maker & Hardware | `#15803D` |
| `3d` | 3D Print & Fab | `#B91C1C` |
| `research` | Forskning & Data | `#1E40AF` |
| `skills` | Skills | `#7C3AED` |

### Panel HTML-mønster

```html
<div class="panel" id="panel-[id]" data-color="[hex]">
  <div class="panel-header">...</div>
  <div class="filter-bar">...</div>
  <div class="section-label">Sektionsnavn</div>
  <div class="grid">
    <!-- kort her -->
  </div>
</div>
```

**Vigtigt:** Hvert panel skal have sin egen `<div class="filter-bar">` — mangler den, bløder kortene over i næste panel.

### Kort HTML-mønster

```html
<div class="card" data-w="[filter]" data-tab-color="[tab-id]" onclick="openModal(this)"
  data-title="Kortnavn"
  data-icon="ti-[tabler-icon]"
  data-desc="Kort beskrivelse til modal-subtitle"
  data-w-label="[Filternavn]"
  data-ex1-title="Eksempel 1 titel"
  data-ex1="Eksempel 1 brødtekst"
  data-ex2-title="Eksempel 2 titel"
  data-ex2="Eksempel 2 brødtekst">
  <div class="card-head">
    <i class="ti ti-[icon]"></i>
    <span class="card-name">Kortnavn</span>
    <span class="badge b-[type]">Label</span>
  </div>
  <div class="card-desc">Beskrivelse på kortet</div>
  <!-- Valgfrit: dependency badge -->
  <div class="dep-badge dep-[type]">🔌 Kræver X</div>
</div>
```

### Maker-kort (udvidet)

Maker-kortet har ekstra data-attributter:
- `data-specs` — HTML-streng med tekniske specs
- `data-alts` — pipe-separerede alternativer: `"Alt 1 (~pris) | Alt 2"`
- `data-compat` — komma-separerede compat-tags: `"board,sensor,display"`
- `data-projects` — projektidéer: `"Projekt A · Projekt B"`
- `data-level` — `"beginner"` eller `"advanced"`
- `data-price` — pris i DKK (tal)
- Klik-handler: `makerCardClick(card)` ikke `openModal(this)`

---

## CSS-konventioner

### Badge-klasser (på kort)

```
b-dig      Digitalt (blå)
b-lyd      Lyd/medie (grøn)
b-fys      Fysisk (orange)
b-ai       AI-forstærket (lilla)
b-iot      IoT generisk (blå)
b-iot-grn  IoT grøn
b-iot-amb  IoT amber
b-iot-red  IoT rød
b-agent    Agentisk
b-sim      Simulation
b-musik    Musik
b-3d       3D Print
b-research Forskning
```

### Dependency badge-klasser

```
dep-extension  Browser-udvidelse (gul)
dep-plugin     Plugin (lilla)
dep-skill      Skill (grøn)
dep-mcp        MCP-connector (blå)
dep-cowork     Cowork/desktop-agent (orange)
dep-api        API-adgang (lyseblå)
```

### Modal farve-overrides

Hvert panel har CSS-overrides til `.example-1` og `.example-2` farver:
```css
[data-tab-color="[id]"] .example-1 { background: ...; border-color: ...; }
[data-tab-color="[id]"] .example-1 .example-title { color: ...; }
```
Skal tilføjes for hvert nyt panel.

---

## JavaScript — nøglefunktioner

```
showTab(name, btn)         Skifter aktivt panel
filterCards(panel, w, btn) Filtrerer kort i panel efter data-w
filterMaker(world, btn)    Maker-specifik filtrering
openModal(card)            Åbner standard-modal med kortdata
makerCardClick(card)       Maker-kortclick (builder/modal logik)
toggleBuilderMode()        Aktiverer/deaktiverer builder-tilstand
generateProjectIdea()      Kalder Claude API med valgte komponenter
TAB_COLORS                 Objekt med hex-farver per tab-id
```

**API-kald:** Bruger `https://api.anthropic.com/v1/messages` med `claude-sonnet-4-20250514`. Kun aktivt i Maker-fanen (project idea generator).

---

## Designprincipper

- **Formål frem for feature** — hvert kort skal besvare "hvornår er det nyttigt?" ikke bare "hvad er det?"
- **Konkrete eksempler** — ingen abstrakte beskrivelser, altid to realistiske use cases
- **Dependency-transparens** — hvis noget kræver en extension, MCP eller Skill, skal det fremgå tydeligt med dep-badge
- **Enkeltfil-princip** — alt forbliver inline i én HTML-fil, ingen CDN-dependencies der kan gå ned (undtagen Tabler Icons CDN og Google Fonts i `<head>`)
- **Dansk** — alt brugervendt tekst er på dansk

---

## Teknologi i `<head>`

```html
<!-- Fonts -->
<link href="https://fonts.googleapis.com/css2?family=DM+Serif+Display&family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
<!-- Icons -->
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@tabler/icons-webfont@latest/tabler-icons.min.css">
```

Ikonklasser: `ti ti-[navn]` — se tabler-icons.io for alle navne.

---

## Kendte gotchas

1. **Manglende `<div class="filter-bar">`** — hvis glemt, bløder kortene ud af panelet i show-all mode
2. **`data-tab-color` skal matche tab-id** — ellers får modal forkert farve
3. **Maker-kort bruger `makerCardClick`** — sættes via `DOMContentLoaded` event listener, ikke inline onclick
4. **`TAB_COLORS`-objektet** skal opdateres når nye tabs tilføjes
5. **Modal farve-overrides** i CSS skal tilføjes for hvert nyt panel
6. **`filterCards('3d', ...)`** — panel-id i filterCards-kald skal matche panel-elementets id uden `panel-`-præfix

---

## Hvad der mangler / næste skridt

- [ ] GitHub Actions auto-deploy workflow
- [ ] GitHub MCP-connector tilsluttet i Claude
- [ ] Skills-fanen: uddybende indhold fra skills-oversigt.html kan integreres
- [ ] Round AMOLED displays (Waveshare ESP32-S3-AMOLED-1.43, LILYGO T-RGB, T-Circle-S3, GC9A01) til Maker-fanen
- [ ] Simulation-fanen: faglig rolleplay (læge/patient, advokat/klient) mangler endnu

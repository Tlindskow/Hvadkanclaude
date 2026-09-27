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
| **Rapport-URL** | `nulpunkt.net/ai-apier` |
| **Server** | Hetzner · Nginx |
| **Repo** | `github.com/Tlindskow/hvadkanclaude` (**offentligt** — bevidst) |
| **Primær fil** | `hvadkanclaude.html` (enkeltfil, alt inline) |
| **Rapport-fil** | `ai-api-rapport.html` (enkeltfil, alt inline) |

```bash
git add -A && git commit -m "<dansk besked>" && git push
ssh hetzner "cd /var/www/hvadkanclaude && git pull"
```

**⚠️ nginx har to EXACT-MATCH locations, ikke en mappe.** `location = /modaliteter`
og `location = /ai-apier` peger hver på én fil med `alias`. Intet andet i
`/var/www/hvadkanclaude/` er tilgængeligt udefra — heller ikke favicons.
**En ny side kræver derfor en ny nginx-location og en reload**, ikke bare en fil.
Backup af configen før ændring ligger som `nulpunkt.net.bak-aiapi-20260927`.

**⚠️ Repoet er offentligt.** Serveren henter over `https://` uden deploy key.
Gøres repoet privat, brækker deployet, indtil det lægges om til en deploy key.

**Ønsket workflow:** GitHub Actions auto-deploy ved push (ikke sat op endnu).

---

## `ai-api-rapport.html` — kortlægning af eksterne AI-API'er

Selvstændig rapport (september 2026) over de API'er, Claude kan kalde UD til:
103 tjenester i 15 kategorier, 16 kombinationsopskrifter med regnestykke,
14 faldgruber, sorterbar prisoversigt. Hænger sammen med modulet gennem
fanen `apier` og et link begge veje.

- **Kataloget og pristabellen genereres fra ÉN datastruktur** (`CATS`, `S`,
  `PIPES` i scriptet). To tal, der skal stemme, deler én beregning.
- **Hver pris er mærket `off` eller `sek`** — officiel pris læst på udbyderens
  egen side, eller sekundær kilde. Et sekundært tal er et skøn, ikke en aftale.
  Bevar den skelnen; den er halvdelen af rapportens værdi.
- **Sektionsskifte frem for ét langt scroll.** Se faldgruben nedenfor.
- Datalaget kan valideres uden browser:
  ```bash
  awk '/^<script>$/{f=1;next} /^<\/script>$/{f=0} f' ai-api-rapport.html > /tmp/rap.js
  node --check /tmp/rap.js
  ```

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
| `apier` | Eksterne AI-API'er | `#4338CA` |

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
7. **En ny fane uden sin egen `.tab[data-tab="x"].active`-regel bliver HVID
   TEKST PÅ HVID BAGGRUND** i det øjeblik, man klikker på den. `.tab.active`
   sætter kun `color: var(--surface)`; baggrunden kommer udelukkende fra
   per-fane-reglen omkring linje 162–178. Tilføj også `--c-x` i `:root`.
   Koden så rigtig ud, og enhver prøve ville have bestået — **det kunne kun
   ses på en skærm.** (27/09-2026)
8. **Modelnavnet bor i konstanten `HKC_MODEL`, ikke i fetch-kaldet.** Det stod
   som `claude-sonnet-4-20250514` direkte i kaldet; da den model blev
   pensioneret, holdt Maker-fanens idégenerator op med at virke. Værre: kun
   401 blev håndteret, så en 404 faldt igennem til `'Ingen svar modtaget.'`
   — **et dødt kald kunne ikke skelnes fra en model uden noget at sige.**
   Fejlgrenen viser nu status og årsag. (27/09-2026)

---

## Faldgruben, der gælder begge filer

**`scroll-behavior: smooth` ANKOMMER ikke over lange afstande.** Målt i Chrome
27/09-2026 på rapporten: et anker 44.175 px nede blev aldrig nået — animationen
stoppede et sted på midten, uden fejl, uden log. Med `scroll-behavior: auto`
rammer den hver gang. Symptomet udefra var, at browserens screenshot-kommando
timede ud, og at den sticky navigation forsvandt ud af billedet.

**Den rigtige kur var ikke at fjerne animationen, men at fjerne afstanden.**
Modulet havde løsningen i forvejen: vis ét panel ad gangen. Rapporten viser nu
én sektion (9.480 px på den største) i stedet for 45.876 px — **79 % mindre at
tegne** — med «Læs alt» til gennemlæsning og print. Et anker ind i en skjult
sektion (fx `#cat-video`) åbner sektionen først og scroller derefter.

> **Og måleredskabet kan være forkert:** første forsøg på at måle scroll-tiden
> brugte `requestAnimationFrame` og hang, fordi rAF ikke kører i en
> baggrundsfane. `setTimeout` gav svaret med det samme. Samme lærepunkt som
> i BRANDHOLD — en fuldførelse må aldrig hænge på rAF.

---

## Hvad der mangler / næste skridt

- [ ] GitHub Actions auto-deploy workflow
- [ ] GitHub MCP-connector tilsluttet i Claude
- [ ] Skills-fanen: uddybende indhold fra skills-oversigt.html kan integreres
- [ ] Round AMOLED displays (Waveshare ESP32-S3-AMOLED-1.43, LILYGO T-RGB, T-Circle-S3, GC9A01) til Maker-fanen
- [ ] Simulation-fanen: faglig rolleplay (læge/patient, advokat/klient) mangler endnu

### Indholdsgennemgang 27/09-2026 — det, der stadig er forældet

De 209 kort blev gennemgået. Fire nye kort om serverside-værktøjer er tilføjet
til Agentisk-fanen, og det døde modelkald er rettet. **Resten står som fund,
ikke som arbejde** — det kræver Thomas' prioritering, hvilke der er værd at
skrive:

- [ ] **Chat-fanens «Langt kontekstvindue»** siger ikke, at 1 mio. tokens nu
      følger med til standardpris fra Claude 4.6 og frem.
- [ ] **Ingen kort om Fast mode** (`speed: "fast"`, research preview på Opus
      5.5/5/4.8 til præmiepris) eller om **inferensgeografi** (`inference_geo`,
      1,1× for US-only) — begge er relevante for et dansk projekt.
- [ ] **Ingen kort om modelfamilierne Fable 5 / Mythos 5.** Modulet nævner kun
      Sonnet 5 og Haiku 4.5 ét sted hver.
- [ ] **Cowork-fanens «Computer Use — Desktopkontrol»** beskriver den gamle
      model; der findes nu et `computer_toolset` OG et `browser_toolset` med
      hver sin token-omkostning pr. kald.
- [ ] **Tokenizer-skiftet fra 4.7 og frem** (~30 % flere tokens for samme
      tekst) er ikke nævnt nogen steder, selv om det ændrer enhver
      prisberegning i Maker-fanen.
- [ ] **Skills-fanen** lister 11 skills + 2 idéer; den bør holdes op mod, hvad
      der faktisk er tilgængeligt i dag.

Kilde til alle punkter: `platform.claude.com/docs/en/about-claude/pricing`,
læst 27/09-2026.

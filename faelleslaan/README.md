# Fælleslån — Blomfelt A/S

Statisk single-page-app til beregning af indfrielsesbeløb og periodeopgørelser
for fælleslån i ejerforeninger.

**Status:** Prototype (v1). Beregningsmotoren skal valideres mod bankens egne
årsopgørelser før den bruges til faktiske indfrielser — Blomfelt kan ifalde
erstatningsansvar ved fejl.

## Arkitektur

- **Frontend-kun** — ingen backend. Data gemmes i browserens `localStorage`
  under nøglen `bf-faelleslaan-v1`.
- **XLSX-persistens** — al data kan eksporteres til ét Excel-ark (fanen *Data*)
  og importeres igen. Det er den anbefalede måde at tage backup på og dele
  data mellem enheder.
- **SheetJS** — loades fra CDN til XLSX-læsning og -skrivning.
- **Blomfelt visuel identitet** — farver fra officiel palette
  (`#2C403A` grøn, `#FDF6E1` cream, `#AFD0E7/FFA899/F9C271/E5C8F8` accent),
  Montserrat som primær skrifttype.

## Funktioner

1. **Opsætning** — CRUD for ejendomme, lån, renteperioder og deltagere
   (inkl. udtræden og tidligere indfrielser).
2. **Indfrielsesberegner** — vælg lån, dato og én eller flere deltagere;
   beregner hver deltagers andel af restgæld + akkumuleret rente siden sidste
   rentetilskrivning + evt. gebyr.
3. **Periodeopgørelse** — vælg lån og periode; genererer opgørelse pr. deltager
   med primo/ultimo-saldo, afdrag og renter i perioden (inkl. pro-rata for
   deltagere der er udtrådt i perioden).
4. **Bogføring** — upload Excel/CSV-export fra bogholderi; systemet matcher
   poster til forventede ydelser og rentetilskrivninger og markerer afvigelser
   og manglende poster.
5. **Data** — eksport/import af al data som Excel. Fuld rundtur: redigér i
   Excel lokalt → importér tilbage.

## Beregningsmodel (kræver validering)

- Daglig renteopløbning: `saldo × rente% / 365` per dag.
- Kvartalsvis rentetilskrivning på 31/3, 30/6, 30/9, 31/12 (sidste dag i
  kvartalsmåneden — banks "sidste bankdag" er approksimeret).
- Månedlig ydelse på sidste dag i måneden; ydelsen dækker først rente,
  derefter afdrag.
- Ved renteændring gen-beregnes ydelsen, så restgæld amortiseres til lånets
  oprindelige slutdato (annuitetsformel). Hvis banken har meddelt en specifik
  ny ydelse, kan den indtastes manuelt i renteperioden.
- Ved ekstraordinær indbetaling reduceres saldoen direkte; løbetiden
  fastholdes og ydelsen sættes ned.

### Kendte begrænsninger i v1

- Bruger kalender-månedens sidste dag som ydelsesdato (ikke "sidste bankdag").
- Antager actual/365 dag-konvention — nogle lån bruger 30/360.
- Håndterer ikke afdragsfrihed eller variable løbetider.
- Runding sker til 2 decimaler ved events; små akkumulerede afvigelser kan
  forekomme. Valider mod bankens årsopgørelse før produktionsbrug.

## Deployment

Værktøjet deployes automatisk til GitHub Pages via
`/.github/workflows/deploy.yml` på push til `main`. Tilgås på
`https://<user>.github.io/<repo>/faelleslaan/`.

## Browserens datapersistens

Data ligger kun i denne browser (localStorage). Det betyder:

- Rens browserens data ⇒ alt er væk.
- Skift til anden enhed eller browser ⇒ ingen data.
- Eksportér regelmæssigt som backup og til deling.

Til fremadrettet hosting anbefales en rigtig backend
(fx Blomfelt Midas dashboard, som allerede har Express + JSON-storage).

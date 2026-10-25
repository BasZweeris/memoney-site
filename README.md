# memoney-site

De publieke site van MeMoney op **https://memoney.app**: een startpagina, de privacyverklaring (`/privacy`) en de voorwaarden (`/voorwaarden`). Statische HTML voor GitHub Pages, zonder framework of build.

De twee juridische pagina's zijn **voorlopig**: ze gelden alleen voor eigen gebruik tijdens de ontwikkeling en zijn nodig voor de registratie van de productie-applicatie bij Enable Banking. In stap 2.6 van de bankkoppeling komen de definitieve teksten op dezelfde adressen; de App Store eist dezelfde links. De inhoud volgt de spec van de bankkoppeling in de (privé) app-repository, sectie "2.1 Site en Enable Banking-applicaties" en beslissing BANK-24.

## Indeling

| Bestand | Adres |
|---|---|
| `index.html` | `/` |
| `privacy/index.html` | `/privacy/` (`/privacy` stuurt door) |
| `voorwaarden/index.html` | `/voorwaarden/` (`/voorwaarden` stuurt door) |
| `style.css` | gedeelde opmaak, licht en donker |
| `CNAME` | het eigen domein `memoney.app` |
| `.nojekyll` | GitHub Pages serveert de bestanden zoals ze zijn, zonder Jekyll |

Alle verwijzingen zijn relatief, zodat elke pagina ook lokaal opent (dubbelklikken in Finder). Doorklikken tussen pagina's werkt lokaal het best met een eenvoudige server in deze map, bijvoorbeeld `python3 -m http.server 8000`, en dan http://localhost:8000.

## Verantwoordelijke en contact

Beide juridische pagina's noemen alleen een naam en een e-mailadres, geen postadres: de AVG vraagt identiteit en contactgegevens, en zolang de site alleen eigen gebruik tijdens de ontwikkeling dekt is er geen bedrijf. Bij de definitieve versie (2.6) komen bij verkoop via de App Store bedrijfsgegevens erbij (naam, vestigingsadres, KvK-nummer); wijzig dan beide pagina's tegelijk.

## GitHub Pages aanzetten

Volgorde volgens de GitHub-documentatie: **eerst het domein in de repository instellen, dan pas de DNS-records zetten.** Andersom kan iemand anders tijdelijk een site op het domein zetten.

1. Repository → **Settings** → **Pages**.
2. Onder "Build and deployment": **Source** = *Deploy from a branch*, branch **`main`**, map **`/ (root)`**, **Save**.
3. Onder "Custom domain": `memoney.app` invullen, **Save**. GitHub controleert daarna de DNS; dat mislukt tot de records hieronder staan.
4. Zodra de controle slaagt en het certificaat er is (kan tot 24 uur duren): **Enforce HTTPS** aanvinken.

Stap 2 en 3 kunnen ook vanaf de command line:

```bash
gh api -X POST repos/BasZweeris/memoney-site/pages -f "source[branch]=main" -f "source[path]=/"
```

```bash
gh api -X PUT repos/BasZweeris/memoney-site/pages -f cname=memoney.app
```

En stap 4, zodra het certificaat er is:

```bash
gh api -X PUT repos/BasZweeris/memoney-site/pages -F https_enforced=true
```

## DNS-records

De DNS-zone van memoney.app staat in Azure DNS (nameservers `ns1-07.azure-dns.com` e.a.), in beheer van de compagnon van de ontwikkelaar; wijzigingen gaan via hem. Ingesteld en gecontroleerd op 25 oktober 2026: de vier A-records en de CNAME voor `www`, zonder AAAA-records; het certificaat is uitgegeven en HTTPS wordt afgedwongen.

Voor het apex-domein `memoney.app` vier A-records (IPv4) en, aanbevolen, vier AAAA-records (IPv6); voor `www` een CNAME naar de standaard Pages-host van het account. Met beide ingesteld stuurt GitHub `www.memoney.app` automatisch door naar `memoney.app`.

| Type | Naam | Waarde |
|---|---|---|
| A | `@` | `185.199.108.153` |
| A | `@` | `185.199.109.153` |
| A | `@` | `185.199.110.153` |
| A | `@` | `185.199.111.153` |
| AAAA | `@` | `2606:50c0:8000::153` |
| AAAA | `@` | `2606:50c0:8001::153` |
| AAAA | `@` | `2606:50c0:8002::153` |
| AAAA | `@` | `2606:50c0:8003::153` |
| CNAME | `www` | `baszweeris.github.io` |

Aandachtspunten:
- **Geen wildcard-record** (`*.memoney.app`): GitHub raadt dat sterk af vanwege het risico op overname van subdomeinen.
- De CNAME voor `www` wijst naar `baszweeris.github.io`, **zonder** repositorynaam, en niet naar het `*.pages.github.io`-adres dat de Pages-instellingen soms tonen.
- Geen andere A-, AAAA- of CNAME-records voor `@` en `www` ernaast (vroeger wezen ze naar een Azure web app).
- `.app` is een HTTPS-only-domein: browsers openen `memoney.app` alleen via HTTPS. De site is dus pas bereikbaar zodra GitHub het certificaat heeft uitgegeven, ook als de DNS al klopt.
- Optioneel en aanbevolen: het domein **verifiëren** onder GitHub → profiel **Settings** → **Pages** → *Add a domain*. GitHub geeft dan een TXT-record voor `_github-pages-challenge-baszweeris.memoney.app` (in de zone naam `_github-pages-challenge-baszweeris`) met een code als waarde; zet het, klik **Verify** en laat het record daarna staan. Dan kunnen alleen repositories van dit account een site op `memoney.app` en zijn directe subdomeinen publiceren, ook als Pages later uit staat.

Controleren of de records staan:

```bash
dig memoney.app +noall +answer -t A
```

```bash
dig www.memoney.app +noall +answer
```

Bron: GitHub Docs, "Managing a custom domain for your GitHub Pages site" en "Verifying your custom domain for GitHub Pages" (gecontroleerd op 25 oktober 2026).

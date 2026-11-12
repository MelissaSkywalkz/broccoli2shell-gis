# Origo Server: kontroll av Node.js och Brotli

**Källkontroll: 8 november 2026. Kodrevisionen är versionslåst.**

**Var ligger sårbarheten?** CVE-2026-111497 gäller **Googles Brotli-avkodare** som kan ingå i **Node.js**. Detta är inte ett fel som här har påvisats i Origo Servers egen applikationskod. Origo Server nämns därför att dess backend använder Node.js och Express; samma beroendefråga gäller många andra Node.js-tjänster utanför GIS.

**Vad vet vi?** Den granskade Origo Server-koden registrerar `express.json()` och `express.urlencoded()`. Express kan avkoda accepterade Brotli-komprimerade HTTP-begärandekroppar genom den körande Node.js-miljön. Det är en **källkodsstödd möjlig väg till den underliggande avkodaren**, inte ett Origo-specifikt CVE-bevis.

**Vad vet vi inte?** Ingen Origo-specifik CVE-utlösning, minneskorruption eller kodkörning har verifierats här. Den faktiska exponeringen beror bland annat på parser, proxy, körande Node-version och vilka begäranden som släpps igenom. En Origo-karta i webbläsaren innebär inte automatiskt att Origo Server används.

**Vad bör göras ändå?** Om installationen kör en Node.js-runtime med berörd Brotli-kod, **uppdatera Node.js** enligt relevant säkerhetsråd utan att invänta Origo-specifikt exploittest. Låt IT/drift verifiera den körande tjänsteprocessens runtime efter omstart och undersök relevanta HTTP-ingångar som separat prioriteringsfråga. [Daterad rättningsstatus](patchkedjan.md).

**Tekniska belägg och vidare läsning:** [README](../README.md) · [HTTP-avkodning](http-angreppsvagar.md) · [Patchkedja](patchkedjan.md)

## Möjlig indata- och avkodningsväg

```text
Extern HTTP-begäran med Brotli-komprimerad kropp
  → eventuell IIS/reverse proxy/lastbalanserare
  → Origo Server: Express-middleware
  → express.json() / express.urlencoded() när parsern matchar
  → Node.js zlib.createBrotliDecompress()
  → den Brotli-avkodare som hör till körande Node.js
```

**Källstöd:** I [Origo Servers `app.js` på revision `bacabe4`](https://github.com/origo-map/origo-server/blob/bacabe4b02611e8cbabf49daf57fe3d220f2e169/app.js) registreras `express.json()` och `express.urlencoded()` före applikationens routes. [Express säkerhetsbesked 31 oktober](https://expressjs.com/en/blog/2026-10-31-security-releases/) beskriver avkodningen genom `zlib.createBrotliDecompress()` för parsers som tar emot `br`-komprimerade kroppar med standardinställningar. Vilka paketversioner som används i en **given installation** ska kontrolleras i dess faktiska `package-lock.json` och runtime.

## Vad måste vara sant för att CVE:n ska kunna utlösas?

- En extern part kan överlämna relevant HTTP-kropp till Origo Servers parser. En proxy kan ha blockerat eller ändrat begäran innan den når Node.js.
- `Content-Type` och middlewarekonfiguration gör att den aktuella parsern verkligen försöker avkoda kroppen. **Alla** URL:er, statiska resurser eller förfrågningar utan begärandekropp går inte automatiskt via denna väg.
- Körande Node.js innehåller berörd Brotli-kod. Att `node` på kommandoraden råkar vara uppdaterat visar inte vilken körfil Windows-tjänsten använder.
- De komprimerade data innehåller det felaktiga formatvillkor som gäller för just CVE:n.

**Vad har inte visats?** Ingen specifik extern Origo-endpoint har här verifierats kunna utlösa CVE:n, och ingen Origo-specifik minneskorruption/RCE eller egen brist i applikationskoden har påvisats för denna CVE. `express.json()` i källkoden motiverar **kontroll av hur en Node.js-komponent avkodar data**, inte en slutsats om att Origo som produkt har en ny separat sårbarhet eller att en incident inträffat.

## Finns rättning?

**Ja, i Node.js.** Node-projektet släppte de första korrigerade utgåvorna **22.23.4**, **24.22.1** och **26.12.1** den 21 oktober 2026. **24.22.2** innehåller också rättningen och ersätter 24.22.1 efter en separat regression. Använd en aktuell stödd version som är kompatibel med tjänsten. [Node.js release](https://nodejs.org/en/blog/vulnerability/october-2026-security-releases) · [Node.js releasehistorik](https://nodejs.org/en/blog/release).

**Distributionsundantag:** En del Linuxdistributioner bygger Node mot systemets Brotli; andra levererar inbyggd Brotli. Uppströms Node-versionsnummer är inte ensamt en fullständig riskbedömning för ett distributionspaket. [Debians CVE-post](https://security-tracker.debian.org/tracker/CVE-2026-111497) · [Red Hat](https://access.redhat.com/security/cve/cve-2026-111497).

Efter patchning ska ansvarig verifiera **processens faktiska körfil och version** efter tjänsteomstart. Om reverse proxy eller lastbalanserare avvisar Brotli-begärandekroppar är det ett möjligt extra skydd för **den här indataingången**, inte en ersättning för patchning av Node.js eller en garanti för andra ingångar.

**Avgränsad slutsats för driftansvariga:** Den relevanta rättningen för den här vägen finns i **Node.js/Brotli**, inte i någon här identifierad ändring av Origo Servers egen kod. En aktiv backend som använder berörd Node-runtime bör uppdateras enligt säkerhetsbeskedet och kontrolleras efter omstart; en publik tjänst med möjlig `br`-ingång prioriteras skyndsamt. Det betyder **inte** att CVE-utlösning, serverkrasch eller kodkörning har visats i Origo Server.

**Kvar att kontrollera:** Körande Node-exekverbar fil och version, eventuell proxy som vidarebefordrar HTTP-kroppar, relevanta Express-parsers och andra eventuella avkodare. Okänd nåbarhet betyder **undersök**, inte att produkten är säker eller exploaterbar.

**Avgränsning:** Den här sidan gäller Brotli och CVE-2026-111497. Funktioner som hämtar externa HTTP-svar behöver bedömas som separata möjliga avkodningsvägar med den klient som faktiskt används.

**Öppen fråga:** Finns i aktuell Origo-installation även en HTTP-klient som automatiskt avkodar angriparpåverkade `br`-svar? Det kan inte fastställas genom att bara läsa Express-parsers i `app.js`.

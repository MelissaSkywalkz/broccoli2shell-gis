# FME – samma CVE via HTTP, Parquet och DuckDB

**Källunderlag: Safes CVE-specifika supportartikel avläst 7 november 2026; anonym kontroll 8 november omdirigerade till inloggning. Öppna nedladdnings- och changelogsidor kontrollerade 8 november 2026.**

**Var finns CVE-felet?** I **Brotli-avkodning i de beroenden och komponenter FME använder**. Safe har bekräftat påverkan i sin produktfamilj; det är inte detsamma som att det offentliga underlaget har påvisat att FME:s egen applikationskod introducerade Brotli-felet. Leverantörsbeskedet är ändå grund för att följa FME:s rättningsråd.

**Vad vet vi?** Enligt Safes CVE-specifika besked, avläst 7 november 2026, rapporteras påverkan i FME Form, Engine, Flow och Hosted. Bland nämnda vägar finns HTTPCaller/OpenAPICaller, förhandsvisning och anslutningar, Parquet samt DuckDB. Dessa kan använda **olika Brotli-kopior**.

**Vad vet vi inte?** Det finns ingen FME-specifik CVE-utlösning eller kodkörning reproducerad i detta repo. Första produktbyggnummer med CVE-specifik rättning: **inte fastställt i öppet kontrollerbara källor här**; det betyder inte att en rättning saknas. En möjlig koppling mellan externa flöden och känsliga integrationer är en undersökningsfråga, inte bevis för åtkomst eller läckage.

**Vad bör göras ändå?** Följ Safes säkerhetsråd för er version och plattform utan att invänta eget exploittest. Inventera aktiva HTTP-/filflöden och körkonton, och be Safe eller driftansvarig bekräfta rättad produktversion samt eventuella separata DuckDB-/Linux-Brotli-kopior. [Daterad rättningsstatus](patchkedjan.md).

**Tekniska belägg och vidare läsning:** [README](../README.md) · [Dataflöde till patch](dataflode-till-patch.md) · [Riskkedja](riskkedja.md) · [Patchkedja](patchkedjan.md)

**Källgräns:** [Safes CVE-artikel](https://support.safe.com/hc/en-us/articles/50040725669517-Brotli-Decoder-CVE-2026-111497-Is-FME-Impacted) var läsbar vid kontroll 7 november men **kräver nu inloggning** vid den anonyma kontrollen 8 november. Uppgifterna ovan är ett **tidigare avläst leverantörsbesked**. Den öppna novemberartikeln om andra säkerhetsuppdateringar styrker inte ett visst CVE-rättat Brotli-byggnummer.

## 1. HTTPCaller, OpenAPICaller och webbanslutningar

```text
Externt HTTP/API-svar med Brotli-kodning
  → FME Workbench-förhandsvisning / FME Engine / tokenförnyelse
  → HTTP-klientens avkodning
  → Brotli (bibliotekskopia beror på FME-plattform)
```

**Nödvändiga villkor:** FME hämtar ett svar från en källa som angriparen kan kontrollera eller kompromettera, svaret hanteras som Brotli-komprimerat och en berörd avkodare används. Vissa Workbench-funktioner kan behandla svar innan ett arbetsflöde uttryckligen körs. Ett internt FME Flow är inte automatiskt skyddat om arbetsytor hämtar externa svar.

**Bevis:** Vägarna återges ur Safes CVE-specifika besked, tidigare läst den **7 november**. Vid anonym kontroll 8 november krävdes inloggning. Detta är återgiven leverantörsinformation, inte en oberoende verifierad avkodning eller offentlig produkt-RCE-demonstration. [Safe – CVE-2026-111497](https://support.safe.com/hc/en-us/articles/50040725669517-Brotli-Decoder-CVE-2026-111497-Is-FME-Impacted) (kan kräva inloggning).

### Exempel: hämta öppna data via ett Swagger-dokumenterat API

En arbetsyta som anropar en offentlig data-API via **HTTPCaller** eller **OpenAPICaller** ska undersökas utifrån **det faktiska HTTP-svaret och FME Engines avkodare**. Swagger/OpenAPI beskriver API:et; det är inte ett särskilt format som i sig gör FME sårbart. Förutsättningarna är att ett `br`-komprimerat, angriparpåverkat svar når en **berörd Brotli-avkodare** och CVE-villkoret uppstår. Kontrollera vilka schemalagda jobb som gör anropet, körkontot och om samma process kan nå andra skyddsvärda resurser – exempelvis en separat personuppgiftsintegration – utan att anta att olika arbetsytor delar behörighet. [Öppna data via Swagger/OpenAPI](http-angreppsvagar.md#öppna-data-via-swaggeropenapi-ett-api-svar-inte-ett-eget-filformat) · [Riskkedjan](riskkedja.md#exempel-öppna-data-från-api-och-separat-känslig-integration).

## 2. Parquet-kolumner

```text
Importerad Parquet-fil med Brotli-komprimerade kolumner
  → FME Engine / Parquet-läsare
  → berört avkodningsbibliotek
```

**Nödvändiga villkor:** En extern fil behandlas och en faktisk datasida/kolumn är Brotli-komprimerad; relevant berörd avkodare används. En Parquet-fil som bara använder en annan codec går inte via denna Brotli-väg.

**Bevis:** Safe pekar ut Parquet i sin produktpåverkananalys; detaljerna om exakt implementerad bibliotekskedja kan variera mellan FME-versioner. Därför märks vägen som **leverantörsrapporterad**, inte som reproducerad i varje FME-byggning.

## 3. DuckDB – särskild kopia

```text
Extern DuckDB-/Parquet-relaterad data
  → FME:s DuckDB-integration
  → DuckDB:s egen medföljande Brotli-kod
```

**Nödvändiga villkor:** En DuckDB-funktion använder den berörda avkodaren för angriparpåverkade data. DuckDB:s eventuella *interna* Brotli kan fortfarande vara gammal efter att system-Brotli uppdaterats.

**Bevis:** Safe beskriver DuckDB som separat rättningsspår. [DuckDB:s arbete med uppströmsrättningen](https://github.com/duckdb/duckdb/issues/27660) är ytterligare stöd för att en inbyggd bibliotekskopia behöver hanteras för sig. En sammanslagen källkodsändring garanterar inte ett rättat FME-paket.

## Verksamhetsrisk: vad händer om en FME-process påverkas?

FME är inte bara ett paket med DuckDB. **Det är arbetsytorna, de data de hämtar och de behörigheter som deras körande FME Engine faktiskt har** som avgör möjlig följdskada. En personuppgiftsintegration kan vara relevant även om ett helt annat FME-flöde står för den angriparpåverkade indatan – **men bara om** rättigheter, hemligheter, nätverk eller andra resurser är åtkomliga från den process eller miljö som angriparen faktiskt skulle kunna påverka. Isolering måste undersökas, inte antas bort.

Förvaltaren bör därför skilja på:

- **Exponering:** vilka arbetsytor, förhandsvisningar och schemalagda jobb som avkodar externa HTTP-svar eller filer med berörd Brotli-kopia.
- **Process och behörighet:** vilken FME Engine/Workbench som behandlar dem, användar- eller tjänstekonto och vilka anslutningar/credentials den processen verkligen kan nå.
- **Skyddsvärden:** personuppgifter, andra verksamhetssystem, filer, skrivbehörigheter, processminne och förmåga att fullfölja kritiska överföringar.
- **Effekt:** en krasch kan ge uteblivet jobb; informationsröjande kräver en separat visad väg; godtycklig kodkörning har **inte** visats specifikt i FME utifrån detta underlag. En sårbar komponent är inte bevis för personuppgiftsincident.
- **Skydd:** rätta berörd FME-/system-/DuckDB-kopia, begränsa onödiga dataingångar, separera privilegier och hemligheter, följ arbetsytefel och återställningsförmåga.

Ett **hypotetiskt** exempel med en verksamhetsintegration som hanterar personuppgifter finns i [riskkedjan](riskkedja.md#exempel-fme-integration-som-behandlar-personuppgifter). Det beskriver **inte** någon verklig organisations konfiguration eller konstaterat dataläckage.

## Var finns patchen?

**Statusen nedan gäller uppgifter avlästa 7–8 november 2026, inte en aktuell garanti om Safes senaste versioner eller en viss installation.** [Samlad daterad patchstatus](patchkedjan.md).

| FME-miljö | Var bör rättningen komma ifrån? | Status enligt kontrollerat underlag |
| --- | --- | --- |
| **Windows / macOS** | Säkerhetsuppdaterad FME-utgåva som ersätter medföljande Brotli | **Produktpåverkan bekräftad av Safe; första CVE-rättade FME-byggnummer inte självständigt verifierat här.** |
| **Linux: HTTP och Parquet** | Linuxdistributionens rättade Brotli-paket där dessa funktioner använder systembiblioteket | **Systemrättningar finns**, men kontrollera körande FME-process och paketkälla. |
| **DuckDB i FME** | DuckDB-rättning måste nå den FME-utgåva som levererar DuckDB-kopian | **FME-specifik leveransstatus kräver aktuellt Safe-besked.** |
| **Flow Hosted** | Plattformens systemuppdatering och relevant omstart; DuckDB separat | Safes äldre besked anger automatiska Ubuntu-paketuppdateringar och behov av omstart för att använda rättningen. |

**Källkritisk versionsanmärkning:** Det Safebesked som dokumenterades den 7 november hade en sammanfattning till och med **2026.3**, medan ett detaljavsnitt angav medföljande Brotli `1.1.0` för Windows/macOS upp till **FME Form 2026.4**. Detta är en olöst motsägelse i det tidigare leverantörsunderlaget. Säkerhetsläget får inte bedömas enbart utifrån sammanfattningstabellen.

Safes [öppna nedladdningssida](https://fme.safe.com/downloads/) listar bland annat `2026.2.1.2`, `2026.1.2.1`, `2025.2.4.2`, `2025.1.3.5`, `2025.0.3.2` och `2024.2.5.2`. Safes [novemberartikel om säkerhetsuppdateringar](https://support.safe.com/hc/en-us/articles/46792031942029-FME-November-2026-Security-Updates) och [changelog](https://fme.safe.com/changelog/) visar att säkerhetsfixar har släppts, men de öppna uppgifterna som kunde läsas här **binder inte dessa byggnummer specifikt till CVE-2026-111497**. Märk dem därför inte som bekräftat rättade för Brotli utan uttryckligt stöd från Safe.

**Tillfällig riskbegränsning om patch saknas eller ännu inte är klarlagd:** Be Safe eller er drift-/leverantörskontakt om **CVE-specifik rekommendation för er version och plattform**. Begränsa vid behov just de arbetsflöden som tar emot angriparpåverkbar Brotli-data, utan att stoppa normal verksamhet i onödan. Kontroll av källor, behörigheter och importer är kompletterande skydd, inte en ersättning för rättning när en relevant uppdatering finns. Ett uteblivet produktspecifikt exploitbevis är inte skäl att avvakta en rekommenderad säkerhetspatch.

## Slutbedömningar för FME-vägarna

**HTTPCaller/OpenAPICaller och Swagger-hämtningar**

**Slutbedömning av denna riskkedja – exempel, inte en verifierad lokal installation:**

| Fält | Bedömning |
| --- | --- |
| **Påverkan** | Safe rapporterar FME-påverkan för dessa HTTP-funktioner; exakt miljös avkodare ska fastställas. |
| **Exponering** | Automatiskt eller manuellt mottagna angriparpåverkade `Content-Encoding: br`-svar; inte alla API-svar. |
| **Konsekvens** | Avbrott i jobb och eventuellt annan skada endast om ytterligare exploatering lyckas inom processens verkliga behörighet. |
| **Prioritet** | **Inventera aktiva externa hämtningar och tillgång till känsliga integrationer först**; patcha nåbar berörd kopia skyndsamt. |
| **Rättning** | FME-/system-Brotli efter plattform; kontrollera CVE-specifik Safe-version innan färdigmarkering. |
| **Kvarvarande osäkerhet** | Faktiskt laddad avkodare, rättat produktbygge och åtkomst från berörd Engine. |

**Parquet-import**

**Slutbedömning av denna riskkedja – exempel, inte en verifierad lokal installation:**

| Fält | Bedömning |
| --- | --- |
| **Påverkan** | Safe rapporterar påverkan i FME:s Parquet-väg; avkodare kan skilja mellan utgåvor. |
| **Exponering** | Kräver att externt Parquet-innehåll med Brotli-sidor importeras med berörd kod. |
| **Konsekvens** | Möjligt import- eller jobbavbrott; RCE och informationsläckage inte visat i FME. |
| **Prioritet** | **Kontrollera aktiva importjobb och filernas ursprung**, därefter patchstatus för den kopian. |
| **Rättning** | FME:s eller systemets Parquet/Brotli-beroende enligt faktiskt bygge. |
| **Kvarvarande osäkerhet** | Vilken codec används i filen och vilket bibliotek laddar aktuell Engine? |

**DuckDB**

**Slutbedömning av denna riskkedja – exempel, inte en verifierad lokal installation:**

| Fält | Bedömning |
| --- | --- |
| **Påverkan** | Safe pekar ut separat DuckDB-spår med potentiellt inbyggd Brotli. |
| **Exponering** | Bara om DuckDB-funktionen avkodar angriparpåverkat innehåll med berörd kod. |
| **Konsekvens** | Möjliga jobbavbrott; vidare konsekvenser villkorade, inte produktvisade. |
| **Prioritet** | **Inventera DuckDB-användning** och prioritera aktivt exponerad, berörd kopia före oanvänd funktion. |
| **Rättning** | Rättningen måste nå FME:s **medföljande DuckDB**, inte enbart operativsystemets Brotli. |
| **Kvarvarande osäkerhet** | FME-levererad DuckDB-version, använd kodväg och CVE-specifik produktpatch. |

**Öppna frågor:** Vilken FME-byggning har rättat den *medföljande* avkodaren? Vilken DuckDB-version levereras och när? Använder aktuell Engine de systembibliotek som uppdaterats? Svar kräver tillgång till aktuell leverantörsdokumentation och den lokala installationen.

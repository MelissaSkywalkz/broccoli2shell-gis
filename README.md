# Broccoli2Shell i GIS: påverkan och uppdateringar

[![CVE](https://img.shields.io/badge/CVE-2026--111497-f12c35)](https://www.cve.org/CVERecord?id=CVE-2026-111497)
[![CVSS](https://img.shields.io/badge/CVSS%204.0-9.2%20Critical-f12c35)](https://www.cve.org/CVERecord?id=CVE-2026-111497)
[![CISA KEV](https://img.shields.io/badge/CISA%20KEV-Listed-d97706)](https://www.cisa.gov/known-exploited-vulnerabilities-catalog?field_cve=CVE-2026-111497)

**Uppdaterad: 12 november 2026**

**CVE-2026-111497** är ett minnesfel i Googles Brotli-avkodare. Felet är inte GIS-specifikt. Brotli används även i filformat, bibliotek och körmiljöer som förekommer i GIS-program.

Broccoli2Shell-GIS hjälper GIS-användare och systemansvariga att förstå påverkan, kontrollera sina installationer och genomföra relevanta uppdateringar. Dokumentationen följer GIS-produkter, deras beroenden och hur rättningen når den använda avkodaren.

GIS-program kan använda Brotli genom exempelvis GDAL/Arrow, Node.js och .NET. Ingen separat brist i QGIS, FME, ArcGIS, Hajk eller Origo Servers egen applikationskod har påvisats här. Produkterna kan ändå behöva uppdateras på grund av sina beroenden. Leverantörsbekräftad påverkan innebär inte att varje angreppsväg är exploaterbar.

En produkt kan innehålla berörd kod utan att avkodaren är nåbar. Rättningen behöver dessutom nå den version som användaren faktiskt kör. Några officiella exempel utanför GIS förklarar skillnaden. [Exempel och avgränsningar](docs/produktoversikt.md#exempel-utanför-gis-och-deras-betydelse-för-gis).

Produktuppgifterna bygger på daterade leverantörsbesked och tidigare dokumenterade kontroller. Länkar och centrala öppna leverantörskällor återkontrollerades 12 november 2026. Safes CVE-artikel lästes senast via inloggning 9 november och kunde inte läsas anonymt den 12 november. De lokala installationsobservationerna är inte nya tester. Kontrollera alltid leverantörens senaste besked och den egna installationen. Tabellen visar inte vilka GIS-produkter som kan exploateras.

## Börja här: behöver du göra något?

**Känt:** En felaktig Brotli-ström som når CVE:ns felvillkor kan orsaka minneskorruption i en berörd avkodare. Vissa leverantörer har bekräftat produktpåverkan och publicerat rättningar. För andra GIS-produkter finns dokumenterade beroenden eller *möjliga* avkodningsvägar, men inte bekräftad produktspecifik exploatering.

**Oklart:** Ett sårbart bibliotek i en installation visar inte om en angripare kan nå det eller utlösa CVE:n där. Avsaknad av en verifierad angreppsväg betyder inte att hela installationen är säker. Underlaget styrker inte generell fjärrkodkörning (RCE) i någon av de undersökta GIS-produkterna.

**Åtgärd:** Installera relevanta säkerhetsuppdateringar från leverantör eller distribution. Vänta inte på ett eget exploateringstest. Använder du GIS: fortsätt med godkända verktyg och normala rutiner för uppdateringar och externa filer. Förvaltar du system: prioritera aktiva servrar, automatiska dataimporter och processer med känsliga behörigheter, och verifiera att rättningen når den *faktiskt använda* avkodaren.

En vanlig GIS-användare behöver inte göra DLL-analyser eller CVE-tester. GIS-förvaltning, IT/drift och leverantören delar på arbetet beroende på vem som äger produkten och dess uppdateringar. [Praktiskt stöd för ansvar och överlämning](docs/borja-har.md).

> [!IMPORTANT]
> **En sårbarhet behöver inte utnyttjas ensam.**
>
> Flera säkerhetsbrister kan förstärka varandra. En brist kan öppna en väg till en berörd Brotli-avkodare. Andra brister eller alltför omfattande behörigheter kan öka följderna om ett angrepp lyckas.
>
> Kontrollera både vad systemet tar emot och vad det självt hämtar. Uppdatera berörda komponenter även om andra skydd finns. Ingen sådan kombinerad angreppskedja har verifierats här för någon viss GIS-produkt.

## 1. Vad är det som kan gå fel?

CVE-2026-111497 är en **out-of-bounds write** (skrivning utanför avsedd minnesbuffert) i Brotli-avkodning. Särskilt felaktig komprimerad indata kan få avkodaren i ett ogiltigt tillstånd och orsaka minneskorruption. **Inte varje trasig fil eller varje komprimerat svar utlöser felet.** [Node.js säkerhetsbesked](https://nodejs.org/en/blog/vulnerability/october-2026-security-releases) · [CVE-post](https://www.cve.org/CVERecord?id=CVE-2026-111497).

När en produkt bedöms behöver fyra frågor hållas isär:

1. **Berört beroende:** en version som omfattas av CVE:n finns i eller används av programvaran.
2. **Nåbar avkodare:** en viss fil, begäran eller datakälla behandlas faktiskt av den berörda funktionen.
3. **CVE-utlösning:** just det specifika felvillkoret kan nås genom denna väg.
4. **Påvisad säkerhetseffekt:** exempelvis krasch, minneskorruption, informationsröjande eller kodkörning har verkligen demonstrerats i den angivna miljön.

Ett berört beroende visar inte om avkodaren kan nås, om CVE:n kan utlösas eller om en säkerhetseffekt har uppstått. Kontrollera också att rättningen finns i den körande installationen, inte bara i en uppströmsrelease.

En krasch kan ge tjänsteavbrott. Minneskorruption *kan* under särskilda förhållanden leda till mer allvarlig påverkan; informationsröjande och kodkörning kräver egna belägg. Forskare har demonstrerat kodkörning i en avgränsad Node.js-testmiljö. Det är **inte** bevis för RCE i QGIS, FME, ArcGIS, Origo Server, Hajk eller GeoServer. [Forskarens redogörelse](https://www.imperva.com/blog/node2shell-from-brotli-data-to-remote-code-execution/) · [Fördjupning om CVE-mekanismen](docs/cve-mekanism.md) · [Konsekvenser och deras beviskrav](docs/konsekvenser.md).

CISA lade till CVE:n i sin katalog över kända exploaterade sårbarheter (KEV) **27 oktober 2026**. Det stärker skälen att prioritera relevanta säkerhetsåtgärder men bevisar **inte** att ett särskilt GIS-system har angripits. [CISA KEV](https://www.cisa.gov/known-exploited-vulnerabilities-catalog?field_cve=CVE-2026-111497).

## 2. Hur kan berörd Brotli-kod nås?

| Möjlig indataväg | Vad krävs för att den ska vara relevant? | Exempel (inte bevis för påverkan) |
| --- | --- | --- |
| **Inkommande HTTP-begäran** | En backend accepterar och avkodar `Content-Encoding: br`, eventuellt före autentisering. | Express/Node.js; separat konfigurerad request-decompression i .NET. |
| **Hämtat HTTP-svar** | En klient avkodar ett Brotli-komprimerat svar med berörd kod. | API/WMS/WFS via en faktisk HTTP-klient i exempelvis FME, QGIS, Python eller en integration. |
| **Filimport** | En fil innehåller Brotli-komprimerade delar som når en berörd filavkodare. | GeoParquet/Parquet via Arrow, eller annan parser med Brotli. Filändelsen behöver inte vara `.br`. |
| **Automatiskt jobb** | Ett schemalagt flöde behandlar ett sådant svar eller en sådan fil utan manuell öppning. | FME Flow, Pythonjobb, integrationsplattform eller importtjänst. |

Swagger/OpenAPI är ett **gränssnitt eller en API-beskrivning**, inte en egen Brotli-avkodare. Det är webbläsaren eller programmet som faktiskt gör HTTP-anropet som behöver kontrolleras. HTTP-komprimering och Brotli *inne i* en Parquet-fil är två skilda vägar. [HTTP-vägar](docs/http-angreppsvagar.md) · [Filvägar och social engineering](docs/fil-angreppsvagar.md) · [Från ett visst dataflöde till rätt patch](docs/dataflode-till-patch.md).

**QGIS utan externa datakällor:** Det minskar vissa exponeringsvägar. En berörd Brotli-kopia i installationen är inte i sig bevis för att den går att angripa. Men även en *lokalt sparad* GeoParquet-fil kan tidigare ha laddats ned manuellt eller skickats av någon annan. En möjlig avkodningsväg visar **inte** att filen kan utlösa CVE:n eller ge RCE i QGIS. Fortsätt använda QGIS normalt och installera relevanta officiella uppdateringar; IT/GIS kan vid behov kontrollera vilken Brotli-kopia installationen använder. [QGIS-bevisningen](docs/qgis.md).

**HTTPS är viktigt men inte ett fullständigt avkodarskydd.** Korrekt TLS/certifikatkontroll skyddar normalt mot manipulation under transport, inte mot data från en komprometterad legitim källa. Filvalidering, viruskontroll och tillfälliga `br`-spärrar kan minska risker men ersätter inte patchning. [Skydd och begränsningar](docs/skydd.md).

## 3. Produkter: vad vet vi, vad vet vi inte och vad bör göras?

**Status nedan gäller källunderlaget från 8 november 2026, inte läsarens installation.** ”Möjlig väg” betyder inte ”bekräftat exploaterbar produkt”. Leverantörens uttryckliga besked om påverkan väger tyngre än en hypotes om ett beroende.

| Produkt eller driftmiljö | Belägg och osäkerheter | Rekommenderad kontroll eller åtgärd |
| --- | --- | --- |
| **[QGIS](docs/qgis.md)** | **Dokumenterad Brotli-avkodning via GDAL/Arrow i specificerade QGIS-byggen.** De fristående Windows-MSI-paketen 3.44.15-2 och 4.2.3-2 innehåller enligt installationskontroller Brotli 1.2.0. Ingen produktspecifik RCE påstås här; andra indatavägar utreds separat. | Följ QGIS säkerhetsbesked. Befintliga MSI-installationer kan ha egen OSGeo4W Setup för paketuppdateringar; prova kontrollerat och verifiera laddad DLL. `brotli 1.2.1-1` rättar inte automatiskt separata GRASS-kopior. [QGIS](docs/qgis.md) |
| **[FME Form/Flow](docs/fme.md)** | Safe bekräftar påverkan i FME genom HTTP-klient, Parquet och DuckDB. Enligt artikel kontrollerad 9 november förbereds rättade byggen för 2026.1 till 2026.3. DuckDB kräver separat rättning. Ingen färdig CVE-rättad FME-utgåva har bekräftats här. | Följ Safes råd för er version. Linux använder system-Brotli för HTTP/Parquet, men DuckDB har en egen kopia. Kontrollera även externa anrop från Workbench och schemalagda Engine-jobb. |
| **[ArcGIS](docs/arcgis.md)** | **Esri bekräftar påverkan via underliggande komponenter** för Server 11.5/12.0/12.1 och Pro 3.5+, med Node.js- och .NET-relaterade rättningskedjor. Server 11.1/11.3 berörs enligt Esri inte av *just Node.js/Brotli-felet*. Ingen specifik extern CVE-endpoint eller generell ArcGIS-RCE redovisas här. | Installera Esris Server-patch där den gäller; följ Esris aktuella besked om Pro och uppdatera relevant .NET separat. |
| **[Origo Server, Node.js i backend](docs/origo-server.md)** | En möjlig väg till **Brotli-avkodaren som följer med Node.js** stöds av Express och granskad källkod. Inget eget CVE-fel i Origos applikationskod eller någon Origo-specifik exploatering har påvisats här. Webbklienten är separat. | Kontrollera om Node-backend körs; uppdatera och verifiera berörd Node.js-runtime enligt driftmiljöns och Node-projektets råd. |
| **[Hajk, Node.js i vissa backenddistributioner](docs/hajk.md)** | Hajks backend kan använda **Brotli-avkodaren som följer med Node.js** via Express. Backend-ZIP använder driftmiljöns Node.js; full officiell Docker innehåller Node.js; statiska `hajk-simple` saknar Node-backend i drift. Inget eget CVE-fel i Hajks kod eller någon Hajk-specifik exploatering har påvisats här. | ZIP: uppdatera driftmiljöns Node.js. Full Docker: välj en aktuell nattlig releaseavbild för er minorversion (exempelvis `nightly-4.3`), **hämta och återskapa** containern och verifiera den körande Node-versionen. Nattliga byggen finns bara för de två senaste minorversionerna. |
| **[GeoServer/GeoWebCache](docs/geoserver.md)** | **Ingen generell produktpåverkan belagd.** Separata Jetty/Brotli4j-moduler eller tillägg *kan* vara relevanta; att standardinstallationen använder en berörd avkodare är inte fastställt. | Inventera bara faktiskt använda moduler/native-kopior och rätta berörda beroenden när de finns. |
| **Frends, Pentaho/Kettle, myCarta, Gaida, EntryScape m.fl.** | **Inte produktspecifikt fastställt här.** Att programmet har HTTP, Java eller .NET är inte tillräcklig bevisning. | Kartlägg aktiv dataingång och avkodare, följ produktleverantörens besked och uppdatera berörda komponenter. [Exempel på verksamhetsflöden](docs/verksamhetsfloden.md). |

**Hajks Docker-uppdateringar:** Hajk bygger nattliga avbilder för de två senaste minorversionerna, exempelvis `nightly-4.3`. De byggs om med aktuell grundavbild för Node.js och operativsystemet, men npm-beroenden följer de låsta versionerna. Bara senaste Hajk-releasen får nya säkerhetsfixar i själva programmet. Vanliga releaseavbilder byggs inte om mellan releaser. Hämta en aktuell avbild, återskapa containern och kontrollera vilken Node.js-version som körs. [Hajks releaseguide](https://github.com/hajkmap/Hajk/blob/master/docs/releases.md#nightly-builds) · [Säkerhetspolicy](https://github.com/hajkmap/Hajk/blob/master/SECURITY.md#official-docker-images).

QGIS, FME och ArcGIS har olika bevisläge. Det finns därför inget stöd för att bedöma dem som lika exploaterbara eller kalla någon av dem säker bara för att en CVE-utlösning inte har demonstrerats offentligt. **En produkt kan påverkas av CVE:n utan att dess utvecklare har orsakat grundfelet.** Ett sakligt besked anger **vilken version, vilket beroende och vilken typ av belägg** det avser, och rekommenderar relevant uppdatering. [Detaljerad produktöversikt med separata möjliga vägar](docs/produktoversikt.md).

## 4. Rättningar och daterade leverantörsbesked

Detta är **tillgängliga versioner eller daterad leverantörsstatus**, inte ett intyg om att en viss installation är uppdaterad. En distribution kan ha bakåtporterade säkerhetsrättningar och flera Brotli-kopior. Följ leverantörens rekommenderade paket för just er version och plattform.

| Komponent | Publicerad rättning eller status per kontrolldatum | Viktig avgränsning / källa |
| --- | --- | --- |
| **Google Brotli** | `1.0.10`, `1.1.1`, `1.2.1` per berörd gren | Grundfix, inte automatiskt rättning för inbyggda kopior. [Advisory](https://github.com/advisories/GHSA-9v8h-qqgm-4m35) |
| **Node.js** | Första Brotli-rättade: `22.23.4`, `24.22.1`, `26.12.1` (21 okt). Senare säkerhetsversioner: `22.23.5`, `24.22.3`, `26.13.1` (11 nov). | Version 24.22.1 hade en separat regression och ska ersättas av en senare säkerhetsuppdaterad 24.x-version. Novemberutgåvorna gäller andra sårbarheter. Kontrollera vilken Node-version tjänsten kör. [Oktober](https://nodejs.org/en/blog/vulnerability/october-2026-security-releases) · [November](https://nodejs.org/en/blog/vulnerability/november-2026-security-releases) |
| **.NET** | `8.0.33`, `9.0.22`, `10.0.14`, `.NET 11 RC3` | Gäller Microsofts angivna Brotli-vägar, inte automatiskt varje HTTP-klient. Berörd HTTP-avkodning är inte alltid påslagen som standard. Self-contained/NativeAOT måste byggas/distribueras om; .NET Framework omfattas inte av just denna advisory. [Microsoft](https://github.com/dotnet/runtime/security/advisories/GHSA-p3jp-9v83-76w9) |
| **PowerShell 7** | `7.4.22`, `7.5.13`, `7.6.9` | Rätta den version schemalagda jobb faktiskt använder. [MSRC](https://www.microsoft.com/en-us/msrc/blog/2026/10/patch-now-php2shell-brotli-emudhra-guidance-for-customers) |
| **Arrow/PyArrow** | `26.0.1` för angivna distribuerade binärer | Systemlänkad Arrow kan använda annan Brotli-kopia. [Arrow](https://arrow.apache.org/blog/2026/11/03/26.0.1-release/) |
| **QGIS/OSGeo4W och GRASS** | `brotli 1.2.1-1` tillgängligt i OSGeo4W; fristående MSI 3.44.15-2 och 4.2.3-2 innehåller enligt separata kontroller Brotli 1.2.0. Senaste kontrollerade stabila GRASS 8.5.0-1 föregår rättningen. | MSI-installationens egen OSGeo4W Setup kan uppdatera paket i dess installationsrot. Verifiera resultatet; ingen rättad komplett MSI eller GRASS-kopia verifierad här per 10 nov. [QGIS](docs/qgis.md) · [OSGeo4W](https://www.norbit.de/osgeo4w/v2/x86_64/release/brotli/) |
| **Debian/Ubuntu** | Säkerhetsbackportar i distributionspaket | En äldre grundversion kan vara rättad. [Debian](https://security-tracker.debian.org/tracker/CVE-2026-111497) · [Ubuntu](https://ubuntu.com/security/CVE-2026-111497) |
| **Brotli4j** | `1.24.0` samt matchande `native-*`-paket | En system-Brotli-fix rättar inte en gammal inbyggd native-kopia. [Advisory](https://github.com/hyperxpro/Brotli4j/security/advisories/GHSA-x524-jrqf-vwq6) |
| **Python Brotli/brotlicffi** | `Brotli 1.2.1` / `brotlicffi 1.2.1.0` | Kontrollera den Pythonmiljö som används. [Advisory](https://github.com/advisories/GHSA-9v8h-qqgm-4m35) |
| **3liz QGIS Server** | Avbild `4.2.3-1.9.7` angavs innehålla rättat paket | Gäller den särskilda avbilden, inte alla QGIS Server-installationer. [Underhållarärende](https://github.com/3liz/py-qgis-server/issues/137) |
| **ArcGIS Server** | Security 2026 Update 4 för `11.5`, `12.0`, `12.1` | Esris produktpatch, inte bevis om lokal installation. [Esri](https://support.esri.com/en-us/patches-updates/2026/arcgis-server-security-2026-update-4-patch) |
| **ArcGIS Mission Server** | Security 2026 Update 1 för `11.5`, `12.0`, `12.1` (9 nov) | Separat från ArcGIS Server-patchen. [Esri](https://support.esri.com/en-us/patches-updates/2026/arcgis-mission-server-security-2026-update-1-patch) |
| **ArcGIS Pro 3.5+** | Produktpatch **väntades** enligt Esris besked uppdaterat **9 november**, återkontrollerat 12 november | Daterat besked, inte ett påstående om aktuell status. Kontrollera även separat .NET. [Esri](https://www.esri.com/arcgis-blog/products/trust-arcgis/administration/brotli-vulnerabilities-broccoli2shell) |
| **FME** | Safe anger påverkan och att rättade 2026.1 till 2026.3-byggen förbereds. DuckDB separat. | Kontrollera Safes [CVE-specifika besked](https://support.safe.com/hc/en-us/articles/50040725669517-Brotli-Decoder-CVE-2026-111497-Is-FME-Impacted) för er version. Novemberartikeln om andra sårbarheter bekräftar inte en Brotli-rättning. |

En rättning i källkoden når inte automatiskt en installerad produkt eller den process som redan kör. Uppdatera via leverantörens kanal, starta om eller driftsätt om vid behov och kontrollera vilken beroendeversion som används. [Patchkedjan med tekniska distributionsvillkor](docs/patchkedjan.md) · [Bibliotek och inbyggda kopior](docs/bibliotek.md) · [Supply chain, hur sårbar kod följer med](docs/supply-chain.md).

## 5. Vad bör jag göra och vem ansvarar?

| Roll eller situation | Rimlig handling |
| --- | --- |
| **Vanlig GIS-användare** | Fortsätt använda godkänd programvara, följ säkerhetsuppdateringar och etablerade rutiner för filer/dataleveranser. Be GIS/IT om hjälp med uppdatering. Du behöver inte själv testa CVE:n. |
| **GIS-/systemförvaltare** | Inventera *aktiva* importvägar, integrationer och servrar. Skilj klient, backend, proxy och körande avkodare. Lämna versions-/driftkontroll till IT eller leverantör där det är deras ansvar. |
| **IT/drift och säkerhet** | Följ leverantörspatchar utan krav på lokal exploatering; prioritera publika aktiva parsrar och interna jobb med känsliga rättigheter. Kontrollera körande runtime, tjänstekonto och faktisk rättning. |
| **Leverantör/utvecklare** | Ange påverkan för specifik produktversion, avkodare, leveransform, säkerhetsrättning och kvarvarande osäkerhet; använd SBOM/VEX vid behov, men skilj beroendefynd från exploaterbarhet. |

**Prioritera aktiva vägar:** En publik tjänst med orättad Node.js-runtime där Express avkodar angriparpåverkade `br`-begäranden före autentisering behöver normalt hanteras skyndsamt. Detta gäller oavsett om tjänsten är ett GIS-system eller en helt annan Node.js-applikation. Ett internt automatiskt jobb med extern indata och känsliga behörigheter kan också vara viktigt. En äldre **oanvänd** lokal Node/npm-installation har normalt lägre akut prioritet för den här specifika vägen; **npm är pakethanteraren, inte en server**. Om användningen är okänd: **undersök**, anta varken ”sårbar” eller ”säker”. För ett ärvt GIS-system: börja med aktiva jobb och servrar, inte varje äldre shapefil.

**Patchrådet gäller oavsett prioriteringsordning:** Vänta inte på att själv reproducera CVE:n innan du installerar en säkerhetsuppdatering som leverantören rekommenderar för er berörda produktversion. Avkodningsvägens nåbarhet styr framför allt brådska, behov av kompletterande skydd och var rättningen måste införas.

Börja med fyra frågor: **Vilka data tar systemet emot och vilka hämtar det självt? Vilken process avkodar innehållet? Vilken Brotli-kopia och version används? Vem ansvarar för uppdateringen?** Om svaren saknas, be leverantören om ett versionsbundet besked i stället för att börja med exploateringstester. [Överlämning till IT/leverantör](docs/borja-har.md) · [Undersök okänd programvara](docs/ny-programvara.md) · [Riskkedja: process, behörighet och möjlig skada](docs/riskkedja.md) · [Skyddsåtgärder](docs/skydd.md).

## 6. Tekniska fördjupningar och källor

Fördjupningarna beskriver hur du kontrollerar beroenden, undersöker datavägar och verifierar rättningar i GIS-miljöer.

| Fråga | Fördjupning |
| --- | --- |
| Hur fungerar felet och vad kan följderna bli? | [CVE-mekanism](docs/cve-mekanism.md) · [Konsekvenser](docs/konsekvenser.md) |
| Hur kan data nå avkodaren? | [HTTP och API](docs/http-angreppsvagar.md) · [Filer/social engineering](docs/fil-angreppsvagar.md) · [Dataflöde till patch](docs/dataflode-till-patch.md) |
| Hur avgörs risk och lämplig skyddsåtgärd? | [Riskkedja](docs/riskkedja.md) · [Verksamhetsflöden](docs/verksamhetsfloden.md) · [Skydd](docs/skydd.md) |
| Hur når rättningen olika produkter? | [Patchkedja](docs/patchkedjan.md) · [Bibliotek](docs/bibliotek.md) · [Supply chain](docs/supply-chain.md) |
| Vilka produktbelägg finns? | [Produktöversikt](docs/produktoversikt.md) · [QGIS](docs/qgis.md) · [FME](docs/fme.md) · [ArcGIS](docs/arcgis.md) · [Origo Server](docs/origo-server.md) · [Hajk](docs/hajk.md) · [GeoServer](docs/geoserver.md) |
| Hur görs en säker undersökning och hur redovisas evidens? | [Metod](docs/metod.md) · [Okänd programvara](docs/ny-programvara.md) · [Källregister](docs/kallor.md) · [Roller och överlämning](docs/borja-har.md) |

**Publiceringsgräns:** GIS-relaterade angreppsvägar och osäkerheter beskrivs utifrån vad underlaget faktiskt visar. Uppgifter som underlättar kontroll, prioritering eller rättning hör hit; nya osamordnade produktfynd, felutlösande reproduktionsdetaljer och privata testloggar gör det inte. Mer forskning kan finnas än vad som publiceras. [Metod och bevisnivåer](docs/metod.md) · [Källregister](docs/kallor.md).

**Exemplen utanför GIS** finns i [produktöversikten](docs/produktoversikt.md#exempel-utanför-gis-och-deras-betydelse-för-gis) och används för att förklara två frågor: när en avkodare faktiskt är exponerad och vem som ansvarar för att uppdatera en medföljande kopia. [DuckDB i vissa QGIS-miljöer](docs/qgis.md#duckdb-i-qgis-pythonmiljö-en-separat-bibliotekskopia) behandlas som ett möjligt separat beroende med eget bevisläge, inte som ett nytt produktspecifikt exploitpåstående.

## Om projektet och författaren

Jag arbetar som GIS-ingenjör i en svensk kommun och driver det här projektet privat. Innehållet är inte ett officiellt besked från min arbetsgivare eller någon leverantör.

Jag började sammanställa CVE-2026-111497 eftersom uppgifter om GIS-program, bibliotek och rättningar var utspridda hos flera leverantörer. Här samlar jag sådant som hjälper andra att kontrollera och uppdatera sina GIS-miljöer. Jag publicerar bara de fynd som är relevanta för GIS och hjälper läsaren att kontrollera eller åtgärda en risk. Sammanställningen ersätter inte kontroll av den egna installationen.

Har du hittat ett fel, något som behöver förtydligas eller ett nytt leverantörsbesked? [Öppna ett ärende på GitHub](https://github.com/MelissaSkywalkz/broccoli2shell-gis/issues) eller skicka en pull request. Ange gärna källa, version och datum. Dela inte interna systemuppgifter, lösenord, personuppgifter eller privata säkerhetstester i öppna ärenden.

**Licens:** Originaltext CC BY 4.0 ([LICENSE](LICENSE)); externa källor omfattas inte av denna licens.

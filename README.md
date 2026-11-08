# Broccoli2Shell i GIS – vad vet vi, vad är osäkert och vad bör göras?

**CVE-2026-111497** är ett minnesfel i **Googles Brotli-avkodare**, inte en GIS-specifik sårbarhet. Brotli används för att packa upp komprimerad information och följer med i många bibliotek, runtimemiljöer, filformat och servertjänster **även utanför GIS**.

**Var finns felet – och varför nämns GIS-produkter?** CVE-felet finns i **Googles Brotli-avkodare**. Bredanvända bibliotek och körmiljöer, exempelvis **GDAL/Arrow, Node.js och .NET**, kan använda eller leverera berörd Brotli-kod genom sina beroenden. Det är inte en separat brist i QGIS, FME, ArcGIS, Hajk eller Origo Servers egen applikationskod som har påvisats här. **Men produkterna kan påverkas genom beroenden och behöva säkerhetsuppdateras.** Leverantörsbekräftad påverkan gäller fortfarande, utan att vara bevis för generell exploaterbarhet.

**Hajk och Origo Server är exempel på Node.js-beroendet.** Vissa installationer använder Node.js i sin backend och kan därför behöva uppdatera den *underliggande Node.js-runtimen* som tjänsten faktiskt kör. Express ger en möjlig väg till avkodaren, men ingen CVE-utlösning eller kodkörning har verifierats specifikt i dessa produkter här. Detsamma kan gälla många andra Node.js-tjänster, även helt utanför GIS.

**Bredden utanför GIS:** CVE-specifika leverantörsbesked eller rättningar finns också för arkivverktygsvarianten [7-Zip ZS (7-Zip-zstd)](https://github.com/mcmilk/7-Zip-zstd/releases/tag/v26.05-v1.5.7-R1) (en särskild arkivverktygsvariant, **inte** vanliga 7-Zip), utvecklarverktyget [Git for Windows](https://github.com/git-for-windows/build-extra/blob/main/ReleaseNotes.md) (medföljande Brotli rättat i 2.56.0(4)), JavaScript-körmiljön [Bun](https://github.com/oven-sh/bun/security/advisories/GHSA-663w-cv8g-jj2p) (rättat i 1.4.4), [PHP i Plesk för Windows](https://support.plesk.com/hc/en-us/articles/44567591006743-Vulnerability-CVE-2026-111497-Out-of-bounds-write-in-Brotli-bundled-with-PHP-on-Plesk-for-Windows) (berört Brotli via PHP:s cURL-tillägg) och brandväggssystemet [PAN-OS](https://security.paloaltonetworks.com/CVE-2026-111497) (villkorad exponering vid både Brotli-inspektion och TLS-dekryptering). [AWS](https://aws.amazon.com/security/security-bulletins/2026-155-aws/) beskriver dessutom påverkan på **Node.js- och .NET-körmiljöer i Lambda**, medan **själva den hanterade Lambda-tjänsten inte är påverkad**. Exemplen visar hur samma underliggande fel når vitt skilda teknikområden; de bevisar inte exploatering av varje produkt eller installation. [Källor och avgränsningar](docs/kallor.md#exempel-utanför-gis).

**Källkontroll: 8 november 2026.** Detta är en fristående vägledning, **inte** ett säkerhetsbesked från någon produktleverantör, en kontroll av din installation eller en lista över bekräftat exploaterbara GIS-program. Produkt- och rättningsstatus gäller det angivna kontrolldatumet; kontrollera senare leverantörsbesked inför en åtgärd.

## Börja här – behöver du göra något?

**Vad vet vi?** En särskilt felaktig Brotli-komprimerad dataström kan orsaka minneskorruption i en berörd avkodare. Vissa leverantörer har bekräftat produktpåverkan och publicerat rättningar. För andra GIS-produkter finns dokumenterade beroenden eller *möjliga* avkodningsvägar, men inte bekräftad produktspecifik exploatering.

**Vad vet vi inte?** Ett sårbart bibliotek i en installation bevisar inte att en angripare kan nå det eller att CVE:n kan utlösas där. Avsaknad av en verifierad angreppsväg betyder samtidigt **inte** att hela installationen är säker. Ingen av de GIS-produkter som kartlagts här har fått generell fjärrkodkörning (RCE) styrkt genom denna dokumentation.

**Vad bör göras ändå?** **Följ relevanta säkerhetsuppdateringar från leverantör eller distribution utan att invänta ett eget exploateringstest.** Använder du GIS: fortsätt med godkända verktyg och normala rutiner för uppdateringar och externa filer. Förvaltar du system: prioritera aktiva servrar, automatiska dataimporter och processer med känsliga behörigheter, och verifiera att rättningen når den *faktiskt använda* avkodaren.

En vanlig GIS-användare behöver inte göra DLL-analyser eller CVE-tester. GIS-förvaltning, IT/drift och leverantören delar på arbetet beroende på vem som äger produkten och dess uppdateringar. [Praktiskt stöd för ansvar och överlämning](docs/borja-har.md).

## 1. Vad är det som kan gå fel?

CVE-2026-111497 är en **out-of-bounds write** (skrivning utanför avsedd minnesbuffert) i Brotli-avkodning. Särskilt felaktig komprimerad indata kan få avkodaren i ett ogiltigt tillstånd och orsaka minneskorruption. **Inte varje trasig fil eller varje komprimerat svar utlöser felet.** [Node.js säkerhetsbesked](https://nodejs.org/en/blog/vulnerability/october-2026-security-releases) · [CVE-post](https://www.cve.org/CVERecord?id=CVE-2026-111497).

Fyra separata påståenden får inte blandas ihop:

1. **Berört beroende:** en version som omfattas av CVE:n finns i eller används av programvaran.
2. **Nåbar avkodare:** en viss fil, begäran eller datakälla behandlas faktiskt av den berörda funktionen.
3. **CVE-utlösning:** just det specifika felvillkoret kan nås genom denna väg.
4. **Påvisad säkerhetseffekt:** exempelvis krasch, minneskorruption, informationsröjande eller kodkörning har verkligen demonstrerats i den angivna miljön.

**Berört beroende ≠ nåbar avkodare ≠ utlöst CVE ≠ bekräftad säkerhetskonsekvens.** En uppdatering är ett separat femte påstående: den måste finnas i **den körande installationen**, inte bara i en uppströmsrelease.

En krasch kan ge tjänsteavbrott. Minneskorruption *kan* under särskilda förhållanden leda till mer allvarlig påverkan; informationsröjande och kodkörning kräver egna belägg. Forskare har demonstrerat kodkörning i en avgränsad testmiljö med en **officiell, oförändrad Node.js 24.22.0-binär på Linux arm64**. Det är **inte** bevis för RCE i QGIS, FME, ArcGIS, Origo Server, Hajk eller GeoServer. [Forskarens redogörelse](https://www.imperva.com/blog/node2shell-from-brotli-data-to-remote-code-execution/) · [Fördjupning om CVE-mekanismen](docs/cve-mekanism.md) · [Konsekvenser och deras beviskrav](docs/konsekvenser.md).

CISA lade till CVE:n i sin katalog över kända exploaterade sårbarheter (KEV) **27 oktober 2026**. Det stärker skälen att prioritera relevanta säkerhetsåtgärder men bevisar **inte** att ett särskilt GIS-system har angripits. [CISA KEV](https://www.cisa.gov/known-exploited-vulnerabilities-catalog?field_cve=CVE-2026-111497).

## 2. Hur kan berörd Brotli-kod nås?

| Möjlig indataväg | Vad krävs för att den ska vara relevant? | Exempel – inte bevis för påverkan |
| --- | --- | --- |
| **Inkommande HTTP-begäran** | En backend accepterar och avkodar `Content-Encoding: br`, eventuellt före autentisering. | Express/Node.js; separat konfigurerad request-decompression i .NET. |
| **Hämtat HTTP-svar** | En klient avkodar ett Brotli-komprimerat svar med berörd kod. | API/WMS/WFS via en faktisk HTTP-klient i exempelvis FME, QGIS, Python eller en integration. |
| **Filimport** | En fil innehåller Brotli-komprimerade delar som når en berörd filavkodare. | GeoParquet/Parquet via Arrow, eller annan parser med Brotli. Filändelsen behöver inte vara `.br`. |
| **Automatiskt jobb** | Ett schemalagt flöde behandlar ett sådant svar eller en sådan fil utan manuell öppning. | FME Flow, Pythonjobb, integrationsplattform eller importtjänst. |

Swagger/OpenAPI är ett **gränssnitt eller en API-beskrivning**, inte en egen Brotli-avkodare. Det är webbläsaren eller programmet som faktiskt gör HTTP-anropet som behöver kontrolleras. HTTP-komprimering och Brotli *inne i* en Parquet-fil är två skilda vägar. [HTTP-vägar](docs/http-angreppsvagar.md) · [Filvägar och social engineering](docs/fil-angreppsvagar.md) · [Från ett visst dataflöde till rätt patch](docs/dataflode-till-patch.md).

**”Jag använder QGIS men hämtar inga externa data.”** Det kan minska exponeringen via automatiska externa tjänster; du är **inte automatiskt exponerad** bara för att en berörd Brotli-kopia har funnits i en distribution. Men även en *lokalt sparad* GeoParquet-fil kan tidigare ha laddats ned manuellt eller skickats av någon annan. En möjlig avkodningsväg visar **inte** att filen kan utlösa CVE:n eller ge RCE i QGIS. Fortsätt använda QGIS normalt och installera relevanta officiella uppdateringar; IT/GIS kan vid behov kontrollera vilken Brotli-kopia installationen använder. [QGIS-bevisningen](docs/qgis.md).

**HTTPS är viktigt men inte ett fullständigt avkodarskydd.** Korrekt TLS/certifikatkontroll skyddar normalt mot manipulation under transport, inte mot data från en komprometterad legitim källa. Filvalidering, viruskontroll och tillfälliga `br`-spärrar kan minska risker men ersätter inte patchning. [Skydd och begränsningar](docs/skydd.md).

## 3. Produkter: vad vet vi, vad vet vi inte och vad bör göras?

**Status nedan gäller källunderlaget från 8 november 2026, inte läsarens installation.** ”Möjlig väg” betyder inte ”bekräftat exploaterbar produkt”. Leverantörens uttryckliga besked om påverkan väger tyngre än en hypotes om ett beroende.

| Produkt eller driftmiljö att kontrollera | Vad vet vi? Och vad saknas? | Vad bör göras ändå? |
| --- | --- | --- |
| **[QGIS](docs/qgis.md)** | **Berörd Brotli-kod via vanliga GIS-bibliotek**, med en dokumenterad GeoParquet → GDAL → Arrow-väg i specificerade byggen. Ingen QGIS-specifik CVE-utlösning eller RCE visad i det offentliga underlaget; andra vägar är mer osäkra. | Uppdatera via officiell QGIS-/OSGeo4W-distribution. Kontrollera faktiskt laddad Brotli-kopia vid behov. |
| **[FME Form/Flow](docs/fme.md)** | **Safe har rapporterat produktpåverkan via Brotli-relaterade komponenter** i HTTPCaller/OpenAPICaller, Parquet och DuckDB. Exakta CVE-rättade FME-byggen är **inte självständigt verifierade i öppet underlag**; Safes CVE-artikel kräver nu inloggning. | Följ Safes säkerhetsråd; begär ett versions- och plattformsspecifikt patchbesked. Prioritera viktiga automatiska flöden. |
| **[ArcGIS](docs/arcgis.md)** | **Esri bekräftar påverkan via underliggande komponenter** för Server 11.5/12.0/12.1 och Pro 3.5+, med Node.js- och .NET-relaterade rättningskedjor. Server 11.1/11.3 berörs enligt Esri inte av *just Node.js/Brotli-felet*. Ingen specifik extern CVE-endpoint eller generell ArcGIS-RCE redovisas här. | Installera Esris Server-patch där den gäller; följ Esris aktuella besked om Pro och uppdatera relevant .NET separat. |
| **[Origo Server – Node.js i backend](docs/origo-server.md)** | En möjlig väg till **Brotli-avkodaren som följer med Node.js** stöds av Express och granskad källkod. Inget eget CVE-fel i Origos applikationskod eller någon Origo-specifik exploatering har påvisats här. Webbklienten är separat. | Kontrollera om Node-backend körs; uppdatera och verifiera berörd Node.js-runtime enligt driftmiljöns och Node-projektets råd. |
| **[Hajk – Node.js i vissa backenddistributioner](docs/hajk.md)** | Hajks backend kan använda **Brotli-avkodaren som följer med Node.js** via Express. Backend-ZIP använder driftmiljöns Node.js; full officiell Docker innehåller Node.js; statiska `hajk-simple` saknar Node-backend i drift. Inget eget CVE-fel i Hajks kod eller någon Hajk-specifik exploatering har påvisats här. | ZIP: uppdatera driftmiljöns Node.js. Full Docker: välj uppdaterad image (t.ex. `nightly-master`), **hämta och återskapa** containern och verifiera den körande Node-versionen. |
| **[GeoServer/GeoWebCache](docs/geoserver.md)** | **Ingen generell produktpåverkan belagd.** Separata Jetty/Brotli4j-moduler eller tillägg *kan* vara relevanta; att standardinstallationen använder en berörd avkodare är inte fastställt. | Inventera bara faktiskt använda moduler/native-kopior och rätta berörda beroenden när de finns. |
| **Frends, Pentaho/Kettle, myCarta, Gaida, EntryScape m.fl.** | **Inte produktspecifikt fastställt här.** Att programmet har HTTP, Java eller .NET är inte tillräcklig bevisning. | Kartlägg aktiv dataingång och avkodare, följ produktleverantörens besked och uppdatera berörda komponenter. [Exempel på verksamhetsflöden](docs/verksamhetsfloden.md). |

**Hajks Docker-uppdateringar:** Enligt [Hajks säkerhetspolicy](https://github.com/hajkmap/Hajk/blob/master/SECURITY.md#official-docker-images) och [Docker-anvisningar](https://github.com/hajkmap/Hajk/blob/master/Docker/README.md#keeping-the-base-image-up-to-date) bygger projektet automatiskt om den **senaste releasen** som `nightly-master` varje natt, med en aktuell grundavbild för Node.js och operativsystemet. Versionsbundna releaseavbilder byggs däremot inte om mellan releaser; en kort versionstagg kan flyttas vid en ny patchrelease. **En nypublicerad image uppdaterar inte en redan körande installation automatiskt:** driftansvarig måste hämta den och återskapa containern, själv eller via ett kontrollerat automatiskt uppdateringsflöde. Nattlig ombyggnad uppdaterar inte heller Hajks npm-beroenden utanför deras låsta versioner. Kontrollera faktisk Node-version efter uppdatering; en viss tagg är inte i sig ett CVE-fixintyg.

**QGIS, FME och ArcGIS har olika bevisstatus men samma bakomliggande CVE-ursprung.** Det vore fel att sammanfatta alla tre som lika exploaterbara – eller att kalla en av dem ”säker” därför att CVE-utlösningen inte demonstrerats offentligt. **En produkt kan påverkas av CVE:n utan att dess utvecklare har orsakat grundfelet.** Ett sakligt besked anger **vilken version, vilket beroende och vilken typ av belägg** det avser, och rekommenderar relevant uppdatering. [Detaljerad produktöversikt med separata möjliga vägar](docs/produktoversikt.md).

## 4. Vilka rättningar och besked fanns den 8 november 2026?

Detta är **tillgängliga versioner eller daterad leverantörsstatus**, inte ett intyg om att en viss installation är uppdaterad. En distribution kan ha bakåtporterade säkerhetsrättningar och flera Brotli-kopior. Följ leverantörens rekommenderade paket för just er version och plattform.

| Komponent | Publicerad rättning eller status per kontrolldatum | Viktig avgränsning / källa |
| --- | --- | --- |
| **Google Brotli** | `1.0.10`, `1.1.1`, `1.2.1` per berörd gren | Grundfix, inte automatiskt rättning för inbyggda kopior. [Advisory](https://github.com/advisories/GHSA-9v8h-qqgm-4m35) |
| **Node.js** | Första rättade i grenarna: `22.23.4`, `24.22.1`, `26.12.1` | Kontrollera den runtime som *tjänsten kör*; 24.22.2 eller senare rekommenderas framför 24.22.1 på grund av separat regression. [Node](https://nodejs.org/en/blog/vulnerability/october-2026-security-releases) |
| **.NET** | `8.0.33`, `9.0.22`, `10.0.14`, `.NET 11 RC3` | Gäller Microsofts angivna Brotli-vägar, inte automatiskt varje HTTP-klient. Berörd HTTP-avkodning är inte alltid påslagen som standard. Self-contained/NativeAOT måste byggas/distribueras om; .NET Framework omfattas inte av just denna advisory. [Microsoft](https://github.com/dotnet/runtime/security/advisories/GHSA-p3jp-9v83-76w9) |
| **PowerShell 7** | `7.4.22`, `7.5.13`, `7.6.9` | Rätta den version schemalagda jobb faktiskt använder. [MSRC](https://www.microsoft.com/en-us/msrc/blog/2026/10/patch-now-php2shell-brotli-emudhra-guidance-for-customers) |
| **Arrow/PyArrow** | `26.0.1` för angivna distribuerade binärer | Systemlänkad Arrow kan använda annan Brotli-kopia. [Arrow](https://arrow.apache.org/blog/2026/11/03/26.0.1-release/) |
| **QGIS/OSGeo4W** | Brotli-paket `1.2.1-1` tillgängligt | Kontrollera distribuerad/installerad DLL. [OSGeo4W](https://download.osgeo.org/osgeo4w/v2/x86_64/setup.ini.bz2) |
| **Debian/Ubuntu** | Säkerhetsbackportar i distributionspaket | En äldre grundversion kan vara rättad. [Debian](https://security-tracker.debian.org/tracker/CVE-2026-111497) · [Ubuntu](https://ubuntu.com/security/CVE-2026-111497) |
| **Brotli4j** | `1.24.0` samt matchande `native-*`-paket | En system-Brotli-fix rättar inte en gammal inbyggd native-kopia. [Advisory](https://github.com/hyperxpro/Brotli4j/security/advisories/GHSA-x524-jrqf-vwq6) |
| **Python Brotli/brotlicffi** | `Brotli 1.2.1` / `brotlicffi 1.2.1.0` | Kontrollera den Pythonmiljö som används. [Advisory](https://github.com/advisories/GHSA-9v8h-qqgm-4m35) |
| **3liz QGIS Server** | Avbild `4.2.3-1.9.7` angavs innehålla rättat paket | Gäller den särskilda avbilden, inte alla QGIS Server-installationer. [Underhållarärende](https://github.com/3liz/py-qgis-server/issues/137) |
| **ArcGIS Server** | Security 2026 Update 4 för `11.5`, `12.0`, `12.1` | Esris produktpatch, inte bevis om lokal installation. [Esri](https://support.esri.com/en-us/patches-updates/2026/arcgis-server-security-2026-update-4-patch) |
| **ArcGIS Pro 3.5+** | Produktpatch **väntades** enligt Esris besked uppdaterat **7 november** | Daterat besked, inte ett påstående om aktuell status. Kontrollera även separat .NET. [Esri](https://www.esri.com/arcgis-blog/products/trust-arcgis/administration/brotli-vulnerabilities-broccoli2shell) |
| **FME** | Påverkan rapporterad; **CVE-specifika rättade byggnummer inte självständigt verifierade här** | ”Inte verifierat” betyder **inte** ”rättning saknas”. Begär produktversionsbesked från Safe. [Säkerhetsuppdateringar](https://support.safe.com/hc/en-us/articles/46792031942029-FME-November-2026-Security-Updates) |

**En upstreamfix, en tillgänglig produktpatch och en verifierat rättad körprocess är tre olika saker.** Uppdatera via rätt leverantörskanal, starta om eller driftsätt om där det krävs och kontrollera faktisk beroendeversion. [Patchkedjan med tekniska distributionsvillkor](docs/patchkedjan.md) · [Bibliotek och inbyggda kopior](docs/bibliotek.md) · [Supply chain – hur sårbar kod följer med](docs/supply-chain.md).

## 5. Vad bör jag göra och vem ansvarar?

| Roll eller situation | Rimlig handling |
| --- | --- |
| **Vanlig GIS-användare** | Fortsätt använda godkänd programvara, följ säkerhetsuppdateringar och etablerade rutiner för filer/dataleveranser. Be GIS/IT om hjälp med uppdatering. Du behöver inte själv testa CVE:n. |
| **GIS-/systemförvaltare** | Inventera *aktiva* importvägar, integrationer och servrar. Skilj klient, backend, proxy och körande avkodare. Lämna versions-/driftkontroll till IT eller leverantör där det är deras ansvar. |
| **IT/drift och säkerhet** | Följ leverantörspatchar utan krav på lokal exploatering; prioritera publika aktiva parsrar och interna jobb med känsliga rättigheter. Kontrollera körande runtime, tjänstekonto och faktisk rättning. |
| **Leverantör/utvecklare** | Ange påverkan för specifik produktversion, avkodare, leveransform, säkerhetsrättning och kvarvarande osäkerhet; använd SBOM/VEX vid behov, men skilj beroendefynd från exploaterbarhet. |

**Snabb prioritering:** En **publik tjänst som använder en opatchad Node.js-runtime och vars Express-parsers avkodar angriparpåverkade `br`-begäranden före autentisering** behöver normalt hanteras skyndsamt. Detta gäller oavsett om tjänsten är ett GIS-system eller en helt annan Node.js-applikation. Ett internt automatiskt jobb med extern indata och känsliga behörigheter kan också vara viktigt. En äldre **oanvänd** lokal Node/npm-installation har normalt lägre akut prioritet för den här specifika vägen; **npm är pakethanteraren, inte en server**. Om användningen är okänd: **undersök**, anta varken ”sårbar” eller ”säker”. För ett ärvt GIS-system: börja med aktiva jobb och servrar, inte varje äldre shapefil.

**Patchrådet gäller oavsett prioriteringsordning:** Vänta inte på att själv reproducera CVE:n innan du installerar en säkerhetsuppdatering som leverantören rekommenderar för er berörda produktversion. Avkodningsvägens nåbarhet styr framför allt brådska, behov av kompletterande skydd och var rättningen måste införas.

För att avgränsa en okänd miljö räcker det ofta att svara på: **Vad tar vi emot? Vilket program/process avkodar det? Vilket bibliotek och vilken version används? Vem ansvarar för uppdateringen?** Om svaret inte går att få lokalt: be leverantören om ett versionsbundet besked i stället för att börja med exploateringstester. [Överlämning till IT/leverantör](docs/borja-har.md) · [Undersök okänd programvara](docs/ny-programvara.md) · [Riskkedja: process, behörighet och möjlig skada](docs/riskkedja.md) · [Skyddsåtgärder](docs/skydd.md).

## 6. Tekniska fördjupningar och källor

README ger den samlade bilden. Dokumenten nedan ger **belägg, tekniska detaljer och exempel**; de ska inte behöva läsas för att förstå huvudslutsatserna ovan.

| Fråga | Fördjupning |
| --- | --- |
| Hur fungerar felet och vad kan följderna bli? | [CVE-mekanism](docs/cve-mekanism.md) · [Konsekvenser](docs/konsekvenser.md) |
| Hur kan data nå avkodaren? | [HTTP och API](docs/http-angreppsvagar.md) · [Filer/social engineering](docs/fil-angreppsvagar.md) · [Dataflöde till patch](docs/dataflode-till-patch.md) |
| Hur avgörs risk och lämplig skyddsåtgärd? | [Riskkedja](docs/riskkedja.md) · [Verksamhetsflöden](docs/verksamhetsfloden.md) · [Skydd](docs/skydd.md) |
| Hur når rättningen olika produkter? | [Patchkedja](docs/patchkedjan.md) · [Bibliotek](docs/bibliotek.md) · [Supply chain](docs/supply-chain.md) |
| Vilka produktbelägg finns? | [Produktöversikt](docs/produktoversikt.md) · [QGIS](docs/qgis.md) · [FME](docs/fme.md) · [ArcGIS](docs/arcgis.md) · [Origo Server](docs/origo-server.md) · [Hajk](docs/hajk.md) · [GeoServer](docs/geoserver.md) |
| Hur görs en säker undersökning och hur redovisas evidens? | [Metod](docs/metod.md) · [Okänd programvara](docs/ny-programvara.md) · [Källregister](docs/kallor.md) · [Roller och överlämning](docs/borja-har.md) |

**Bevis- och publiceringsgräns:** Officiella leverantörsbesked, öppet dokumenterade kod-/beroendekedjor och ännu obekräftade angreppsvägar redovisas separat. Inga privata testloggar, känsliga organisationsuppgifter eller osamordnade CVE-utlösande reproduktionsdetaljer ingår. Frånvaron av offentlig produktspecifik bevisning betyder **inte** att en angreppsväg är omöjlig. Se [metoden](docs/metod.md) och [källregistret](docs/kallor.md).

**Licens:** Originaltext CC BY 4.0 ([LICENSE](LICENSE)); externa källor omfattas inte av denna licens.

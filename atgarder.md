# Kontroller och åtgärder

**Lägesbild: 6 november 2026** · [Till produktöversikten](README.md)

Här redovisas källbelagd påverkan, tillgängliga rättningar och kontroller för olika installationsformer. **Rättningen kan levereras genom programmet, en separat körmiljö, ett operativsystemspaket eller en containeravbild. Ett separat säkerhetsbesked från GIS-projektet är inte en förutsättning för att en beroendeuppdatering ska finnas.** Felet gäller **uppackning** av Brotli: för att utlösa det behöver programmet behandla manipulerade, felaktigt formaterade Brotli-data från en källa som en angripare kan påverka, till exempel ett HTTP-svar eller en fil. Att en komponent har en sårbar version visar inte ensamt vilka sådana datavägar som finns i produkten eller att kodkörning har demonstrerats. En publicerad rättning är inte heller bevis för att den installerats eller används i drift.

[FME](#fme-form-och-flow) · [ArcGIS](#arcgis) · [QGIS/OSGeo4W](#qgis-och-osgeo4w) · [Origo Server](#origo-server) · [Hajk](#hajk) · [Pythonpaket](#pythonpaket-i-gis-miljöer)

## FME Form och Flow

**Bekräftad påverkan enligt Safe Software:** FME Engine, FME Form, FME Flow och FME Flow Hosted **2024.2–2026.3**. FME Flow berörs genom de FME Engines som kör arbetsytor; Safe anger att Flow-webbapplikationen inte använder Brotli och att FME inte levererar Node.js.

**Källa:** [Safes säkerhetsbesked om CVE-2026-111497](https://support.safe.com/hc/en-us/articles/50040725669517-Brotli-Decoder-CVE-2026-111497-Is-FME-Impacted), kontrollerat 6 november 2026.

### Vad kan beröras?

Safe anger att FME Workbench kan påverkas även utan en körande arbetsyta, genom förhandsvisning i HTTPCaller/OpenAPICaller och skapande eller förnyelse av vissa webbanslutningar. I FME Engine är möjliga vägar HTTPCaller/OpenAPICaller, tokenförnyelse, Parquet med Brotli-komprimerade kolumner samt DuckDB:s läsare och skrivare.

**Versionsavgränsning:** Safes sammanfattningstabell slutar vid **2026.3**, men artikelns detaljtext anger att **2024.2–2026.4 på Windows och macOS** innehåller Brotli `1.1.0`. Därför bör 2026.4 inte betraktas som opåverkad enbart för att den saknas i tabellen. Kontrollera aktuell versions- och bygginformation hos Safe.

### Uppdatering och tillfälliga åtgärder

- **Windows och macOS:** Safe förbereder rättade FME-byggen för **2026.1, 2026.2 och 2026.3** med Brotli `1.2.1`. Tidigare versioner hänvisas till uppgradering när rättade byggen finns. Att uppdatera operativsystemets Brotli rättar inte den kopia som FME levererar här.
- **Linux:** För HTTP-klienten och Parquet använder FME operativsystemets Brotli. Följ Linuxdistributionens säkerhetsuppdateringar. **DuckDB använder en egen kopia**, som kräver en FME-uppdatering. Safe anger att rättningen i DuckDB-projektet ännu inte ingick i någon utgiven version vid kontrollen.
- **FME Flow Hosted:** Ubuntu-paketet uppdateras automatiskt på körande instanser, men Safe anger att **FME Flow behöver startas om eller instansen startas om** för att rättningen ska användas. Äldre Ubuntu än 22.04 behöver uppgraderas. **DuckDB kräver fortfarande en separat FME-rättning**.

I väntan på rättning rekommenderar Safe att endast använda betrodda tjänster för HTTPCaller, OpenAPICaller och webbanslutningar, bara läsa Parquet- och DuckDB-data från betrodda källor samt undvika Workbench-förhandsvisning mot obetrodda adresser. Granska även schemalagda arbetsytor och begränsa där det är möjligt Brotli-komprimerade HTTP-svar till FME med proxy eller brandvägg. Intern åtkomst till Flow hindrar inte att en arbetsyta hämtar externa data.

## ArcGIS

Esri bekräftar påverkan på **ArcGIS Enterprise** och **ArcGIS Pro 3.5 och senare**. Det krävs olika åtgärder för ArcGIS Server-patchen och de .NET-körmiljöer som Esri anger ska uppdateras separat.

**Källor:** [Esris produktbesked, uppdaterat 5 november 2026](https://www.esri.com/arcgis-blog/products/trust-arcgis/administration/brotli-vulnerabilities-broccoli2shell), [ArcGIS Server Security 2026 Update 4 Patch, publicerad 5 november 2026](https://support.esri.com/en-us/patches-updates/2026/arcgis-server-security-2026-update-4-patch) och [Microsofts säkerhetsbesked](https://github.com/dotnet/announcements/issues/494).

### ArcGIS Server – produktpatch

Enligt Esris patchsida från **5 november** finns versionsanpassade nedladdningslänkar för **ArcGIS Server 11.5, 12.0 och 12.1**, på både **Windows och Linux**. Patchen åtgärdar `BUG-000188308` genom att uppdatera Node.js för CVE-2026-111497.

Installera rätt patch för **version och plattform** enligt [Esris installationsanvisningar](https://support.esri.com/en-us/patches-updates/2026/arcgis-server-security-2026-update-4-patch). Esri anger att ArcGIS Server-tjänsten stoppas och startas av patchinstallationen, att Linuxpatcharnas ordning spelar roll och att patchen ska installeras på varje berörd ArcGIS Server-installation. [PatchFinder](https://support.esri.com/en-us/patches-updates/2026/arcgis-server-security-2026-update-4-patch#how-to-identify-which-arcgis-products-are-installed) visar installerade produkter och patchar. En patch på en server bekräftar inte att andra servrar är uppdaterade.

**ArcGIS Pro 3.5 och senare:** Påverkad, men patch inväntas enligt Esris produktbesked uppdaterat **5 november**. Kontrollera nyare status direkt hos Esri.

**Custom Data Feeds:** Esri anger att funktionen kan återaktiveras **efter att relevant ArcGIS Server-patch installerats**, om den tidigare stängts av som tillfällig åtgärd. Funktionen är enligt Esris tidigare besked avstängd som standard. Följ Esris anvisningar för den version som används.

### .NET – separat uppdatering

Esri anger att följande använder en **.NET-körmiljö som användaren uppdaterar separat**: **ArcGIS Pro**, **ArcGIS Enterprise Web Adaptor (IIS)**, **Enterprise SDK SOE/SOI** och **ArcGIS Cloud Builder**. Esri anger också att appar byggda med **ArcGIS Maps SDK for .NET 200.0 eller senare** för Windows eller Android kan beröras; för dessa är Esris åtgärd att uppdatera .NET och **bygga om appen**, inte att uppdatera SDK:t.

Microsoft anger följande rättade .NET-versioner för CVE-2026-111497: **`8.0.33`, `9.0.22` och `10.0.14`**. [Microsofts besked](https://github.com/dotnet/announcements/issues/494) anger också att applikationer ska startas om efter uppdatering. Om en applikation distribuerats *self-contained* behöver den byggas om och distribueras igen med rättad körmiljö. Detta är en generell .NET-regel; kontrollera Esris råd för den konkreta produkten.

Att en .NET-körmiljö finns installerad innebär inte i sig exploaterbarhet. Microsoft anger att programmet måste behandla angriparstyrda Brotli-data genom den berörda avkodningen.

## QGIS och OSGeo4W

QGIS-miljöer har **dokumenterade beroenden till Brotli**, bland annat via curl (HTTP) och libjxl (JPEG XL) i OSGeo4W. **QGIS-projektet underhåller själv Windowsdistributionen via OSGeo4W**, där en rättad Brotli-version finns tillgänglig. På Ubuntu/Debian hanteras säkerhetsuppdateringar av systembibliotek genom respektive Linuxdistribution. QGIS beskriver denna ansvarsfördelning på sin säkerhetssida. **Kontrollera och uppdatera de beroendepaket som den aktuella installationen använder.**

**Källor:** [QGIS säkerhetsinformation](https://www.qgis.org/resources/support/security/), [OSGeo4W:s curl-byggskript](https://github.com/jef-n/OSGeo4W/blob/c8514a953de4b50a6dd4887fb02a874c8d74eec9/src/curl/osgeo4w/package.sh), [libjxl-byggskript](https://github.com/jef-n/OSGeo4W/blob/c8514a953de4b50a6dd4887fb02a874c8d74eec9/src/libjxl/osgeo4w/package.sh), [OSGeo4W:s paketregister](https://download.osgeo.org/osgeo4w/v2/x86_64/setup.ini.bz2), [Debians säkerhetsspårning](https://security-tracker.debian.org/tracker/CVE-2026-111497) och [Ubuntus säkerhetsbesked](https://ubuntu.com/security/CVE-2026-111497).

### Windows

I QGIS-projektets Windowsmiljö **OSGeo4W** erbjöds det rättade Brotli-paketet **`1.2.1-1`** vid kontrollen av paketregistret den **5 november 2026**. OSGeo4W kan uppdatera enskilda bibliotek utan att hela QGIS behöver installeras om. Kontrollera vilken OSGeo4W-installation som används och uppdatera relevanta paket enligt [QGIS säkerhetsanvisningar](https://www.qgis.org/resources/support/security/) och [OSGeo4W:s dokumentation](https://trac.osgeo.org/osgeo4w/). Paketregistret visar vad som **erbjuds**, inte vad som redan har installerats eller används av en körande process.

QGIS erbjuder också **fristående Windows-installerare**. Följ [QGIS installationsanvisningar](https://qgis.org/download/) för rätt installationsform; det går inte att fastställa den fristående installationens Brotli-status från OSGeo4W:s paketregister.

### Linux

För Debian och Ubuntu tillhandahålls rättningen via **distributionernas egna säkerhetsuppdateringar**, enligt QGIS-projektets anvisningar för systemberoenden. Följande Brotli-paket anges som rättade av respektive distribution:

| Distribution | Rättat Brotli-paket enligt distributionen |
| --- | --- |
| Debian 12 | `1.0.9-2+deb12u1` |
| Debian 13 | `1.1.0-2+deb13u1` |
| Ubuntu 22.04 LTS | `1.0.9-2ubuntu0.1` |
| Ubuntu 24.04 LTS | `1.1.0-2ubuntu0.1` |

Uppdatera via distributionens säkerhetsuppdateringar och kontrollera installerat paket. En rättad systemversion verifierar inte eventuella bibliotek som ett program distribuerar separat.

### 3liz QGIS Server – konkret paketkorrigering

I [3liz/py-qgis-server ärende #137](https://github.com/3liz/py-qgis-server/issues/137) rapporteras att containeravbilden `3liz/qgis-map-server:4.2.2-1.9.7` hade ett Brotli-paket utan Debians säkerhetsrättning. Rapportören beskriver även att `libcurl` och Qt:s nätverksbibliotek är länkade mot Brotli-avkodaren och pekar på externa WMS/WFS-svar som en möjlig dataväg. En [projektföreträdare bekräftar](https://github.com/3liz/py-qgis-server/issues/137#issuecomment-6437344689) att avbilden ersattes av `4.2.3-1.9.7` med rättat `libbrotli1`. Det är en dokumenterad paketkorrigering i denna distribution, **inte ett demonstrerat utnyttjande av CVE:n eller en bedömning av alla QGIS Server-installationer**. Kontrollera att en uppdaterad avbild verkligen används av den körande containern.

## Origo Server

Origo Servers [installationsanvisning](https://github.com/origo-map/origo-server) kräver Node.js och beskriver både manuell start och Windows-tjänst. Vid [granskad kodrevision `bacabe4`](https://github.com/origo-map/origo-server/blob/bacabe4b02611e8cbabf49daf57fe3d220f2e169/app.js) använder servern `express.json()` och `express.urlencoded()`. [Express beskriver](https://expressjs.com/en/blog/2026-10-31-security-releases/#a-note-on-brotli-decompression-cve-2026-111497) att dessa som standard kan avkoda förfrågningar med `Content-Encoding: br` genom Node.js:s Brotli-avkodare. **Det visar en möjlig kodväg, inte bekräftad påverkan i en viss Origo-installation.**

### Kontrollera Node.js

Enligt [Node.js-projektets säkerhetsrelease den 21 oktober](https://nodejs.org/en/blog/vulnerability/october-2026-security-releases) är de första rättade versionerna:

| Versionsgren | Första rättade version |
| --- | --- |
| `22.x` | `22.23.4` |
| `24.x` | `24.22.1` |
| `26.x` | `26.12.1` |

Äldre och utgångna Node.js-grenar, till exempel **20.x**, får inte dessa rättningar från Node.js-projektet. Följ [projektets supportstatus](https://nodejs.org/en/about/eol) och välj en stödd, kompatibel version.

**Välj rätt uppdateringsväg:** Dessa versionsnummer gäller Node.js-projektets utgåvor. [Debian](https://security-tracker.debian.org/tracker/CVE-2026-111497) anger att dess Node.js-paket använder systemets Brotli. [Red Hat](https://access.redhat.com/security/cve/cve-2026-111497) beskriver skillnader mellan RHEL-versioner och Node.js-paket: vissa använder systembiblioteket, andra en inbyggd kopia. Kontrollera därför Node.js **och relevant Brotli-paket** mot den aktuella distributionens säkerhetsbesked.

### Kontrollera den körmiljö som Origo använder

Om Origo Server körs som Windows-tjänst, identifiera **den Node.js-process som hör till tjänsten** och dess körfil. Exempel på en **läsande fråga** i PowerShell, baserad på [Microsofts `Win32_Process`-dokumentation](https://learn.microsoft.com/en-us/windows/win32/cimwin32prov/win32-process):

```powershell
Get-CimInstance Win32_Process -Filter "Name = 'node.exe'" |
  Select-Object ProcessId, CreationDate, ExecutablePath, CommandLine
```

Identifiera rätt `ProcessId` via tjänstens startkonfiguration och kommandoraden. Kontrollera sedan versionen på **just den körfilen**; ersätt `1234` med processens ID:

```powershell
$p = Get-CimInstance Win32_Process -Filter "ProcessId = 1234"
& $p.ExecutablePath --version
```

**Begränsning:** Kommandot startar en ny process och visar **filens version på disk**. Det bevisar inte vilken version en redan körande process laddade vid sin start. Uppdatera enligt distributionsanvisningen, **starta om Origo Server** och identifiera sedan den **nya** processen och dess körfil igen. Om sökvägen inte kan läsas eller processen inte kan kopplas till Origo Server är kontrollen inte slutförd. Kommandot har granskats mot dokumentation men inte funktionstestats på en Origo Server-installation.

## Hajk

Enligt [Hajks release-dokumentation](https://github.com/hajkmap/Hajk/blob/master/docs/releases.md) finns följande distributionsformer:

| Paket eller avbild | Vad innehåller det vid drift? |
| --- | --- |
| `hajk-vX.Y.Z-nodejs.zip` | Backend, klient och admin; **Node.js-körmiljön ingår inte** |
| `hajk-vX.Y.Z-simple.zip` | Endast statiska klientfiler, utan backend |
| `ghcr.io/hajkmap/hajk` | Fullständig Docker-avbild med Node.js-backend och körmiljö |
| `ghcr.io/hajkmap/hajk-simple` | Statiska klientfiler via nginx, **ingen Node.js-körmiljö i driftavbilden** |

**ZIP med backend:** Kontrollera och uppdatera Node.js **separat** i den miljö där backend körs. Se [Node.js-versioner och distributionsskillnader](#kontrollera-nodejs). Att ersätta ZIP-paketet uppdaterar inte automatiskt Node.js-körmiljön.

**Officiell Docker:** Enligt [Hajks säkerhetspolicy](https://github.com/hajkmap/Hajk/blob/master/SECURITY.md#official-docker-images) byggs vanliga releaseavbilder en gång. `nightly-master` bygger om den senaste releasen varje natt med en aktuell grundavbild för att få med Node.js- och OS-rättningar mellan Hajk-releaser. Hajk beskriver också versionsserietaggar för produktion. Välj rätt avbild och tagg enligt [projektets anvisningar](https://github.com/hajkmap/Hajk/blob/master/docs/releases.md), hämta uppdaterad avbild och återskapa containern. Kontrollera efteråt vilken avbild och Node.js-version som faktiskt körs. För `hajk-simple` gäller ingen Node.js-backend vid drift.

**Versionsstöd:** Hajk ger säkerhetsrättningar endast till den senaste releasen. Äldre versionsserier får inga nya rättningar när en ny release har kommit; uppgradera då till den senaste releasen. [Hajks säkerhetspolicy](https://github.com/hajkmap/Hajk/blob/master/SECURITY.md#supported-versions).

## Pythonpaket i GIS-miljöer

GitHubs [säkerhetsråd för CVE-2026-111497](https://github.com/advisories/GHSA-9v8h-qqgm-4m35) anger följande rättade **PyPI-paket**:

| Paket | Berörda versioner | Rättad version |
| --- | --- | --- |
| `Brotli` | `>=1.0.3`, `<1.2.1` | `1.2.1` |
| `brotlicffi` | `>=1.0.3`, `<1.2.1.0` | `1.2.1.0` |

Kontrollera **den Pythonmiljö som GIS-programmet använder**. En systeminstallerad Python kan skilja sig från programmets miljö. För QGIS via OSGeo4W används den aktuella installationens OSGeo4W Shell; ArcGIS Pro har en [Python Command Prompt och conda-miljöer](https://pro.arcgis.com/en/pro-app/3.6/arcpy/get-started/what-is-conda.htm). Följ Esris anvisningar om klonade miljöer i stället för att ändra ArcGIS Pros standardmiljö direkt.

I den aktuella Pythonmiljön kan installerade Pythonpaket listas utan att ändras:

```cmd
python -m pip show Brotli brotlicffi pyarrow
```

För en ArcGIS Pro-miljö som använder conda kan även conda-installerade paket kontrolleras med ett [dokumenterat namnfilter](https://docs.conda.io/projects/conda/en/stable/commands/list.html):

```cmd
conda list "brotli|pyarrow"
```

Kommandona visar paketversioner **men verifierar inte vilken Brotli-avkodare applikationen faktiskt använder**. Uppdatera enligt distributions- eller produktanvisningarna.

### PyArrow

[Apache Arrow bekräftar](https://arrow.apache.org/blog/2026/11/03/26.0.1-release/) att sårbarheten berörde Brotli-koden i förbyggda **PyArrow-hjul**. **PyArrow 26.0.1**, publicerad **3 november 2026**, innehåller en rättad medföljande Brotli-version. Andra distributioner kan paketera biblioteken annorlunda; kontrollera paketkällan innan uppdatering.
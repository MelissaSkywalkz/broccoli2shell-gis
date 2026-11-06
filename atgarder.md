# Kontroller och åtgärder

**Uppdaterad: 6 november 2026** · [Till produktöversikten](README.md)

Den här sidan beskriver kontroller och åtgärder för Broccoli2Shell (CVE-2026-111497) per produkt. Källorna är leverantörernas säkerhetsbesked eller dokumenterade paket och beroenden. Paketuppgifter visar vilka komponenter som ingår, men bekräftar inte hur en viss produkt påverkas.

Välj produkt nedan. Varje avsnitt börjar med en kort beskrivning och vad som behöver kontrolleras. Om du inte sköter installationen själv kan du använda kontrollpunkterna som underlag för den som gör det. Tekniska detaljer finns längre ned i respektive avsnitt. Följ anvisningarna för den version och plattform som används. En version som saknas i en patchlista är inte därmed bekräftat opåverkad.

Gå direkt till: [FME](#fme-form-och-flow) · [ArcGIS](#arcgis) · [QGIS/OSGeo4W](#qgis-och-osgeo4w) · [Origo Server](#origo-server) · [Hajk](#hajk) · [Pythonpaket](#pythonpaket-i-gis-miljöer)

## FME Form och Flow

Safe bekräftar påverkan för FME Engine, FME Form, FME Flow och FME Flow Hosted från **2024.2 till och med 2026.3**. Kontrollera vilken FME-version som används och följ Safes aktuella åtgärdsråd.

**Källa:** [Safe Softwares säkerhetsbesked](https://support.safe.com/hc/en-us/articles/50040725669517-Brotli-Decoder-CVE-2026-111497-Is-FME-Impacted), kontrollerat 6 november 2026.

### Kontroll och åtgärd

Safe förbereder rättade byggen av **FME 2026.1, 2026.2 och 2026.3** med Brotli 1.2.1. Användare av äldre versioner rekommenderas att planera uppgradering till någon av dessa när rättade byggen finns. DuckDB har enligt Safe fått en rättning i projektet, men en utgiven version med rättningen saknades vid kontrollen.

I väntan på rättning rekommenderar Safe bland annat att:

- använda HTTPCaller och webbanslutningar bara mot tjänster man litar på
- läsa Parquet- och DuckDB-data bara från betrodda källor
- undvika förhandsvisning av HTTP-anrop mot obetrodda adresser i Workbench
- se över schemalagda eller automatiskt startade arbetsytor med externa HTTP-anrop eller externa Parquet- och DuckDB-data
- där det är möjligt blockera Brotli-komprimerade svar till FME med proxy eller brandvägg
- ansluta FME AI Assist bara till AI- och MCP-servrar man litar på.

En **FME Flow som bara nås internt** är enligt Safe skyddad mot angripare utanför nätverket, men arbetsytor som hämtar externa data kan ändå ta emot skadliga Brotli-data. För Linux och FME Flow Hosted gäller även de särskilda anvisningarna nedan.

### Berörda funktioner och versionsuppgifter

Safe skriver också att FME 2024.2–2026.4 på Windows och macOS innehåller Brotli 1.1.0. Det är en annan övre gräns än i artikelns uppgift om berörda versioner. Kontrollera därför exakt version och byggnummer i Safes artikel.

Safe beskriver följande vägar till Brotli-avkodaren:

- I Workbench kan förhandsvisning med HTTPCaller och OpenAPICaller samt skapande eller förnyelse av webbanslutningar med OAuth 2.0 eller tokenbaserad inloggning beröras. Det gäller även när ingen arbetsyta körs.
- I FME Engine kan HTTPCaller och OpenAPICaller ta emot Brotli-komprimerade svar. Webbanslutningar som hämtar nya åtkomsttoken kan också beröras.
- Parquet-filer med Brotli-komprimerade kolumner och DuckDB:s egen kopia av Brotli berörs.

FME Flow berörs enligt Safe genom de FME Engines som kör arbetsytor. Safe anger att Flow-webbapplikationen och tjänsterna inte använder Brotli och att FME inte levererar Node.js.

### Linux och FME Flow Hosted

På **Linux** använder FME enligt Safe operativsystemets Brotli-bibliotek, utom i DuckDB. Safe rekommenderar att systemets Brotli-paket uppdateras. Distributioner kan ha rättat en äldre versionsgren; kontrollera distributionsbeskedet i stället för att enbart jämföra versionsnummer. Uppdateringen hanterar enligt Safe vägarna via HTTP-klienten och Parquet; DuckDB kräver FME-uppdateringen.

För **FME Flow Hosted** anger Safe att Ubuntus säkerhetsuppdateringar installeras automatiskt på körande instanser, men att FME Flow eller instansen behöver startas om. Instanser med Ubuntu äldre än 22.04 behöver enligt Safe uppgraderas till en stödd version. Basavbildens innehåll och DuckDB behöver bedömas separat. Följ Safes plattformsspecifika anvisningar i säkerhetsbeskedet ovan.

### Separat säkerhetsbesked från Safe

Safe har också publicerat [FME November 2026 Security Updates](https://support.safe.com/hc/en-us/articles/46792031942029-FME-November-2026-Security-Updates). Det gäller andra sårbarheter i Safes egen kod och innehåller rättningar även för äldre FME-versioner. Dessa uppdateringar behöver kontrolleras och hanteras separat. En novemberpatch är inte i sig en bekräftad rättning av Broccoli2Shell (CVE-2026-111497).

## ArcGIS

Esri bekräftar påverkan för **ArcGIS Enterprise** och **ArcGIS Pro 3.5 eller senare**. Kontrollera vilken produktversion som används och vilka produktpatchar och .NET-uppdateringar Esri rekommenderar. För ArcGIS Pro anger Esri att **patch ännu inväntas** den 6 november 2026. För **ArcGIS Online** anger Esri att begränsande åtgärder införts och att uppdateringar pågår.

**Källor:** [Esris säkerhetsbesked](https://www.esri.com/arcgis-blog/products/trust-arcgis/administration/brotli-vulnerabilities-broccoli2shell), [Esris patchsida](https://support.esri.com/en-us/patches-updates/2026/arcgis-server-security-2026-update-4-patch) och [Microsofts .NET-besked](https://github.com/dotnet/announcements/issues/494). Esris besked, patchsida och Microsofts besked kontrollerades 6 november 2026.

### Kontroll och åtgärd

Enligt Esri åtgärdar **ArcGIS Server Security 2026 Update 4 Patch** CVE-2026-111497 genom en uppdatering av Node.js (`BUG-000188308`). Esri listar ArcGIS Server `11.5`, `12.0` och `12.1`. Kontrollera på Esris patchsida ovan vilka nedladdningar som är tillgängliga för din version.

Rättningen gäller ArcGIS Server, inte automatiskt andra delar av ArcGIS Enterprise. Följ Esris anvisningar för varje komponent. Esri rekommenderar att äldre versioner i livscykelfaserna Mature eller Retired uppgraderas.

Esri rekommenderar också att **ArcGIS Enterprise SDK Custom Data Feeds** stängs av i väntan på relevant patch. Funktionen är enligt Esri avstängd som standard. Kontrollera även .NET-körmiljöerna som Esri anger behöver uppdateras separat, se avsnittet nedan.

### .NET och särskilda fall

Esri anger att bland annat ArcGIS Pro, ArcGIS Web Adaptor för IIS och Enterprise SDK SOE/SOI använder .NET, som behöver uppdateras separat. Microsoft anger rättade .NET-versioner **`8.0.33`, `9.0.22` och `10.0.14`**. .NET Framework omfattas enligt Microsoft inte av denna sårbarhet.

Enligt Microsoft kräver exponering att programmet packar upp angriparstyrda Brotli-data, exempelvis med `BrotliStream` eller särskilt aktiverad automatisk uppackning. Microsofts beskrivning gäller .NET generellt, inte inställningarna i varje ArcGIS-komponent. Följ Esris rekommendation om att uppdatera .NET.

## QGIS och OSGeo4W

Börja med att kontrollera hur QGIS är installerat. För QGIS via OSGeo4W på Windows finns ett uppdaterat Brotli-paket att kontrollera. På Debian och Ubuntu gäller distributionens säkerhetsuppdateringar.

Vi har inte hittat något QGIS- eller OSGeo4W-specifikt säkerhetsbesked om denna CVE. Uppgifterna bygger på paket- och beroendeinformation, inte på bekräftad påverkan från QGIS-projektet.

**Källor:** [QGIS säkerhetssida](https://www.qgis.org/resources/support/security/), [OSGeo4W:s paketregister](https://download.osgeo.org/osgeo4w/v2/x86_64/setup.ini.bz2), [OSGeo4W:s byggskript](https://github.com/jef-n/OSGeo4W/tree/c8514a953de4b50a6dd4887fb02a874c8d74eec9/src), [Debian](https://security-tracker.debian.org/tracker/CVE-2026-111497) och [Ubuntu](https://ubuntu.com/security/CVE-2026-111497). Paketregistret kontrollerades 5 november, övriga underlag 6 november 2026.

### Kontroll och åtgärd på Windows

OSGeo4W:s paketregister erbjuder det rättade paketet `brotli` `1.2.1-1`.

För installationer som hanteras av **OSGeo4W Network Installer**:

1. Stäng QGIS och öppna installeraren med `Advanced Install`.
2. Kontrollera installationskatalogen och sök efter paketet `brotli`.
3. Uppdatera till `1.2.1-1` eller en senare rättad version vid behov.
4. Kontrollera paketversionen och testa QGIS efter uppdateringen.

Frivillig kontroll i **OSGeo4W Shell** för den installation som ska granskas:

```cmd
findstr /B /I /C:"brotli " "%OSGEO4W_ROOT%\etc\setup\installed.db"
```

Kommandot visar Brotli-paketets installerade version enligt OSGeo4W:s register och ändrar ingenting. Det funktionstestades lokalt den 4 november 2026. Kontrollen visar inte vilken Brotli-fil andra bibliotek läser in eller vad andra QGIS-installationer innehåller.

För fristående QGIS-installationer och andra plattformar, följ [QGIS installations- och uppdateringsanvisningar](https://qgis.org/resources/installation-guide/).

### Kontroll och åtgärd på Debian och Ubuntu

QGIS hänvisar till distributionens pakethanterare för beroenden på Debian och Ubuntu. Uppdatera relevanta paket enligt distributionens instruktioner. Säkerhetsrättningar kan ha förts in i äldre grundversioner:

| Distribution | Rättad paketversion |
| --- | --- |
| Debian 12 | `1.0.9-2+deb12u1` |
| Debian 13 | `1.1.0-2+deb13u1` |
| Ubuntu 22.04 LTS | `1.0.9-2ubuntu0.1` |
| Ubuntu 24.04 LTS | `1.1.0-2ubuntu0.1` |

För Ubuntu 20.04 och äldre anger Ubuntu att säkerhetsrättningen för relevanta paket kan kräva Ubuntu Pro. Kontrollera paketstatus för den version som används.

### Tekniska beroenden i OSGeo4W

OSGeo4W:s byggskript vid commit `c8514a9` anger `brotli` som körberoende för Arrow, som GDAL använder för Parquet, och för libjxl (JPEG XL). Arrow byggs med systemets Brotli, och för libjxl anges `JPEGXL_FORCE_SYSTEM_BROTLI=ON`. Skripten beskriver byggkonfigurationen vid den commiten, inte vilka filer en viss installation innehåller.

## Origo Server

Origo Server använder Node.js, som har fått säkerhetsrättningar för sårbarheten. Kontrollera vilken Node.js-version tjänsten kör och uppdatera vid behov. Kopplingen bygger på projektets källkod och säkerhetsbesked för dess komponenter, inte på ett separat säkerhetsbesked från Origo-projektet.

**Källor:** [Origo Servers källkod](https://github.com/origo-map/origo-server), [Express säkerhetsbesked](https://expressjs.com/en/blog/2026-10-31-security-releases/#a-note-on-brotli-decompression-cve-2026-111497) och [Node.js säkerhetsrelease](https://nodejs.org/en/blog/vulnerability/october-2026-security-releases). Kontrollerat 6 november 2026.

### Kontroll och uppdatering av Node.js

**Första rättade Node.js-versioner** enligt Node.js-projektet:

| Versionsgren | Första rättade version |
| --- | --- |
| `22.x` | `22.23.4` |
| `24.x` | `24.22.1` |
| `26.x` | `26.12.1` |

1. Kontrollera vilken Node.js-installation **Origo Server-tjänsten faktiskt använder**, exempelvis med `node --version` i motsvarande miljö.
2. Kontrollera hur Node.js distribueras. Officiella Node.js-byggen och Linuxdistributionernas Node.js-paket kan använda Brotli på olika sätt. Följ distributionens säkerhetsbesked för **både Node.js och Brotli** där sådana paket används.
3. Uppdatera till en rättad och kompatibel version enligt aktuell leverantörs- eller distributionsinformation.
4. Starta om tjänsten eller skapa om containern från en uppdaterad avbild. Kontrollera sedan den version som faktiskt körs.

För Node.js-versioner utanför support, se [Node.js EOL-information](https://nodejs.org/en/about/eol). Express beskriver en tillfällig spärr för inkommande Brotli-förfrågningar när Node.js inte kan uppdateras och applikationen inte behöver sådana förfrågningar. Spärren skyddar inte utgående HTTP-anrop och ersätter inte en uppdatering.

### Tekniska belägg och avgränsningar

Vid [Origo Server-commit `bacabe4`](https://github.com/origo-map/origo-server/blob/bacabe4b02611e8cbabf49daf57fe3d220f2e169/package.json) anges Express `^5.2.1`. [app.js](https://github.com/origo-map/origo-server/blob/bacabe4b02611e8cbabf49daf57fe3d220f2e169/app.js) använder `express.json()` och `express.urlencoded()` utan ändrad inställning för uppackning.

Express anger att standardinställningarna kan packa upp förfrågningar med `Content-Encoding: br`. Node.js beskriver möjliga följder när en angripare kan tillföra data till avkodaren: krasch, röjt minnesinnehåll och kodkörning. En installerad Origo Server kan skilja sig från den granskade koden.

## Hajk

Hajk finns med eller utan en serverdel som kör Node.js. Kontrollera vilken variant som används. Om serverdelen ingår behöver Node.js-versionen kontrolleras. Simple Edition saknar den serverdelen vid drift.

**Källor:** [Hajks releases-dokumentation](https://github.com/hajkmap/Hajk/blob/master/docs/releases.md), [säkerhetspolicy](https://github.com/hajkmap/Hajk/blob/master/SECURITY.md), [Dockerfile](https://github.com/hajkmap/Hajk/blob/badc79df70407a95669aed59c8632f9b3419185d/Dockerfile) och [Node.js säkerhetsrelease](https://nodejs.org/en/blog/vulnerability/october-2026-security-releases). Kontrollerat 6 november 2026.

### Kontrollera distributionsform

| Paket eller avbild | Node.js vid drift enligt Hajks dokumentation |
| --- | --- |
| `hajk-vX.Y.Z-nodejs.zip` | Backend, klient och admin; Node.js-backend |
| `hajk-vX.Y.Z-simple.zip` | Endast statiska klientfiler; ingen Node.js-backend |
| `ghcr.io/hajkmap/hajk` | Fullständig avbild med Node.js-backend |
| `ghcr.io/hajkmap/hajk-simple` | Statiska klientfiler via nginx; ingen Node.js-backend |

### Kontroll och uppdatering av Hajk

För zip-installation med backend: kontrollera Node.js-versionen, uppdatera vid behov enligt [tabellen över rättade Node.js-versioner](#kontroll-och-uppdatering-av-nodejs), starta om backend och verifiera versionen.

För Docker-installationer: kontrollera vilken avbild och tagg som används. Hajks [versionsanvisningar](https://github.com/hajkmap/Hajk/blob/master/docs/releases.md) beskriver de officiella avbilderna och hur de uppdateras. Enligt projektet byggs releaseavbilder en gång, medan `nightly-master` byggs om varje natt med en aktuell grundavbild. Det fastställer inte vilken Node.js-version en viss publicerad avbild innehåller.

Se [Hajks paketregister](https://github.com/hajkmap/Hajk/pkgs/container/hajk) för avbilder och taggar. Kontrollera vilken Node.js-version den avbild som används faktiskt innehåller och jämför med [rättade Node.js-versioner](#kontroll-och-uppdatering-av-nodejs). Följ Hajks anvisningar vid uppdatering. En redan körande container behöver återskapas från den uppdaterade avbilden.

### Teknisk bakgrund

För `hajk-simple` används Node.js enligt [Dockerfile.simple](https://github.com/hajkmap/Hajk/blob/badc79df70407a95669aed59c8632f9b3419185d/Docker/Dockerfile.simple) när avbilden byggs, men inte i den färdiga avbildens driftmiljö. Kontrollera byggmiljön separat.

Hajks backend anger Express `^5.2.1` och använder `Express.json()` och `Express.urlencoded()` vid [commit `badc79d`](https://github.com/hajkmap/Hajk/blob/badc79df70407a95669aed59c8632f9b3419185d/apps/backend/package.json). Express beskriver hur dessa funktioner kan använda Node.js:s Brotli-avkodare, se [Origo Servers tekniska belägg](#tekniska-belägg-och-avgränsningar).

## Pythonpaket i GIS-miljöer

QGIS, ArcGIS Pro och andra GIS-program kan innehålla egna Pythonmiljöer även om du inte använder Python själv. Den som hanterar miljön behöver kontrollera om de berörda paketen finns installerade. Använd kontrollkommandona nedan i rätt Pythonmiljö.

**Källor:** [GitHubs säkerhetsråd](https://github.com/advisories/GHSA-9v8h-qqgm-4m35), [urllib3:s källkod](https://github.com/urllib3/urllib3/blob/c5347bbd39203d68d7b0e3ffb83ced3e3b4b9109/src/urllib3/util/request.py), [Apache Arrows releasebesked](https://arrow.apache.org/blog/2026/11/03/26.0.1-release/), [Esris Python-dokumentation](https://pro.arcgis.com/en/pro-app/3.6/arcpy/get-started/what-is-conda.htm) och [Esris paketkanal](https://anaconda.org/esri/libbrotlidec). Kontrollerat 6 november 2026.

### Kontroll och åtgärd

| Paket | Berörda versioner enligt GitHubs säkerhetsråd | Rättad version |
| --- | --- | --- |
| `Brotli` | Från `1.0.3` till före `1.2.1` | `1.2.1` |
| `brotlicffi` | Från `1.0.3` till före `1.2.1.0` | `1.2.1.0` |

Kontrollera **den Pythonmiljö som GIS-programmet faktiskt använder**, inte bara den Python som ligger först i systemets `PATH`.

- **QGIS via OSGeo4W på Windows:** Öppna OSGeo4W Shell för den aktuella QGIS-installationen och kör kommandot nedan.
- **ArcGIS Pro på Windows:** Öppna **Python Command Prompt** för den aktiva ArcGIS Pro-miljön. Kontrollera först vilken conda-miljö som är aktiv och kör sedan båda kommandona nedan.
- **Andra Pythonmiljöer:** Aktivera den miljö som ska granskas innan kommandot körs.

```cmd
python -m pip show Brotli brotlicffi pyarrow
```

För ArcGIS Pro, kontrollera även paket som installerats med conda:

```cmd
conda list brotli
```

`conda list brotli` visar paket vars namn innehåller `brotli`, även C-bibliotek som `libbrotlidec` i [Esris paketkanal](https://anaconda.org/esri/libbrotlidec). `pip show` visar inte alla conda-paket. Kommandona ändrar inget och visar paketversioner, inte vilken avkodare ett program faktiskt använder. Följ Esris anvisningar för ändringar i ArcGIS Pro-miljön. Paket från Linuxdistributionen kan ha rättningar som inte syns i grundversionsnumret; följ då distributionsbeskedet.

### pyarrow och Apache Arrow

[Apache Arrow](https://arrow.apache.org/blog/2026/11/03/26.0.1-release/) bekräftar att sårbarheten berörde PyArrow-hjul (förbyggda Pythonpaket). **PyArrow 26.0.1** innehåller en rättad version av den medföljande Brotli-koden. För en berörd PyArrow-installation från PyPI är 26.0.1 därför rättad version. Kontrollera versionen med kommandot ovan och uppdatera enligt paketets anvisningar vid behov.

Andra distributioner kan använda systemets Brotli i stället för en inbyggd kopia. För pyarrow i OSGeo4W, se [QGIS och OSGeo4W](#qgis-och-osgeo4w).

### Varför Pythonpaket kan beröras

När `Brotli` eller `brotlicffi` är installerat kan urllib3 enligt källkoden begära Brotli-komprimerade HTTP-svar. Även program som använder urllib3, exempelvis `requests`, kan beröras. Den faktiska exponeringen beror på vilka data programmet behandlar.
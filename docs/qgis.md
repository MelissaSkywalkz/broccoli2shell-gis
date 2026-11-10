# QGIS: filer, HTTP och separata Brotli-kopior

**Kontrollerat: 10 november 2026 · Windows-paket och separata indatavägar**  

Felet finns i Brotli-avkodaren som vissa QGIS-distributioner använder genom GDAL och Arrow. Någon separat brist i QGIS egen applikationskod har inte påvisats här. Kontrollera vilken Brotli-kopia installationen använder och var den kan nås av indata.

**Känt:** En beroende- och avkodningsväg från GeoParquet genom GDAL och Arrow till berörd Brotli-kod är dokumenterad i vissa QGIS-distributioner. HTTP via Qt/libcurl, JPEG XL och QGIS Server är **separata** vägar med annat bevisläge.

**Oklart:** Det offentliga underlaget här visar ingen QGIS-specifik CVE-utlösning eller kodkörning. Det visar varken att exploatering är omöjlig eller att QGIS generellt kan exploateras. Exponeringen skiljer sig också mellan arbetssätt och distributioner.

**Åtgärd:** Följ QGIS och OSGeo4W:s säkerhetsbesked och uppdatera berörda beroenden. En fristående Windows-MSI kan ha en medföljande OSGeo4W Setup (`bin/setup.bat`) för paketuppdatering i just den installationsroten. Pröva detta först i en isolerad testmiljö, granska föreslagna paketändringar och verifiera rätt laddad DLL och vanliga QGIS-arbetsflöden efteråt. Betrakta inte huvudpaketet Brotli 1.2.1 som bevis för att alla kopior, särskilt GRASS, är rättade. Undvik opålitliga Parquet-/GeoParquet-filer tills relevant rättning kunnat verifieras. Avinstallera inte QGIS enbart som CVE-åtgärd och byt inte DLL-filer manuellt. [Rättningsstatus per 10 november](patchkedjan.md).

**Källor och fördjupning:** [README](../README.md) · [Filimport](fil-angreppsvagar.md) · [Beroenden](bibliotek.md) · [Patchkedja](patchkedjan.md)

## Uppdatera QGIS på Windows och kontrollera GRASS samt andra bibliotek

**Fristående Windows-MSI, kontroll 10 november 2026:** QGIS nedladdningsarkiv listar `QGIS-OSGeo4W-3.44.15-2.msi` och `QGIS-OSGeo4W-4.2.3-2.msi`, daterade 6 oktober 2026. Enligt separata installationskontroller innehåller båda `brotlidec.dll` från Brotli 1.2.0. Det finns ingen senare rättad fristående MSI verifierad i detta underlag. Den medföljande OSGeo4W-pakethanteraren kan användas för att uppdatera paket i MSI-installationens egen rot. Det är ännu inte verifierat som en komplett CVE-rättning för dessa QGIS-versioner. Detta är en distributionsobservation, inte ett publicerat besked från QGIS Security eller bevis för RCE. [QGIS paketarkiv](https://qgis.org/downloads-list/) · [OSGeo4W-underhållarens besked](https://trac.osgeo.org/osgeo4w/ticket/893#comment:2) · [Paketeringsskript](https://github.com/jef-n/OSGeo4W/blob/master/src/setup/osgeo4w/package.sh) · [Brotli 1.2.1](https://github.com/google/brotli/releases/tag/v1.2.1).

**GRASS i OSGeo4W:** En lokal inventering efter uppdatering av huvudpaketet `brotli` till `1.2.1-1` visade en separat GRASS-medföljande avkodare som fortfarande var Brotli 1.2.0. GRASS Windows-paketering kan kopiera egna DLL-beroenden från byggmiljön. I det kontrollerade OSGeo4W-arkivet var senaste stabila `grass`-paketet `8.5.0-1` (12 juni 2026), medan `grass-dev` hade nyare byggningar. Ingen av dessa observationer bekräftar en rättad GRASS-kopia eller en aktiv CVE-ingång via GRASS. [GRASS-paket](https://www.norbit.de/osgeo4w/v2/x86_64/release/grass/) · [Brotli-paket](https://www.norbit.de/osgeo4w/v2/x86_64/release/brotli/) · [GRASS byggskript](https://github.com/OSGeo/grass/blob/main/mswindows/osgeo4w/package.sh).

**För användare:** installera QGIS genom den officiella distributionsvägen eller via GIS/IT. **För förvaltare:** kontrollera vilken installation som faktiskt används, vilka paket som innehåller Brotli och om en rättad version har nått dem. En hittad fil bevisar inte att avkodaren används. En publicerad patch bevisar inte att installationen har uppdaterats.

1. **Identifiera distributionen.** Skilj mellan [OSGeo4W v2 och fristående QGIS-installationer](https://www.qgis.org/resources/installation-guide/), parallella versioner och eventuellt separat [GRASS](https://grass.osgeo.org/download/). En fristående QGIS-katalog som innehåller OSGeo4W-paketmanifest ska inte därför uppdateras med OSGeo4W:s nätverksinstallerare.
2. **Använd rätt installerare.** Uppdatera OSGeo4W v2 med dess officiella nätverksinstallerare i avsedd installationsrot. Även en fristående MSI kan ha `bin/setup.bat` som startar OSGeo4W:s pakethanterare med installationens egen rot; detta bekräftas av paketeringsansvarig. I en isolerad testmiljö: säkerhetskopiera, stäng QGIS, starta medföljande Setup, kontrollera installationsroten och granska hela paketförslaget före eventuell uppdatering. Verifiera därefter faktisk Brotli-DLL och ordinarie QGIS-arbetsflöden. Det är ingen verifierat fullständig CVE-rättning, särskilt eftersom GRASS kan innehålla egen äldre kopia. Byt inte DLL-filer manuellt. [OSGeo4W-ärende #893](https://trac.osgeo.org/osgeo4w/ticket/893#comment:2) · [Setup-byggskript](https://github.com/jef-n/OSGeo4W/blob/master/src/setup/osgeo4w/package.sh).
3. **Verifiera berörda kopior.** OSGeo4W:s `etc/setup/installed.db` kan visa paketversioner och `*.lst.gz` kan visa paketägarskap. GRASS kan leverera egna `libbrotlidec`-filer oberoende av huvudpaketet `brotli`. Fastställ vilka kopior som tillhör aktiva program och kontrollera leverantörens rättning för respektive bygge. Uppdatera eller avinstallera äldre oanvända installationer genom ordinarie hantering; ersätt inte DLL-filer manuellt.
4. **Kontrollera drift och resultat.** En genväg, ett PyQGIS-skript eller ett schemalagt jobb kan starta en annan installation än den som nyss uppdaterades. Notera använd programväg och, där det är motiverat, vilka bibliotek processen faktiskt laddar. Ett normalt funktionstest visar att arbetsflödet fungerar, inte att CVE:n är rättad. [Inventering och bevisnivåer](ny-programvara.md).

### Bekräfta paketägarskap utan CVE-test

På en **egen, behörigt administrerad** Windows-installation kan ansvarig läsa `etc/setup/installed.db` och paketmanifesten `etc/setup/*.lst.gz` för att skilja ett `grass`-ägt bibliotek från en fil som levereras av `brotli`. Manifest och filnamn är **inventeringsbelägg**, inte bevis för versionen hos statiskt inbyggd kod eller en aktiv angreppsväg. Vid behov kan ett godkänt processverktyg, exempelvis [Microsoft Process Explorer](https://learn.microsoft.com/en-us/sysinternals/downloads/process-explorer), visa vilka DLL-sökvägar som faktiskt används. Kontrollera hela distributionskedjan före uppdatering och efter eventuell omstart.

## DuckDB i QGIS Pythonmiljö: en separat bibliotekskopia

Vissa QGIS-distributioner kan innehålla en **separat DuckDB-modul i Pythonmiljön** utöver GDAL/Arrow. DuckDB kan levereras med **inbyggd Brotli-kod**, som inte nödvändigtvis rättas när ett separat `brotlidec.dll`-paket uppdateras. [DuckDB:s ärende #27660](https://github.com/duckdb/duckdb/issues/27660) dokumenterade en tidigare vendorerad kopia utan upstreamrättning; ärendet stängdes efter [PR #27791](https://github.com/duckdb/duckdb/pull/27791). Det bekräftar inte vilken DuckDB-version som ingår i en given QGIS-distribution eller att en viss avkodare är nåbar via dess arbetsflöden.

**Kontroll:** om DuckDB ingår i installationen, identifiera dess faktiska paket/version och följ distributionsleverantörens rättning separat från system-Brotli. Installerad modul eller inbyggda avkodarsignaturer visar **beroende**, inte CVE-utlösning eller säkerhetseffekt. Vilka kopior en enskild installation innehåller måste kontrolleras där; privata inventeringsloggar och produktspecifika tester redovisas inte här. [Inventeringsmetod](ny-programvara.md) · [Patchkedja](patchkedjan.md).

## Om du bara arbetar med lokala data

Om du arbetar med egna, kontrollerade lokala data utan externa tjänster är du **inte automatiskt exponerad** via extern hämtning; det kan minska flera möjliga ingångar. Det betyder **inte** att hela QGIS-installationen är säker mot CVE:n: även en fil som öppnas lokalt kan ha laddats ned från ett forum, och projekt eller tillägg kan hämta nätverksdata. En GeoParquet-fil kan innehålla Brotli-komprimerade datasidor; det ger en möjlig indataväg, men ingen CVE-utlösning via en sådan fil har visats i det publika underlaget för QGIS. Fortsätt normal GIS-användning och uppdatera programmet även om du inte brukar hämta externa data. [Filer och social påverkan](fil-angreppsvagar.md) · [Supply chain](supply-chain.md#4-hur-ska-man-tänka-om-programmet-bara-används-lokalt).

## 1. GeoParquet → GDAL → Arrow → Brotli

```text
Externt tillförd GeoParquet-/Parquet-fil
  → QGIS / GDAL:s OGR-datakälla
  → GDAL Parquet-driver
  → Apache Arrow C++ Parquet-dekomprimering
  → berörd Brotli-avkodare (om datasidan använder Brotli)
```

**Varför kan data nå Brotli?** GeoParquet är Parquet med geodatametadata. Enskilda kolumndatasidor kan använda codec `BROTLI`. GDAL:s Parquet-driver använder **Arrow C++**. I den versionslåsta OSGeo4W-konfigurationen för Arrow finns `ARROW_PARQUET=ON`, `ARROW_WITH_BROTLI=ON` och `ARROW_DEPENDENCY_SOURCE=SYSTEM`. Det visar *hur* berört Brotli kan ingå i ett Windows-bygge. [GDAL Parquet](https://gdal.org/en/stable/drivers/vector/parquet.html) · [OSGeo4W Arrow byggrecept](https://github.com/jef-n/OSGeo4W/blob/c8514a953de4b50a6dd4887fb02a874c8d74eec9/src/arrow-cpp/osgeo4w/package.sh).

**Nödvändiga förutsättningar:** Filen kommer från en källa som angriparen kan påverka; relevant Parquet-läsare öppnar filen och begär den komprimerade sidan; sidan använder Brotli och en *berörd* avkodare laddas; felaktig komprimerad indata kan nå dess CVE-specifika kodväg. Enbart filändelsen `.parquet`, GeoParquet-metadata eller en DLL i installationskatalogen räcker inte. Parquet kan också använda exempelvis Snappy, ZSTD eller GZIP.

**Bevisgräns:** De offentliga bygg- och formatkällorna belägger beroende och teknisk avkodningsmekanism. De är **inte i sig** bevis för minneskorruption eller RCE i ett oförändrat QGIS-bygge. Nya, ännu inte samordnade produktspecifika testdetaljer inkluderas inte i detta publika repo. Sådan rapportering hör hemma hos [QGIS Security](https://www.qgis.org/resources/support/security/).

**Patchens placering:**

- **OSGeo4W:s separata Brotli-paket:** rättat `brotli 1.2.1-1` är tillgängligt. Kontrollera den DLL som processerna faktiskt laddar. Ett GRASS-paket kan innehålla en separat äldre DLL även efter denna uppdatering; uppdaterat huvudpaket är därför inte fullständig verifikation av hela QGIS-/GRASS-miljön. [Brotli-arkiv](https://www.norbit.de/osgeo4w/v2/x86_64/release/brotli/).
- **Arrow med egen inbyggd Brotli:** Apache Arrow **26.0.1** rättar den medföljande koden i berörda förbyggda PyArrow-wheelpaket och R-binärer. Detta säger *inte* att alla Arrow C++-byggen eller fristående QGIS-installationer automatiskt rättas. [Apache Arrow](https://arrow.apache.org/blog/2026/11/03/26.0.1-release/).
- **Fristående QGIS-MSI på Windows:** enligt verifierade paketkontroller innehåller `3.44.15-2` och `4.2.3-2` Brotli 1.2.0. Ingen nyare rättad fristående MSI har verifierats i QGIS nedladdningsarkiv per 10 november 2026. Medföljande OSGeo4W Setup kan uppdatera enskilda paket i samma MSI-installationsrot men kräver lokal efterkontroll; det är inte bekräftat som en komplett säkerhetsrättning. [QGIS arkiv](https://qgis.org/downloads-list/) · [underhållarbesked](https://trac.osgeo.org/osgeo4w/ticket/893#comment:2).
- **Linux:** Debian/Ubuntu levererar backportade Brotli-korrigeringar. [Debian](https://security-tracker.debian.org/tracker/CVE-2026-111497) · [Ubuntu](https://ubuntu.com/security/CVE-2026-111497). Exempel på distro-paket som listats som rättade i källunderlaget: Debian 12 `1.0.9-2+deb12u1`, Debian 13 `1.1.0-2+deb13u1`, Ubuntu 22.04 `1.0.9-2ubuntu0.1`, Ubuntu 24.04 `1.1.0-2ubuntu0.1`. Kontrollera aktuell distributionspost.

## 2. Externa HTTP-svar → Qt Network eller libcurl

```text
Externt WMS/WFS/WMTS-/API-svar med Content-Encoding: br
  → den nätverkskomponent som just detta anrop använder
  → eventuell automatisk Brotli-dekomprimering
  → QGIS behandlar avkodat innehåll
```

**Varför är spåret relevant?** Qt Network har funktionalitet för Brotli-dekomprimering av HTTP-svar. libcurl kan också avkoda `br`, men först när den relevanta avkodningskonfigurationen används. Vilken klient QGIS faktiskt använder kan skilja sig mellan en OGC-provider, GDAL:s nätverkssökvägar och andra tillägg. [Qt QNetworkRequest](https://doc.qt.io/qt-6/qnetworkrequest.html) · [libcurl Accept-Encoding](https://curl.se/libcurl/c/CURLOPT_ACCEPT_ENCODING.html).

En teknisk rapport för **en särskild 3liz QGIS Server-avbild** visar att både `libQt6Network` och `libcurl` länkas mot `libbrotlidec.so.1`. Rapportören pekar på externa WMS/WFS-svar som möjlig ingång. Detta ger stöd för att *undersöka* vägen, inte för att en manipulerad WMS/WFS-respons bevisligen utlöser CVE:n i en viss QGIS-installation. [3liz-ärende #137](https://github.com/3liz/py-qgis-server/issues/137).

**Nödvändiga förutsättningar:** Applikationen hämtar data från en angriparkontrollerad/komprometterad tjänst; klienten använder Brotli-avkodning för det svaret; rätt (berörd) Brotli-kopia laddas; felaktig komprimerad data passerar tidigare kontroller och når den sårbara avkodaren. HTTPS gör inte en komprometterad källas innehåll pålitligt.

**Bevisgräns:** Det offentliga källunderlaget som redovisas här verifierar inte en QGIS-specifik CVE-utlösning via WMS/WFS/HTTP. Det är **inte** ett bevis för att vägen är omöjlig. Att Qt har en kontroll för extrem expansionsgrad vid avkodning är inte ett verifierat skydd mot denna CVE.

**Patch:** Samma **faktiskt använda** Brotli-kopia behöver rättas som för andra programvägar; ibland räcker OS-paketet, ibland ligger biblioteket i en egen distribution. Parquet-rättningen ensam garanterar inget om nätverksstacken.

## 3. JPEG XL och andra bildvägar

`OSGeo4W → libjxl → Brotli` är en dokumenterad beroendemöjlighet, skild från Parquet och HTTP. JPEG XL kan innehålla Brotli-komprimerad metadata. Forskare från JFrog har visat ett CVE-relevant angrepp genom **FFmpeg → libjxl**, men det resultatet gäller inte automatiskt QGIS. [OSGeo4W libjxl](https://github.com/jef-n/OSGeo4W/blob/c8514a953de4b50a6dd4887fb02a874c8d74eec9/src/libjxl/osgeo4w/package.sh) · [JFrog-rapport](https://research.jfrog.com/post/brotli2shell-beyond-node2shell-and-http2shell/).

**Öppna frågor:** Vilka QGIS-funktioner/insticksprogram öppnar JPEG XL, vilken libjxl- och Brotli-version laddas, och når utifrån påverkad metadata verkligen den aktuella avkodaren?

## 4. QGIS Server i container: kontrollera avbildens rättning

[3liz/py-qgis-server #137](https://github.com/3liz/py-qgis-server/issues/137) dokumenterar att `3liz/qgis-map-server:4.2.2-1.9.7` hade ett Debian-Brotli-paket utan rättning. En projektföreträdare uppgav den **27 oktober 2026** att den ersattes av `4.2.3-1.9.7`, där `libbrotli1` korrigerats. Detta är en **bekräftad paketkorrigering i angiven avbild**, inte bekräftad RCE i QGIS Server eller garanti för andra 3liz-taggar. En ny image måste faktiskt hämtas och körande container återskapas.

## Samlad bedömning av de fyra vägarna

| Angreppsväg | Vad är belagt i det offentliga underlaget? | Vad återstår att klarlägga? | Rättning och prioritering |
| --- | --- | --- | --- |
| **GeoParquet → GDAL/Arrow** | Beroende-/avkodningskedjan till Brotli är dokumenterad för specificerade byggen. | Vilken kopia laddas i den aktuella QGIS-installationen? Nåbar CVE-specifik utlösning och säkerhetseffekt är inte offentligt styrkta här. | Följ officiell QGIS-/OSGeo4W-uppdatering och kontrollera berörd Brotli-/Arrow-kopia; prioritera aktiva importer av externa filer. |
| **HTTP/WMS/WFS → Qt/libcurl** | Brotli-stöd i nätverksbibliotek och länkning i en specificerad distribution är belagda. | Vilka QGIS-funktioner avkodar faktiska `br`-svar med en berörd kopia? Ingen QGIS-specifik CVE-utlösning redovisas här. | Uppdatera rätt nätverksbibliotek/distribution; undersök de HTTP-flöden som används. |
| **JPEG XL → libjxl** | Separat beroendesamband och en CVE-relevant demonstration i FFmpeg, **inte i QGIS**. | Använder QGIS-bygget bildvägen, och når attackerbar indata den berörda avkodaren? | Uppdatera berörda bibliotek enligt distributionen; inventera användningen om JPEG XL hanteras. |
| **QGIS Server / 3liz** | Berört paket i en äldre avbild och rättningsbesked för `4.2.3-1.9.7`. | Vilken avbild körs i drift och finns en faktisk CVE-relevant avkodningsväg? | Driftsätt rättad avbild, återskapa containern och verifiera körande version. |

**Bevisgräns:** Ingen av raderna innebär att CVE-specifik utlösning, minneskorruption eller RCE i QGIS har demonstrerats i **den publika dokumentation som redovisas här**. Frånvaron av ett sådant offentligt bevis utesluter inte en möjlig angreppsväg. Ett berört bibliotek eller en fungerande vanlig avkodning bevisar å andra sidan inte exploaterbarhet.

**Råd till användare och förvaltare:** Fortsätt normal GIS-användning och installera relevanta officiella säkerhetsuppdateringar utan att invänta egen CVE-reproduktion. Prioritera kartläggning av faktiska externa filimporter, HTTP-hämtningar och körande servrar efter exponering och konsekvens. GIS/IT kan kontrollera vilken bibliotekskopia processen faktiskt använder. [Versions- och patchstatus med källkontrolldatum](patchkedjan.md).

**Kontrollfråga för driftansvarig:** Vilket QGIS-bygge behandlar vilken angriparpåverkbar datakälla med vilken Brotli-kopia? Om uppgiften saknas är just den vägbedömningen ofullständig, inte ett bevis för vare sig säkerhet eller exploaterbarhet.

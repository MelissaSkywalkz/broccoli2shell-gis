# QGIS – filinläsning, HTTP-svar och flera Brotli-kopior

**Kontrollerat: 8 november 2026 · Bedömningen gäller separata indatavägar**  

**Var finns CVE-felet?** I **Brotli-avkodaren som vissa QGIS-distributioner använder genom standardbibliotek som GDAL och Arrow**, inte i någon här påvisad separat brist i QGIS egen applikationskod. Dessa bibliotek används även i många andra program. Det är den **faktiska paketeringen och avkodningen**, inte produktnamnet i sig, som behöver kontrolleras.

**Vad vet vi?** I vissa QGIS-distributioner är en beroende- och avkodningsväg via GeoParquet, GDAL och Arrow till berörd Brotli-kod dokumenterad. HTTP via Qt/libcurl, JPEG XL och QGIS Server är **separata** vägar med annat bevisläge.

**Vad vet vi inte?** Det offentliga underlaget här visar ingen QGIS-specifik CVE-utlösning eller kodkörning. Det är varken bevis för att exploatering är omöjlig eller för att QGIS generellt kan exploateras. Exponeringen skiljer sig också mellan arbetssätt och distributioner.

**Vad bör göras ändå?** Använd QGIS normalt och installera relevanta uppdateringar genom den officiella distributionen. GIS/IT bör vid behov kontrollera vilken Brotli-kopia installationen faktiskt använder; ett tillgängligt OSGeo4W-paket betyder inte att varje QGIS-installation redan är rättad. [Rättningsstatus per 8 november](patchkedjan.md).

**Tekniska belägg och vidare läsning:** [README](../README.md) · [Filimport](fil-angreppsvagar.md) · [Beroenden](bibliotek.md) · [Patchkedja](patchkedjan.md)

## Om du bara arbetar med lokala data

Om du arbetar med egna, kontrollerade lokala data utan externa tjänster är du **inte automatiskt exponerad** via extern hämtning; det kan minska flera möjliga ingångar. Det betyder **inte** att hela QGIS-installationen är säker mot CVE:n: även en fil som öppnas lokalt kan ha laddats ned från ett forum, och projekt eller tillägg kan hämta nätverksdata. En GeoParquet-fil kan innehålla Brotli-komprimerade datasidor; detta visar en möjlig indataväg, **inte** att en sådan fil har visats utlösa CVE:n i QGIS. Fortsätt normal GIS-användning och uppdatera programmet även om du inte brukar hämta externa data. [Filer och social påverkan](fil-angreppsvagar.md) · [Supply chain](supply-chain.md#4-hur-ska-man-tänka-om-programmet-bara-används-lokalt).

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

- **OSGeo4W med separat bibliotek:** rättat Brotli-paket `1.2.1-1` finns enligt paketunderlag kontrollerat 7 november. Kontrollera sedan *vilken* `brotlidec.dll` och vilket Arrow-bygge installationen använder. [OSGeo4W](https://trac.osgeo.org/osgeo4w/) · [paketregister](https://download.osgeo.org/osgeo4w/v2/x86_64/setup.ini.bz2) · [Brotli-recept](https://github.com/jef-n/OSGeo4W/blob/master/src/brotli/osgeo4w/package.sh).
- **Arrow med egen inbyggd Brotli:** Apache Arrow **26.0.1** rättar den medföljande koden i berörda förbyggda PyArrow-wheelpaket och R-binärer. Detta säger *inte* att alla Arrow C++-byggen eller fristående QGIS-installationer automatiskt rättas. [Apache Arrow](https://arrow.apache.org/blog/2026/11/03/26.0.1-release/).
- **Fristående QGIS på Windows:** paketregistret för OSGeo4W verifierar inte av sig självt DLL:erna i en redan installerad fristående version. Följ QGIS-installationens uppdateringsväg.
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

## 4. QGIS Server i container – en egen distributionsfråga

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

# Källregister och kontrolldatum

**Kontrollperiod: 7–8 november 2026.** Register över underlagen i den här versionen. En webbsidas hämtdatum är inte samma sak som dess publiceringsdatum. Säkerhetsläget kan ändras mellan kontroller. **Källgräns:** Brotli-/runtime-advisories styrker var den underliggande CVE:n sitter; produktleverantörers besked styrker angiven produktpåverkan; teknisk kodgranskning kan visa en möjlig beroendeväg. Dessa belägg ska inte förvandlas till en anklagelse mot en produktutvecklare eller till bevis för produktspecifik exploatering.

[Till översikten](../README.md) · [Supply chain](supply-chain.md) · [CVE-mekanism](cve-mekanism.md) · [HTTP](http-angreppsvagar.md) · [filer](fil-angreppsvagar.md)

## CVE:n och uppströmsrättningen

| Underlag | Används för | Status |
| --- | --- | --- |
| [CVE-2026-111497](https://www.cve.org/CVERecord?id=CVE-2026-111497) | CVE-identitet, felbeskrivning | Kontrollkälla |
| [GitHub Advisory GHSA-9v8h-qqgm-4m35](https://github.com/advisories/GHSA-9v8h-qqgm-4m35) | Brotli-versioner och Pythonpaket | Kontrollerad |
| [CISA KEV](https://www.cisa.gov/known-exploited-vulnerabilities-catalog?field_cve=CVE-2026-111497) | Känd exploatering av CVE:n generellt | Listad 27 oktober 2026; säger inget om enskilt GIS-program |
| [Node.js – 21 oktober 2026](https://nodejs.org/en/blog/vulnerability/october-2026-security-releases) | Orsak, konsekvens, berörda Node-grenar, rättade utgåvor | Officiell säkerhetsrelease |
| [Brotli 1.0.10](https://github.com/google/brotli/releases/tag/v1.0.10), [1.1.1](https://github.com/google/brotli/releases/tag/v1.1.1), [1.2.1](https://github.com/google/brotli/releases/tag/v1.2.1) | Uppströmsrättningar | Versionsbundna |
| [Debian tracker](https://security-tracker.debian.org/tracker/CVE-2026-111497), [Ubuntu security](https://ubuntu.com/security/CVE-2026-111497) | Paket med backportade rättningar | Distro-versioner måste alltid kontrolleras på nytt |

## Produkt- och beroendekedjor

| Produkt/spår | Primär källa | Vad den **inte** bevisar |
| --- | --- | --- |
| GDAL/GeoParquet | [GDAL:s Parquet-driver](https://gdal.org/en/stable/drivers/vector/parquet.html) | Produktspecifik CVE-utlösning/RCE i QGIS |
| QGIS Windows och beroenden | [QGIS Security](https://www.qgis.org/resources/support/security/), [OSGeo4W:s Arrow-recept](https://github.com/jef-n/OSGeo4W/blob/c8514a953de4b50a6dd4887fb02a874c8d74eec9/src/arrow-cpp/osgeo4w/package.sh) | Att alla installerade QGIS-byggen använder samma DLL |
| QGIS HTTP | [Qt Network](https://doc.qt.io/qt-6/qnetworkrequest.html), [libcurl](https://curl.se/libcurl/c/CURLOPT_ACCEPT_ENCODING.html) | Att QGIS WMS/WFS utlöser CVE:n |
| QGIS Server container | [3liz ärende #137](https://github.com/3liz/py-qgis-server/issues/137) | Att alla QGIS Server-versioner har samma status |
| Bildformat | [OSGeo4W libjxl-recept](https://github.com/jef-n/OSGeo4W/blob/c8514a953de4b50a6dd4887fb02a874c8d74eec9/src/libjxl/osgeo4w/package.sh), [JFrog separat studie av FFmpeg/libjxl](https://research.jfrog.com/post/brotli2shell-beyond-node2shell-and-http2shell/) | QGIS-exploiterbarhet via JPEG XL |
| Apache Arrow | [Arrow 26.0.1, 3 november 2026](https://arrow.apache.org/blog/2026/11/03/26.0.1-release/) | Att systemlänkat Arrow är rättat utan systempaket |
| FME | [Safes CVE-artikel](https://support.safe.com/hc/en-us/articles/50040725669517-Brotli-Decoder-CVE-2026-111497-Is-FME-Impacted), **läst 7 nov 2026; anonym åtkomst krävde inloggning 8 nov**. Tidigare avläst leverantörsuppgift om Form/Engine/Flow/Hosted samt HTTPCaller/OpenAPICaller, Parquet, DuckDB | Oberoende reproduktion av CVE:n i FME, fullständig publik verifierbarhet eller exakt CVE-rättat FME-byggnummer |
| FME tillgängliga utgåvor | [Downloads](https://fme.safe.com/downloads/), [changelog](https://fme.safe.com/changelog/), [novemberuppdateringar](https://support.safe.com/hc/en-us/articles/46792031942029-FME-November-2026-Security-Updates), öppet läsbara **8 nov 2026** | Att listade säkerhetsfixar eller hotfix-byggnummer är en bekräftad rättning för **denna CVE** |
| ArcGIS Server/Pro | [Esri produktbesked, uppdaterat 7 nov och avläst 8 nov 2026](https://www.esri.com/arcgis-blog/products/trust-arcgis/administration/brotli-vulnerabilities-broccoli2shell), [Security 2026 Update 4-patch, publicerad 5 nov](https://support.esri.com/en-us/patches-updates/2026/arcgis-server-security-2026-update-4-patch) | Vilken extern endpoint som når Node-avkodaren eller att Pro-patchens status är oförändrad senare |
| .NET | [Microsofts säkerhetsbesked](https://github.com/dotnet/announcements/issues/494) | Att Brotli avkodas automatiskt i standardkonfiguration |
| Origo Server | [`app.js`, revision bacabe4](https://github.com/origo-map/origo-server/blob/bacabe4b02611e8cbabf49daf57fe3d220f2e169/app.js), [Express](https://expressjs.com/en/blog/2026-10-31-security-releases/) | Att en specifik Origo-endpoint kan utlösa CVE:n |
| Hajk Backend och officiella images | [server.js](https://github.com/hajkmap/Hajk/blob/master/apps/backend/server/common/server.js), [Hajks säkerhetspolicy](https://github.com/hajkmap/Hajk/blob/master/SECURITY.md#official-docker-images), [releaser](https://github.com/hajkmap/Hajk/blob/master/docs/releases.md#docker-images), [Docker-anvisningar](https://github.com/hajkmap/Hajk/blob/master/Docker/README.md#keeping-the-base-image-up-to-date), [nattligt byggflöde](https://github.com/hajkmap/Hajk/blob/master/.github/workflows/nightly-master.yml), [utvecklarens förtydligande 6 nov](https://github.com/hajkmap/Hajk/commit/badc79df70407a95669aed59c8632f9b3419185d) och [PR i denna vägledning](https://github.com/MelissaSkywalkz/broccoli2shell-gis/pull/2) | Att alla Hajk-utgåvor innehåller Node-runtime, att npm-beroenden patchas vid varje nattbygge eller att en körande container uppdateras när en ny image publiceras |
| GeoServer/Jetty | [GeoServer 3.0.2 pom](https://github.com/geoserver/geoserver/blob/3.0.2/src/pom.xml), [Jetty start.ini](https://github.com/geoserver/geoserver/blob/3.0.2/src/release/jetty/start.ini), [Jetty moduler](https://jetty.org/docs/jetty/12.1/operations-guide/modules/standard.html) | Att Brotlis native-kod laddas av GeoServer som standard |
| Brotli4j | [GHSA-x524-jrqf-vwq6](https://github.com/hyperxpro/Brotli4j/security/advisories/GHSA-x524-jrqf-vwq6) | Att varje Java-/GeoServer-installation använder Brotli4j |
| DuckDB | [Offentlig rättningsdiskussion](https://github.com/duckdb/duckdb/issues/27660) | Att kodändringen levereras i viss FME-version |
| **myCarta** | Ingen verifierad versionslåst beroendekälla i denna granskning | All produktpåverkan, aktiv avkodning och patchstatus; används endast som inventeringsexempel |

## Exempel utanför GIS

Exemplen illustrerar **bredden hos samma Brotli-beroende**, inte en kartläggning av angrepp mot dessa produkter. Produktbeskedens villkor och rättningar gäller angivna versioner, distributionsformer och konfigurationer; de får inte tolkas som bekräftad kodkörning i varje program.

| Program eller tjänst | Primärkälla och verifierat besked | Viktig avgränsning |
| --- | --- | --- |
| **7-Zip ZS (7-Zip-zstd)** | [Ärende #579](https://github.com/mcmilk/7-Zip-zstd/issues/579) · [release 2 nov 2026](https://github.com/mcmilk/7-Zip-zstd/releases/tag/v26.05-v1.5.7-R1): uppdaterar medföljande Brotli till 1.2.1. | Gäller varianten ZS; inte belägg för generell påverkan i vanliga 7-Zip eller ZS-specifik RCE. |
| **Git for Windows** | [Officiella versionsanteckningar](https://github.com/git-for-windows/build-extra/blob/main/ReleaseNotes.md): 2.56.0(4), 26 okt 2026, uppdaterar medföljande Brotli till 1.2.1 för denna CVE. | En beroenderättning; visar inte i sig någon praktisk exploitväg i Git. |
| **Bun** | [Säkerhetsbesked GHSA-663w-cv8g-jj2p](https://github.com/oven-sh/bun/security/advisories/GHSA-663w-cv8g-jj2p), 23 okt 2026: berörda versioner från 1.0.21 före 1.4.4; rättat i 1.4.4. | Felet ligger i medföljande Brotli; Bun anger uttryckligen att Bun-specifik exploatering inte testats. |
| **PHP i Plesk för Windows** | [Plesks säkerhetsbesked](https://support.plesk.com/hc/en-us/articles/44567591006743-Vulnerability-CVE-2026-111497-Out-of-bounds-write-in-Brotli-bundled-with-PHP-on-Plesk-for-Windows), uppdaterat 29 okt 2026: rättade Windows-PHP-paket med Brotli 1.2.1. | Relevant för PHP:s cURL-tillägg med aktiverad automatisk HTTP-svarsavkodning; inkommande webbtrafik utlöser inte felet i sig. |
| **Palo Alto Networks PAN-OS** | [Leverantörens CVE-besked](https://security.paloaltonetworks.com/CVE-2026-111497), uppdaterat 31 okt 2026: beskriver påverkade versioner och rättningar. | För PAN-OS krävs att både Brotli-inspektion och TLS/SSL-dekryptering är aktiva; hanteringsgränssnittet anges inte vara berört. |
| **AWS Lambda** | [AWS Security Bulletin 2026-155](https://aws.amazon.com/security/security-bulletins/2026-155-aws/), uppdaterat 5 nov 2026: relevanta Node.js- och .NET-runtimes har fått uppdateringar. | **Själva Lambda-tjänsten är inte berörd.** Kontrollera runtime-uppdateringsläge; egna containeravbilder behöver byggas och distribueras om. |

## Från valfritt dataflöde till avkodaren – runtimes och patchar

| Källa | Styrker | Styrker inte |
| --- | --- | --- |
| [Microsofts .NET out-of-band-uppdatering 25 oktober 2026](https://devblogs.microsoft.com/dotnet/dotnet-october-2026-oob-security-updates/) · [detaljerad advisory](https://github.com/dotnet/runtime/security/advisories/GHSA-p3jp-9v83-76w9) | CVE-specifika första rättade versioner 8.0.33 / 9.0.22 / 10.0.14 / 11 RC3; default-inställningar; self-contained/NativeAOT; .NET Framework ej berört. | Att varje Frends-/ArcGIS-/.NET-integration faktiskt använder den sårbara avkodaren. |
| [urllib3:s Brotli-stöd](https://urllib3.readthedocs.io/en/latest/advanced-usage.html#brotli-encoding) · [brotlicffi 1.2.1.0](https://pypi.org/project/brotlicffi/) | Frivilligt `brotli`/`brotlicffi`-beroende och rättad brotlicffi-utgåva. | Att alla Python-skript eller HTTP-klienter automatiskt använder Google Brotli. |
| [Pentaho REST Client](https://docs.pentaho.com/pdia-data-integration/10.2-data-integration/pdi-transformation-steps-reference-overview/rest-client-step) · [HTTP-Javadoc](https://javadoc.pentaho.com/kettle102/kettle-engine-10.2.0.0-218-javadoc/org/pentaho/di/trans/steps/http/HTTP.html) | Kettle/PDI:s HTTP-/REST-funktionalitet och Apache HttpClient-typer i dokumenterad HTTP-implementation. | Att en viss Kettle-version aktiverar Brotli eller har CVE-utlösning/publik säkerhetspatch. |
| [Microsoft MSRC om automatiska PowerShell-anrop](https://www.microsoft.com/en-us/msrc/blog/2026/10/patch-now-php2shell-brotli-emudhra-guidance-for-customers) | PowerShell 7:s `Invoke-WebRequest`/`Invoke-RestMethod` har annan Brotli-standardinställning än vanliga `HttpClient`; kontrollera rätt faktisk runtime även för automatiserade skript. | Att Windows Aktivitetsschemaläggaren i sig är en Brotli-avkodare. |

## API-hämtning och HTTP-avkodning (generellt, inte GIS-specifik exploitbevisning)

| Källa | Visar | Visar inte |
| --- | --- | --- |
| [Swagger UI – Configuration](https://swagger.io/docs/open-source-tools/swagger-ui/usage/configuration/) | Try it out/Execute gör faktiska API-begäranden, normalt från webbläsarmiljön. | Att Swagger UI innehåller sårbar Brotli eller att något visst myndighets-API skickar angriparkontrollerade byte. |
| [MDN – Content-Encoding](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Content-Encoding) | HTTP-svarets använda kodning, exempelvis `br` eller `gzip`. | Komprimering inuti ett filformat som GeoParquet. |
| [MDN – Accept-Encoding](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Accept-Encoding) | Vilken kodning klienten kan acceptera. | Att svaret verkligen komprimeras eller att klienten använder sårbar Google Brotli. |
| [FME och HTTPCaller/OpenAPICaller](fme.md) | Dokumenterad leverantörspåverkan enligt daterat Safe-underlag; härledd riskkedja vid API-hämtning. | Att en viss API-källa eller personuppgiftsintegration har exploaterats. |

## Analysmetod – forsknings- och vägledningskällor

- [OWASP – Threat Modeling](https://cheatsheetseries.owasp.org/cheatsheets/Threat_Modeling_Cheat_Sheet.html): dataflöden, attackytor och förtroendegränser.
- [OWASP – Dependency Graph & SBOM](https://cheatsheetseries.owasp.org/cheatsheets/Dependency_Graph_SBOM_Cheat_Sheet.html): direkt/indirekt beroendekedja och SBOM-komplettering.
- [CISA – Minimum Requirements for VEX](https://www.cisa.gov/sites/default/files/2023-04/minimum-requirements-for-vex-508c.pdf): skilj komponentförekomst från leverantörens påverkanbedömning.
- [O'Donoghue m.fl. (2024), *Assessing Security Risks of Software Supply Chains Using Software Bill of Materials*](https://consensus.app/papers/assessing-security-risks-of-software-supply-chains-using-odonoghue-reinhold/39b7b414b386511f91815349c4b8bd3b/): empiriskt underlag om beroendebaserad sårbarhetsanalys.
- [Xia m.fl. (2023), *An Empirical Study on Software Bill of Materials*](https://consensus.app/papers/an-empirical-study-on-software-bill-of-materials-where-we-xia-bi/8b499d5ed29b5af2a45565d907aec1f6/): användningsproblem och evidensgränser för SBOM.


## Verksamhetsflöden (funktionell källa, inte CVE-bevis för produkten)

| Exempel | Kontrollerad källa | Källans räckvidd och gräns |
| --- | --- | --- |
| Frends | [HTTP-tasks](https://tasks.frends.com/integrations/HTTP), [plattform och .NET](https://docs.frends.com/hybrid-integration-architecture/technologies), [HTTP-trigger](https://docs.frends.com/reference/triggers/http-trigger), [Microsofts AutomaticDecompression](https://learn.microsoft.com/en-us/dotnet/api/system.net.http.httpclienthandler.automaticdecompression) | Visar HTTP-funktioner och möjlighet till Brotli i .NET; **inte** att en viss Frends-task har aktiverad sårbar avkodare. |
| ArcGIS Pro / Lantmäteriets STAC | [Esri Sveriges STAC-guide](https://www.esri.se/arcgis-bloggen/stac-lantmateriet-arcgis-pro/), [Esri Pro STAC](https://doc.esri.com/en/arcgis-pro/latest/help/data/imagery/create-a-stac-connection.html) | Bekräftar separata katalog- och raster-/lagringsanrop; **inte** `br` på endera svar eller CVE-utlösning i Pro. |
| Origo WFS-T / GeoServer / MSSQL | [Origo Editor](https://github.com/origo-map/origo-documentation/blob/master/content/controls.md), [GeoServer WFS-säkerhet](https://docs.geoserver.org/main/en/user/production/config/), [SQL Server extension](https://docs.geoserver.org/stable/en/user/data/database/sqlserver/) | Verifierar redigerings- och databaskopplingar, **inte** Brotli i WFS-T eller MS SQL Server. |
| Gaida/SBI (FB) | [Gaida fastighetsinformation](https://help.gaida.se/sv/for-administratoren/sbi/fastighetsinformation), [Gaida Geotorget](https://help.gaida.se/sv/for-administratoren/sbi/guider/bestallning-via-geotorget) | Verifierar lokalt FB-lager, direkt-API och schemalagd import; ingen sårbar avkodare/patch i Gaida verifierad. |
| EntryScape / dataportal | [EntryScape API-introduktion](https://docs.entryscape.com/en/api/), [EntryScape API-referens](https://entryscape.org/api/), [Sveriges dataportal](https://docs.dataportal.se/registry/api/) | Dokumenterar skrivbara DCAT-/metadata-API:er; inte serverns eventuella Brotli-avkodning. |

[Riskkedjorna för dessa arbetsflöden](verksamhetsfloden.md).

## Begränsningar i källkontrollen

- **Safe Software:** CVE-artikeln är inte åtkomlig anonymt i nuvarande webbläsning. Produktspecifik påverkan och fördelning mellan system-Brotli och DuckDB bygger på daterad läsning **7 november 2026**. Inga FME-versioner utpekas här som *CVE-rättade* utan separat CVE-koppling.
- **QGIS:** offentliga beroenden och paketregister kan förklara en relevant angreppsväg; de är inte bevis för godtycklig produkt-RCE. Detaljer från ännu inte samordnade produktspecifika tester publiceras inte.
- **Övriga produkter:** avsaknad av ett leverantörsbesked eller en sökträff innebär inte att sårbar kod är frånvarande. Generella risker gäller bara där de angivna villkoren uppfylls.
- **myCarta:** inga beroenden, angreppsvägar eller produktversioner har verifierats. Namnet används endast för att visa hur en okänd installation utreds.
- **Kontrolldatum:** produkternas patchstatus kan ändras efter **8 november 2026**, och lokala installationer måste bedömas individuellt.

## Riskkedjans bedömningsmodell

- [OWASP Threat Modeling](https://cheatsheetseries.owasp.org/cheatsheets/Threat_Modeling_Cheat_Sheet.html): angriparkontroll, processer, förtroendegränser och skyddsvärden. Används metodiskt, inte för att styrka produkter eller faktiska attacker.
- [OWASP File Upload](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html): filvägar, importkontroller och riskbegränsning.
- [OWASP TLS](https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Security_Cheat_Sheet.html): skydd mot ändring under transport, inte mot farligt innehåll från en autentiserad källa.
- [Node.js CVE-besked](https://nodejs.org/en/blog/vulnerability/october-2026-security-releases) och [Impervas avgränsade test](https://www.imperva.com/blog/node2shell-from-brotli-data-to-remote-code-execution/): skilda säkerhetseffekter, utan att generalisera RCE till GIS.
- [Safe CVE-artikel](https://support.safe.com/hc/en-us/articles/50040725669517-Brotli-Decoder-CVE-2026-111497-Is-FME-Impacted) (kan kräva inloggning): leverantörsrapporterad påverkan på FME, inte belägg för personuppgiftsläcka eller åtkomst i en verklig integration.

## Angriparmodeller, effekter och försvar

| Källa | Vad källan stödjer | Vad den inte bevisar |
| --- | --- | --- |
| [Node.js – 21 okt 2026](https://nodejs.org/en/blog/vulnerability/october-2026-security-releases) | CVE-felmekanism och möjliga säkerhetseffekter | Produktspecifik RCE i GIS |
| [Imperva – Node2Shell-demonstration](https://www.imperva.com/blog/node2shell-from-brotli-data-to-remote-code-execution/) | Kodkörning i *specificerat* sårbart Node.js 24.22.0 Linux arm64-test med inkommande komprimerad begäran/mottaget svar | Att samma utfall gäller alla Node-installationer eller GIS-tjänster |
| [Express – br-komprimerade begärandekroppar](https://expressjs.com/en/blog/2026-10-31-security-releases/#a-note-on-brotli-decompression-cve-2026-111497) | Aktiva body-parsers kan nå `node:zlib`; råd om runtime-patch och tillfälligt stopp före parser | Att alla publika Node-servrar automatiskt har sådan parser |
| [OWASP TLS](https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Security_Cheat_Sheet.html) | Konfidentialitet och integritet för trafik mellan verifierade TLS-parter | Säkerhet hos innehåll från legitim men komprometterad datakälla |
| [OWASP File Upload](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html) | Filbaserade hot, säker import, skanning och isolering | Att skanning upptäcker varje CVE-relevant komprimerad fil |
| [OWASP SSRF](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html) | Kontroller för URL-hämtningar och nätverksåtkomst | Att SSRF och Brotli-CVE är samma sårbarhet |
| [Microsoft .NET](https://github.com/dotnet/announcements/issues/494) | Default-inställningar för request/response-dekomprimering och patched runtimes | Automatisk exponering i alla .NET-appar |
| [JFrog – mer än Node/HTTP](https://research.jfrog.com/post/brotli2shell-beyond-node2shell-and-http2shell/) | Separata angreppsvägar genom andra native-program och filformat | Exploiterbarhet i QGIS |

## Forskningsstöd för metod

[O'Donoghue m.fl. 2024](https://consensus.app/papers/assessing-security-risks-of-software-supply-chains-using-odonoghue-reinhold/39b7b414b386511f91815349c4b8bd3b/) visar hur SBOM-baserad inventering kan användas för att hitta transitiva risker. [Xia m.fl. 2023](https://consensus.app/papers/an-empirical-study-on-software-bill-of-materials-where-we-xia-bi/8b499d5ed29b5af2a45565d907aec1f6/) belyser praktiska begränsningar. Varken forskning eller generisk riskmodell bekräftar en enskild produkt-CVE-trigger. Vi använder VEX/OWASP som **metodstöd**, inte som ersättning för produktspecifik bevisning.

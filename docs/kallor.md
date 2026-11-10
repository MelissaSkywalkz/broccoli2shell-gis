# Källregister och kontrolldatum

**Kontrollperiod: 7 till 10 november 2026.** Källorna är hämtade vid olika tidpunkter; hämtdatum och publiceringsdatum är inte samma sak. Brotli- och runtime-besked visar var grundfelet finns, leverantörernas besked gäller deras produkter och kodgranskning kan visa hur ett beroende används. Dessa belägg ska inte förvandlas till en anklagelse mot en produktutvecklare eller till bevis för produktspecifik exploatering.

[Till översikten](../README.md) · [Supply chain](supply-chain.md) · [CVE-mekanism](cve-mekanism.md) · [HTTP](http-angreppsvagar.md) · [filer](fil-angreppsvagar.md)

## CVE:n och uppströmsrättningen

| Underlag | Används för | Status |
| --- | --- | --- |
| [CVE-2026-111497](https://www.cve.org/CVERecord?id=CVE-2026-111497) | CVE-identitet, felbeskrivning | Kontrollkälla |
| [GitHub Advisory GHSA-9v8h-qqgm-4m35](https://github.com/advisories/GHSA-9v8h-qqgm-4m35) | Brotli-versioner och Pythonpaket | Kontrollerad |
| [OSV: CVE-2026-111497](https://osv.dev/vulnerability/CVE-2026-111497), [maskinläsbar post](https://api.osv.dev/v1/vulns/CVE-2026-111497) | Samlar CVE-post, relaterade advisory-ID:n och länkar till distributionsspecifika säkerhetsbesked; kontrollerad 9 november 2026 (posten senast ändrad 8 november). | Register för vidare kontroll, inte ett separat produktspecifikt säkerhetsbesked. Många poster kan avse samma beroende i olika paket och bevisar inte att ett GIS-program kan exploateras. |
| [CISA KEV](https://www.cisa.gov/known-exploited-vulnerabilities-catalog?field_cve=CVE-2026-111497) | Känd exploatering av CVE:n generellt | Listad 27 oktober 2026; säger inget om enskilt GIS-program |
| [Node.js, 21 oktober 2026](https://nodejs.org/en/blog/vulnerability/october-2026-security-releases) | Orsak, konsekvens, berörda Node-grenar, rättade utgåvor | Officiell säkerhetsrelease |
| [Brotli 1.0.10](https://github.com/google/brotli/releases/tag/v1.0.10), [1.1.1](https://github.com/google/brotli/releases/tag/v1.1.1), [1.2.1](https://github.com/google/brotli/releases/tag/v1.2.1) | Uppströmsrättningar | Versionsbundna |
| [Debian tracker](https://security-tracker.debian.org/tracker/CVE-2026-111497), [Ubuntu security](https://ubuntu.com/security/CVE-2026-111497) | Paket med backportade rättningar | Distro-versioner måste alltid kontrolleras på nytt |
| [Red Hat, CVE-2026-111497](https://access.redhat.com/security/cve/cve-2026-111497), senast ändrad 2 nov 2026, kontrollerad 8 nov | Vilka angivna Node.js- och .NET-paket i RHEL 8/9/10 som använder systemets respektive medföljande Brotli; backportar och separata systemuppdateringar | Att dessa paketregler gäller andra Linuxdistributioner eller att en viss RHEL-installation är rättad |

## Produkt- och beroendekedjor

| Produkt/spår | Primär källa | Vad den **inte** bevisar |
| --- | --- | --- |
| GDAL/GeoParquet | [GDAL:s Parquet-driver](https://gdal.org/en/stable/drivers/vector/parquet.html) | Produktspecifik CVE-utlösning/RCE i QGIS |
| QGIS Windows, OSGeo4W och GRASS | [QGIS nedladdningsarkiv, 10 nov](https://qgis.org/downloads-list/), [OSGeo4W Brotli](https://www.norbit.de/osgeo4w/v2/x86_64/release/brotli/), [stabilt GRASS](https://www.norbit.de/osgeo4w/v2/x86_64/release/grass/), [GRASS byggskript](https://github.com/OSGeo/grass/blob/main/mswindows/osgeo4w/package.sh), [OSGeo4W-ärende #893](https://trac.osgeo.org/osgeo4w/ticket/893#comment:2), [Setup-byggskript](https://github.com/jef-n/OSGeo4W/blob/master/src/setup/osgeo4w/package.sh) | Paketdatum och distributionskedja bekräftar inte versionen på varje DLL eller aktiv CVE-väg. Brotli 1.2.0 i de två MSI-paketen bygger på separat dokumenterad paketinspektion. |
| QGIS HTTP | [Qt Network](https://doc.qt.io/qt-6/qnetworkrequest.html), [libcurl](https://curl.se/libcurl/c/CURLOPT_ACCEPT_ENCODING.html) | Att QGIS WMS/WFS utlöser CVE:n |
| QGIS Server container | [3liz ärende #137](https://github.com/3liz/py-qgis-server/issues/137) | Att alla QGIS Server-versioner har samma status |
| Bildformat | [OSGeo4W libjxl-recept](https://github.com/jef-n/OSGeo4W/blob/c8514a953de4b50a6dd4887fb02a874c8d74eec9/src/libjxl/osgeo4w/package.sh), [JFrog separat studie av FFmpeg/libjxl](https://research.jfrog.com/post/brotli2shell-beyond-node2shell-and-http2shell/) | QGIS-exploiterbarhet via JPEG XL |
| Apache Arrow | [Arrow 26.0.1, 3 november 2026](https://arrow.apache.org/blog/2026/11/03/26.0.1-release/) | Att systemlänkat Arrow är rättat utan systempaket |
| FME | [Safes CVE-artikel](https://support.safe.com/hc/en-us/articles/50040725669517-Brotli-Decoder-CVE-2026-111497-Is-FME-Impacted), **läst igen 9 november 2026 via inloggad webbläsarflik**. Safe bekräftar påverkan i Form/Engine/Flow/Hosted genom HTTP-klient, Parquet och inbyggd DuckDB. Rättade FME-byggen för 2026.1 till 2026.3 anges som under arbete. | Oberoende CVE-reproduktion i FME, ett faktiskt installerat rättat bygge eller vilken senare FME-utgåva som innehåller DuckDB-rättningen |
| FME tillgängliga utgåvor | [Downloads](https://fme.safe.com/downloads/), [changelog](https://fme.safe.com/changelog/), [novemberuppdateringar](https://support.safe.com/hc/en-us/articles/46792031942029-FME-November-2026-Security-Updates), öppet läsbara **8 nov 2026** | Att listade säkerhetsfixar eller hotfix-byggnummer är en bekräftad rättning för **denna CVE** |
| ArcGIS Server/Pro | [Esri produktbesked, uppdaterat 7 nov och kontrollerat 9 nov 2026](https://www.esri.com/arcgis-blog/products/trust-arcgis/administration/brotli-vulnerabilities-broccoli2shell), [Security 2026 Update 4-patch, publicerad 5 nov](https://support.esri.com/en-us/patches-updates/2026/arcgis-server-security-2026-update-4-patch) | Vilken extern endpoint som når Node-avkodaren eller att Pro-patchens status är oförändrad senare |
| ArcGIS Mission Server | [Esri, Security 2026 Update 1, 9 november 2026](https://support.esri.com/en-us/patches-updates/2026/arcgis-mission-server-security-2026-update-1-patch) | Produktpatch för 11.5/12.0/12.1 och `BUG-000188431`, inte bevis om produktspecifik RCE |
| .NET | [Microsofts säkerhetsbesked](https://github.com/dotnet/announcements/issues/494) | Att Brotli avkodas automatiskt i standardkonfiguration |
| Origo Server | [`app.js`, revision bacabe4](https://github.com/origo-map/origo-server/blob/bacabe4b02611e8cbabf49daf57fe3d220f2e169/app.js), [Express](https://expressjs.com/en/blog/2026-10-31-security-releases/) | Att en specifik Origo-endpoint kan utlösa CVE:n |
| Hajk Backend och officiella images | [server.js](https://github.com/hajkmap/Hajk/blob/master/apps/backend/server/common/server.js), [Hajks säkerhetspolicy](https://github.com/hajkmap/Hajk/blob/master/SECURITY.md#official-docker-images), [releaser](https://github.com/hajkmap/Hajk/blob/master/docs/releases.md#docker-images), [Docker-anvisningar](https://github.com/hajkmap/Hajk/blob/master/Docker/README.md#keeping-the-base-image-up-to-date), [nattligt byggflöde](https://github.com/hajkmap/Hajk/blob/master/.github/workflows/nightly-master.yml), [utvecklarens förtydligande 6 nov](https://github.com/hajkmap/Hajk/commit/badc79df70407a95669aed59c8632f9b3419185d) och [PR i denna vägledning](https://github.com/MelissaSkywalkz/broccoli2shell-gis/pull/2) | Att alla Hajk-utgåvor innehåller Node-runtime, att npm-beroenden patchas vid varje nattbygge eller att en körande container uppdateras när en ny image publiceras |
| GeoServer/Jetty | [GeoServer 3.0.2 pom](https://github.com/geoserver/geoserver/blob/3.0.2/src/pom.xml), [Jetty start.ini](https://github.com/geoserver/geoserver/blob/3.0.2/src/release/jetty/start.ini), [Jetty moduler](https://jetty.org/docs/jetty/12.1/operations-guide/modules/standard.html) | Att Brotlis native-kod laddas av GeoServer som standard |
| GeoServer ImageI/O-Ext och native GDAL | [GeoServers GDAL-tillägg](https://docs.geoserver.org/stable/en/user/data/raster/gdal/), [standardformat för raster](https://docs.geoserver.org/stable/en/user/data/raster/), [GDAL:s cURL-bygge](https://gdal.org/en/stable/development/building_from_source.html) | Att `gs-gdal`, `GDAL Version` eller `CURL_ENABLED=YES` bevisar ett inläst Brotli-bibliotek, aktivt användande eller CVE-utlösning |
| GeoServer och Tomcat, uppgradering och moduler (kontrollerat 9 nov 2026) | [GeoServer 2.27.1:s release med GeoTools 33.1](https://geoserver.org/announcements/vulnerability/2025/05/13/geoserver-2-27-1-released.html), [GeoServers Java-anvisningar](https://docs.geoserver.org/main/en/user/production/java/), [Tomcats Host-konfiguration](https://tomcat.apache.org/tomcat-9.0-doc/config/host.html) | Att en `SNAPSHOT`-sträng ensam visar sårbarhet, att en katalog automatiskt är en nåbar webbapp, eller att en Java-uppdatering rättar separat native GDAL/Brotli |
| Brotli4j | [GHSA-x524-jrqf-vwq6](https://github.com/hyperxpro/Brotli4j/security/advisories/GHSA-x524-jrqf-vwq6) | Att varje Java-/GeoServer-installation använder Brotli4j |
| DuckDB | [Offentlig rättningsdiskussion](https://github.com/duckdb/duckdb/issues/27660) | Att kodändringen levereras i viss FME-version |
| **myCarta** | Ingen verifierad versionslåst beroendekälla i denna granskning | All produktpåverkan, aktiv avkodning och patchstatus; används endast som inventeringsexempel |

## Exempel utanför GIS

Unity, Siemens, Microsoft och AWS används för att förklara olika villkor för exponering och uppdatering i [GIS-produktöversikten](produktoversikt.md#exempel-utanför-gis-och-deras-betydelse-för-gis).

| Primärkälla, kontrollerad 9 nov 2026 | Vad exemplet tillför GIS-bedömningen |
| --- | --- |
| [Unitys säkerhetsbesked](https://activation.unity3d.com/security/unity-brotli-decoder-vulnerability), uppdaterat 7 nov | En sårbar avkodare kan följa med i en runtime utan att standardfunktionerna når den. Berörda applikationer kan behöva byggas om och distribueras på nytt. |
| [Microsoft MSRC](https://www.microsoft.com/en-us/msrc/blog/2026/10/patch-now-php2shell-brotli-emudhra-guidance-for-customers) | Uppdatering av ett operativsystem rättar inte nödvändigtvis separat paketerade program och runtimes. |
| [Siemens ProductCERT SSA-443410](https://productcert.siemens.com/productcert/html/ssa-443410.html), uppdaterat 6 nov | Berörd kod kan finnas utan exponerad funktion; installerade tillägg styr åtgärdsbehovet. |
| [AWS Security Bulletin 2026-155](https://aws.amazon.com/security/security-bulletins/2026-155-aws/) | Hanterad molntjänst, tillhandahållen runtime och kundens containeravbild kan ha olika uppdateringsansvar. |

**Direkt GIS-relevant bibliotek:** [Apache Arrow 26.0.1](https://arrow.apache.org/blog/2026/11/03/26.0.1-release/) beskriver rättad medföljande Brotli i specificerade PyArrow-paket och R-binärer, inte automatisk rättning av systemlänkat Arrow C++. **Primärkälla till grundfelet:** [Project Zero](https://project-zero.issues.chromium.org/issues/565308158) och [Googles kodrättning](https://github.com/google/brotli/commit/0ed4268a4401abcb4472096340477cf4af7d1a2b). **Metodstöd:** [Cloudflares HTTP-komprimering](https://developers.cloudflare.com/speed/optimization/content/compression/) och [Trail of Bits Brotli Checker](https://github.com/trailofbits/brotli-checker) för begränsad beroendeinventering; ingen av dessa källor belägger produktspecifik GIS-exploatering.

## Från dataflöde till avkodare och rättning

| Källa | Styrker | Styrker inte |
| --- | --- | --- |
| [Microsofts .NET out-of-band-uppdatering 25 oktober 2026](https://devblogs.microsoft.com/dotnet/dotnet-october-2026-oob-security-updates/) · [detaljerad advisory](https://github.com/dotnet/runtime/security/advisories/GHSA-p3jp-9v83-76w9) | CVE-specifika första rättade versioner 8.0.33 / 9.0.22 / 10.0.14 / 11 RC3; default-inställningar; self-contained/NativeAOT; .NET Framework ej berört. | Att varje Frends-/ArcGIS-/.NET-integration faktiskt använder den sårbara avkodaren. |
| [urllib3:s Brotli-stöd](https://urllib3.readthedocs.io/en/latest/advanced-usage.html#brotli-encoding) · [brotlicffi 1.2.1.0](https://pypi.org/project/brotlicffi/) | Frivilligt `brotli`/`brotlicffi`-beroende och rättad brotlicffi-utgåva. | Att alla Python-skript eller HTTP-klienter automatiskt använder Google Brotli. |
| [Pentaho REST Client](https://docs.pentaho.com/pdia-data-integration/10.2-data-integration/pdi-transformation-steps-reference-overview/rest-client-step) · [HTTP-Javadoc](https://javadoc.pentaho.com/kettle102/kettle-engine-10.2.0.0-218-javadoc/org/pentaho/di/trans/steps/http/HTTP.html) | Kettle/PDI:s HTTP-/REST-funktionalitet och Apache HttpClient-typer i dokumenterad HTTP-implementation. | Att en viss Kettle-version aktiverar Brotli eller har CVE-utlösning/publik säkerhetspatch. |
| [Microsoft MSRC om automatiska PowerShell-anrop](https://www.microsoft.com/en-us/msrc/blog/2026/10/patch-now-php2shell-brotli-emudhra-guidance-for-customers) | PowerShell 7:s `Invoke-WebRequest`/`Invoke-RestMethod` har annan Brotli-standardinställning än vanliga `HttpClient`; kontrollera rätt faktisk runtime även för automatiserade skript. | Att Windows Aktivitetsschemaläggaren i sig är en Brotli-avkodare. |

## API-hämtning och HTTP-avkodning (generellt, inte GIS-specifik exploitbevisning)

| Källa | Visar | Visar inte |
| --- | --- | --- |
| [Swagger UI, Configuration](https://swagger.io/docs/open-source-tools/swagger-ui/usage/configuration/) | Try it out/Execute gör faktiska API-begäranden, normalt från webbläsarmiljön. | Att Swagger UI innehåller sårbar Brotli eller att något visst myndighets-API skickar angriparkontrollerade byte. |
| [MDN, Content-Encoding](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Content-Encoding) | HTTP-svarets använda kodning, exempelvis `br` eller `gzip`. | Komprimering inuti ett filformat som GeoParquet. |
| [MDN, Accept-Encoding](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Accept-Encoding) | Vilken kodning klienten kan acceptera. | Att svaret verkligen komprimeras eller att klienten använder sårbar Google Brotli. |
| [FME och HTTPCaller/OpenAPICaller](fme.md) | Dokumenterad leverantörspåverkan enligt daterat Safe-underlag; härledd riskkedja vid API-hämtning. | Att en viss API-källa eller personuppgiftsintegration har exploaterats. |

## Källor för analysmetod

- [OWASP, Vulnerability Disclosure](https://cheatsheetseries.owasp.org/cheatsheets/Vulnerability_Disclosure_Cheat_Sheet.html): samordnad leverantörskontakt, verifiering och återhållsam publicering av osamordnade tekniska detaljer.

- [OWASP, Threat Modeling](https://cheatsheetseries.owasp.org/cheatsheets/Threat_Modeling_Cheat_Sheet.html): dataflöden, attackytor och förtroendegränser.
- [OWASP, Dependency Graph & SBOM](https://cheatsheetseries.owasp.org/cheatsheets/Dependency_Graph_SBOM_Cheat_Sheet.html): direkt/indirekt beroendekedja och SBOM-komplettering.
- [CISA, Minimum Requirements for VEX](https://www.cisa.gov/sites/default/files/2023-04/minimum-requirements-for-vex-508c.pdf): skilj komponentförekomst från leverantörens påverkanbedömning.
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

- **Safe Software:** CVE-artikeln kontrollerades via inloggad webbläsarflik 9 november. Safe anger påverkan och planerade rättningar för FME 2026.1 till 2026.3, men inget slutligt CVE-rättat byggnummer. Artikeln anger även Brotli 1.1.0 för Windows/macOS till och med 2026.4 i detaljtexten, medan sammanfattningstabellen slutar på 2026.3. Versionsskillnaden är inte utredd. Novemberartikeln om andra säkerhetsbrister bekräftar inte en Brotli-rättad version.
- **QGIS:** offentliga beroenden och paketregister kan förklara en relevant angreppsväg; de är inte bevis för godtycklig produkt-RCE. Detaljer från ännu inte samordnade produktspecifika tester publiceras inte.
- **Övriga produkter:** avsaknad av ett leverantörsbesked eller en sökträff innebär inte att sårbar kod är frånvarande. Generella risker gäller bara där de angivna villkoren uppfylls.
- **myCarta:** inga beroenden, angreppsvägar eller produktversioner har verifierats. Namnet används endast för att visa hur en okänd installation utreds.
- **Kontrolldatum:** de viktigaste leverantörsbeskeden kontrollerades på nytt 9 november 2026. Det säger inget om enskilda installationers patchstatus.

## Angriparmodeller, effekter och försvar

| Källa | Vad källan stödjer | Vad den inte bevisar |
| --- | --- | --- |
| [Node.js, 21 okt 2026](https://nodejs.org/en/blog/vulnerability/october-2026-security-releases) | CVE-felmekanism och möjliga säkerhetseffekter | Produktspecifik RCE i GIS |
| [Imperva, Node2Shell-demonstration](https://www.imperva.com/blog/node2shell-from-brotli-data-to-remote-code-execution/) | Kodkörning i *specificerat* sårbart Node.js 24.22.0 Linux arm64-test med inkommande komprimerad begäran/mottaget svar | Att samma utfall gäller alla Node-installationer eller GIS-tjänster |
| [Express, br-komprimerade begärandekroppar](https://expressjs.com/en/blog/2026-10-31-security-releases/#a-note-on-brotli-decompression-cve-2026-111497) | Aktiva body-parsers kan nå `node:zlib`; råd om runtime-patch och tillfälligt stopp före parser | Att alla publika Node-servrar automatiskt har sådan parser |
| [OWASP TLS](https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Security_Cheat_Sheet.html) | Konfidentialitet och integritet för trafik mellan verifierade TLS-parter | Säkerhet hos innehåll från legitim men komprometterad datakälla |
| [OWASP File Upload](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html) | Filbaserade hot, säker import, skanning och isolering | Att skanning upptäcker varje CVE-relevant komprimerad fil |
| [OWASP SSRF](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html) | Kontroller för URL-hämtningar och nätverksåtkomst | Att SSRF och Brotli-CVE är samma sårbarhet |
| [Microsoft .NET](https://github.com/dotnet/announcements/issues/494) | Default-inställningar för request/response-dekomprimering och patched runtimes | Automatisk exponering i alla .NET-appar |
| [JFrog, mer än Node/HTTP](https://research.jfrog.com/post/brotli2shell-beyond-node2shell-and-http2shell/) | Separata angreppsvägar genom andra native-program och filformat | Exploiterbarhet i QGIS |

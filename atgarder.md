# Åtgärder för GIS-installationer

Uppdaterad 4 november 2026. [Produktöversikt](README.md)

Kontrollera leverantörens senaste besked för er produktversion och plattform före uppdatering.

## QGIS

### Windows med OSGeo4W

Gäller QGIS som uppdateras med OSGeo4W Installer. Låt IT uppdatera centralt hanterade installationer.

1. Ta reda på vilken QGIS-installation som används. Kontrollera varje installation separat om datorn har flera.
2. Stäng QGIS och andra program som använder installationen. Hämta OSGeo4W Installer via [QGIS installationsguide](https://qgis.org/resources/installation-guide/). Välj `Advanced Install` och kontrollera installationskatalogen.
3. Under `Select Packages`, sök efter `brotli`. Kontrollera att `New` visar den rättade versionen `1.2.1-1` eller en senare rättad version. Om `Current` redan visar en rättad version behöver du inte uppdatera just det paketet.
4. Granska paketändringarna och slutför uppdateringen. Byt inte DLL-filer manuellt.
5. Öppna installeraren igen och kontrollera att `Current` visar en rättad version i samma installation. Starta QGIS och testa projekt, datakällor och bearbetningar som används hos er.

### Frivillig snabbkontroll i OSGeo4W Shell

Kör kommandot i OSGeo4W Shell för den installation du vill kontrollera:

```cmd
findstr /B /I /C:"brotli " "%OSGEO4W_ROOT%\etc\setup\installed.db"
```

Kommandot läser paketregistret `installed.db` och visar raden för det installerade `brotli`-paketet, inklusive versionen i paketfilnamnet. Det ändrar inga filer. Om ingen rad visas, kontrollera att rätt OSGeo4W Shell används och att registret finns. Uteblivet resultat visar inte i sig om installationen är säker.

Kontrollen visar inte separata Brotli-kopior eller andra QGIS-installationer. Filen `installed.db` används för att inventera installerade paket, se [exempel på inventering via OSGeo4W:s paketregister](https://gis.stackexchange.com/questions/494508/search-more-deeply-than-package-name-in-osgeo4w-package-repository/494578). Kommandot har funktionstestats i OSGeo4W Shell den 4 november 2026. Det visar paketversionen som registret innehåller, inte om en nyare version finns tillgänglig.

OSGeo4W-paketet `brotli` rättar inte separata kopior i exempelvis DuckDB eller Qt WebEngine. Att QGIS fungerar efter uppdateringen visar inte om dessa kopior är rättade.

Fristående QGIS-installationer för Windows bygger också på OSGeo4W-paket, men uppdateras normalt med en ny fristående installerare. Använd inte OSGeo4W Installer för att ändra dem utan stöd i distributionsanvisningarna. För macOS, Linux, Conda, Flatpak och containrar gäller andra uppdateringsmetoder. Paketversionen `1.2.1-1` gäller OSGeo4W. Andra distributioner kan ha rättningen i andra paketversioner.

Källor: [OSGeo4W:s paketregister](https://download.osgeo.org/osgeo4w/v2/x86_64/setup.ini) · [QGIS säkerhetsinformation](https://qgis.org/resources/support/security/)

## Origo Server

Origo Server använder Node.js. Kontrollera vilken Node.js-installation som kör servern, inte bara versionen i en terminal.

1. Identifiera tjänstens Node.js-installation via startkonfiguration, tjänstehanterare eller container.
2. Kontrollera versionen. Node.js har rättningen i `22.23.4`, `24.22.1` och `26.12.1` samt senare rättade versioner inom respektive gren. Välj en stödd version som fungerar med er Origo Server-installation.
3. Uppdatera Node.js och starta om Origo Server. `npm install` uppdaterar inte Node.js-körmiljön.
4. Kontrollera versionen som den omstartade tjänsten använder. Testa serverns API:er och övriga funktioner som används hos er.

Node.js är berört, men varje Origo Server-installation är inte automatiskt sårbar för angrepp. Det beror på om servern avkodar Brotli-data som en angripare styr.

Källor: [Node.js säkerhetsbesked](https://nodejs.org/en/blog/vulnerability/october-2026-security-releases) · [Origo Server](https://github.com/origo-map/origo-server)

## FME

FME Form påverkas genom Workbench och Engine. FME Flow påverkas genom sina Engines, inte genom webbapplikationen. Produkterna levererar inte Node.js.

På Windows och macOS innehåller berörda versioner Brotli `1.1.0`. Safe förbereder rättade byggen för FME 2026.1, 2026.2 och 2026.3. DuckDB innehåller en egen Brotli-kopia. Rättningen slogs ihop i [DuckDB:s huvudgren den 3 november 2026](https://github.com/duckdb/duckdb/pull/27791), men det bekräftar inte att den ingår i en utgiven DuckDB-version eller i FME:s levererade paket. Se [Safes Brotli-besked](https://support.safe.com/hc/en-us/articles/50040725669517-Brotli-Decoder-CVE-2026-111497-Is-FME-Impacted).

### Berörda funktioner

- `HTTPCaller` och `OpenAPICaller` kan ta emot HTTP-svar med `Content-Encoding: br`.
- Webbanslutningar med OAuth 2.0 eller tokenbaserad autentisering använder den berörda HTTP-klienten vid hämtning av åtkomsttoken.
- Parquet kan läsa Brotli-komprimerade kolumner.
- DuckDB läser data med en egen Brotli-kopia.
- Förhandsvisning av HTTP-anrop i Workbench kan behandla Brotli-komprimerade svar.

### Åtgärder

1. Ta fram version, byggnummer och plattform för FME Form, Flow och separata Engines.
2. Kontrollera [Safes Brotli-besked](https://support.safe.com/hc/en-us/articles/50040725669517-Brotli-Decoder-CVE-2026-111497-Is-FME-Impacted) för rättade byggen och tillfälliga åtgärder. Versionsgren räcker inte.
3. I väntan på rättning: använd bara betrodda HTTP-tjänster, webbanslutningar och Parquet-/DuckDB-filer. Undvik förhandsvisning i Workbench mot okända adresser och granska schemalagda jobb som hämtar externa data. Där det är möjligt kan en utgående proxy eller brandvägg blockera Brotli-kodade HTTP-svar till FME. Det skyddar inte mot Brotli i Parquet eller DuckDB.
4. Uppdatera enligt Safe. Starta om berörda Engines och tjänster. Kontrollera byggnummer och testa arbetsytor, anslutningar och schemalagda jobb.

### Linux och Flow Hosted

På Linux använder FME operativsystemets Brotli-bibliotek för HTTP och Parquet. Installera distributionens säkerhetsrättning och starta om FME-processerna. DuckDB behöver en separat FME-rättning.

På Flow Hosted installeras Ubuntus säkerhetsuppdateringar automatiskt, men FME Flow eller instansen behöver startas om för att använda rättningen. Ubuntu-versioner utan säkerhetsstöd måste uppgraderas. Se [Safes anvisningar](https://support.safe.com/hc/en-us/articles/50040725669517-Brotli-Decoder-CVE-2026-111497-Is-FME-Impacted).

En FME Flow-installation som bara nås internt kan fortfarande läsa data från externa källor. Safes artikel anger olika övre versionsgränser i tabellen och löptexten: 2026.3 respektive 2026.4. Kontrollera därför exakt byggnummer. [Safes säkerhetsuppdateringar för november 2026](https://support.safe.com/hc/en-us/articles/46792031942029-FME-November-2026-Security-Updates) gäller även andra rättningar och bekräftar inte att Brotli-felet är löst.

## ArcGIS

Esri anger att ArcGIS Enterprise och ArcGIS Pro påverkas. Rättningarna skiljer sig mellan produkter och versioner.

| Produkt eller komponent | Status och åtgärd |
| --- | --- |
| ArcGIS Server 12.1 | Security 2026 Update 4 Patch finns. Installera enligt Esri. |
| ArcGIS Server 12.0 och 11.5 | Motsvarande patchar inväntas. |
| ArcGIS Pro 3.5 och senare | Påverkan bekräftad. Produktpatch inväntas. |
| ArcGIS Online | Esri uppger att begränsningsåtgärder införts och att uppdateringar pågår. |
| Komponenter som använder .NET | Uppdatera den .NET-körmiljö som används. |

### Åtgärder

1. Inventera ArcGIS Server och övriga Enterprise-komponenter, Pro, Web Adaptor, Enterprise SDK-tillägg och egna appar.
2. Kontrollera [Esri Japans besked från 4 november](https://support.esrij.com/news/38149), som också länkar till patchsidan. Där bekräftar Esri att rättningen finns för ArcGIS Server 12.1. Patchar för 12.0 och 11.5 inväntas. Installera rätt patch för er version och kontrollera varje nod.
3. Uppdatera .NET separat där det används av ArcGIS Pro, Web Adaptor (IIS), Enterprise SDK SOE/SOI och egna .NET-appar. Rättade versioner är `8.0.33`, `9.0.22` och `10.0.14` samt senare rättade versioner inom respektive gren.
4. Använder ni Enterprise SDK custom data feeds rekommenderar Esri att funktionen tillfälligt inaktiveras tills rätt patch finns. Kontakta Esri om den är verksamhetskritisk.
5. Starta om berörda tjänster. Kontrollera installerade patchar och den .NET-körmiljö som används på samtliga noder. Testa karttjänster och projekt.

.NET-uppdateringar ersätter inte Esris produktpatchar. En patch för ArcGIS Server rättar inte automatiskt andra Enterprise-komponenter. Bygg om och distribuera appar och containrar som har en egen .NET-körmiljö. Microsoft anger att .NET Framework inte påverkas av denna CVE.

Enligt [Microsofts säkerhetsbesked](https://github.com/dotnet/announcements/issues/494) dekomprimerar ASP.NET Core och HttpClient inte Brotli automatiskt med standardinställningarna. Exponering kräver att en applikation avkodar angriparstyrda Brotli-data, exempelvis med aktiverad dekomprimering eller `BrotliStream`. Detta beskriver .NET:s möjliga angreppsvägar, inte hur Esris komponenter är konfigurerade. Följ fortsatt Esris rekommendationer om uppdatering.

Källor: [Esris säkerhetsbesked](https://www.esri.com/arcgis-blog/products/trust-arcgis/administration/brotli-vulnerabilities-broccoli2shell) · [Esri Japan: patchstatus 4 november](https://support.esrij.com/news/38149) · [Microsofts .NET-uppdateringar](https://devblogs.microsoft.com/dotnet/dotnet-october-2026-oob-security-updates/)

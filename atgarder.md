# Kontroller och åtgärder

**Uppdaterad: 5 november 2026** · [Till produktöversikten](README.md)

Informationen avser Broccoli2Shell (CVE-2026-111497) och bygger på leverantörernas säkerhetsbesked eller projektens dokumenterade komponenter. Kontrollera alltid aktuella anvisningar för den version och plattform som används. En version som saknas i en patchlista är inte automatiskt opåverkad.

## FME Form och Flow

[Safe Software bekräftar](https://support.safe.com/hc/en-us/articles/50040725669517-Brotli-Decoder-CVE-2026-111497-Is-FME-Impacted) att FME Form påverkas genom Workbench och FME Engine och att FME Flow påverkas genom de Engines som kör arbetsytor. Safe beskriver bland annat HTTPCaller, OpenAPICaller, webbanslutningar, Brotli-komprimerade Parquet-filer och DuckDB som relevanta funktioner. FME levererar inte Node.js; påverkan gäller andra Brotli-komponenter.

**Rättningsstatus:** Safe förbereder rättade byggen av FME **2026.1, 2026.2 och 2026.3** med Brotli 1.2.1. Safe rekommenderar att användare av äldre FME-versioner planerar att uppgradera till någon av dessa utgåvor när rättningarna finns tillgängliga. DuckDB innehåller en separat Brotli-kopia vars rättning enligt Safe inväntar en utgiven DuckDB-version. Det går därför inte att behandla enbart den gemensamma Brotli-uppdateringen som en bekräftad fullständig rättning för DuckDB.

**I väntan på rättning rekommenderar Safe bland annat att:**

- Använda HTTPCaller och webbanslutningar mot betrodda tjänster.
- Bara läsa Parquet- och DuckDB-data från betrodda källor.
- Undvika förhandsvisning av HTTP-anrop mot obetrodda adresser i Workbench.
- Se över schemalagda FME Flow-jobb som hämtar externa HTTP-data eller läser externa Parquet- och DuckDB-filer.
- Där det är möjligt begränsa Brotli-komprimerade HTTP-svar till FME med proxy eller brandvägg.

På **Linux** rekommenderar Safe att operativsystemets Brotli-paket uppdateras för HTTP och Parquet. Det rättar inte DuckDB:s separat paketerade kopia. För **FME Flow Hosted** anger Safe att Ubuntus säkerhetsuppdatering installeras automatiskt, men att FME Flow eller instansen behöver startas om för att börja använda den. Följ [Safes plattformsspecifika anvisningar](https://support.safe.com/hc/en-us/articles/50040725669517-Brotli-Decoder-CVE-2026-111497-Is-FME-Impacted).

**Separat säkerhetsbesked:** [FME November 2026 Security Updates](https://support.safe.com/hc/en-us/articles/46792031942029-FME-November-2026-Security-Updates) gäller andra allvarliga sårbarheter. Safe tillhandahåller även rättningar för äldre versioner och anger särskilda krav på säkerhetsinställningar. **Dessa uppdateringar ska inte förväxlas med en bekräftad rättning av Broccoli2Shell.**

## ArcGIS

[Esri bekräftar](https://www.esri.com/arcgis-blog/products/trust-arcgis/administration/brotli-vulnerabilities-broccoli2shell) påverkan från CVE-2026-111497 för angivna ArcGIS-produkter och versioner.

**ArcGIS Server:** [ArcGIS Server Security 2026 Update 4 Patch](https://support.esri.com/en-us/patches-updates/2026/arcgis-server-security-2026-update-4-patch) åtgärdar enligt Esri CVE-2026-111497 genom att uppdatera Node.js (`BUG-000188308`). Nedladdningsstatus på Esris patchsida den 5 november 2026:

| ArcGIS Server | Status |
| --- | --- |
| `12.1` | Patch tillgänglig |
| `12.0` | Patch annonserad, nedladdning inväntas |
| `11.5` | Patch tillgänglig |

Tabellen gäller **ArcGIS Server**, inte alla delar av ArcGIS Enterprise. Om en version inte finns med är den inte därmed bekräftat opåverkad. Följ Esris fortsatta versionsspecifika besked.

**Övriga produkter:** Esri anger att **ArcGIS Pro 3.5 och senare** påverkas och att patch förbereds. För **ArcGIS Online** uppger Esri att begränsande åtgärder har införts och att ytterligare uppdateringar pågår.

**Tillfälliga och kompletterande åtgärder:** Esri rekommenderar att berörda **Microsoft .NET-körmiljöer uppdateras separat** och att **ArcGIS Enterprise SDK Custom Data Feeds inaktiveras där funktionen används** i väntan på relevant patch.

## Pythonpaket i GIS-miljöer

QGIS, ArcGIS Pro och FME kan använda separata Pythonmiljöer med egna paket. Där paketen `Brotli` eller `brotlicffi` används kan de behöva uppdateras till de rättade versionerna `1.2.1` respektive `1.2.1.0` enligt [GitHubs säkerhetsråd för CVE-2026-111497](https://github.com/advisories/GHSA-9v8h-qqgm-4m35).

## QGIS / OSGeo4W

[OSGeo4W:s paketregister](https://download.osgeo.org/osgeo4w/v2/x86_64/setup.ini.bz2) erbjuder det rättade Brotli-paketet **`1.2.1-1`**. Det är en bekräftad paketuppdatering, inte ett produktspecifikt säkerhetsbesked om QGIS.

För QGIS-installationer som hanteras genom **OSGeo4W Network Installer**:

1. Stäng QGIS, öppna installeraren och välj `Advanced Install`.
2. Kontrollera installationskatalogen och sök efter paketet `brotli`.
3. Uppdatera till `1.2.1-1` eller en senare rättad version vid behov.
4. Kontrollera installerad version och testa QGIS efter uppdateringen.

**Frivillig kontroll av installerat OSGeo4W-paket**

Kör i OSGeo4W Shell för den installation du vill kontrollera:

```cmd
findstr /B /I /C:"brotli " "%OSGEO4W_ROOT%\etc\setup\installed.db"
```

Kommandot visar den installerade Brotli-paketversionen enligt OSGeo4W:s paketregister. Det ändrar ingenting och har testats lokalt.

Kontrollen omfattar inte andra QGIS-installationer eller separat paketerade Brotli-bibliotek.

För fristående QGIS-installationer och andra plattformar gäller [QGIS installations- och uppdateringsanvisningar](https://qgis.org/resources/installation-guide/).

## Origo Server

[Origo Servers källkod](https://github.com/origo-map/origo-server/blob/master/app.js) visar att servern använder Node.js och Express. Enligt [Express säkerhetsbesked](https://expressjs.com/en/blog/2026-10-31-security-releases/) kan dess standardfunktioner för inkommande HTTP-anrop använda Node.js Brotli-avkodare. Express rekommenderar att Node.js uppdateras för CVE-2026-111497.

**Första rättade Node.js-versioner enligt [Node.js-projektet](https://nodejs.org/en/blog/vulnerability/october-2026-security-releases):**

| Versionsgren | Första rättade version |
| --- | --- |
| `22.x` | `22.23.4` |
| `24.x` | `24.22.1` |
| `26.x` | `26.12.1` |

Kontrollera vilken Node.js-version **Origo Server faktiskt kör**, uppdatera till en rättad och kompatibel version vid behov, starta om tjänsten och verifiera versionen efteråt. [Node.js-versioner som inte längre får säkerhetsuppdateringar](https://nodejs.org/en/about/eol) behöver ersättas med en stödd och rättad version. För containerbaserad drift gäller kontrollen Node.js i den körande containern.

Uppgifterna ovan är dokumenterade komponentberoenden och rekommendationer från Node.js och Express, **inte ett separat säkerhetsbesked från Origo Server-projektet om exploaterbarhet**.

## Hajk

Enligt [Hajkprojektets dokumentation](https://github.com/hajkmap/Hajk/blob/master/docs/releases.md) innehåller **NodeJS Edition** en Node.js-backend. **Simple Edition** består av statiska klientfiler och saknar egen backend. Den fullständiga containerdistributionen innehåller också en Node.js-backend.

För installationer med backend: kontrollera vilken Node.js-version som faktiskt körs och jämför med [Node.js rättade versioner](#origo-server). Använd en stödd och rättad Node.js-version och uppdatera körmiljön enligt Hajks distributionsanvisningar och kontrollera versionen efteråt.

[Hajks säkerhetspolicy](https://github.com/hajkmap/Hajk/blob/master/SECURITY.md) anger att endast den senaste produktutgåvan får säkerhetsrättningar i projektets egen kod. Node.js-körmiljön hör till den driftmiljö som organisationen själv ansvarar för.

Här redovisas ett dokumenterat Node.js-beroende, **inte en bekräftad produktspecifik exploaterbarhet eller en särskild Hajk-rättning för CVE-2026-111497**.

## Om informationen

Vägledningen sammanställer leverantörernas publicerade besked och tydligt angivna komponentuppgifter. Den avgör inte om en enskild installation är exploaterbar och ersätter inte leverantörernas aktuella rekommendationer.
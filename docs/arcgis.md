# ArcGIS – bekräftad produktpåverkan, men flera olika avkodare

**Källkontroll: Esris produktbesked uppdaterat 7 november och ArcGIS Server-patch 5 november 2026; Microsofts .NET-besked kontrollerat 8 november**  

**Var finns CVE-felet?** I **Brotli-avkodning som följer med underliggande komponenter**, bland annat Node.js i ArcGIS Server och en separat .NET-kedja för berörda ArcGIS-komponenter. Esris bekräftelse på **produktpåverkan** ska behållas; den innebär inte att Esri har påvisats ha orsakat grundfelet i Brotli.

**Vad vet vi?** Esri rapporterar påverkan för ArcGIS Server **11.5, 12.0 och 12.1** samt ArcGIS Pro **3.5 och senare** (besked uppdaterat 7 november och avläst 8 november 2026). **ArcGIS Server Security 2026 Update 4** rättar den berörda Node.js-komponenten. Den kundhanterade .NET-körmiljön är en **separat** rättningskedja.

**Vad vet vi inte?** Esris produktpåverkan bevisar inte att varje ArcGIS-endpoint kan utlösa CVE:n eller att RCE har demonstrerats i ArcGIS. Esris avlästa besked angav att produktpatch för ArcGIS Pro 3.5+ fortfarande väntades **7 november**; det är inte en garanti för status vid senare läsning.

**Vad bör göras ändå?** Installera Esris säkerhetspatch för berörda Server-versioner och rätta separat använd .NET-runtime enligt Microsoft. Följ Esris aktuella produktbesked om ArcGIS Pro; kontrollera faktiska produkt- och runtimeversioner. **Vänta inte på egen CVE-reproduktion.** [Esris besked](https://www.esri.com/arcgis-blog/products/trust-arcgis/administration/brotli-vulnerabilities-broccoli2shell) · [Daterad patchöversikt](patchkedjan.md).

**Tekniska belägg och vidare läsning:** [README](../README.md) · [Patchkedja](patchkedjan.md) · [STAC-flöde](verksamhetsfloden.md#2-arcgis-pro--lantmäteriets-stac)

## ArcGIS Server → Node.js

```text
ArcGIS Server-funktion som använder berörd Node.js/Brotli
  → Node.js-runtime
  → inbyggd Brotli-avkodare
```

**Vad är känt?** Esri klassificerar ArcGIS Enterprise som påverkad och beskriver ArcGIS Server 11.5, 12.0 och 12.1. Esris patch **ArcGIS Server Security 2026 Update 4** åtgärdar `BUG-000188308` genom att uppdatera Node.js. Esri anger att **11.1 och 11.3 inte påverkas av just detta Node.js/Brotli-fel**. [Esris produktbesked](https://www.esri.com/arcgis-blog/products/trust-arcgis/administration/brotli-vulnerabilities-broccoli2shell) · [patchsida](https://support.esri.com/en-us/patches-updates/2026/arcgis-server-security-2026-update-4-patch).

**Omständigheter:** För direkt CVE-utlösning måste angriparpåverkad komprimerad indata nå den berörda Node.js-avkodaren genom en faktiskt aktiverad funktion. Vilken specifik extern begäran eller ArcGIS-funktion som uppfyller det villkoret är **inte fullständigt kartlagd i Esris offentliga besked**. Leverantörens bekräftade påverkan är tillräcklig grund för att rätta, men inte för att påstå RCE via en bestämd ArcGIS-endpoint.

**Patch finns:** Ja. Esris patchsida listar filer och SHA-256 för **ArcGIS Server 11.5, 12.0 och 12.1 på Windows och Linux** (publicerad 5 november). Välj patch efter **både produktversion och plattform** och tillämpa på varje berörd ArcGIS Server-installation. Kontrollera installerad patch med Esris verktyg och att tjänsterna har startats om. En uppdaterad server innebär inte att resten av sajten är rättad.

## ArcGIS Pro och .NET

```text
Funktion i ArcGIS Pro / Web Adaptor / SDK-baserad tjänst
  → .NET-API med aktiverad Brotli-avkodning
  → .NET:s medföljande Brotli-avkodare
```

**Omständigheter:** Microsoft anger att **ASP.NET Core request decompression och HttpClient automatic decompression inte är påslagna som standard**. Relevanta vägar är exempelvis uttryckligen aktiverad avkodning av inkommande begäranden, `HttpClient` med Brotli i `AutomaticDecompression`, eller direkt användning av `BrotliStream`/`BrotliDecoder` på angriparpåverkade data. När dessa vägar inte används kan ett närvarande .NET-paket vara irrelevant för just denna attackväg. [Microsofts advisory](https://github.com/dotnet/announcements/issues/494).

**Vad är känt?** Enligt Esris [produktbesked, uppdaterat 7 november 2026 och avläst 8 november 2026](https://www.esri.com/arcgis-blog/products/trust-arcgis/administration/brotli-vulnerabilities-broccoli2shell) anges påverkan för **ArcGIS Pro 3.5 och senare** samt att separat hanterad .NET-runtime används av bland annat ArcGIS Pro, **Enterprise Web Adaptor (IIS)**, SOE/SOI-tillägg och Cloud Builder. Det finns **inte** någon generell slutsats att varje sådan produktkonfiguration har en internetnåbar Brotli-ingång. Esris senaste avlästa produktbesked (uppdaterat 7 november) anger att produktpatch för **ArcGIS Pro 3.5+ väntas**. Kontrollera senare ändringar hos Esri före publicering eller driftbeslut.

**.NET-patch finns:** Microsoft anger de första rättade versionerna **8.0.33**, **9.0.22** och **10.0.14** (och angiven rättad 11-förhandsversion). Uppdatera den runtime som produkten faktiskt använder. **Self-contained** .NET-applikationer kan kräva ombyggnad och ny distribution: enbart rättad delad runtime räcker inte. Uppdatering av .NET ersätter inte Esris separata ArcGIS-produktpatch. [.NET-advisory](https://github.com/dotnet/announcements/issues/494).

## ArcGIS Online och SDK-baserade appar

**ArcGIS Online:** Esri anger införda begränsningsåtgärder och pågående uppdateringar; det är en leverantörshanterad tjänst och inte en lokal Node/.NET-patch för kunden. **ArcGIS Maps SDK for .NET 200.0+:** Esri anger att appar kan beröras; åtgärden är rättad .NET-runtime och **ombyggnad av appen**, inte nödvändigtvis uppgradering av SDK-versionen. [Esri](https://www.esri.com/arcgis-blog/products/trust-arcgis/administration/brotli-vulnerabilities-broccoli2shell).

## Sammanfattning

| Del | Påverkansbesked | CVE-ingång till avkodare | Patch |
| --- | --- | --- | --- |
| ArcGIS Server 11.5/12.0/12.1 | **Bekräftad** | Node.js-komponent; exakt extern ingång inte beskriven | **Ja**, Security 2026 Update 4 |
| ArcGIS Server 11.1/11.3 | Esri anger **inte** berörda av just denna Node.js-fråga | Ingen generell slutsats om andra delar eller framtida byggen | Följ produktens övriga support-/patchstatus |
| ArcGIS Pro 3.5+ | **Bekräftad** | Produktspecifika dataflöden inte fullt kartlagda | Produktpatch **väntades 7 nov**; .NET separat rättad |
| .NET i Web Adaptor, SOE/SOI m.fl. | Esri anger kundhanterad runtime | Bara konfigurationer som avkodar angriparpåverkad Brotli | **Ja**, .NET 8.0.33 / 9.0.22 / 10.0.14 |

## Slutbedömningar för ArcGIS-kedjorna

**ArcGIS Server och Node.js**

**Slutbedömning av denna riskkedja – exempel, inte en verifierad lokal installation:**

| Fält | Bedömning |
| --- | --- |
| **Påverkan** | **Leverantörsbekräftad** påverkan för ArcGIS Server 11.5, 12.0 och 12.1 via Node-komponent. |
| **Exponering** | Exakt angriparstyrd avkodningsingång är inte visad i underlaget; kontrollera faktiskt använda funktioner. |
| **Konsekvens** | Tjänstavbrott och möjlig vidare skada är villkorade; ingen generell ArcGIS Server-RCE visad här. |
| **Prioritet** | **Patcha berörda aktiva ArcGIS Server-utgåvor skyndsamt** enligt Esris säkerhetsbesked, även om exakt endpoint inte publicerats. |
| **Rättning** | **ArcGIS Server Security 2026 Update 4**, vald för produktversion och OS; kontrollera installerad patch. |
| **Kvarvarande osäkerhet** | Vilka externa indata når Node-avkodaren i just er driftsättning? |

**ArcGIS Pro och .NET/andra beroenden**

**Slutbedömning av denna riskkedja – exempel, inte en verifierad lokal installation:**

| Fält | Bedömning |
| --- | --- |
| **Påverkan** | Esri rapporterar påverkan i Pro 3.5+; .NET har separat Microsoft-rättning. |
| **Exponering** | Manual/automatisk inläsning eller HTTP-hämtning kräver egen kontroll av faktisk Brotli-avkodning. |
| **Konsekvens** | Möjligt avbrott i GIS-klienten; kodkörning/röjande i Pro inte bevisat av dessa källor. |
| **Prioritet** | **Inventera exponerade Pro-arbetsflöden och produktbyggen**, följ samtidigt Esris aktuella säkerhetspatchar. |
| **Rättning** | Esris Pro-produktuppdatering när bekräftad för berörd version; .NET 8.0.33/9.0.22/10.0.14 där denna runtime används. |
| **Kvarvarande osäkerhet** | Vilken kodväg avkodar i Pro, aktuell Pro-patch och om distribuerade .NET-appar måste byggas om? |

**Öppna frågor:** Vilken komponent avkodar en specifik extern begäran eller tjänsterespons i respektive ArcGIS-version? Vilket runtime-bygge kör varje driftsatt produkt? Vad är den nyaste produktpatchstatusen för Pro? Besvara via aktuell Esri-dokumentation och installationens versioner, inte antaganden från CVSS.

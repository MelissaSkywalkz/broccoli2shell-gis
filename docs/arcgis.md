# ArcGIS: berörda versioner och separata rättningar

**Källstatus: 21 november 2026.** Esris säkerhetsbesked publicerades 29 oktober och uppdaterades 17 november med ArcGIS Enterprise 12.2 och instruktioner för kundhanterad Node.js. Esri Japan bekräftade 18 november att ArcGIS Server Security 2026 Update 4 även tillhandahålls för 12.2. Enligt Esris besked uppdaterat **20 november** påverkas ArcGIS Pro **3.5, 3.6 och 3.7**, medan **3.8.1 och senare** samt **ArcGIS Online** klassificeras som **inte påverkade**. Microsofts separata .NET-besked publicerades 25 oktober.

Felet finns i Brotli-avkodning som följer med underliggande komponenter, bland annat Node.js i ArcGIS Server och .NET i vissa andra ArcGIS-komponenter. Esri bekräftar produktpåverkan, men grundfelet ligger i Brotli.

**Vad vet vi?** Esri rapporterar påverkan för ArcGIS Server **11.5, 12.0, 12.1 och 12.2** samt ArcGIS Pro **3.5, 3.6 och 3.7** enligt besked uppdaterat 20 november 2026. Esri klassificerar **3.8.1 och senare** som inte påverkade. **ArcGIS Server Security 2026 Update 4** rättar den berörda Node.js-komponenten. Den kundhanterade .NET-körmiljön är en **separat** rättningskedja.

**Bevisgräns:** Esris produktbesked visar inte att varje ArcGIS-ingång kan utlösa CVE:n eller att kodkörning har demonstrerats i ArcGIS. Esris besked anger att produktpatchar för ArcGIS Pro 3.5–3.7 fortfarande väntas **20 november**; det är inte en garanti för status vid senare läsning.

**Åtgärd:** Installera Esris säkerhetspatch för berörda Server-versioner. Uppdatera separat använd .NET-runtime enligt Microsoft och kontrollera Esris aktuella besked om ArcGIS Pro. Verifiera installerade produkt- och runtimeversioner. [Esris besked](https://www.esri.com/arcgis-blog/products/trust-arcgis/administration/brotli-vulnerabilities-broccoli2shell) · [Patchöversikt](patchkedjan.md).

**Källor och fördjupning:** [README](../README.md) · [Patchkedja](patchkedjan.md) · [STAC-flöde](verksamhetsfloden.md#2-arcgis-pro-lantmäteriets-stac)

## ArcGIS Server → Node.js

```text
ArcGIS Server-funktion som använder berörd Node.js/Brotli
  → Node.js-runtime
  → inbyggd Brotli-avkodare
```

**Vad är känt?** Esri klassificerar ArcGIS Enterprise som påverkad och beskriver ArcGIS Server 11.5, 12.0, 12.1 och 12.2. Esris patch **ArcGIS Server Security 2026 Update 4** åtgärdar `BUG-000188308` genom att uppdatera Node.js. Esri anger att **11.1 och 11.3 inte påverkas av just detta Node.js/Brotli-fel**. [Server-patch](https://support.esri.com/en-us/patches-updates/2026/arcgis-server-security-2026-update-4-patch) · [Esri Japan, uppdatering 18 november](https://support.esrij.com/news/38149).

**Omständigheter:** För direkt CVE-utlösning måste angriparpåverkad komprimerad indata nå den berörda Node.js-avkodaren genom en faktiskt aktiverad funktion. Vilken specifik extern begäran eller ArcGIS-funktion som uppfyller det villkoret är **inte fullständigt kartlagd i Esris offentliga besked**. Leverantörens bekräftade påverkan är tillräcklig grund för att rätta, men inte för att påstå RCE via en bestämd ArcGIS-endpoint.

**Patch finns:** Ja. Esris ursprungliga patchsida listar filer och SHA-256 för **ArcGIS Server 11.5, 12.0 och 12.1 på Windows och Linux** (publicerad 5 november). Esri Japan bekräftade 18 november att patchen även finns för **12.2**; kontrollera den versionsspecifika nedladdningen eftersom leverantörens olika sidor inte nödvändigtvis uppdateras samtidigt. Välj patch efter **både produktversion och plattform** och tillämpa på varje berörd ArcGIS Server-installation. Kontrollera installerad patch med Esris verktyg och att tjänsterna har startats om. En Server-patch rättar inte automatiskt andra ArcGIS-komponenter.

**Tidigare begränsningsåtgärd:** Esri anger att den som tillfälligt stängt av *ArcGIS Enterprise SDK custom data feeds* kan aktivera funktionen igen **när Security 2026 Update 4 har installerats** på berörd ArcGIS Server. Kontrollera att patchen är installerad innan funktionen återaktiveras.

## ArcGIS Mission Server: separat säkerhetspatch

Esri publicerade **ArcGIS Mission Server Security 2026 Update 1** den **9 november 2026** för Mission Server **11.5, 12.0 och 12.1** på Windows och Linux. I säkerhetsbeskedet uppdaterat 17 november omfattas även **12.2**. Patchen åtgärdar `BUG-000188431`, som Esri kopplar till CVE-2026-111497. Det är en **egen produktpatch**, inte samma patch som ArcGIS Server Security 2026 Update 4 (`BUG-000188308`). Installera endast den Mission Server-patch som motsvarar produktversion och plattform; följ Esris anvisningar för kontroll efter installation. Detta leverantörsbesked verifierar inte en specifik extern angreppsväg eller RCE i Mission Server. [Esris Mission Server-patch](https://support.esri.com/en-us/patches-updates/2026/arcgis-mission-server-security-2026-update-1-patch).

## Kundhanterad Node.js i Enterprise SDK och Experience Builder

Esri har i beskedet uppdaterat **17 november 2026** lagt till instruktioner för Node.js som kunden själv installerar och underhåller.

**ArcGIS Enterprise SDK 11.1, 11.3, 11.5, 12.0 och 12.1:** Esri anger att lösningar som byggts med SDK:n och driftsatts på ArcGIS Server med **Security 2026 Update 4 installerad** inte är sårbara för den aktuella frågan. I utvecklingsmiljöer med SDK före 12.2 kan en äldre kompatibel Node.js-version behövas vid installationen; därefter ska Node.js uppdateras till en rättad version enligt Esris instruktioner.

**ArcGIS Experience Builder:** Om Node.js används behöver både applikationsserverns och utvecklingsmiljöns kundhanterade Node.js-installationer kontrolleras och uppdateras.

En ArcGIS Server-patch uppdaterar inte automatiskt en separat Node.js-runtime i Experience Builder eller en utvecklingsmiljö. Kontrollera den version som faktiskt körs. [Esris säkerhetsbesked](https://www.esri.com/arcgis-blog/products/trust-arcgis/administration/brotli-vulnerabilities-broccoli2shell).

## ArcGIS Pro och .NET

```text
Funktion i ArcGIS Pro / Web Adaptor / SDK-baserad tjänst
  → .NET-API med aktiverad Brotli-avkodning
  → .NET:s medföljande Brotli-avkodare
```

**Omständigheter:** Microsoft anger att **ASP.NET Core request decompression och HttpClient automatic decompression inte är påslagna som standard**. Relevanta vägar är exempelvis uttryckligen aktiverad avkodning av inkommande begäranden, `HttpClient` med Brotli i `AutomaticDecompression`, eller direkt användning av `BrotliStream`/`BrotliDecoder` på angriparpåverkade data. När dessa vägar inte används kan ett närvarande .NET-paket vara irrelevant för just denna attackväg. [Microsofts advisory](https://github.com/dotnet/announcements/issues/494).

**Vad är känt?** Enligt Esris besked uppdaterat **20 november 2026** påverkas **ArcGIS Pro 3.5, 3.6 och 3.7**, och produktspecifika patchar väntas för dessa versioner. **ArcGIS Pro 3.8.1 och senare** klassificeras som **inte påverkade** av denna CVE. Esri beskriver separat hanterad .NET-runtime för bland annat ArcGIS Pro, **Enterprise Web Adaptor (IIS)**, SOE/SOI-tillägg och Cloud Builder. Det finns **inte** någon generell slutsats att varje sådan produktkonfiguration har en internetnåbar Brotli-ingång. Kontrollera både installerad produktversion och faktiskt använd .NET-runtime; en produktklassificering verifierar inte alla lokala beroenden.

**.NET-patch finns:** Microsoft anger de första rättade versionerna **8.0.33**, **9.0.22** och **10.0.14** (och angiven rättad 11-förhandsversion). Uppdatera den runtime som produkten faktiskt använder. **Self-contained** .NET-applikationer kan kräva ombyggnad och ny distribution: enbart rättad delad runtime räcker inte. Uppdatering av .NET ersätter inte Esris separata ArcGIS-produktpatch. [.NET-advisory](https://github.com/dotnet/announcements/issues/494).

## ArcGIS Online och SDK-baserade appar

**ArcGIS Online:** Esri klassificerar tjänsten som **inte påverkad** enligt säkerhetsbeskedet uppdaterat **20 november 2026**. Tidigare angavs begränsningsåtgärder och pågående uppdateringar. Tjänsten är leverantörshanterad och kräver inte lokal Node/.NET-patchning av kunden för själva ArcGIS Online. **ArcGIS Maps SDK for .NET 200.0+:** Esri anger att appar kan beröras; åtgärden är rättad .NET-runtime och **ombyggnad av appen**, inte nödvändigtvis uppgradering av SDK-versionen.

## Sammanfattning

| Del | Påverkansbesked | CVE-ingång till avkodare | Patch |
| --- | --- | --- | --- |
| ArcGIS Server 11.5/12.0/12.1/12.2 | **Bekräftad** | Node.js-komponent; exakt extern ingång inte beskriven | **Ja**, Security 2026 Update 4 |
| ArcGIS Mission Server 11.5/12.0/12.1/12.2 | **Bekräftad enligt separat Esri-patch** | Produktspecifik extern ingång inte visad här | **Ja**, Security 2026 Update 1 |
| ArcGIS Server 11.1/11.3 | Esri anger **inte** berörda av just denna Node.js-fråga | Ingen generell slutsats om andra delar eller framtida byggen | Följ produktens övriga support-/patchstatus |
| ArcGIS Pro 3.5/3.6/3.7 | **Bekräftad** | Produktspecifika dataflöden inte fullt kartlagda | Produktpatchar **väntar enligt besked 20 nov**; .NET separat |
| ArcGIS Pro 3.8.1 och senare | **Inte påverkade enligt Esri** | Avser Esris produktklassificering, inte en verifiering av varje lokal runtime | Ingen CVE-specifik Pro-patch behövs enligt beskedet |
| ArcGIS Online | **Inte påverkad enligt Esri 20 nov** | Leverantörshanterad tjänst | Ingen kundinstallerad produktpatch |
| Enterprise SDK / Experience Builder | Kundhanterad Node.js kan behöva uppdateras separat | Exakt exponering beror på driftsättning och använd runtime | Följ Esris instruktioner från 17 nov; kontrollera körande Node.js-version |
| .NET i Web Adaptor, SOE/SOI m.fl. | Esri anger kundhanterad runtime | Bara konfigurationer som avkodar angriparpåverkad Brotli | **Ja**, .NET 8.0.33 / 9.0.22 / 10.0.14 |

## Slutbedömningar för ArcGIS-kedjorna

För varje installation behöver ansvarig kontrollera produktversion, tillämplig Esri-patch och faktiskt använd .NET-runtime. Driftsätt Server-patchen på samtliga berörda noder och kontrollera resultatet efter omstart. För ArcGIS Pro krävs fortsatt kontroll av Esris produktspecifika patchstatus; en uppdaterad .NET-runtime ersätter inte en eventuell Pro-patch.

Den exakta väg som angriparpåverkad data kan ta till Brotli-avkodaren i en viss ArcGIS-installation är fortfarande okänd i detta underlag. Det påverkar bedömningen av exponering, men är inte skäl att avvakta leverantörens rättning.

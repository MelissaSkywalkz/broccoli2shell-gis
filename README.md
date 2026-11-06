# Broccoli2Shell i GIS

[![CVE](https://img.shields.io/badge/CVE-2026--111497-f12c35)](https://www.cve.org/CVERecord?id=CVE-2026-111497)
[![CVSS](https://img.shields.io/badge/CVSS-9.2%20Critical-f12c35)](https://www.cve.org/CVERecord?id=CVE-2026-111497)
[![CISA KEV](https://img.shields.io/badge/CISA%20KEV-Listed-d97706)](https://www.cisa.gov/known-exploited-vulnerabilities-catalog?search=CVE-2026-111497)

**Uppdaterad: 6 november 2026**

Broccoli2Shell (CVE-2026-111497), även kallad [Node2Shell och HTTP2Shell](https://www.esri.com/arcgis-blog/products/trust-arcgis/administration/brotli-vulnerabilities-broccoli2shell), är en kritisk sårbarhet i Google Brotli, ett bibliotek för att komprimera och packa upp data. Om en sårbar avkodare packar upp särskilt utformade Brotli-data kan det leda till att programmet kraschar, röjer minnesinnehåll eller potentiellt kör kod som angriparen styr. Google anger CVSS 9,2. Sårbarheten finns i [CISA:s katalog över kända utnyttjade sårbarheter](https://www.cisa.gov/known-exploited-vulnerabilities-catalog?search=CVE-2026-111497).

Brotli används i komponenter som förekommer i GIS-miljöer. Kontrollera vilka versioner och komponenter som används och följ leverantörernas anvisningar. [CERT-SE](https://www.cert.se/2026/10/kritisk-sarbarhet-i-google-brotli.html) rekommenderar skyndsam uppdatering av berörda instanser.

## GIS-produkter

Lägesbild den 6 november 2026. Tabellen skiljer leverantörernas säkerhetsbesked från dokumenterade beroenden och paketuppgifter.

| Produkt | Vad är känt? | Originalkälla | Kontroller och åtgärder |
| --- | --- | --- | --- |
| **FME Form och Flow** | **Leverantörsbesked.** Safe bekräftar påverkan för flera versioner och förbereder rättade byggen. | [Safe](https://support.safe.com/hc/en-us/articles/50040725669517-Brotli-Decoder-CVE-2026-111497-Is-FME-Impacted) | [FME](atgarder.md#fme-form-och-flow) |
| **ArcGIS** | **Leverantörsbesked.** Esri bekräftar påverkan för ArcGIS Enterprise och ArcGIS Pro 3.5 och senare. Kontrollera produktpatchar och de separata .NET-uppdateringar Esri anger. Patch för ArcGIS Pro inväntas. | [Esri](https://www.esri.com/arcgis-blog/products/trust-arcgis/administration/brotli-vulnerabilities-broccoli2shell) | [ArcGIS](atgarder.md#arcgis) |
| **QGIS och OSGeo4W** | **Paket och byggskript.** För QGIS-installationer via OSGeo4W på Windows finns ett rättat Brotli-paket. Linuxdistributioner har också publicerat uppdateringar. Kontrollera hur QGIS är installerat. | [QGIS](https://www.qgis.org/resources/support/security/) · [OSGeo4W](https://download.osgeo.org/osgeo4w/v2/x86_64/setup.ini.bz2) | [QGIS](atgarder.md#qgis-och-osgeo4w) |
| **Origo Server** | **Källkod och komponentbesked.** Origo Server använder Node.js, som har fått en rättning för sårbarheten. Kontrollera vilken Node.js-version servern kör. | [Origo Server](https://github.com/origo-map/origo-server) · [Express](https://expressjs.com/en/blog/2026-10-31-security-releases/) | [Origo Server](atgarder.md#origo-server) |
| **Hajk** | **Projektets dokumentation och källkod.** Hajks zipfiler innehåller ingen Node.js-runtime, bara statiska JS-filer. Den som driftar Hajk från zipfil ansvarar själv för att hålla driftmiljön, inklusive Node.js, uppdaterad. Hajks Docker-avbilder innehåller däremot en Node.js-runtime. Där rekommenderar projektet avbilden `nightly-master`: den senaste officiella releasen (4.3.0), byggd om varje natt med de senaste paketen i grundavbilden. | [Hajk](https://github.com/hajkmap/Hajk/blob/master/docs/releases.md) · [Säkerhetspolicy](https://github.com/hajkmap/Hajk/blob/master/SECURITY.md#official-docker-images) | [Hajk](atgarder.md#hajk) |

QGIS, ArcGIS Pro och andra GIS-program kan också ha egna Pythonmiljöer. Se [Pythonpaket i GIS-miljöer](atgarder.md#pythonpaket-i-gis-miljöer) för paket som kan behöva kontrolleras.

## Vad behöver jag göra?

Om du inte vet hur produkten är installerad, börja med produktavsnittet. Där framgår vad den som hanterar installationen behöver kontrollera.

1. Ta reda på vilka GIS-produkter och versioner som används.
2. Läs tillgängliga källor och rekommendationer för produkten. Se [kontroller och åtgärder per produkt](atgarder.md).
3. Uppdatera enligt leverantörens eller distributionens instruktioner. Starta om där det behövs och kontrollera vilken version som faktiskt körs.
4. Om rättning saknas, följ leverantörens tillfälliga åtgärder. [CERT-SE](https://www.cert.se/2026/10/kritisk-sarbarhet-i-google-brotli.html) rekommenderar att begränsa åtkomsten till Brotli eller undvika exponering mot osäkra nätverk när uppdatering inte är möjlig.

Om flera installationer berörs, prioritera enligt leverantörernas besked. Se särskilt över system som kan ta emot eller hämta Brotli-komprimerade data från källor du inte styr.

## Viktigt att känna till

- Att Brotli finns installerat betyder inte automatiskt att en installation kan angripas. Angriparen måste kunna tillföra data som programmet packar upp. [Node.js](https://nodejs.org/en/blog/vulnerability/october-2026-security-releases) och [Microsoft](https://github.com/dotnet/announcements/issues/494) beskriver detta villkor för sina komponenter.
- En tjänst med enbart intern åtkomst kan ändå behandla externa data. [Safe](https://support.safe.com/hc/en-us/articles/50040725669517-Brotli-Decoder-CVE-2026-111497-Is-FME-Impacted) beskriver detta för FME-arbetsytor som hämtar HTTP-svar eller läser filer.
- Linuxdistributioner kan ha säkerhetsrättningar i äldre paketversioner. Kontrollera paketstatus hos [Debian](https://security-tracker.debian.org/tracker/CVE-2026-111497) eller [Ubuntu](https://ubuntu.com/security/CVE-2026-111497) i stället för att jämföra enbart grundversionen.

En version som saknas i en patchlista är inte därmed bekräftat opåverkad.

## Rättade versioner av Google Brotli

Tabellen gäller själva Brotli-biblioteket. Pythonpaketen `Brotli` och `brotlicffi` har separata rättningsuppgifter, se [Pythonpaket](atgarder.md#pythonpaket-i-gis-miljöer).

| Berörda versioner | Rättad version | Utgåva |
| --- | --- | --- |
| `1.0.3`–`1.0.9` | `1.0.10` | [v1.0.10](https://github.com/google/brotli/releases/tag/v1.0.10) |
| `1.1.0` | `1.1.1` | [v1.1.1](https://github.com/google/brotli/releases/tag/v1.1.1) |
| `1.2.0` | `1.2.1` | [v1.2.1](https://github.com/google/brotli/releases/tag/v1.2.1) |

Produkter och Linuxdistributioner kan ha andra paketversioner. Källa: [CVE-posten](https://www.cve.org/CVERecord?id=CVE-2026-111497).

## Håll dig uppdaterad

Följ originalkällorna i tabellen och [åtgärdsfilen](atgarder.md). [CERT-SE](https://www.cert.se/prenumerera/) har prenumeration på säkerhetsinformation.

## Om vägledningen

Det här är en fristående sammanställning med fokus på GIS. Den representerar ingen arbetsgivare, myndighet eller leverantör och utfärdar inga egna säkerhetsbesked. Uppgifter från källkod och paketregister är inte bekräftelser från produktens leverantör om påverkan. Följ alltid de aktuella originalkällorna.

Har du en relevant officiell källa eller rättelse? Öppna gärna ett [GitHub Issue](https://github.com/MelissaSkywalkz/broccoli2shell-gis/issues). Publicera inte interna systemuppgifter, känsliga loggar eller uppgifter om enskilda installationers säkerhet.

## Licens

Vägledningens originalinnehåll är licensierat under [Creative Commons Erkännande 4.0 Internationell (CC BY 4.0)](LICENSE). Licensen omfattar inte material från externa källor.
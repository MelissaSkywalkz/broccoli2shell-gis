# Broccoli2Shell i GIS

[![CVE](https://img.shields.io/badge/CVE-2026--111497-f12c35)](https://www.cve.org/CVERecord?id=CVE-2026-111497)
[![CVSS](https://img.shields.io/badge/CVSS-9.2%20Critical-f12c35)](https://www.cve.org/CVERecord?id=CVE-2026-111497)
[![CISA KEV](https://img.shields.io/badge/CISA%20KEV-Listed-d97706)](https://www.cisa.gov/known-exploited-vulnerabilities-catalog?search=CVE-2026-111497)

**Uppdaterad: 5 november 2026**

Broccoli2Shell (CVE-2026-111497), även kallad Node2Shell eller HTTP2Shell, är en kritisk sårbarhet i Google Brotli, ett bibliotek för komprimering och dekomprimering av data. Google har klassat sårbarheten som kritisk (CVSS 9,2). Om en sårbar avkodare behandlar särskilt utformade Brotli-data kan det bland annat leda till att program kraschar, minnesinnehåll röjs eller kod körs.

Sårbarheten finns i [CISA:s katalog över kända exploaterade sårbarheter](https://www.cisa.gov/known-exploited-vulnerabilities-catalog). [CERT-SE](https://www.cert.se/2026/10/kritisk-sarbarhet-i-google-brotli.html) rekommenderar att berörda installationer uppdateras enligt leverantörernas anvisningar.

Brotli används också i komponenter som förekommer i GIS-miljöer. **Kontrollera leverantörernas besked, vilka versioner som faktiskt används och uppdatera vid behov.**

## GIS-produkter

Lägesbild per 5 november 2026. Tabellen skiljer mellan leverantörsbekräftad påverkan och dokumenterade beroenden eller paketuppdateringar.

| Produkt | Vad är bekräftat? | Originalkälla | Kontroller och åtgärder |
| --- | --- | --- | --- |
| **FME Form / Flow** | Safe bekräftar påverkan och förbereder Brotli-rättningar. | [Safe](https://support.safe.com/hc/en-us/articles/50040725669517-Brotli-Decoder-CVE-2026-111497-Is-FME-Impacted) | [FME](atgarder.md#fme-form-och-flow) |
| **ArcGIS** | Esri bekräftar påverkan för angivna produkter och versioner. | [Esri](https://www.esri.com/arcgis-blog/products/trust-arcgis/administration/brotli-vulnerabilities-broccoli2shell) | [ArcGIS](atgarder.md#arcgis) |
| **QGIS / OSGeo4W** | Rättat Brotli-paket finns i OSGeo4W. Det är paketinformation, inte ett besked om QGIS som helhet. | [OSGeo4W](https://download.osgeo.org/osgeo4w/v2/x86_64/setup.ini.bz2) | [QGIS](atgarder.md#qgis--osgeo4w) |
| **Origo Server** | Serverkoden använder Node.js och Express. Node.js har fått en Brotli-rättning. | [Origo Server](https://github.com/origo-map/origo-server) · [Node.js](https://nodejs.org/en/blog/vulnerability/october-2026-security-releases) | [Origo Server](atgarder.md#origo-server) |
| **Hajk** | NodeJS Edition använder Node.js i drift. Simple Edition saknar egen Node.js-backend. | [Hajk](https://github.com/hajkmap/Hajk/blob/master/docs/releases.md) | [Hajk](atgarder.md#hajk) |

## Varför är det viktigt att kontrollera?

Broccoli2Shell är en allvarlig sårbarhet och berörda leverantörer har publicerat säkerhetsbesked, rättningar eller tillfälliga åtgärder. Det är därför viktigt att ta reda på vad som gäller för de produkter och versioner som används.

En säkerhetsuppdatering kan kräva planering, tester eller en större versionsuppgradering. **Att en version saknas i en patchlista innebär inte att den är bekräftat opåverkad eller att inga åtgärder behövs.** Följ leverantörens fortsatta besked eller kontakta supporten för versionsspecifik vägledning.

En leverantörs besked om att den egna produkten inte påverkas utesluter inte att bakomliggande komponenter behöver uppdateras.

## Rättade versioner av Google Brotli

| Berörda versioner | Rättad version |
| --- | --- |
| `1.0.3`–`1.0.9` | `1.0.10` |
| `1.1.0` | `1.1.1` |
| `1.2.0` | `1.2.1` |

Källa: [Googles CVE-post](https://www.cve.org/CVERecord?id=CVE-2026-111497) och [officiella utgåvor](https://github.com/google/brotli/releases).

Dessa är versioner av själva Brotli-biblioteket. Produkter och Linuxdistributioner kan använda andra paketversioner eller ha infört rättningen i en äldre versionsgren. Följ därför respektive leverantörs eller distributions säkerhetsbesked.

## Om vägledningen

Detta är en fristående sammanställning med fokus på GIS. Den representerar inte någon arbetsgivare, myndighet eller leverantör och utfärdar inga egna säkerhetsbesked.

Att ett bibliotek eller beroende förekommer i en produkt är inte i sig en bekräftelse på att en viss installation är exploaterbar. Där produktspecifik påverkan inte har bekräftats skiljer vägledningen uttryckligen på det och dokumenterade tekniska beroenden.

Rättelser kan följas i Git-historiken. Har du en relevant officiell källa eller rättelse? Öppna gärna ett [GitHub Issue](https://github.com/MelissaSkywalkz/broccoli2shell-gis/issues). Publicera inte interna systemuppgifter, känsliga loggar eller uppgifter om enskilda installationers säkerhet.
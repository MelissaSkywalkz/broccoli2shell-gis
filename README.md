# Broccoli2Shell i GIS

[![CVE](https://img.shields.io/badge/CVE-2026--111497-f12c35)](https://www.cve.org/CVERecord?id=CVE-2026-111497)
[![CVSS](https://img.shields.io/badge/CVSS-9.2%20Critical-f12c35)](https://github.com/advisories/GHSA-9v8h-qqgm-4m35)
[![CISA KEV](https://img.shields.io/badge/CISA%20KEV-Listed-d97706)](https://www.cisa.gov/known-exploited-vulnerabilities-catalog?search=CVE-2026-111497)

**Lägesbild: 6 november 2026**

Broccoli2Shell (**CVE-2026-111497**) är en kritisk sårbarhet i Googles Brotli-avkodare, som används för att packa upp komprimerade data. Om ett program använder en sårbar avkodare för att packa upp manipulerade Brotli-komprimerade data från en källa som en angripare kan påverka, kan programmet krascha, röja minnesinnehåll eller potentiellt köra kod som angriparen styr. Google anger **CVSS 9,2** och CISA har fört upp sårbarheten i sin [KEV-katalog](https://www.cisa.gov/known-exploited-vulnerabilities-catalog?search=CVE-2026-111497). [Källa: GitHubs säkerhetsråd](https://github.com/advisories/GHSA-9v8h-qqgm-4m35).

**Att ett program innehåller Brotli betyder inte automatiskt att det är exploaterbart.** Sårbarheten förutsätter att programmet använder den berörda avkodaren för data som en angripare kan påverka. En produktuppdatering, ett uppdaterat operativsystemspaket och en uppdaterad containeravbild kan dessutom vara olika åtgärder. Kontrollera den **komponent som tjänsten faktiskt använder**, och starta om eller återskapa tjänsten enligt produktens anvisningar. [Node.js](https://nodejs.org/en/blog/vulnerability/october-2026-security-releases) · [Microsoft .NET](https://github.com/dotnet/announcements/issues/494) · [Hajks Docker-anvisningar](https://github.com/hajkmap/Hajk/blob/master/docs/releases.md)

## GIS-produkter

Uppgifterna nedan skiljer mellan bekräftad produktpåverkan, dokumenterade komponenter och tillgängliga rättningar. **En rättning kan komma från produktprojektet, en beroendeleverantör eller en distribution utan ett separat säkerhetsbesked för GIS-produkten.** Kontrollera vad som har rättats och vilken version som faktiskt används i din installation.

| Produkt | Bekräftat underlag och vad som behöver kontrolleras | Originalkälla | Åtgärder |
| --- | --- | --- | --- |
| **FME Form och Flow** | **Bekräftad påverkan.** Safe anger påverkan genom FME Engine och Workbench; rättade FME-byggen förbereds. Uppdateringsvägen skiljer sig mellan Windows/macOS, Linux och DuckDB. | [Safe](https://support.safe.com/hc/en-us/articles/50040725669517-Brotli-Decoder-CVE-2026-111497-Is-FME-Impacted) | [FME](atgarder.md#fme-form-och-flow) |
| **ArcGIS** | **Bekräftad påverkan.** Esri anger påverkan för ArcGIS Enterprise och ArcGIS Pro 3.5 eller senare. Patchfiler för ArcGIS Server 11.5, 12.0 och 12.1 listas på Esris patchsida från 5 november; berörda .NET-körmiljöer uppdateras separat. | [Esri](https://www.esri.com/arcgis-blog/products/trust-arcgis/administration/brotli-vulnerabilities-broccoli2shell) · [Patchsida](https://support.esri.com/en-us/patches-updates/2026/arcgis-server-security-2026-update-4-patch) | [ArcGIS](atgarder.md#arcgis) |
| **QGIS och OSGeo4W** | **Rättningar finns i distributionspaket.** QGIS-projektet underhåller Windowsmiljön via OSGeo4W, som erbjuder rättat Brotli-paket. På Linux uppdateras systemberoenden via distributionen. Även 3liz har rättat Brotli i en utpekad QGIS Server-avbild. Kontrollera och uppdatera den installationsform som används. | [QGIS säkerhet](https://www.qgis.org/resources/support/security/) · [OSGeo4W](https://download.osgeo.org/osgeo4w/v2/x86_64/setup.ini.bz2) · [3liz](https://github.com/3liz/py-qgis-server/issues/137) | [QGIS](atgarder.md#qgis-och-osgeo4w) |
| **Origo Server** | **Dokumenterade beroenden.** Origo Server använder Node.js och Express. Kontrollera hur Node.js levereras och vilken körmiljö Origo-processen faktiskt använder. | [Origo Server](https://github.com/origo-map/origo-server) · [Express](https://expressjs.com/en/blog/2026-10-31-security-releases/) | [Origo Server](atgarder.md#origo-server) |
| **Hajk** | **Projektets dokumentation.** ZIP med backend kräver separat Node.js-körmiljö. Den fullständiga officiella Docker-avbilden innehåller Node.js; `hajk-simple` serverar statiska filer med nginx. Kontrollera distributionsform och avbild. | [Hajk](https://github.com/hajkmap/Hajk/blob/master/docs/releases.md) · [Säkerhetspolicy](https://github.com/hajkmap/Hajk/blob/master/SECURITY.md#official-docker-images) | [Hajk](atgarder.md#hajk) |

Pythonpaket kan också behöva kontrolleras i den miljö som GIS-programmet använder. Se [Pythonpaket och PyArrow](atgarder.md#pythonpaket-i-gis-miljöer).

## Vad behöver jag göra?

1. Identifiera **produkt, version, plattform och installationsform**.
2. Kontrollera enligt [produktens avsnitt](atgarder.md) vilken komponent som omfattas och vilken rättning leverantören eller distributionen tillhandahåller.
3. Uppdatera enligt den aktuella produktens anvisningar. **Starta om** berörda processer eller **återskapa containern från en uppdaterad avbild** där det behövs.
4. Kontrollera efteråt vilken komponent och version som faktiskt används. Ett paketregister eller en installerad ny version någon annanstans på datorn verifierar inte en körande tjänst.

## Rättade versioner av Google Brotli

Detta gäller **Googles Brotli-bibliotek**, inte automatiskt program som innehåller eller använder det. Distributioner kan ha fört in rättningen i paket med äldre grundversioner; kontrollera deras egna säkerhetsbesked.

| Berörd version | Rättad version | Google-utgåva |
| --- | --- | --- |
| `1.0.3`–`1.0.9` | `1.0.10` | [v1.0.10](https://github.com/google/brotli/releases/tag/v1.0.10) |
| `1.1.0` | `1.1.1` | [v1.1.1](https://github.com/google/brotli/releases/tag/v1.1.1) |
| `1.2.0` | `1.2.1` | [v1.2.1](https://github.com/google/brotli/releases/tag/v1.2.1) |

## Om vägledningen

Detta är en fristående GIS-inriktad sammanställning, **inte ett säkerhetsbesked från produkternas leverantörer**. Följ de länkade originalkällorna och kontrollera din egen installation.

## Licens

Vägledningens originalinnehåll är licensierat under [Creative Commons Erkännande 4.0 Internationell (CC BY 4.0)](LICENSE). Licensen omfattar inte material från externa källor.
# Broccoli2Shell i GIS

[![CVE](https://img.shields.io/badge/CVE-2026--111497-f12c35)](https://www.cve.org/CVERecord?id=CVE-2026-111497)
[![CVSS](https://img.shields.io/badge/CVSS%204.0-9.2%20Critical-f12c35)](https://www.cve.org/CVERecord?id=CVE-2026-111497)
[![CISA KEV](https://img.shields.io/badge/CISA%20KEV-Listed-d97706)](https://www.cisa.gov/known-exploited-vulnerabilities-catalog?field_cve=CVE-2026-111497)
[![EPSS 82,64 % den 6 november 2026](https://img.shields.io/badge/EPSS%202026--11--06-82%2C64%20%25-58616b)](https://api.first.org/data/v1/epss?cve=CVE-2026-111497&date=2026-11-06)

**Uppdaterad: 7 november 2026**

Broccoli2Shell (**CVE-2026-111497**) är en kritisk sårbarhet i Googles Brotli-avkodare, som används för att packa upp komprimerade data. Om ett program använder en sårbar avkodare för att packa upp manipulerade Brotli-komprimerade data från en källa som en angripare kan påverka, kan programmet krascha, röja minnesinnehåll eller potentiellt köra kod som angriparen styr. [CVE-posten](https://www.cve.org/CVERecord?id=CVE-2026-111497)

Google anger **9,2 enligt CVSS 4.0** i CVE-posten. Samma poäng visas i GitHubs säkerhetsråd. CISA lade till sårbarheten i KEV-katalogen den 27 oktober 2026. [CVE-posten](https://www.cve.org/CVERecord?id=CVE-2026-111497) · [GitHubs allvarlighetsbedömning](https://github.com/advisories/GHSA-9v8h-qqgm-4m35#severity) · [CISA KEV](https://www.cisa.gov/known-exploited-vulnerabilities-catalog?field_cve=CVE-2026-111497). [CERT-SE:s notis](https://www.cert.se/2026/10/kritisk-sarbarhet-i-google-brotli.html) sammanfattar sårbarheten och rekommenderar att berörda instanser uppdateras enligt leverantörens instruktioner.

EPSS-badgen visar FIRST:s daterade uppskattning av sannolikheten för att exploateringsaktivitet för sårbarheten observeras under de följande 30 dagarna. [Så tolkar du EPSS](https://www.first.org/epss/faq)

**Att ett program innehåller Brotli betyder inte automatiskt att det är exploaterbart.** Sårbarheten förutsätter att programmet använder den berörda avkodaren för data som en angripare kan påverka. En produktuppdatering, ett uppdaterat operativsystemspaket och en uppdaterad containeravbild kan dessutom vara olika åtgärder. Kontrollera den **komponent som tjänsten faktiskt använder**, och starta om eller återskapa tjänsten enligt produktens anvisningar. [Node.js](https://nodejs.org/en/blog/vulnerability/october-2026-security-releases) · [Microsoft .NET](https://github.com/dotnet/announcements/issues/494) · [Hajks Docker-anvisningar](https://github.com/hajkmap/Hajk/blob/master/docs/releases.md)

## GIS-produkter

Uppgifterna nedan skiljer mellan bekräftad produktpåverkan, dokumenterade komponenter och tillgängliga rättningar. **En rättning kan komma från produktprojektet, en beroendeleverantör eller en distribution utan ett separat säkerhetsbesked för GIS-produkten.** Kontrollera vad som har rättats och vilken version som faktiskt används i din installation.

| Produkt | Bekräftat underlag och vad som behöver kontrolleras | Originalkälla | Åtgärder |
| --- | --- | --- | --- |
| **FME Form och Flow** | **Leverantörsrapporterad påverkan.** Uppdateringsvägen beror på plattform och på om FME använder systemets eller en medföljande Brotli-kopia. Safe anger medföljande Brotli 1.1.0 för FME Form 2026.4 på Windows/macOS, men sammanfattningen slutar vid 2026.3. Rättade byggnummer har ännu inte bekräftats i det granskade beskedet. | [Safe](https://support.safe.com/hc/en-us/articles/50040725669517-Brotli-Decoder-CVE-2026-111497-Is-FME-Impacted) | [FME](atgarder.md#fme-form-och-flow) |
| **ArcGIS** | **Bekräftad påverkan.** Esri anger påverkan för ArcGIS Enterprise och ArcGIS Pro 3.5 eller senare. Esris kumulativa säkerhetspatch från 5 november 2026 gäller ArcGIS Server 11.5, 12.0 och 12.1 på Windows och Linux. Berörda .NET-körmiljöer uppdateras separat. | [Esri](https://www.esri.com/arcgis-blog/products/trust-arcgis/administration/brotli-vulnerabilities-broccoli2shell) · [Patchsida](https://support.esri.com/en-us/patches-updates/2026/arcgis-server-security-2026-update-4-patch) · [BUG-000188308](https://support.esri.com/en-us/bug/update-nodejs-to-address-brotli-cve2026111497-bug-000188308) | [ArcGIS](atgarder.md#arcgis) |
| **QGIS och OSGeo4W** | **Rättningar finns i distributionspaket.** QGIS-projektet underhåller Windowsmiljön via OSGeo4W, som erbjuder rättat Brotli-paket. På Linux uppdateras systemberoenden via distributionen. Även 3liz har rättat Brotli i en utpekad QGIS Server-avbild. Kontrollera och uppdatera den installationsform som används. | [QGIS säkerhet](https://www.qgis.org/resources/support/security/) · [OSGeo4W](https://download.osgeo.org/osgeo4w/v2/x86_64/setup.ini.bz2) · [3liz](https://github.com/3liz/py-qgis-server/issues/137) | [QGIS](atgarder.md#qgis-och-osgeo4w) |
| **Origo Server** | **Dokumenterade beroenden.** Origo Server använder Node.js och Express. Kontrollera hur Node.js levereras och vilken körmiljö Origo-processen faktiskt använder. | [Origo Server](https://github.com/origo-map/origo-server) · [Express](https://expressjs.com/en/blog/2026-10-31-security-releases/) | [Origo Server](atgarder.md#origo-server) |
| **Hajk** | **Projektets dokumentation.** ZIP med backend kräver separat Node.js-körmiljö. Den fullständiga officiella Docker-avbilden innehåller Node.js; `hajk-simple` serverar statiska filer med nginx. Kontrollera distributionsform och avbild. | [Hajk](https://github.com/hajkmap/Hajk/blob/master/docs/releases.md) · [Säkerhetspolicy](https://github.com/hajkmap/Hajk/blob/master/SECURITY.md#official-docker-images) | [Hajk](atgarder.md#hajk) |
| **GeoServer** | **Produktpåverkan ej fastställd.** Kontrollera faktisk distribution, servletcontainer och installerade tillägg för berörda Brotli-avkodare. | [GeoServers installationsformer](https://docs.geoserver.org/stable/en/user/installation/index.html) | [GeoServer](atgarder.md#geoserver) |

Även [Pythonpaket och PyArrow](atgarder.md#pythonpaket-i-gis-miljöer) och [Java-biblioteket Brotli4j](atgarder.md#java-bibliotek-och-brotli4j) kan behöva kontrolleras i GIS-miljöer.

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

**Källor och rättningsstatus:** [CVE-register](https://www.cve.org/) och [GitHub Advisory Database](https://docs.github.com/en/code-security/concepts/vulnerability-reporting-and-management/global-security-advisories) hjälper oss att identifiera sårbarheter och berörda paket, men omfattar inte automatiskt alla GIS-produkter. Uppgifterna bygger på leverantörernas säkerhetsbesked, projektens dokumentation och säkerhetspolicyer, distributionernas paketinformation samt granskad källkod. Ett separat produktspecifikt säkerhetsbesked krävs inte för att en beroendeuppdatering ska finnas. Den som ansvarar för en installation behöver kontrollera vilka rättningar som är relevanta och se till att de faktiskt används. Vi skiljer mellan **rättning under arbete**, **publicerad rättning** och **verifierad rättning i den aktuella installationen**.

**Nya, ännu inte offentliggjorda säkerhetsfynd** publiceras inte här utan [hanteras genom samordnad rapportering](https://docs.github.com/en/code-security/concepts/vulnerability-reporting-and-management/coordinated-disclosure) med ansvariga underhållare.

Detta är en fristående GIS-inriktad sammanställning, **inte ett säkerhetsbesked från produkternas leverantörer**. Följ de länkade originalkällorna och kontrollera din egen installation.

## Licens

Vägledningens originalinnehåll är licensierat under [Creative Commons Erkännande 4.0 Internationell (CC BY 4.0)](https://github.com/MelissaSkywalkz/broccoli2shell-gis/blob/483bd94cde326f6d687fdd58d17ae977912e57c4/LICENSE). Licensen omfattar inte material från externa källor.

# Broccoli2Shell i GIS

[![CVE-2026-111497](https://img.shields.io/badge/CVE-2026--111497-f12c35.svg)](https://www.cve.org/CVERecord?id=CVE-2026-111497)
[![CVSS 4.0: 9.2 Critical](https://img.shields.io/badge/CVSS%204.0-9.2%20Critical-f12c35.svg)](https://www.cert.se/2026/10/kritisk-sarbarhet-i-google-brotli.html)

Uppdaterad 4 november 2026

Broccoli2Shell (CVE-2026-111497), även kallad Node2Shell eller HTTP2Shell, är en kritisk sårbarhet i Google Brotli. Brotli används för att komprimera och packa upp data. Särskilt utformade Brotli-data kan krascha program som använder en sårbar avkodare. Angrepp kan också röja minnesinnehåll eller leda till att angriparens kod körs. CVSS 9,2.

Brotli ingår också i GIS-program och deras beroenden. Kontrollera vilka komponenter som används i installationen, inte bara GIS-programmets versionsnummer.

CISA lade till sårbarheten i sin [KEV-katalog](https://www.cisa.gov/known-exploited-vulnerabilities-catalog) den 27 oktober 2026. Katalogen listar sårbarheter som har utnyttjats i verkliga angrepp. Prioritera rättning enligt leverantörernas anvisningar.

[CERT-SE:s information om Google Brotli](https://www.cert.se/2026/10/kritisk-sarbarhet-i-google-brotli.html)

## Berörda Brotli-versioner

| Gren | Sårbara versioner | Rättad version |
| --- | --- | --- |
| 1.0 | `1.0.3` till `1.0.9` | `1.0.10` |
| 1.1 | `1.1.0` | `1.1.1` |
| 1.2 | `1.2.0` | `1.2.1` |

Versionerna kommer från [Googles CVE-post](https://www.cve.org/CVERecord?id=CVE-2026-111497). Distributioner kan ha rättat sårbarheten genom att föra in ändringen i äldre paketversioner, vilket exempelvis [Debian](https://security-tracker.debian.org/tracker/CVE-2026-111497) har gjort. Kontrollera därför distributionens säkerhetsbesked innan ni bedömer ett installerat paket enbart utifrån versionsnumret.

## Produktöversikt

| Produkt | Läget | Åtgärder |
| --- | --- | --- |
| QGIS / OSGeo4W | Brotli kan uppdateras separat. Andra komponenter kan ha egna kopior. | [QGIS](atgarder.md#qgis) |
| Origo Server | Node.js har fått säkerhetsrättningar. Kontrollera serverns körmiljö. | [Origo Server](atgarder.md#origo-server) |
| FME Form / Flow | Safe beskriver berörda funktioner och förbereder rättade byggen. | [FME](atgarder.md#fme) |
| ArcGIS | Esri anger påverkan på Enterprise och Pro. Rättningar varierar mellan versioner och komponenter. | [ArcGIS](atgarder.md#arcgis) |

Identifiera berörda installationer och följ [åtgärderna per produkt](atgarder.md). Kontrollera sedan installerad version och vilken version programmet eller tjänsten använder.

En installation är inte automatiskt möjlig att angripa bara för att den innehåller Brotli. Risken beror på om sårbar avkodning kan nås med data som en angripare styr.

## Bidra

Bidra gärna med källor om nya rättningar, fler berörda GIS-produkter eller dokumenterade åtgärder.

Öppna ett [issue](https://github.com/MelissaSkywalkz/broccoli2shell-gis/issues) med källhänvisning eller skicka en [pull request](https://github.com/MelissaSkywalkz/broccoli2shell-gis/pulls) med en rättelse. Ange produkt och version. Lägg inte upp interna systemuppgifter, lösenord eller känsliga loggar.

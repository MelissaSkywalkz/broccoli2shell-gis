# Bibliotek och runtime – när rättas fel kopia?

**Källkontroll: 8 november 2026**  

**Den generella modellen:** [CVE-felet](cve-mekanism.md) · [HTTP-ingångar](http-angreppsvagar.md) · [filingångar](fil-angreppsvagar.md) · [konsekvenser](konsekvenser.md) · [skydd](skydd.md) · [patchkedja](patchkedjan.md). Varje produktväg är en tillämpning av dessa villkor, inte ett oberoende säkerhetsbesked.

[Till attackvägsöversikten](produktoversikt.md) · [Metod](metod.md)

Den här sidan gäller **bibliotek som kan ingå i GIS-program och många andra applikationer**. Den aktuella CVE:n avser **Brotli-avkodaren i beroendekedjan**; en namngiven applikation blir inte därmed upphov till felet. För att bedöma exploaterbarhet måste man dessutom undersöka om angriparpåverkad komprimerad data kan nå den berörda avkodaren. Berörda leverantörsuppdateringar gäller ändå.

## Google Brotli / systembibliotek

```text
Applikation → delat libbrotlidec.so / brotlidec.dll
```

**Berört:** upstream 1.0.3–1.0.9, 1.1.0 och 1.2.0. **Rättat:** 1.0.10, 1.1.1 respektive 1.2.1. Distributioner kan backporta rättningen med gammalt grundversionsnummer. [GitHub advisory](https://github.com/advisories/GHSA-9v8h-qqgm-4m35) · [Debian](https://security-tracker.debian.org/tracker/CVE-2026-111497) · [Ubuntu](https://ubuntu.com/security/CVE-2026-111497).

**Särskilt villkor:** Programmet måste *använda* den sårbara avkodaren, inte bara ha biblioteket installerat. En rättad systemversion säger inget om separata inbyggda kopior i andra paket.

## Apache Arrow och PyArrow

```text
Parquet-datasida med codec BROTLI
  → Arrow Parquet-läsare
  → system-Brotli eller Arrow/PyArrow:s medföljande Brotli
```

**Varför är denna avkodningsväg relevant?** En angriparpåverkad Parquet-sida kan nå avkodningen när relevant kodväg används. Det kända felet sitter i Brotli-kopian, inte i ett här påvisat separat fel i Arrows tolkning av GIS-filer.

**Patch:** [Apache Arrow 26.0.1](https://arrow.apache.org/blog/2026/11/03/26.0.1-release/) (3 november 2026) uppdaterade **medföljande Brotli i berörda PyArrow-wheelpaket och R-binärer**. Det är inte en universell rättningsgaranti för alla Arrow C++-distributioner, särskilt sådana som länkar mot **system-Brotli**. Kontrollera paketformat och byggsätt.

## Python `Brotli` och `brotlicffi`

```text
Python-program eller plugin → avkodningsfunktion → native Brotli
```

**Berörda paket enligt [GHSA](https://github.com/advisories/GHSA-9v8h-qqgm-4m35):** PyPI-paketet `Brotli` rättat i **1.2.1**, `brotlicffi` rättat i **1.2.1.0**. En föråldrad Pythonmiljö behöver dock faktiskt användas för att behandla angriparpåverkad Brotli-data. Program som QGIS och ArcGIS Pro kan använda en separat Python-/conda-miljö; ett `pip`-resultat från system-Python gäller inte automatiskt den integrerade applikationen.

## Node.js

```text
HTTP-klient, zlib-API eller Express-body-parser
  → Node.js:s Brotli-avkodning
```

**Patch:** första rättade Node.js-versioner i stödda grenar **22.23.4**, **24.22.1**, **26.12.1**. Version **24.22.2** rättar dessutom en separat regression i 24.22.1. Äldre osupporterade grenar får inte automatiskt dessa uppströmsrättningar. Kontrollera distributionspaketens undantag för systemlänkad Brotli. [Node.js](https://nodejs.org/en/blog/vulnerability/october-2026-security-releases) · [Express](https://expressjs.com/en/blog/2026-10-31-security-releases/#a-note-on-brotli-decompression-cve-2026-111497).

**Viktig distinktion:** Utgående HTTP-svar som avkodas av klienten och inkommande begärandekroppar som avkodas av serverns parser är **två olika attackvägar**. Att servern bara *komprimerar utgående data* bevisar inte att den använder sårbar avkodning.

## .NET

```text
Aktiverad ASP.NET Core request decompression
  eller HttpClient med AutomaticDecompression=Brotli
  eller explicit BrotliStream/BrotliDecoder
  → .NET:s medföljande Brotli
```

Microsoft anger att ASP.NET Core och `HttpClient` **inte** aktiverar de relevanta automatiska avkodningsvägarna i standardkonfigurationen. Applikationen är därför inte automatiskt praktiskt exponerad bara för att .NET finns installerat. [Microsofts CVE-advisory](https://github.com/dotnet/announcements/issues/494).

**Patch:** .NET **8.0.33**, **9.0.22**, **10.0.14** (och specificerad .NET 11 RC3). **Self-contained** och AOT-byggen kräver särskild ombyggnad och distribution enligt Microsofts anvisningar. [.NET-advisory](https://github.com/dotnet/announcements/issues/494).

## Brotli4j / Java JNI

```text
Java-applikation med Brotli4j
  → native-* bibliotek
  → medföljande Google Brotli-avkodare
```

**Patch:** `brotli4j` och motsvarande `native-*` **1.24.0**; tidigare versioner till och med **1.23.0** anges som berörda. Operativsystemets Brotli-uppdatering rättar inte den separata native-kopian. Även Java-applikationen måste faktiskt exponera avkodningsfunktionen för angriparpåverkade data. [Brotli4j advisory](https://github.com/hyperxpro/Brotli4j/security/advisories/GHSA-x524-jrqf-vwq6).

## DuckDB och andra inbyggda kopior

En del program levererar egen Brotli-källkod inuti ett annat bibliotek, till exempel DuckDB. En [DuckDB-ändring för CVE:n](https://github.com/duckdb/duckdb/issues/27660) gäller den inbyggda kopian. **Fix i källkoden, fix i en DuckDB-release och fix i det GIS-program som levererar DuckDB är tre separata steg.** Den som kör FME måste därför kontrollera Safes aktuella produkter och byggnummer, inte bara ett DuckDB-commit.

## Praktisk slutsats

Skriv aldrig en åtgärd som enbart säger “uppdatera Brotli” utan att identifiera **vilken kopia och vilket paket som innehåller den**. Avkodningens dataflöde avgör angreppsvägen; distributionskedjan avgör om patchen faktiskt träffar rätt kod.

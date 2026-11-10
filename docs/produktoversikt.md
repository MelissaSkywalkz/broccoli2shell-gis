# GIS-produkter: beroenden och avkodningsvägar

**Källkontroll: 9 november 2026.** [README](../README.md) innehåller versions- och rättningsöversikten. Tabellen jämför relevanta beroenden och visar vad som är belagt och vad som återstår att kontrollera. Den styrker inte exploaterbarhet i enskilda GIS-produkter.

## Bevisstatus för olika avkodningsvägar

| Teknisk väg | Vad som är belagt och inte belagt | Vad behöver fortfarande kontrolleras? |
| --- | --- | --- |
| **[QGIS: GeoParquet → GDAL/Arrow → Brotli](qgis.md)** | **Dokumenterad avkodningsväg** i specificerade byggen. Ingen QGIS-specifik CVE-utlösning eller RCE verifierad i offentligt underlag. | Vilken avkodare som är installerad och används, samt att relevant patch nått den. |
| **[QGIS: WMS/WFS via Qt/libcurl och JPEG XL](qgis.md)** | Möjliga beroendevägar. Ingen CVE-utlösning har visats i QGIS via dessa vägar. | Om just den använda klient-/bildavkodaren når berörd Brotli-kod. |
| **[FME: HTTPCaller/OpenAPICaller, Parquet, DuckDB](fme.md)** | Safe bekräftar påverkan i sitt CVE-specifika besked, kontrollerat 9 november via inloggad webbläsare. Rättade FME-byggen för 2026.1 till 2026.3 anges fortfarande som under arbete; något rättat produktbyggnummer är inte bekräftat här. | Vilken FME-komponent avkodar och vilken rättning bekräftar Safe för aktuell version och plattform? DuckDB kräver en separat rättning. |
| **[ArcGIS Server/Node.js och ArcGIS Pro/.NET](arcgis.md)** | **Leverantörsbekräftad produktpåverkan** från Esri; Server-patch angiven. Ingen ArcGIS-specifik exploaterbar ingång eller RCE är visad här. | Vilken produkt, version och runtime används och är leverantörens rättning installerad? |
| **[Node.js/Brotli i Origo Server-backend](origo-server.md)** · **[Hajk-backend](hajk.md)** | **Möjliga Express-vägar till berörd Node.js-avkodare** stöds av kod och dokumentation. CVE-felet ligger i Brotli/Node-beroendet, inte i någon här påvisad egen brist i Hajks eller Origos applikationskod. Ingen produktspecifik CVE-utlösning har verifierats. | Vilken Node-runtime kör den aktuella backendmiljön? Accepteras relevant `br`-indata? Statiska Hajk-varianter saknar denna Node-backend. |
| **[GeoServer/GeoWebCache med Jetty/Brotli4j eller native GDAL/JNI](geoserver.md)** | **Möjliga, villkorade vägar**. En installerad GDAL-extension kan ha cURL-stöd, men detta verifierar inte Brotli eller en CVE-ingång; ingen generell bekräftad produktpåverkan. | Finns en aktiv berörd avkodare i servletcontainern, GDAL-/cURL-/bildformatskedjan eller annat tillägg? Används den mot angriparpåverkade data? |
| **Andra program: Frends, Kettle, myCarta, Gaida, EntryScape** | **Inte produktspecifikt fastställt här.** En nätverksfunktion eller ett programmeringsspråk räcker inte för att påstå påverkan. | Följ data till faktisk process, avkodare och rättning; begär leverantörsbesked vid okända beroenden. [Verksamhetsflöden](verksamhetsfloden.md) |

**QGIS med DuckDB:** En QGIS-distribution kan innehålla DuckDB som separat Pythonmodul med en egen Brotli-kopia. En uppdaterad system-DLL rättar inte automatiskt den kopian. Fyndet gäller beroenden och paketering. En CVE-utlösning via DuckDB i QGIS är inte verifierad här. [QGIS och DuckDB](qgis.md#duckdb-i-qgis-pythonmiljö-en-separat-bibliotekskopia) · [DuckDB:s rättningsdiskussion](https://github.com/duckdb/duckdb/issues/27660).

## Exempel utanför GIS och deras betydelse för GIS

**Kontrollera om avkodaren används.** [Unity](https://activation.unity3d.com/security/unity-brotli-decoder-vulnerability) beskriver hur den berörda avkodaren kan ingå i en spelruntime utan att vanliga nätverks- och resursfunktioner använder den. [Siemens](https://productcert.siemens.com/productcert/html/ssa-443410.html) skiljer mellan Industrial Edge-installationer där biblioteket finns men den berörda funktionen inte exponeras och sådana där installerade applikationer kräver åtgärd. I en GIS-miljö behöver ett biblioteksfynd därför följas till den avkodning som faktiskt används. Möjliga ingångar ska fortfarande undersökas.

**Rättningen behöver nå den använda kopian.** [Microsofts besked](https://www.microsoft.com/en-us/msrc/blog/2026/10/patch-now-php2shell-brotli-emudhra-guidance-for-customers) visar varför en Windows-uppdatering inte rättar varje separat installerad runtime eller applikation. [AWS](https://aws.amazon.com/security/security-bulletins/2026-155-aws/) skiljer mellan hanterad molntjänst, tillhandahållen runtime och kundens egna miljöer. Samma uppdelning gäller för GIS-system med systempaket, inbyggda bibliotek, containrar och lokalt installerade komponenter. [Apache Arrow 26.0.1](https://arrow.apache.org/blog/2026/11/03/26.0.1-release/) är ett direkt relevant GIS-beroendeexempel: rättningen i specificerade PyArrow-paket säger inte att varje systemlänkat Arrow C++-bygge är rättat.

Exemplen styrker inte exploaterbarhet i enskilda GIS-installationer. Se även [källregistret](kallor.md#exempel-utanför-gis).

När bara ett bibliotek eller en möjlig avkodningsväg har identifierats räcker det inte att ange statusen som ”sårbar” eller ”inte sårbar”. ”Leverantörsbekräftad”, ”beroende/avkodningsväg belagt”, ”möjlig” och ”ej verifierad” har olika innebörd. Avsaknad av offentlig CVE-reproduktion utesluter inte en framtida upptäckt, och en identifierad beroendeväg styrker inte automatiskt exploatering.

**Uppdatering:** Följ leverantörens råd utan att invänta egen CVE-reproduktion. Daterade uppgifter om publicerade produkt- och biblioteksrättningar finns i [README:s rättningsöversikt](../README.md#4-vilka-rättningar-och-besked-fanns-den-10-november-2026) och [tekniska patchkedjan](patchkedjan.md). För hur en okänd produkt bedöms: [supply chain](supply-chain.md) och [inventeringsmetod](ny-programvara.md).

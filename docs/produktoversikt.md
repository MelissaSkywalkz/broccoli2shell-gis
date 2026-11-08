# Produktfördjupning – jämför tekniska avkodningsvägar

**Källkontroll: 8 november 2026.** [README](../README.md) ger **hela sammanställningen** av vad som är känt, osäkert och vad som bör göras, inklusive versions- och rättningsöversikt. Här jämförs **hur det underliggande Brotli-felet kan beröra produkter genom standardbibliotek, runtimemiljöer och inbyggda kopior**. Att en leverantör bekräftar påverkan är inte en anklagelse om att leverantören orsakat grundfelet. Tabellen är **inte** en lista över bekräftat exploaterbara GIS-produkter.

## Bevisstatus för olika avkodningsvägar

| Teknisk väg | Vad som är belagt – och inte | Vad behöver fortfarande kontrolleras? |
| --- | --- | --- |
| **[QGIS: GeoParquet → GDAL/Arrow → Brotli](qgis.md)** | **Dokumenterad avkodningsväg** i specificerade byggen. Ingen QGIS-specifik CVE-utlösning eller RCE verifierad i offentligt underlag. | Vilken avkodare som är installerad och används, samt att relevant patch nått den. |
| **[QGIS: WMS/WFS via Qt/libcurl och JPEG XL](qgis.md)** | **Möjliga ytterligare beroendevägar**, inte produktspecifik CVE-trigger-beläggning. | Om just den använda klient-/bildavkodaren når berörd Brotli-kod. |
| **[FME: HTTPCaller/OpenAPICaller, Parquet, DuckDB](fme.md)** | **Tidigare avläst leverantörsrapporterad påverkan** (Safe). Exakt CVE-rättat FME-produktbygge **ej verifierad** i öppet underlag. | Vilken FME-komponent avkodar och vilken rättning bekräftar Safe för version/plattform? |
| **[ArcGIS Server/Node.js och ArcGIS Pro/.NET](arcgis.md)** | **Leverantörsbekräftad produktpåverkan** från Esri; Server-patch angiven. En allmänt användbar exploiterbar endpoint eller RCE i ArcGIS är inte visad. | Vilken produkt, version och runtime används och är leverantörens rättning installerad? |
| **[Node.js/Brotli i Origo Server-backend](origo-server.md)** · **[Hajk-backend](hajk.md)** | **Möjliga Express-vägar till berörd Node.js-avkodare** stöds av kod och dokumentation. CVE-felet ligger i Brotli/Node-beroendet, inte i någon här påvisad egen brist i Hajks eller Origos applikationskod. Ingen produktspecifik CVE-utlösning har verifierats. | Vilken Node-runtime kör den aktuella backendmiljön? Accepteras relevant `br`-indata? Statiska Hajk-varianter saknar denna Node-backend. |
| **[GeoServer/GeoWebCache med eventuell Jetty/Brotli4j-modul](geoserver.md)** | **Möjlig, villkorad väg**. Ingen generell bekräftad påverkan på GeoServers standardinstallation. | Finns en aktiv berörd native-avkodare/tillägg i den faktiska servern? |
| **Andra program: Frends, Kettle, myCarta, Gaida, EntryScape** | **Inte produktspecifikt fastställt här.** En nätverksfunktion eller ett programmeringsspråk räcker inte för att påstå påverkan. | Följ data till faktisk process, avkodare och rättning; begär leverantörsbesked vid okända beroenden. [Verksamhetsflöden](verksamhetsfloden.md) |

**Detta är inte GIS-specifikt:** många andra Node.js/Express-applikationer kan behöva motsvarande kontroll och uppdatering. Produktnamnen här visar möjliga *användningsmiljöer* för ett berört beroende, inte vilka projekt som orsakat CVE:n.

**Bevisstatus ska inte förkortas till ”sårbar/inte sårbar”** när bara ett bibliotek eller en möjlig väg har identifierats. ”Leverantörsbekräftad”, ”beroende/avkodningsväg belagt”, ”möjlig” och ”ej verifierad” har olika innebörd. Avsaknad av offentlig CVE-reproduktion utesluter inte en framtida upptäckt, och en identifierad beroendeväg styrker inte automatiskt exploatering. Även en installatör eller AI-tjänst som bara läser denna tabell behöver bevara den skillnaden.

**Patchråd:** Följ leverantörens relevanta uppdatering utan att invänta egen CVE-reproduktion. För en komplett, daterad sammanställning av publicerade produkt- och biblioteksrättningar: [README:s rättningsöversikt](../README.md#4-vilka-rättningar-och-besked-fanns-den-8-november-2026) och [tekniska patchkedjan](patchkedjan.md). För hur en okänd produkt bedöms: [supply chain](supply-chain.md) och [inventeringsmetod](ny-programvara.md).

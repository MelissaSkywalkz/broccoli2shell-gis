# GeoServer och GeoWebCache: möjliga Brotli-beroenden

**Källkontroll: 8 november 2026; uppgraderingskomplettering 9 november 2026.**

Felet finns i den underliggande Brotli-avkodaren, som kan ingå i till exempel ett separat Brotli4j/native-beroende. Det granskade underlaget visar varken ett separat fel i GeoServers kod eller generell påverkan på standardinstallationen.

**Känt:** I det granskade underlaget finns inget generellt leverantörsbesked som bekräftar påverkan på GeoServer eller GeoWebCache. Jetty har en separat Brotli-modul och Brotli4j har en dokumenterad rättning på biblioteksnivå, men detta bevisar inte att GeoServers standardinstallation använder berörd avkodning.

**Oklart:** En viss installation kan ha egna servletcontainerinställningar, tillägg eller native-bibliotek. Påverkan på en enskild installation är därför okänd tills avkodare, version och datavägar har kontrollerats.

**Kontroll:** Inventera aktiva moduler och bibliotek i servermiljön. Om en berörd Brotli4j/native-kopia används, installera relevant rättning enligt bibliotekets eller leverantörens besked utan att invänta CVE-reproduktion. Kontrollera dessutom driftversion och tjänsteomstart. [Daterad rättningsstatus](patchkedjan.md).

**Källor och fördjupning:** [README](../README.md) · [Bibliotek](bibliotek.md) · [Patchkedja](patchkedjan.md) · [WFS-T som separat fråga](verksamhetsfloden.md#3-origo-redigering--geoserver--mssql)

## Möjliga men obekräftade vägar

### Servletcontainer och inkommande HTTP

```text
Komprimerad extern HTTP-begäran
  → Jetty/Tomcat/proxy eller annan servletcontainer
  → endast om Brotli-avkodningsmodul faktiskt är aktiverad
  → exempelvis Brotli4j/native → berörd Brotli-kopia
```

Jettys `compression-brotli` är en separat modul som måste vara aktiverad för att användas. GeoServer kan också installeras som WAR i en kundvald servletcontainer. Att [GeoServers byggfil för 3.0.2](https://github.com/geoserver/geoserver/blob/3.0.2/src/pom.xml) anger Jetty 12.1.14 visar en version i det projektets byggkedja, inte att slutlig distribution har en aktiv Brotli-avkodare. Den [granskade `start.ini`](https://github.com/geoserver/geoserver/blob/3.0.2/src/release/jetty/start.ini) aktiverar inte Brotli-modulen. Kontrollera ändå den **faktiskt distribuerade** WAR/BIN/containern. [Jetty-moduler](https://jetty.org/docs/jetty/12.1/operations-guide/modules/standard.html).

### Utgående HTTP och källor till kartdata

GeoServer och GeoWebCache kan ingå i kedjor som hämtar externa kartdata eller tjänstesvar. Det innebär inte att de automatiskt avkodar Brotli via Googles C-bibliotek. För en sådan påverkansbedömning krävs identifierad HTTP-klient, förhandlad `Content-Encoding: br`, faktisk parser och den laddade avkodarens implementation. Samma gäller eventuella GDAL-/native-tillägg.

### GDAL som separat rastertillägg

GeoServers [Image/I/O-Ext GDAL Coverage Extension](https://docs.geoserver.org/stable/en/user/data/raster/gdal/) kan använda en **separat installerad native GDAL-miljö via Java/JNI**, exempelvis för vissa rasterformat. Det är relevant eftersom dess cURL- och andra beroenden kan ha **en annan rättningskedja än GeoServers Java-kod och servletcontainer**. Det är inte belagt här att standardinstallationen av GeoServer använder en sårbar Brotli-avkodare eller att alla rasterlager går genom GDAL-tillägget.

**Kontroll för driftansvariga:** ta reda på om tillägget är installerat och används för aktuella rasterlager. Granska dess GDAL-bygge, beroenden och eventuella laddade bibliotek med godkända administrationsverktyg. `CURL_ENABLED=YES` visar cURL-stöd, **inte** Brotli-avkodning, CVE-utlösning eller exploaterbarhet. Om en berörd Brotli-kopia kan beläggas, uppdatera den enligt **den komponentleverantör som ansvarar för native-paketet**, och verifiera att tillägg och ordinarie rastertjänster fungerar efteråt. Ersätt inte enskilda DLL-filer manuellt. [GDAL:s byggberoenden](https://gdal.org/en/stable/development/building_from_source.html) · [Patchkedjan](patchkedjan.md).

### Tillägg och separat installerade bibliotek

Ett tillägg kan inkludera Brotli4j eller andra native-bibliotek, även om grunddistributionen inte gör det. Inventera `WEB-INF/lib`, servletcontainerns gemensamma bibliotek, pluginmoduler, native-runtime och aktuella containerlager. En Maven- eller testdependency är inte bevis för att avkodaren används i driftsatt server.

## Är Brotli4j sårbart och finns patch?

**Ja, på biblioteksnivå.** [Brotli4j:s säkerhetsbesked](https://github.com/hyperxpro/Brotli4j/security/advisories/GHSA-x524-jrqf-vwq6) anger att versioner till och med `1.23.0` innehåller berörd native Brotli och att `1.24.0` är rättad. Även tillhörande `native-*`-paket behöver uppdateras. **Uppdatering av operativsystemets libbrotli rättar inte en gammal medföljande Brotli4j-native-kopia**.

Detta är **inte** ett påstående att GeoServer 3.0.2 innehåller eller aktiverar just Brotli4j i sin standarddistribution. Det är ett undersökningsvillkor för miljöer där biblioteket faktiskt används.

## Vid uppgradering: håll Java och native-bibliotek isär

Kontrollera att GeoServer, servletcontainern, Java och eventuella **GDAL/JNI-/rastertillägg** har kompatibla, underhållna versioner. En Java- eller GeoServer-uppgradering rättar inte automatiskt separat installerad native GDAL eller dess beroenden. Dokumentera **vilka komponenter som faktiskt körs**, välj respektive leverantörs uppdatering och verifiera relevanta lager och tjänster efter driftbytet. Att ett ECW-raster fungerar visar inte att det finns någon Brotli-ingång; även fungerande tjänster kräver separat bedömning av bibliotekets rättningsstatus. [GeoServers GDAL-stöd](https://docs.geoserver.org/stable/en/user/data/raster/gdal/) · [Java-krav](https://docs.geoserver.org/main/en/user/production/java/).

## Bedömning och patchstatus

| Omständighet | Bedömning |
| --- | --- |
| GeoServer/GeoWebCache utan berörd Brotli-avkodare i faktisk drift | **Ingen visad CVE-väg via Brotli**; frånvaro måste fastställas, inte antas |
| Aktiverad Jetty Brotli-avkodning med berörd native-kopia | **Möjlig CVE-väg** om angriparpåverkad `br`-indata når avkodaren |
| Installerat Brotli4j <= 1.23.0 och faktiskt avkodar obetrodda data | **Känd berörd komponent** och en relevant dataväg; produktspecifik exploatering fortfarande en separat fråga |
| Brotli4j >= 1.24.0 i körande Java-miljö | **Bibliotekspatch tillgänglig/använd om verifierad**; kontrollera även andra Brotli-kopior |

**Öppna frågor:** Finns Brotli-modul i den konkreta GeoServer-/GeoWebCache-distributionen? Är den aktiverad för inkommande eller utgående trafik? Vilket native-bibliotek används? Finns ytterligare inbäddade beroenden? Utan svar går det inte att utfärda ett generellt “sårbar” eller “inte sårbar” för produkten.

**Slutbedömning av denna riskkedja (exempel, ingen verifierad lokal installation):**

| Fält | Bedömning |
| --- | --- |
| **Påverkan** | **Ingen generell GeoServer-påverkan belagd.** Berörd Brotli4j-kopia kan dock förekomma i tillägg/aktiverad modul. |
| **Exponering** | Bara om en berörd avkodare faktiskt används mot angriparpåverkade `br`-data i driftsatt server. |
| **Konsekvens** | Möjligt tjänsteavbrott; åtkomst till datastore endast vid separat lyckad vidare exploatering och faktiska rättigheter. |
| **Prioritet** | **Inventera moduler, JAR/native och aktiv HTTP-väg först**; patcha vid påvisat berört beroende. |
| **Rättning** | Brotli4j **1.24.0** med matchande native-paket där det används; annars rättad system-/produktkopia. |
| **Kvarvarande osäkerhet** | Finns överhuvudtaget berörd avkodare och nåbar funktion i driftsättningen? |

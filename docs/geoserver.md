# GeoServer och GeoWebCache – villkorad påverkan, inte en generell produkt-CVE

**Källkontroll: 8 november 2026**  

**Var finns CVE-felet?** I den **underliggande Brotli-avkodaren**, som exempelvis kan följa med ett separat Brotli4j/native-beroende. Det finns **inget belägg här för att GeoServers egen kod introducerat detta fel**, eller för en generell berörd standardinstallation.

**Vad vet vi?** Det granskade underlaget visar **ingen generell leverantörsbekräftad CVE-påverkan** i GeoServer eller GeoWebCache. Jetty har en separat Brotli-modul och Brotli4j har en dokumenterad rättning på biblioteksnivå, men detta bevisar inte att GeoServers standardinstallation använder berörd avkodning.

**Vad vet vi inte?** En viss installation kan ha egna servletcontainerinställningar, tillägg eller native-bibliotek. Utan kontroll av faktisk avkodare, version och indata går det inte att friskriva eller kalla just den installationen exploaterbar.

**Vad bör göras ändå?** Inventera aktiva moduler och bibliotek i den egna servermiljön. Om en berörd Brotli4j/native-kopia används, installera relevant rättning enligt bibliotekets eller leverantörens besked utan att invänta CVE-reproduktion. Kontrollera dessutom driftversion och tjänsteomstart. [Daterad rättningsstatus](patchkedjan.md).

**Tekniska belägg och vidare läsning:** [README](../README.md) · [Bibliotek](bibliotek.md) · [Patchkedja](patchkedjan.md) · [WFS-T som separat fråga](verksamhetsfloden.md#3-origo-redigering--geoserver--mssql)

## Möjliga men obekräftade vägar

### Servletcontainer och inkommande HTTP

```text
Komprimerad extern HTTP-begäran
  → Jetty/Tomcat/proxy eller annan servletcontainer
  → endast om Brotli-avkodningsmodul faktiskt är aktiverad
  → exempelvis Brotli4j/native → berörd Brotli-kopia
```

Jetty har en separat `compression-brotli`-modul. Den är **inte per automatik aktiv i varje GeoServer-installation**. GeoServer kan också installeras som WAR i en kundvald servletcontainer. Att [GeoServers byggfil för 3.0.2](https://github.com/geoserver/geoserver/blob/3.0.2/src/pom.xml) anger Jetty 12.1.14 visar en version i det projektets byggkedja – inte att slutlig distribution har en aktiv Brotli-avkodare. Den [granskade `start.ini`](https://github.com/geoserver/geoserver/blob/3.0.2/src/release/jetty/start.ini) aktiverar inte Brotli-modulen. Kontrollera ändå den **faktiskt distribuerade** WAR/BIN/containern. [Jetty-moduler](https://jetty.org/docs/jetty/12.1/operations-guide/modules/standard.html).

### Utgående HTTP och källor till kartdata

GeoServer och GeoWebCache kan ingå i kedjor som hämtar externa kartdata eller tjänstesvar. Det innebär inte att de automatiskt avkodar Brotli via Googles C-bibliotek. För en sådan påverkansbedömning krävs identifierad HTTP-klient, förhandlad `Content-Encoding: br`, faktisk parser och den laddade avkodarens implementation. Samma gäller eventuella GDAL-/native-tillägg.

### Tillägg och separat installerade bibliotek

Ett tillägg kan inkludera Brotli4j eller andra native-bibliotek, även om grunddistributionen inte gör det. Inventera `WEB-INF/lib`, servletcontainerns gemensamma bibliotek, pluginmoduler, native-runtime och aktuella containerlager. En Maven- eller testdependency är inte bevis för att avkodaren används i driftsatt server.

## Är Brotli4j sårbart och finns patch?

**Ja, på biblioteksnivå.** [Brotli4j:s säkerhetsbesked](https://github.com/hyperxpro/Brotli4j/security/advisories/GHSA-x524-jrqf-vwq6) anger att versioner till och med `1.23.0` innehåller berörd native Brotli och att `1.24.0` är rättad. Även tillhörande `native-*`-paket behöver uppdateras. **Uppdatering av operativsystemets libbrotli rättar inte en gammal medföljande Brotli4j-native-kopia**.

Detta är **inte** ett påstående att GeoServer 3.0.2 innehåller eller aktiverar just Brotli4j i sin standarddistribution. Det är ett undersökningsvillkor för miljöer där biblioteket faktiskt används.

## Bedömning och patchstatus

| Omständighet | Bedömning |
| --- | --- |
| GeoServer/GeoWebCache utan berörd Brotli-avkodare i faktisk drift | **Ingen visad CVE-väg via Brotli**; frånvaro måste fastställas, inte antas |
| Aktiverad Jetty Brotli-avkodning med berörd native-kopia | **Möjlig CVE-väg** om angriparpåverkad `br`-indata når avkodaren |
| Installerat Brotli4j <= 1.23.0 och faktiskt avkodar obetrodda data | **Känd berörd komponent** och en relevant dataväg; produktspecifik exploatering fortfarande en separat fråga |
| Brotli4j >= 1.24.0 i körande Java-miljö | **Bibliotekspatch tillgänglig/använd om verifierad**; kontrollera även andra Brotli-kopior |

**Öppna frågor:** Finns Brotli-modul i den konkreta GeoServer-/GeoWebCache-distributionen? Är den aktiverad för inkommande eller utgående trafik? Vilket native-bibliotek används? Finns ytterligare inbäddade beroenden? Utan svar går det inte att utfärda ett generellt “sårbar” eller “inte sårbar” för produkten.

**Slutbedömning av denna riskkedja – exempel, inte en verifierad lokal installation:**

| Fält | Bedömning |
| --- | --- |
| **Påverkan** | **Ingen generell GeoServer-påverkan belagd.** Berörd Brotli4j-kopia kan dock förekomma i tillägg/aktiverad modul. |
| **Exponering** | Bara om en berörd avkodare faktiskt används mot angriparpåverkade `br`-data i driftsatt server. |
| **Konsekvens** | Möjligt tjänsteavbrott; åtkomst till datastore endast vid separat lyckad vidare exploatering och faktiska rättigheter. |
| **Prioritet** | **Inventera moduler, JAR/native och aktiv HTTP-väg först**; patcha vid påvisat berört beroende. |
| **Rättning** | Brotli4j **1.24.0** med matchande native-paket där det används; annars rättad system-/produktkopia. |
| **Kvarvarande osäkerhet** | Finns överhuvudtaget berörd avkodare och nåbar funktion i driftsättningen? |

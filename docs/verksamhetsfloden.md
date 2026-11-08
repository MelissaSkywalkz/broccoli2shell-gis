# Riskkedjor i vanliga GIS- och integrationsflöden

**CVE-2026-111497 · Källkontroll 8 november 2026.** [CVE-mekanismen](cve-mekanism.md) · [HTTP-avkodning](http-angreppsvagar.md) · [Riskkedjan](riskkedja.md) · [Supply chain](supply-chain.md) · [Patchkedjan](patchkedjan.md)

För det generella fallet **”vi hämtar X-data från Y via FME, Frends, ArcGIS, schemalagd Python eller Kettle”**, börja i [dataflöde → avkodare → rätt patch](dataflode-till-patch.md).

Den här sidan börjar i **verksamhetens dataflöde**, inte i en produktvarning. Den ska hjälpa en systemförvaltare att upptäcka en relevant avkodningsväg i integrationsplattformar, GIS-klienter, kartredigering, fastighetsinformationssystem och metadata-publicering. **Ett arbetsflöde eller en namngiven produkt är inte orsaken till den här CVE:n; felet ligger i en berörd Brotli-avkodare som eventuellt används i flödet.** **Ingen av de fem konkreta miljöerna nedan har testats för CVE-utlösning här.** Dokumenterade produktfunktioner är verifierade mot källor; Brotli-vägar som inte har bekräftats i respektive installation anges som **villkorade**.

**Två olika kedjor behöver alltid följas:**

1. **Indatakedjan:** Vem kan påverka komprimerade byte → vilken HTTP-klient, parser eller filimport tar emot dem → vilken *faktiskt körande* Brotli-avkodare används? Ett JSON-, XML- eller rasterformat är inte i sig ett CVE-bevis.
2. **Risk- och rättningskedjan:** Om CVE:n faktiskt utlöses, vilken process påverkas → vad får den läsa eller ändra → vad kan gå fel → var finns rättningen och vilka kontroller begränsar följderna? **Krasch, läckage och RCE är olika effekter.** En sårbar komponent innebär inte att någon redan har åtkomst till processen.

| Arbetsflöde | Rimligt att undersöka | Det som **inte** följer automatiskt |
| --- | --- | --- |
| [Frends-integrationer](#1-frends--http-och-integrationer) | HTTP-svar, filhämtning, HTTP-trigger, .NET-agent, rättigheter och hemligheter | Att .NET eller en HTTP-task alltid avkodar Brotli |
| [ArcGIS Pro + Lantmäteriets STAC](#2-arcgis-pro--lantmäteriets-stac) | Separata anrop för STAC-metadata och COG/rasterfil, HTTP-stack, GDAL och eventuellt filformat | Att STAC i sig använder Brotli eller att COG/LAZ är Brotli-format |
| [Origo-redigering → GeoServer → MSSQL](#3-origo-redigering--geoserver--mssql) | Klientens svar, serverns eventuella HTTP-avkodning och tillägg; databasbehörighet som möjlig följd | Att WFS-T, SQL Server eller geometriändring i sig utlöser Brotli-CVE:n |
| [Gaida / SBI (tidigare FB)](#4-gaida--sbi-före-detta-fb) | Lantmäteriets direkt-API:er, schemalagda importer och faktisk klient-/importparser | Att en lokal FB-databas är sårbar bara för att den lagrar importerad information |
| [Publicera metadata via EntryScape-API](#5-publicera-öppna-data-via-entryscape-api) | HTTP-svaret på POST/PUT, vilken klient som tar emot det; separat serverparser vid komprimerad inkommande kropp | Att `POST`/`PUT`, DCAT eller EntryScape i sig betyder Brotli-avkodning |

## 1. Frends – HTTP och integrationer

**Belagd produktfunktion:** Frends har `HTTP.Request`, `DownloadFile` och andra HTTP-tasks; integrationsprocesser körs i Frends Agents på .NET och kan startas med en HTTP-trigger. Ett HTTP-anrop kan hämta data från en extern källa och skriva dem till ett annat system. [Frends HTTP-integrationer](https://tasks.frends.com/integrations/HTTP) · [Frends teknisk plattform](https://docs.frends.com/hybrid-integration-architecture/technologies) · [HTTP-trigger](https://docs.frends.com/reference/triggers/http-trigger).

```text
Leverantör/API → HTTP-svar (eventuellt Content-Encoding: br)
    → Frends Agent / HTTP-task → faktiskt vald .NET-/native-avkodare?
    → transformerat innehåll → annan databas, filyta eller verksamhetssystem

Alternativt: extern POST → HTTP-trigger → begärandekropp
    → Brotli-avkodning endast OM konfigurerad parser/middleware gör det
```

- **Angreppsväg:** skilj **mottaget HTTP-svar**, **nedladdad fil som senare tolkas** och **inkommande HTTP-trigger**. Kontrollera vilken Frends-task/version och agent som används, HTTP-huvuden och faktisk avkodning. `HttpClientHandler.AutomaticDecompression` har konfigurerbar Brotli-hantering; dess existens bevisar inte att Frends har aktiverat den. [Microsoft](https://learn.microsoft.com/en-us/dotnet/api/system.net.http.httpclienthandler.automaticdecompression).
- **Påverkad process:** den specifika Frends Agent-/taskprocess som läser indatan; inte automatiskt alla integrationer i miljön. Ingen Frends-specifik CVE-påverkan eller rättad Frends-version har bekräftats här.
- **Åtkomliga resurser:** utred den agentens körkonto, anslutningshemligheter, nätverksvägar, databasrättigheter och andra integrationer. En personuppgiftsintegration kräver separat kontroll av behörighet och processisolering.
- **Möjlig skada:** en krasch kan stoppa synkronisering; eventuell lyckad exploatering skulle kunna nå de resurser processen faktiskt har rätt till. **Inget läckage eller RCE är visat i Frends** av detta underlag.
- **Skydd och patch:** kontrollera aktuell Frends- och .NET-runtime, HTTP-konfiguration, rättad Brotli-kopia, separerade agents/konton, minsta privilegium, TLS, övervakning och eventuell separat filparser. Utpeka inte en Frends-patch utan Frends-specifikt besked.


**Samlad bedömning – Frends (hypotetiskt flöde):** Ingen Frends-specifik CVE-påverkan har fastställts. Nästa kontroll är aktiv **HTTP-task/Agent**, om `br`-avkodning verkligen sker och vilken .NET-/native-version som används. En krasch kan stoppa ett jobb; annan skada kräver ytterligare effekt och faktiska Agent-behörigheter. Uppdatera relevant komponent enligt leverantörsbesked; en Frends-produktpatch ska inte pekas ut utan källstöd.

## 2. ArcGIS Pro – Lantmäteriets STAC

**Belagd produktfunktion:** ArcGIS Pro kan skapa `.astac`-anslutningar till STAC API. Esri Sverige beskriver hur Pro 3.5 ansluter till Lantmäteriets STAC-kataloger med **separat Cloud Storage Connection** för att läsa exempelvis markhöjdmodell eller ortofoto. STAC-API:et returnerar katalog-/objektinformation; själva rasterfilen läses via den länk som anges för objektet. [Esris instruktion](https://www.esri.se/arcgis-bloggen/stac-lantmateriet-arcgis-pro/) · [ArcGIS Pro – STAC](https://doc.esri.com/en/arcgis-pro/latest/help/data/imagery/create-a-stac-connection.html).

```text
STAC API (katalog och objektmetadata i HTTP-svar)
     → ArcGIS Pro:s faktiska HTTP-klient → Brotli-avkodning OM br används
     → tillgång till asset-länk
     → separat begäran till Cloud Storage / COG eller annan fil
     → eventuell HTTP-avkodning; därefter raster/GDAL/filformatets egen avkodning
```

- **Angreppsväg:** verifiera `Content-Encoding` på **både metadataanrop och filhämtning** samt vilken klientstack som används i varje steg. Kontrollerna är oberoende. **COG, GeoTIFF och LAZ är inte per definition Brotli-komprimerade**, och metadataformatet STAC är ingen CVE-trigger i sig.
- **Påverkad process:** ArcGIS Pro, eller separat bibliotek/process som hämtar och avkodar filen. Esri rapporterar påverkan för delar av ArcGIS Pro/.NET, men detta är **inte bevis** för CVE-utlösning genom just Lantmäteriets STAC-anslutning. [Esri säkerhetsbesked](https://www.esri.com/arcgis-blog/products/trust-arcgis/administration/brotli-vulnerabilities-broccoli2shell).
- **Åtkomliga resurser:** användarens filer och projekt, eventuella behöriga databasanslutningar och åtkomstuppgifter i den körande miljön. Kontrollera faktisk lagring och separation av autentiseringsuppgifter; anta inte att de är exponerade.
- **Möjlig skada:** avbrott i nedladdning och bearbetning; vidare informations- eller systempåverkan **endast om relevant exploatering lyckas** och rättigheter medger det.
- **Skydd och patch:** följ Esris patchbesked för aktuell ArcGIS Pro/.NET-uppsättning, håll andra berörda native-bibliotek uppdaterade, använd korrekta HTTPS-/autentiseringsinställningar, avgränsa rättigheter och kontrollera faktisk filparser. Lantmäteriets STAC-endpoint är ingen automatiskt verifierad skadlig källa.


**Samlad bedömning – ArcGIS Pro/STAC (hypotetiskt flöde):** Esri bekräftar påverkan i delar av ArcGIS, **inte** CVE-utlösning genom denna specifika STAC-anslutning. Kontrollera **både metadataanrop och separat filhämtning**, inklusive respektive avkodare och lokala behörigheter. Följ Esris rekommenderade produkt-/runtime-rättningar **utan att invänta STAC- eller exploittest**; osäkerheten gäller dataväg och installationens patchläge.

## 3. Origo-redigering → GeoServer → MSSQL

**Belagd produktfunktion:** Origo har en editor för WFS-lager. GeoServer stöder transaktionella WFS-operationer (WFS-T) och kan använda ett SQL Server-datastore via en särskild extension. [Origo Editor](https://github.com/origo-map/origo-documentation/blob/master/content/controls.md) · [GeoServer WFS-säkerhet](https://docs.geoserver.org/main/en/user/production/config/) · [GeoServer SQL Server](https://docs.geoserver.org/stable/en/user/data/database/sqlserver/).

```text
Behörig kollega ändrar objekt i Origos webbkarta
    → webbläsaren skickar WFS-T Transaction (vanligen XML/HTTP)
    → GeoServer validerar behörighet och skriver till MSSQL via SQL Server-datastore

SEPARAT CVE-fråga: Finns en inkommande eller utgående HTTP-/filväg
    i webbläsare, reverse proxy, GeoServer/JVM eller tillägg som faktiskt
    avkodar angriparpåverkad Brotli med berörd implementation?
```

- **Angreppsväg:** en vanlig WFS-T-redigering med XML och databaslagring är **inte i sig en Brotli-trigger**. `Content-Encoding: br` på ett svar till webbläsaren innebär klientavkodning; inkommande `br`-komprimerad begäran kräver en särskilt aktiv serveravkodare. Granska sådan funktion innan GeoServer anses berört. Origo Server/Node kan vara en **separat** komponent om redigeringsflödet faktiskt passerar den; det får inte antas.
- **Påverkad process:** webbläsare, relevant proxy, GeoServer/JVM eller ett tillägg – endast den som verkligen avkodar. **MSSQL lagrar data i kedjan men är inte visat som CVE-berörd Brotli-avkodare.**
- **Åtkomliga resurser:** om GeoServer-processen skulle komprometteras, kontrollera datastore-kontots läs-/skrivrättigheter till berörda tabeller, publicerade lager, konfigurationsfiler och andra anslutningar. Webbläsarens behörigheter skiljer sig från serverns.
- **Möjlig skada:** en vanlig obehörig WFS-T-transaktion kan ändra data **utan någon CVE-utlösning**. En hypotetiskt komprometterad server skulle eventuellt kunna skriva eller läsa enligt sina faktiska rättigheter; en krasch kan störa kartredigering. Detta är skilda hot.
- **Skydd och patch:** skydda `WFS.Transaction` med autentisering, tjänste-/lagerrättigheter och databasprincipen minsta privilegium; separera läs- och skrivrättigheter när det går. Kontrollera eventuella Brotli-moduler och uppdatera **den avkodande komponenten** om berörd – inte MSSQL bara för att den är sista länken. Det här är också [GeoServers officiella rekommendation](https://docs.geoserver.org/main/en/user/production/config/).


**Samlad bedömning – Origo/GeoServer/MSSQL (hypotetiskt flöde):** WFS-T och SQL-skrivning bevisar ingen Brotli-påverkan. **Säkra WFS-T-behörigheter oavsett CVE.** För Brotli-frågan behöver den faktiskt avkodande klienten, proxyn, servermodulen eller tillägget identifieras och vid behov uppdateras. MSSQL:s roll som datalager är inte i sig en Brotli-angreppsväg; databasrättigheter blir följdfråga först vid verklig serverpåverkan.

## 4. Gaida / SBI (före detta FB)

**Belagd produktfunktion:** Gaidas dokumentation skiljer mellan **SBI-databasen (FB)**, som håller fastighetsuppgifter lokalt, och **Lantmäteriets direktåtkomst-API:er**. Gaida beskriver även **schemalagda importer av fastighetsaviseringar** via FB Admin/FB-schemaläggaren. [Gaida fastighetsinformation](https://help.gaida.se/sv/for-administratoren/sbi/fastighetsinformation) · [nya nedladdningstjänster](https://help.gaida.se/sv/for-administratoren/sbi/guider/bestallning-via-geotorget).

```text
A. Direktfråga → Lantmäteriets API-svar → Gaidas faktiska HTTP-klient
      → eventuell br-avkodning → verksamhetsfunktion
B. Schemalagd avisering → nedladdat underlag → importkomponent
      → eventuell HTTP-avkodning ELLER avkodning inuti importfilen
      → uppdatering av SBI/FB-databasen
```

- **Angreppsväg:** fastställ **vilken av A/B** och vilken applikation/tjänst som gör hämtningen. `gzip`, ZIP, andra interna filkomprimeringar och Brotli är inte utbytbara. Kontrollera både svarshuvuden och importbibliotek. Här finns **inget bevis** för berörd Brotli-implementation i Gaida/FB.
- **Påverkad process:** den faktiska Gaida-klienttjänsten, schemaläggaren eller importprocessen; inte själva databasen per automatik.
- **Åtkomliga resurser:** fastighetsinformation och eventuellt personuppgifter, tjänstekonton och den lokala databasen i den mån processen verkligen har åtkomst. Undvik att anta att alla moduler delar samma behörighet.
- **Möjlig skada:** avbruten avisering och föråldrat fastighetsunderlag; vid en *faktisk ytterligare kompromettering* även risk för obehörig läsning/ändring inom processens behörigheter. **Ingen Gaida-specifik RCE eller läcka har belagts.**
- **Skydd och patch:** granska källval, schemalagda jobb, installerade komponenter och underliggande .NET/Java/native-avkodare; verifiera rättning hos respektive leverantör; isolera importkonton och begränsa databasrättigheter. Utgå inte från att en rättning av en annan GIS-produkt automatiskt rättar Gaida.


**Samlad bedömning – Gaida/SBI (hypotetiskt flöde):** Ingen Gaida-specifik berörd Brotli-avkodare har verifierats. Börja med att skilja **aktiv direkt-API-fråga** från **schemalagd FB-import** och identifiera klient-/importparsern. En eventuell krasch kan avbryta avisering; åtkomst till fastighetsdata förutsätter ytterligare säkerhetseffekt och faktiska rättigheter. Begär versionsbundet besked innan produktspecifik CVE-påverkan anges.

## 5. Publicera öppna data via EntryScape API

**Belagd produktfunktion:** EntryScape/EntryStore erbjuder REST-endpoints för att skapa och uppdatera kataloger, dataset och distributioner, med autentiserad skrivning. Sveriges dataportal dokumenterar EntryScape Registry/EntryStore som registerteknik för metadata. [EntryScape API](https://docs.entryscape.com/en/api/) · [API-referens](https://entryscape.org/api/) · [Sveriges dataportal](https://docs.dataportal.se/registry/api/).

```text
Katalog-/metadatajobb (FME, Frends, eget skript eller annan klient)
    → POST/PUT med DCAT-metadata (t.ex. JSON-LD/RDF) över HTTPS
    → EntryScape/EntryStore API → svar till avsändaren

CVE-väg 1: svaret har Content-Encoding: br och avsändarens klient
      avkodar med sårbar Brotli-kopia.
CVE-väg 2: API-servern avkodar inkommande Content-Encoding: br
      med sårbar implementation (kräver faktisk konfiguration).
```

- **Angreppsväg:** ett vanligt `POST` eller `PUT` med JSON-LD/RDF innebär **inte** automatiskt Brotli, varken på sändar- eller serversidan. Kontrollera verkliga `Content-Encoding` i båda riktningarna, klientbibliotek, HTTP-proxy och API-serverns eventuella request-decompression. Ett HTTP-svar kan avkodas av samma FME-/Frends-/skriptprocess som skickar POST-anropet.
- **Påverkad process:** utgående integrationsjobb om det avkodar svaret; EntryScape-backend endast om relevant serveravkodning faktiskt är implementerad och berörd. EntryScapes faktiska Brotli-beroenden och patchstatus **är inte verifierade**.
- **Åtkomliga resurser:** avsändarens API-token, behörighet att ändra eller avpublicera metadata, eventuella andra integrationsanslutningar; på serversidan gäller de verkliga rättigheterna för den processen. Metadata-POST är normalt **inte samma sak som överföring av hela underliggande geodatamängden**.
- **Möjlig skada:** avbruten metadata-publicering, felaktig kataloginformation eller otillåtna ändringar om behörigheter missbrukas efter en verklig kompromettering; ingen av effekterna är bevisad via CVE:n i EntryScape.
- **Skydd och patch:** uppdatera faktiskt avkodande klient/runtime, skydda och begränsa skrivtoken, ge endast avsedda katalogrättigheter, använd korrekt verifierad HTTPS och logga publiceringsförändringar. Om servern tar emot komprimerad request-body, granska exakt parser och patch den; annars finns ingen visad CVE-väg genom den punkten.


**Samlad bedömning – EntryScape (hypotetiskt flöde):** `POST`/`PUT` med metadata bevisar inte Brotli. Kontrollera först om **avsändarens HTTP-klient avkodar API-svaret** eller om **servern avkodar en inkommande `br`-kropp**; det är olika processer och möjliga rättningar. Begränsa skrivtoken och uppdatera relevant avkodare om den påvisas; EntryScapes beroenden och CVE-patchstatus är inte fastställda här.

## Praktisk slutsats för nya flöden

**Håll isär tre frågor** varje gång du hittar en tjänst eller integration:

1. **Funktionen finns:** dokumentationen visar att ett program kan hämta via HTTP, ta emot POST, läsa filer eller skriva till databaser.
2. **CVE-vägen finns:** du har verifierat att just den körande processen tar angriparpåverkade **Brotli-komprimerade byte** till en **berörd avkodare**. Detta kräver mer än produktnamn, API-användning eller `Accept-Encoding: br`.
3. **Följdskadan är möjlig:** om felvillkoret *och* en faktisk säkerhetseffekt uppstår, kan den påverkade eller komprometterade processen nå särskilda resurser. Krasch är inte liktydigt med RCE eller dataläckage.

**Patchbeslut:** Installera säkerhetsuppdateringar som leverantören rekommenderar för er berörda produkt och version enligt ordinarie säkerhetsprocess – **vänta inte på egen reproduktion av CVE:n eller fullständig kartläggning av en angreppsväg**. Undersök vilken avkodarkopia och distributionsform som används (systembibliotek, inbyggd runtime, DLL, container, tillägg eller produktpatch) för att träffa rätt komponent. Nåbarhet, exponering och verksamhetskonsekvenser avgör **vad som prioriteras först och vilka tillfälliga skydd som behövs**, inte om en relevant rekommenderad rättning ska hoppas över. För program med okänd implementation: inventera eller begär versionsbunden SBOM/VEX-/säkerhetsinformation från leverantören. [Allmän metod](ny-programvara.md) · [Patchkedjan](patchkedjan.md).

**Publiceringsgräns:** arbetsflödena ovan är *generiska exempel*. Verkliga organisationsadresser, API-tokens, databasscheman, nätverksregler och faktiska exploaterbara endpoints ska inte publiceras. För den egna verksamhetens känsliga detaljer krävs en intern riskanalys.

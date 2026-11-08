# Vi hämtar X via Y – vad behöver vi kontrollera?

**CVE-2026-111497 · Praktisk inventering från dataflöde till patch. Källkontroll: 8 november 2026.**

[Översikt](../README.md) · [Riskkedjan](riskkedja.md) · [Supply chain](supply-chain.md) · [Patchkedjan](patchkedjan.md) · [Källor](kallor.md)

Du behöver **inte veta vad Brotli, DuckDB eller Arrow är i förväg**. Börja med en mening från verksamheten: *”Det här jobbet hämtar X-data från Y och skriver resultatet till Z.”* Samma frågor gäller ett manuellt klick i ArcGIS, ett FME-flöde och ett nattligt jobb i Windows Aktivitetsschemaläggaren. **En HTTP-hämtning är inte i sig en CVE-sårbarhet.** Bedömning kräver en *verklig avkodningsväg* till en berörd Brotli-implementation och angriparpåverkad data.

## 1. Från uppgiften till den faktiska avkodaren

| Fråga i rätt ordning | Vad du faktiskt undersöker | Vad svaret betyder |
| --- | --- | --- |
| **1. Vilka data hämtas och varifrån?** | API-URL/tjänst, WFS/STAC, exportfil eller delad mapp; dataägare; hur mycket källan kan påverkas av användare eller tredje part. | Bestämmer förtroendegränsen – en betrodd myndighet kan ha komprometterad datakälla utan att varje svar är farligt. |
| **2. Hur kommer byte in?** | HTTP **svar** till klienten, HTTP **begäran** till servern, nedladdad fil eller filens interna komprimering. Ett POST från dig är inte samma väg som svaret på POST:et. | Bestämmer vem som skulle avkoda och på vilken sida av anslutningen. |
| **3. Vem kör det verkliga jobbet?** | Webbläsare, FME Engine, Frends Agent, ArcGIS Pro-process, `python.exe`, Kettle/Pentaho Java-process, proxy eller server. För schemalagda jobb: **exakt körbar fil och körkonto**. | Schemaläggaren och Swagger är vanligtvis **inte** den process som avkodar svaret. |
| **4. Förekommer Brotli på just den vägen?** | På HTTP-nivå: svarets faktiska `Content-Encoding: br` och klientens avkodningsstöd. På filnivå: exempelvis en Parquet-kolumn med `BROTLI`. Kontrollera även proxy- och mellanlagers avkodning. | `Accept-Encoding: br` är ett erbjudande; `Content-Encoding: br` visar ett komprimerat svar, **inte** vilken implementation som använts eller om den är sårbar. |
| **5. Vilket bibliotek avkodar?** | HTTP-klient och version; Pythonpaket, .NET-API, Java-JAR/native-bibliotek, Node-runtime, inbyggd Arrow/DuckDB eller annan avkodare. Följ indirekta beroenden. | En installerad Brotli-kopia och en **använd** Brotli-kopia är inte samma sak. |
| **6. Är just den kopian berörd och nåbar?** | Paket-/runtimeversion, leverantörsbesked, anropad kodväg, om någon kan påverka den komprimerade datan. Dokumentera **bekräftat**, **möjligt** eller **okänt**. | Du kan motivera en patchprioritet utan att påstå att CVE-utlösning, läckage eller RCE är bevisad i programmet. |
| **7. Vad får processen åtkomst till?** | Körkonto, API-token, anslutningsuppgifter, databaser, nätverk, personuppgifter, skrivytor och jobbens tillgänglighetskrav. | En hypotetisk kompromettering bedöms efter faktiska behörigheter. Krasch, minnesröjande och RCE är **skilda scenarier**. |
| **8. Var ska rättningen sitta?** | Rättat Pythonhjul, Java-/native-JAR, Windows-/Linuxpaket, produktpatch, .NET-/Node-runtime, self-contained-bygge eller containerimage. | Verifiera **den körande** komponenten efter ombyggnad/omstart, inte bara vad som går att ladda ner. |

### Två avkodningar kan finnas i samma hämtning

```text
API svarar Content-Encoding: br
  → HTTP-klienten avkodar HTTP-svaret [kontrollpunkt A]
  → fil sparas (t.ex. Parquet/GeoParquet)
  → senare filimport avkodar interna Brotli-komprimerade delar [kontrollpunkt B]
```

Om HTTP-svaret använder `gzip` eller saknar `Content-Encoding` finns ingen påvisad Brotli-avkodning **i just det HTTP-steget**. En senare filimport kan ändå vara relevant. `Content-Type: application/json`, `Swagger/OpenAPI`, `GET`, `POST`, `STAC` eller `HTTPS` visar inte i sig om Brotli används. [HTTP-fördjupning](http-angreppsvagar.md) · [Filvägar](fil-angreppsvagar.md).

## 2. Samma data – olika program: kontrollera här

| Om hämtningen sker i … | Börja i den faktiska körningen | Vad avgör CVE-exponeringen? | Hur kan rättningen nå processen? |
| --- | --- | --- | --- |
| **FME Form/Flow** | Workspace och transformer (`HTTPCaller`, `OpenAPICaller`, filimport etc.), faktiskt exekverande FME Engine, version och anslutningar. | Vilken HTTP-/filkodväg avkodar Brotli, via vilken komponent? Kartlägg andra anslutningar som just Engine-processen kan nå. | Säkerhetsfix i **rätt FME-utgåva**, eventuellt särskild system-Brotli; verifiera mot Safe. Vi har **inte** självständigt verifierat CVE-specifika rättade FME-byggnummer i tillgängligt leverantörsunderlag. [FME](fme.md). |
| **Frends** | Process, `HTTP.Request`/nedladdning/HTTP-trigger, **vilken Agent och vilken task-version** som körs. | Är Brotli-komprimerade svar eller begäranden faktiskt konfigurerade att avkodas? Följer tasken .NET:s `HttpClient`-väg, ett eget bibliotek eller annan kod? **Okänt tills kontrollerat**. | Kontrollera Frends-leverans och .NET-runtime **om det är den berörda avkodaren**; självständig applikationsdistribution kan kräva ny agent/build. Ingen Frends-specifik rättning verifierad här. [Frends](verksamhetsfloden.md#1-frends--http-och-integrationer). |
| **ArcGIS Pro / STAC** | `.astac`-anslutning, metadataanrop, separat asset-URL/rasterhämtning, programversion. | Kontrollpunkt A för **varje** HTTP-svar; kontrollpunkt B vid eventuell fil-/rasteravkodning. Undersök **vilken** native-/HTTP-/ .NET-kodväg som faktiskt körs. | Esris **produktpatch** där relevant, separat .NET-runtime och andra berörda bibliotek. Ingen bevisad CVE-trigger enbart av STAC. [ArcGIS](arcgis.md). |
| **Python via Aktivitetsschemaläggaren** | I Windows Aktivitetsschemaläggaren: aktivitetens **Åtgärder** → `Program/script`, argument och **Starta i**, körkonto och den Pythonmiljö som faktiskt startas. | Kör HTTP-koden med `requests`/`urllib3` eller annan klient? Är `Brotli`/`brotlicffi` installerade och används de för `br`-svar? Läser skriptet senare Parquet via PyArrow eller annan parser? | Uppdatera **paketen i just den interpreter/miljön** (och eventuella native-bibliotek); starta om jobben. Att Python körs via Task Scheduler gör inte .NET-patchen till dess automatiska lösning. |
| **PowerShell 7 via Aktivitetsschemaläggaren** | Kontrollera om aktiviteten kör `pwsh.exe` (PowerShell 7) och vilka `Invoke-WebRequest`/`Invoke-RestMethod`-anrop den gör. Skilj från schemalagd `python.exe`. | Microsoft anger att dessa PowerShell 7-kommandon har **automatisk HTTP-svarsavkodning på som standard**, till skillnad från standardkonfigurerad .NET `HttpClient`. Kontrollera även nedladdade filers parser. | Microsoft anger rättade PowerShell-utgåvor **7.4.22 / 7.5.13 / 7.6.9**; uppdatera den PowerShell-installation som jobbet faktiskt använder. [MSRC](https://www.microsoft.com/en-us/msrc/blog/2026/10/patch-now-php2shell-brotli-emudhra-guidance-for-customers). |
| **Pentaho Data Integration / Kettle** | `.ktr`-transformation, `.kjb`-jobb, **REST Client/HTTP Client** eller filsteg; körning via Spoon, Pan, Kitchen eller server, samt vilken JVM och vilka JAR/native-bibliotek den använder. | PDI kan konsumera REST/HTTP, men dess stöd för Brotli och **faktiskt använd avkodare i det specifika bygget** är inte fastställt här. Följ valt HTTP-steg och dess Java-beroenden; separat filparser kan vara relevant. | Rätta **påverkad Java-/native-komponent eller Pentaho-distribution** om en berörd avkodare hittas. En Microsoft .NET-patch rättar inte en Java-process utan särskild koppling. Ingen Kettle-specifik Brotli-patch verifierad. |
| **Webbläsare / Swagger UI** | Vilken webbläsare kör *Try it out* och vilket API-anrop gjorde den verkligen? | Webbläsarens HTTP-stack avkodar svaret (inte API-specifikationen); kontrollera klientuppdatering och faktiskt svar. | Uppdatera berörd webbläsare/underliggande komponent enligt leverantör, **inte** Swagger JSON-specifikationen. |
| **Egen .NET-tjänst eller klient** | Körande app, .NET-version, exakt `HttpClientHandler`/`SocketsHttpHandler`-/ASP.NET Core-konfiguration och eventuell `BrotliStream`. | Microsoft: automatisk HTTP-svarsavkodning och request-dekomprimering **är av som standard**; enbart .NET-installation eller `HttpClient` räcker inte. | .NET 8.0.33 / 9.0.22 / 10.0.14 / 11 RC3, **för relevant distributionsform**. [Microsoft .NET](#3-microsofts-net-rättning-vad-ska-uppdateras). |

**Python, försiktig lokal kontroll:** kör kommandona med *exakt den interpreter som aktiviteten använder*, inte en godtycklig `python` från din interaktiva terminal. Detta läser bara miljöinformation och ska inte anropa externa system:

```text
"C:\\sökväg\\till\\python.exe" -c "import sys; print(sys.executable)"
"C:\\sökväg\\till\\python.exe" -m pip show Brotli brotlicffi urllib3 requests pyarrow
```

Listan visar **installerade Pythonpaket, inte garanterat den avkodare som faktiskt laddas**, och säger inget ensamt om en externt använd filparser eller server. Granska skriptets `requests`/`urllib3`-anrop och eventuella separata filimporter. Urllib3 kan stödja `br` när `brotli` eller `brotlicffi` finns installerat; avkodning beror fortfarande på det faktiska svaret och klientinställningarna. [urllib3](https://urllib3.readthedocs.io/en/latest/advanced-usage.html#brotli-encoding).

**Kettle/Java, försiktig lokal kontroll:** PDI:s REST Client och HTTP Client kan göra nätverksanrop; PDI:s 10.2-Javadoc visar Apache HttpClient-typer i HTTP-steget. Det fastställer **inte** att Pentahos aktuella HTTP-implementation har aktiverad Brotli-avkodning, vilken version som används eller att Google Brotli ingår. Identifiera **just den JVM som processen använder**, kontrollera faktiskt lastade/bundlade JAR:er och eventuella native-kopior. En allmän `java -version` från en annan terminal kan visa fel JVM. [Pentaho REST Client](https://docs.pentaho.com/pdia-data-integration/10.2-data-integration/pdi-transformation-steps-reference-overview/rest-client-step) · [PDI 10.2 HTTP-Javadoc](https://javadoc.pentaho.com/kettle102/kettle-engine-10.2.0.0-218-javadoc/org/pentaho/di/trans/steps/http/HTTP.html).

## 3. Microsofts .NET-rättning: vad ska uppdateras?

Microsoft släppte **25 oktober 2026** en separat säkerhetsuppdatering för **CVE-2026-111497**. Felet kan nå .NET:s Brotli-avkodning genom *konfigurerad* ASP.NET Core request decompression, `HttpClient` med Brotli i `AutomaticDecompression` eller uttryckliga `System.IO.Compression`-anrop som `BrotliStream`/`BrotliDecoder`. **Varken ASP.NET Core request decompression eller automatisk HttpClient-svarsavkodning är aktiverad som standard. `.NET Framework` berörs inte av just den Microsoft-advisoryn.** Det utesluter inte separat sårbar native-Brotli som en produkt kan paketera. [Microsofts säkerhetsbesked](https://github.com/dotnet/runtime/security/advisories/GHSA-p3jp-9v83-76w9) · [Microsofts publicering](https://devblogs.microsoft.com/dotnet/dotnet-october-2026-oob-security-updates/).

| .NET-gren | Första rättade runtime |
| --- | --- |
| .NET 8 | `8.0.33` |
| .NET 9 | `9.0.22` |
| .NET 10 | `10.0.14` |
| .NET 11 förhandsversion | `11.0.0-rc.3.26522.148` |

**Vilken patch behövs i min installation?**

- **Framework-dependent app:** uppdatera den **delade runtime-version som appen faktiskt laddar** och starta om. `dotnet --list-runtimes` är en inventeringshjälp, **inte ett bevis för vilken runtime en given tjänst använder**.
- **Self-contained app:** bygg om och distribuera om appen mot rättad runtime; en patchad delad runtime på servern ändrar inte automatiskt dess medföljande kod.
- **NativeAOT:** uppdatera SDK, rätta explicita compiler-/runtime-pack-referenser och publicera/distribuera om exekverbar fil. En delad runtime på maskinen kan inte rätta kod som redan kompilerats in.
- **Leverantörsprodukt (t.ex. agent eller GIS-program):** kontrollera med leverantören om .NET är paketerad/inbyggd och **vilken** artefakt som ska uppdateras. Uppdaterad .NET ersätter inte en **separat produktpatch** eller en annan sårbar native-kopia.
- **Java eller Python utan .NET i denna kodväg:** patcha den **rätta avkodaren**, inte .NET bara för att den finns på Windowsdatorn.

Temporärt kan man **om funktionen redan är aktiverad** stänga av hantering av angriparstyrd Brotli på den specifika vägen, innan någon sårbar avkodare körs. **Slå inte på request-dekomprimering för att kunna stänga av den** och räkna inte med att bara ändra `Accept-Encoding` stoppar alla felaktiga `br`-svar. Microsoft rekommenderar att själva runtime-/applikationsfixen installeras. [Advisory och förutsättningar](https://github.com/dotnet/runtime/security/advisories/GHSA-p3jp-9v83-76w9).

## 4. Slutsatsen för ditt eget X → Y-jobb

Skriv ett **kort resultat för varje faktisk avkodningspunkt** – inte en produktgissning:

> - **Data/källa och mottagare:** …
> - **Program, aktivitet och körande process/konto:** …
> - **HTTP-svar/begäran eller intern filkomprimering:** …
> - **Faktiskt avkodande komponent och version:** …
> - **Status:** verifierat rättad / relevant möjlig exponering / ingen påvisad väg i detta avgränsade steg / fortfarande okänt.
> - **Åtkomliga resurser och möjlig verksamhetsskada:** …
> - **Patch på rätt nivå:** tillgänglig / installerad / verifierad efter omstart.
> - **Tillfälliga skydd och kvarstående osäkerhet:** …

**Uppdatering ska inte invänta ett färdigt testresultat.** Om leverantören redan rekommenderar en säkerhetsuppdatering för er produktversion genomförs den enligt säkerhetsrutinen medan den tekniska kartläggningen pågår. Inventeringen avgör vilken komponent som ska verifieras och vilka flöden som behöver prioriteras först. [Patchstatus och källor](patchkedjan.md).

### Samla resultatet i sex fält

Efter kontrollerna ovan: skriv **en slutsats per faktiskt avkodningssteg**, även om samma jobb har både HTTP-Brotli och intern Brotli i en fil. Använd samma rubriker som i [riskkedjans slutbedömning](riskkedja.md#slutbedömning--från-inventering-till-åtgärdsbeslut): **Påverkan, Exponering, Konsekvens, Prioritet, Rättning, Kvarvarande osäkerhet**. Prioritera **åtgärden** (*rätta skyndsamt*, *inventera snarast*, *ordinarie uppdatering*, *verifiera patch*) framför en påhittad risksiffra.

**Exempel – schemalagd Pythonhämtning där Brotli ännu inte är undersökt:** Påverkan: **okänd**. Exponering: automatisk hämtning, men `br` och avkodare okända. Konsekvens: jobbavbrott möjligt om avkodare är berörd; RCE ej visad. Prioritet: **inventera Pythonmiljön och avkodningen först**. Rättning: välj paket/build först när den verkliga kopian är identifierad. Kvarvarande osäkerhet: bibliotek, inkommande svar och resursbehörigheter. Detta är inte en bekräftad Kettle-, Python- eller Frends-sårbarhet.

**Exempel – schemalagt Pythonjobb som hämtar öppna data och sparar dem i en databas:** URL:en eller Swagger-dokumentet är inte tillräcklig information. Ta reda på faktisk Python-interpreter, HTTP-klient, `Content-Encoding`, eventuell filimport, avkodare, paketversion och jobbets databasrättigheter. En angripare behöver fortfarande kunna få relevanta felaktiga data hela vägen till den sårbara avkodaren. **Enbart att jobbet har databasåtkomst visar inte intrång eller personuppgiftsläcka.**

**Exempel – Kettle hämtar samma data:** samma frågor om API-svar och filformat, men den avkodande komponenten kan ligga i Kettles Java-/native-beroenden. Du ska **inte** utgå från att Microsofts .NET-fix rättar en Java-installation – och inte heller påstå att Kettle är CVE-sårbart innan beroende och indataväg är fastställda.

**Samla inte in åtkomsttoken eller personuppgifter i diagnostikloggen. Skicka inte manipulerad Brotli-data till en levande extern tjänst för att prova CVE-utlösning.** Ofarlig versions-, konfigurations- och beroendeinventering räcker långt för att välja korrekt uppgradering eller dokumentera vad som återstår att verifiera.

# HTTP – hur når angriparpåverkade svar och begäranden Brotli?

**Kontrollerat: 8 november 2026.** [Översikt](../README.md) · [Skydd](skydd.md)

**Två olika angreppsvägar ska aldrig blandas ihop:** en server som **tar emot** en Brotli-komprimerad begärandekropp och en klient som **hämtar** ett Brotli-komprimerat HTTP-svar. Båda kan leda till avkodaren, men via olika kod och olika förtroendegränser.

## Inkommande HTTP-begäranden

```text
Extern klient → HTTP-proxy/lastbalanserare → serverappens parser
  → accepterad Content-Encoding: br → Node.js/.NET/native Brotli-avkodare
```

**Exponeringsvillkor:** (1) En utomstående kan nå den relevanta tjänsten eller en tillåten uppladdningsfunktion. (2) Begärandekroppar med `Content-Encoding: br` accepteras av en parser som avkodar dem. (3) Innehållstyp, middlewareordning, storleksregler och eventuell proxy tillåter data att nå parsern. (4) Den körande avkodaren är berörd. (5) Den aktuella komprimerade strömmen utlöser CVE-villkoret.

[Express](https://expressjs.com/en/blog/2026-10-31-security-releases/#a-note-on-brotli-decompression-cve-2026-111497) dokumenterar att standardparsers som `express.json()` och `express.urlencoded()` kan nå `zlib.createBrotliDecompress()` vid relevanta Brotli-komprimerade begärandekroppar. Parsern kan vara registrerad *före* en autentiserande route, men sådan ordning måste kontrolleras i den enskilda applikationen. En publikt nåbar Node.js-tjänst med gammal, berörd runtime och aktiv Brotli-parser kan därför ha en **potentiell oautentiserad serverväg**. Det är en **allmän beroendeväg för Node.js/Express-tjänster**, inte en unik sårbarhet i någon namngiven GIS-produkts egen applikationskod. Att bara köra gammal Node.js eller vara publik räcker däremot **inte** som exploateringsbevis.

Microsoft anger att .NET:s ASP.NET Core request decompression **inte** aktiveras som standard; konfigurationen måste faktiskt använda relevant Brotli-avkodning. [Microsoft .NET-advisory](https://github.com/dotnet/announcements/issues/494).

**Skydd som bryter just denna väg:** uppdaterad runtime; att upstream avvisar Brotli-komprimerade begärandekroppar före någon avkodare om funktionen inte behövs; granskning av middleware och proxy; trafikbegränsning. En WAF som inte avkodar/inspekterar rätt lager är inte en tillförlitlig CVE-patch. [Express råd om tillfällig begränsning](https://expressjs.com/en/blog/2026-10-31-security-releases/).

## Utgående anrop och mottagna HTTP-svar

```text
GIS-app/automatiserat jobb begär WMS/WFS/WMTS/API eller annan URL
  → server returnerar Content-Encoding: br
  → Qt Network / libcurl / Node fetch / konfigurerad .NET HttpClient
  → eventuellt sårbar Brotli-avkodare
  → GIS- eller applikationskod tolkar det avkodade svaret
```

**Exponeringsvillkor:** någon kan styra eller påverka HTTP-svarets **komprimerade byte**, den valda klientstacken utför verkligen Brotli-avkodning, och den berörda avkodarkopian används. `Accept-Encoding: br` anger vad klienten kan acceptera; `Content-Encoding: br` anger kodningen hos svaret. **Enbart det ena eller andra HTTP-huvudet verifierar inte att avkodning sker i den undersökta applikationen.** [libcurl CURLOPT_ACCEPT_ENCODING](https://curl.se/libcurl/c/CURLOPT_ACCEPT_ENCODING.html) · [Qt QNetworkRequest](https://doc.qt.io/qt-6/qnetworkrequest.html) · [Microsoft](https://github.com/dotnet/announcements/issues/494).

**Varför även till en betrodd källa?** Källans namn och TLS-certifikat säger inte att innehållet är ofarligt för en bibliotekavkodare. Det kan vara en komprometterad dataleverantör, ett behörigt konto som manipulerats, en uppströmsdatakedja med angriparpåverkan eller en tjänst som legitimt vidarebefordrar data från andra. HTTPS autentiserar normalt transportens motpart och skyddar trafiken mot ändring *mellan dessa TLS-ändpunkter*, inte mot farliga byte som den autentiserade källan själv skickar.

## Öppna data via Swagger/OpenAPI: ett API-svar, inte ett eget filformat

**Scenario:** någon går till en myndighets eller annan dataägares Swagger UI och väljer **Try it out/Execute**, eller använder samma dokumenterade OpenAPI-endpoint i FME, Python eller ett GIS-program. Swagger/OpenAPI beskriver API-anropen; **det är inte Swagger i sig som avgör vilken Brotli-avkodare som används**. Vilken **klient som faktiskt skickar begäran och tar emot svaret** avgör var en eventuell sårbar avkodning sker. [Swagger UI:s dokumentation](https://swagger.io/docs/open-source-tools/swagger-ui/usage/configuration/).

```text
Öppet API → HTTP-svar, t.ex. GeoJSON/JSON/CSV eller en nedladdningsfil
  → eventuell HTTP-kodning: Content-Encoding: br
  → webbläsare / FME HTTPCaller / Pythonbibliotek / GIS-klient
  → klientens faktiska Brotli-avkodare, om den används
  → tolkning eller sparande av innehållet
  → eventuellt en ANDRA avkodningsväg vid senare filimport (t.ex. Parquet)
```

| Så hämtas öppna data | Var behöver man leta efter avkodaren? | Vad är inte styrkt enbart av hämtningen? |
| --- | --- | --- |
| **Swagger UI:s Try it out i webbläsaren** | Webbläsarens nätverksstack för det faktiska API-svaret; håll webbläsaren uppdaterad. | Att webbläsaren använder just den **sårbara Google Brotli-koden** eller att ett normalt API-svar kan utlösa CVE:n. |
| **FME HTTPCaller/OpenAPICaller** | FME Engine/Workbench och den HTTP-klient/Brotli-kopia som den installerade FME-versionen använder. | Att alla arbetsytor använder Brotli eller att andra integrationers data automatiskt exponeras. [FME](fme.md). |
| **Python eller eget skript** | HTTP-biblioteket, dess eventuella Brotli-beroende och aktuell körmiljö. | Att varje Python HTTP-klient automatiskt avkodar `br`. |
| **QGIS eller annan GIS-klient** | Vilken nätverkskodväg (t.ex. Qt eller libcurl) som används för just anropet. | Att QGIS-specifik CVE-trigger via API/WMS/WFS har demonstrerats. [QGIS](qgis.md). |
| **Nedladdad fil öppnas senare** | Både eventuell HTTP-avkodning vid hämtningen och den *separata* filparserns avkodning, exempelvis Parquet/Arrow. | Att `Content-Encoding` säger något om komprimeringen **inne i filformatet**. [Filvägar](fil-angreppsvagar.md). |

**Kontrollera utan att skicka felaktiga data:** I webbläsarens utvecklarverktyg, **Network**, välj den verkliga API-begäran (inte bara Swagger-sidans HTML), och titta efter `Content-Encoding: br`, `gzip` eller frånvaro av sådan kodning. `Accept-Encoding` visar vad klienten annonserar, inte nödvändigtvis vad servern skickar. Ett `br`-svar visar att Brotli är relevant **på HTTP-nivån**, men inte vilken biblioteksimplementation som används eller att den är sårbar. Om `gzip` eller ingen HTTP-komprimering används finns **ingen Brotli-avkodning på just det HTTP-steget**; eventuell intern Parquet-/filkomprimering återstår att undersöka. Webbläsaren kan redan ha avkodat svaret när det visas. [MDN Content-Encoding](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Content-Encoding) · [MDN Accept-Encoding](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Accept-Encoding).

**Så kontrollerar du samma API-hämtning i FME, Frends, ArcGIS, Python eller Kettle:** [från dataflöde till patch](dataflode-till-patch.md). [Fler exempel på API-flöden](verksamhetsfloden.md) skiljer metadatahämtning, filhämtning, POST och POST-svar.

**Riskkedjan:** en legitim API-källa kan vara betrodd men ändå leverera angriparpåverkade data, exempelvis efter ett intrång hos leverantören eller i dess uppströmsflöde. Korrekt HTTPS hindrar normalt en extern nätverksangripare från att byta ut svaret på vägen, men inte sådant innehåll från den autentiserade källan. Det krävs fortfarande en **sårbar och nåbar avkodare** samt indatan som utlöser CVE-villkoret. Om en FME Engine både hämtar öppna data och kör en separat känslig integration ska processens faktiska åtkomster kartläggas; detta är **inte ett påstående om personuppgiftsläcka**. [Riskkedja](riskkedja.md#exempel-öppna-data-från-api-och-separat-känslig-integration).

**Skydd:** patcha avkodaren i rätt klient/runtime, använd korrekt verifierad HTTPS och kontroll över API-källor, begränsa privilegier och nätverksåtkomst för automatiska jobb, och följ upp om patchen verkligen används. Att stänga av `br` i ett visst klientflöde kan minska just den ingången om det genomförs **före avkodning** och verifieras, men är inte en generell ersättning för patchning.

### Man-in-the-middle (MITM), HTTP och HTTPS

- **Oskyddad HTTP:** En nätverksangripare med relevant möjlighet att påverka trafiken kan i princip ändra ett svar och därmed försöka nå klientens Brotli-avkodare. Det förutsätter att klienten använder avkodningsvägen. Använd korrekt verifierad HTTPS.
- **Korrekt verifierad HTTPS:** En vanlig angripare på nätverksvägen kan inte normalt byta ut svarsinnehåll utan att TLS-valideringen misslyckas. DNS-ompekning ensam räcker inte. Kvarvarande risker rör exempelvis en komprometterad källa, illvilligt innehåll från legitim API-leverantör, lokalt betrodd TLS-inspektionsproxy eller bristfällig certifikatvalidering. Det är *skilda* scenarier.
- **Social engineering:** En angripare kan försöka få en operatör att lägga till en annan datatjänst, ändra en URL eller köra ett arbetsflöde som hämtar externa data. Det är en möjlig leveransmetod, inte en CVE-specifik exploatering som har verifierats mot en viss produkt.

Källa: [OWASP TLS](https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Security_Cheat_Sheet.html). **Kryptering och källautentisering är nödvändiga skydd men ingen ersättning för att rätta avkodaren.**

## Intern tjänst och automatiska jobb

En intern FME Flow, GIS Server eller ETL-tjänst kan vara exponerad **indirekt**: en schemalagd hämtning eller filimport behandlar data från internet, en integrerad leverantör eller en annan intern tjänst. Direkt internetåtkomst till serverns inkommande port krävs inte för att en *klientbaserad* avkodningsväg ska vara relevant. Även funktioner som hämtar användarangivna URL:er kan förstärka påverkan, men SSRF/URL-auktorisering är **en separat kontrollfråga**, inte samma CVE. [OWASP SSRF](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html).

**Kontrollera i den egna miljön:** vilka källor begärs, vem får ändra dem, vem kontrollerar svarens data, utför klienten automatisk `br`-avkodning, vilken bibliotekskopia laddas och vilken patch är installerad? Se [andra program](ny-programvara.md) och [produktkopplingarna](produktoversikt.md).

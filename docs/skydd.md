# Skydd – vad kan förhindra att angriparstyrda Brotli-data gör skada?

**Kontrollerat: 8 november 2026.** [Översikt](../README.md) · [Patchkedja](patchkedjan.md)

**För helhetsbilden:** [README](../README.md). Behöver du lämna en avgränsad fråga till IT: [ansvar och överlämning](borja-har.md).

**Huvudåtgärd: installera en version som innehåller den korrigerade avkodaren och säkerställ att processen använder den.** Alla övriga kontroller är villkorade riskbegränsningar eller skademinimering. Använd [riskkedjan](riskkedja.md) för att bedöma både **vad som stoppar den angriparpåverkade indatan** och **vad som begränsar skadan om en process trots detta påverkas**.

| Skydd | Vilken väg bryter eller begränsar det? | Vad garanterar det inte? |
| --- | --- | --- |
| **Uppdatera rätt Brotli-kopia och starta om/bygg om** | Tar bort det kända avkodningsfelet i rättad kod | Att en gammal kopia inte fortfarande används på annat håll |
| **Avvisa inkommande `Content-Encoding: br` före parsers**, om det inte behövs | Kan bryta den berörda inkommande HTTP-vägen | Att tjänsten inte har en separat HTTP-klient eller filparser |
| **TLS med korrekt certifikatverifiering** | Hindrar normalt nätverks-MITM att manipulera HTTP-svar under transport | Att en autentiserad eller komprometterad källa skickar säker data |
| **Begränsa vilka externa källor och URL:er som får hämtas** | Minskar angriparens kontroll över externa svar och oavsiktliga vidarebefordringsvägar | Att tillåtna källor aldrig komprometteras; fullständig CVE-riskfrihet |
| **Karantän, filtypskontroll, lämplig skanning och isolerad import** | Minskar exponeringen och effekten från manipulerade filer | Att skanning/magic bytes upptäcker varje parserutlösande fel |
| **Minsta privilegium, servicekonton, sandbox/container, nätverkssegmentering** | Begränsar möjliga följder efter krasch eller lyckad kodkörning | Att ett fel inte kan utlösas eller en process inte kan komprometteras |
| **Rate limiting, observation av krascher och larm** | Underlättar upptäckt och dämpar vissa avbrott | Att en enstaka felaktig avkodning stoppas; fullständigt RCE-skydd |
| **Verifierad återläsning och återställningsbara offlinebackuper** | Reducerar påverkan av senare dataskada/ransomware | Att intrånget eller CVE-utlösningen förhindras |

Källor: [Express – tillfällig spärr före body-parser](https://expressjs.com/en/blog/2026-10-31-security-releases/), [OWASP TLS](https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Security_Cheat_Sheet.html), [OWASP File Upload](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html), [OWASP SSRF](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html).

## Prioritera aktiva riskkedjor

**Två installationer med samma berörda Brotli-version kan behöva helt olika akut prioritet.** Värdera inte enbart CVSS, programnamn, intern/publik nätverksplacering eller ett gammalt versionsnummer. Ett relevant CVE-angrepp kräver att en **körande eller senare startad process** avkodar angriparpåverkade Brotli-data med berörd kod och att det särskilda felvillkoret nås. Det är *inte* samma sak som att exploatering är visad i produkten.

| Fråga | Högre risk / tidigare åtgärd | Lägre eller ännu obestyrkt exponering |
| --- | --- | --- |
| **1. Aktivitet** | Tjänst kör kontinuerligt eller jobb startas automatiskt och hämtar data | Endast kvarlämnad lokal installation, utan verifierad användning eller schemalagda anrop |
| **2. Angriparkontroll och nåbarhet** | Publikt tillgänglig parser kan avkoda inkommande `br` utan autentisering; eller automatiskt jobb avkodar externa/komprometterbara svar | Ingen identifierad väg till berörd avkodare; detta måste beläggas, inte antas |
| **3. Konsekvenser vid möjlig påverkan** | Processen har känsliga anslutningar, stora rättigheter, personuppgifter eller verksamhetskritisk tillgänglighet | Isolerad process med begränsade rättigheter; krasch kan ändå vara relevant |
| **4. Rättningsläge** | Orättad, faktiskt använd avkodare och tillgänglig leverantörsrättning | Rättad avkodare verifierad i körande process, eller avkodningsvägen bevisligen saknas |

**Praktisk prioriteringsordning:** rätta först aktiva och nåbara avkodningsvägar med stora möjliga konsekvenser. En **publik opatchad Node.js/Express-server** som avkodar angriparstyrda komprimerade begärandekroppar före autentisering är normalt akut. En **intern FME-/Python-/Node-automatisering** som avkodar data från externa källor kan vara minst lika viktig när den kan nå känsliga system. En **gammal lokal Node-/npm-installation som bara användes för felsökning och inte körs** är normalt lägre akut prioriterad för *den här* CVE:n; uppdatera eller avinstallera den enligt ordinarie hantering. Men kontrollera först att inga tjänster, Aktivitetsschemaläggaren-jobb, CI-flöden eller andra program använder den.

**Viktigt för patchbeslut:** Om leverantören rekommenderar en rättning för den berörda produktversionen ska den genomföras enligt er ordinarie säkerhetsprocess även utan eget exploitbevis. Prioriteringen gäller **ordningen och behovet av extra skydd**, inte om en rekommenderad rättning ska hoppas över. Begränsat användningssätt sänker ibland exponeringen men ger inget generellt säkerhetsintyg.

**npm är inte en server:** `npm` hanterar paket, medan Node.js är en runtime som kan starta kod. En gammal npm-version visar varken att en offentlig tjänst körs eller vilken Brotli-kopia en annan applikation använder. Kontrollera också om ett program levererar **en separat inbäddad Node-runtime**, som kräver egen uppdatering. Ett stoppat program är inte en pågående attackyta för den vägen, men kan bli exponerat när det startas igen.

**Om information saknas: skriv "okänd – undersök"**, inte "säker", "sårbar" eller "låg risk". En publik port eller gammal Node-version räcker inte ensam för att visa Brotli-exploaterbarhet; ett icke-publikt system blir inte riskfritt om det läser angriparpåverkad data. Att CVE:n har demonstrerad RCE i särskilda Node-testmiljöer belägger inte RCE i just den installerade produkten. [Express om Brotli och rättning i Node](https://expressjs.com/en/blog/2026-10-31-security-releases/) · [Node.js säkerhetsreleaser](https://nodejs.org/en/blog/vulnerability/october-2026-security-releases) · [riskkedjan](riskkedja.md).

**Slutsatsen ska innehålla ett beslut även när påverkan är okänd.** Avsluta varje identifierad väg med [sex fält: påverkan, exponering, konsekvens, prioritet, rättning och kvarvarande osäkerhet](riskkedja.md#slutbedömning--från-inventering-till-åtgärdsbeslut). Vid okänd implementation är första beslutet **inventera**, inte ”avvakta tills någon visar RCE”. Dokumenterad frånvaro av en nåbar kodväg kan motivera lägre akut prioritet för *den vägen*; det betyder inte att framtida användning eller andra avkodare är säkra.

## Prioritering per ingång

**Publik Node.js/Express-tjänst:** kontrollera om komprimerade begärandekroppar kan nå parsern, rätta Node.js och kontrollera hur reverse proxy/lastbalanserare hanterar `br`. En kantspärr måste ligga *före* avkodningen för att bryta vägen. Att applikationen använder `Content-Encoding: gzip` eller komprimerar **utgående** svar räcker inte för att dra en slutsats om Brotli-inmatning.

**GIS-klient som hämtar WMS/WFS/API:** rätta nätverksstackens faktiska Brotli-kopia, använd TLS och betrodda konfigurationer, begränsa vem som får ändra externa tjänster och kontrollera eventuell automatisk avkodning. Betrodda källor är en riskreduktion, inte bevis för säker parser.

**Import av GeoParquet och andra filer:** rätta Arrow/Parquet, DuckDB, libjxl eller den faktiska Brotli-kopia som används; validera och isolera externa dataset innan behandlingen där det är praktiskt möjligt. En regel som enbart förbjuder `.br` hjälper inte när Brotli ligger inuti andra filformat.

**Intern automatisering:** kartlägg FME- och andra batchjobbs externa inhämtningar och filflöden. Internt placerad server betyder inte frånvaro av angriparpåverkad indata.

## Skydda resurserna om processen ändå påverkas

Identifiera den faktiska processen och körkontot, kontrollera dess åtkomst till filer, anslutningshemligheter, databaser och personuppgifter. Ge endast nödvändiga behörigheter, avgränsa integrationskonton och egress, separera känsliga arbetsflöden när arkitekturen medger det och kontrollera att återställning av viktiga jobb fungerar. **Dessa åtgärder förhindrar inte själva out-of-bounds-felet**, men kan minska skadan av krasch eller ett senare lyckat intrång. Att en integration behandlar personuppgifter är inte i sig bevis för att CVE:n läcker dem. [Riskkedjan](riskkedja.md).

## Efter uppdateringen

Kontrollera *vilken* process som kör, dess körfil, tillhörande bibliotek och containeravbild/digest, inklusive om flera installationer finns på samma maskin. Kontrollera att ingen äldre medföljande DLL, Pythonmiljö eller inbyggd native-kopia används. Ett nytt paket i registret eller uppdatering på disken utan omstart verifierar inte det processminne som redan körs. [Patchkedjan](patchkedjan.md).

**Kompletterande skydd ersätter inte leverantörens rättning.** En WAF eller filskanner kan misslyckas mot okända eller inbäddade felmönster. Undvik att själv mata publika produktionssystem med CVE-reproducerande indata för kontroll; följ leverantörens verifieringsmetoder och eventuell samordnad rapportering. [Metod](metod.md).

# Bedöm ett annat program – CVE-först utan att gissa produktsårbarheter

**För en samlad översikt:** [README](../README.md). Om du saknar driftansvar: [avgränsa och lämna vidare](borja-har.md).

**Praktisk ingång för ett bestämt API, ett jobb eller en nedladdning:** [Vi hämtar X via Y – vad behöver vi kontrollera?](dataflode-till-patch.md).

**Metod för program som inte nämns i produktkartan.** CVE-felet finns i Brotli; ett okänt program kan använda samma vanliga bibliotek utan att det har en egen rapporterad kodbrist. Fastställ beroende och möjlig påverkan, och följ relevanta rättningar utan att först försöka exploatera produkten. [Översikt](../README.md) · [Supply chain](supply-chain.md) · **[Riskkedja](riskkedja.md)** · [Bevisnivåer](metod.md)

## Frågan är inte "har programmet Brotli?" utan "kan angriparpåverkad komprimerad data nå just den berörda avkodaren?"

Följ **samma ordning** oavsett om det gäller en intern GIS-tjänst, en webbkarta, en ETL-motor, ett Python-skript, en container, en HTTP-proxy eller ett annat program:

1. **Vilken CVE?** Identifiera CVE-2026-111497, relevant felaktig *avkodning* och upstream-rättad kod. [Felmekanism](cve-mekanism.md).
2. **Var kan angriparen påverka byte?** Finns inkommande komprimerade begäranden, hämtade svar, filer, uppladdningar, importer eller automatiska jobb? Vem får styra URL/källa eller filinnehåll?
3. **Var korsar innehållet en förtroendegräns?** Finns HTTP utan TLS, extern tjänst, användarstyrd hämtning, delad mapp eller leverantörsdata? Förutsätt inte att intern infrastruktur betyder betrodd indata.
4. **Vilken komponent *avkodar*?** Följ data från parser/klient/filformat via direkt och transitivt beroende till avkodningsfunktionen. Skilj t.ex. `Content-Encoding: br` från en Parquet-datasida med codec BROTLI.
5. **Vilken Brotli-kopia körs?** Systemlänkad `libbrotlidec` eller `brotlidec.dll`, statiskt inbyggd kod, Node/.NET, Python-wheel, Brotli4j native-JAR, containeravbild? Inventera faktiskt byggares och körda artefakters ursprung.
6. **Vad är belagt?** Leverantörspåverkan, dokumenterad kodväg, verifierad normal avkodning, CVE-specifik utlösning, minneskorruption eller RCE måste redovisas som *olika observationer*. Okänd väg är inte "säker".
7. **Vilken process och vilka resurser påverkas?** Identifiera körkonto, anslutningar, databaser, filrättigheter, hemligheter och eventuella personuppgifter som *just denna process* faktiskt har åtkomst till. Skilj krasch från endast möjlig RCE och från belagt röjande. [Riskkedja](riskkedja.md).
8. **Vilken verksamhetsskada är möjlig?** Stoppar en krasch ett viktigt jobb? Skulle en verklig informationsläcka exponera skyddsvärda data? Vad begränsar skadan om kodkörning trots allt uppnås?
9. **Vilken rättning träffar kopian?** Upstream-, systempaket-, runtime-, produkt-, image- eller native-paketpatch? Finns rättning *för aktuellt bygge* och är den i drift? [Patchkedja](patchkedjan.md).
10. **Vad bryter vägen före patch och vad begränsar följdskadan?** Ange både skydd mot inträde och kontroller för minsta behörighet, isolering och återställning. [Skydd](skydd.md).

## Börja i funktionen, inte produktnamnet

Om en kollega säger ”jag använder myCarta” eller nämner ett annat program utan eget säkerhetsbesked finns ännu inte tillräcklig information för att avgöra just denna CVE. **Frågan är vilken variant, vilka tillägg och vilken avkodare som används.** Produktnamnet ensamt avgör inte påverkan.

För ett system såsom **myCarta** är status här **okänd**: vi har inte verifierat programversion, bibliotek, aktiv HTTP-/filavkodning eller CVE-specifik patch. Inventera server, proxy, webbläsarklient och eventuella import-/pluginflöden som **separata byggningar**; anta inte att någon av dem använder Brotli bara för att de bygger en webbkarta. Om exempelvis HTTP-hämtning eller filimport sker, identifiera först vilken komponent som avkodar datan. Det är *en hypotes att kontrollera*, inte ett påstående om myCarta.

En användare som endast hanterar självproducerade lokala filer i QGIS kan sakna angriparpåverkad indata på en viss avkodningsväg, men det säger inte att installationens samtliga tillägg, importer och nätverksfunktioner saknar andra vägar. **Begränsad praktisk exponering och uppdateringsbehov är olika bedömningar.** [Supply chain och lokala arbetsflöden](supply-chain.md#4-hur-ska-man-tänka-om-programmet-bara-används-lokalt).

**Schemalagd Python, Kettle/Pentaho, FME, Frends och ArcGIS:** se [verktygsoberoende hämtning → process → avkodare → patch](dataflode-till-patch.md#2-samma-data--olika-program-kontrollera-här). Windows Aktivitetsschemaläggaren är inte en Brotli-decoder; dess **Åtgärder** identifierar vilket program, skript och körkonto som faktiskt kör. Kettle är normalt en Java-miljö: anta inte att Microsofts .NET-patch rättar dess Java-/native-bibliotek.

## Användbart dokumentationskort

> **Program/version/plattform:** …
> **Ingång och angriparkontroll:** …
> **Dataflöde till Brotli:** …
> **Avkodare och paketering:** …
> **Vilka villkor krävs?** …
> **Evidensnivå / källa / kontrolldatum:** …
> **Vad *är inte* visat?** …
> **Påverkad process / konto / arbetsyta:** …
> **Åtkomliga resurser och skyddsvärden:** …
> **Möjlig skada vid krasch, separat vid visat röjande eller möjlig RCE:** …
> **Rättning: upstream / i produkten / verifierad i drift:** …
> **Tillfälligt skydd och kvarvarande frågor:** …

**Avsluta även en utredning av okänd programvara med sex fält**, inte bara en lista med installerade bibliotek: **Påverkan** (bekräftad/möjlig/okänd), **Exponering** (aktiv, nåbar, ej visad), **Konsekvens** (processens faktiska skyddsvärden), **Prioritet** (nästa kontroll eller rättning), **Rättning** (vilken distributionskopia och driftstatus) och **Kvarvarande osäkerhet** (vad som återstår att visa). Använd [det gemensamma avslutskortet](riskkedja.md#slutbedömning--från-inventering-till-åtgärdsbeslut). **Okänt är inte lika med säkert.**

## Exempel på härledning av uppgradering utan produktspecifikt CVE-besked

**Egen Node.js API-tjänst:** Om kod eller middleware avkodar inkommande `br`-begäranden och servern kör berörd Node.js-runtime finns en relevant väg; om enbart statiska filer skickas eller utgående svar komprimeras är samma väg inte påvisad. Dessutom måste eventuella utgående `fetch()`-anrop undersökas var för sig. [Express](https://expressjs.com/en/blog/2026-10-31-security-releases/) · [Node.js](https://nodejs.org/en/blog/vulnerability/october-2026-security-releases).

**Pythonbearbetning av Parquet:** Ta reda på om PyArrow är installerat och om den aktuella bygget använder medföljande eller system-Brotli. En förbyggd wheel är inte likvärdig med systemlänkad Arrow C++. Kontrollera om någon faktiskt importerar Brotli-komprimerade Parquet-sidor från extern källa. [Arrow](https://arrow.apache.org/blog/2026/11/03/26.0.1-release/).

**Hämtning av öppna data från Swagger/OpenAPI:** Identifiera den **faktiska HTTP-klienten**. Om en användare trycker *Try it out* i en webbläsare sker svarets HTTP-avkodning i webbläsarmiljön; om en schemalagd FME Engine eller ett Python-skript använder API-URL:en sker den i deras respektive HTTP-bibliotek. Kontrollera faktisk `Content-Encoding`, berörd avkodare, process och skyddsvärden. Swagger som dokumentationsgränssnitt och `Content-Type: application/json` bevisar varken Brotli eller exploaterbarhet. En senare nedladdad Parquet-fil kan innehålla **separat intern** Brotli-komprimering. [HTTP och öppna data](http-angreppsvagar.md#öppna-data-via-swaggeropenapi-ett-api-svar-inte-ett-eget-filformat) · [Riskkedja](riskkedja.md#exempel-öppna-data-från-api-och-separat-känslig-integration).

**Java-baserad webbtjänst:** Titta efter installerade native-komponenter och aktiv Brotli-avkodning (exempelvis Brotli4j, inte bara Jetty som namn). En `brotli4j`-version ≤1.23.0 kan ha sårbar native-kod; det säger ännu inget om applikationens tillgängliga dataflöden. [Brotli4j](https://github.com/hyperxpro/Brotli4j/security/advisories/GHSA-x524-jrqf-vwq6).

**Containeriserat program:** En paketfynd i Dockerfile eller på registry-taggen är inte samma sak som bilden som körs. Granska digest, paket i den körande bilden, inbyggda beroenden och om en ombyggnad/start har gjorts.

## Vad betyder "inte påverkad"?

En sådan slutsats kräver ett **avgränsat, positivt skäl**, till exempel att berörd komponent inte ingår, att den berörda koden inte finns i den aktuella byggningen eller att en specifik väg inte kan nå avkodningen enligt verifierat underlag. Frånvaro av sökträff eller ett misslyckat syntetiskt test räcker inte. Använd VEX-status endast om den kommer från belagt underlag och påståendets produkt/version/konfiguration är tydliga. [CISA VEX](https://www.cisa.gov/sites/default/files/2023-04/minimum-requirements-for-vex-508c.pdf).

**Rör inte obehöriga levande tjänster med felaktig komprimerad indata för att verifiera attackvägen.** Prioritera leverantörskällor, kod- och beroendeanalys och ofarlig inventering; ny produktspecifik säkerhetsevidens ska samordnas med ansvariga innan publik detaljpublicering.

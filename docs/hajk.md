# Hajk: Express-backend, Node.js och olika distributioner

**Källkontroll: 8 november 2026**  

**Den generella modellen:** [CVE-felet](cve-mekanism.md) · [HTTP-ingångar](http-angreppsvagar.md) · [filingångar](fil-angreppsvagar.md) · [konsekvenser](konsekvenser.md) · [skydd](skydd.md) · [patchkedja](patchkedjan.md). Varje produktväg är en tillämpning av dessa villkor, inte ett oberoende säkerhetsbesked.

[Till attackvägsöversikten](produktoversikt.md) · [Metod](metod.md)

**Bevisstatus: källkodsstödd möjlig Express/Node-avkodningsväg; ingen Hajk-specifik CVE-utlösning eller RCE belagd här.** En berörd installerad runtime bör säkerhetsuppdateras enligt ansvarig leverantör/distribution utan att invänta exploittest.

**Kort svar:** Hajks backend använder Express och registrerar parsers som kan avkoda komprimerade begärandekroppar via Node.js. Det ger en relevant villkorad Brotli-väg. Däremot innehåller statiska `hajk-simple` inte samma Node.js-backend, och ZIP-/Docker-utgåvor kräver olika rättning.

## Möjlig CVE-väg genom inkommande HTTP

```text
Inkommande HTTP-begäran med Content-Encoding: br
  → proxy/ingång till Hajk Backend
  → Express.json()/urlencoded()/text(), om parsern matchar
  → Node.js zlib → medföljande eller distribuerad Brotli
```

Hajks [backend-serverkod](https://github.com/hajkmap/Hajk/blob/master/apps/backend/server/common/server.js) registrerar `Express.json()`, `Express.urlencoded()` och `Express.text()` i middlewarekedjan. [Express-projektet](https://expressjs.com/en/blog/2026-10-31-security-releases/#a-note-on-brotli-decompression-cve-2026-111497) dokumenterar att de kan behandla Brotli-komprimerade begärandekroppar. Den verkliga exponeringen påverkas av `Content-Type`, body-parsers konfiguration, reverse proxy och den runtime som faktiskt körs.

Det här bevisar **inte** CVE-utlösning i ett visst Hajk-bygge, en internetåtkomlig exploiterbar endpoint eller RCE. Även eventuella externa HTTP-anrop från backend måste analyseras **separat**: att middleware avkodar inkommande begäranden betyder inte att en annan HTTP-klient avkodar hämtade svar med samma inställning.

## Patch: identifiera distributionsformen först

| Hajk-distribution | Kör Node.js-backend? | Vad behöver rättas? |
| --- | --- | --- |
| `hajk-vX.Y.Z-nodejs.zip` | **Ja**, om backend startas | Node.js-runtime installeras **separat** och måste uppdateras i driftmiljön |
| `hajk-vX.Y.Z-simple.zip` | **Nej**, statiska klientfiler | Ingen Node.js-backend följer med denna ZIP; externa webbservrar och tillägg bedöms separat |
| `ghcr.io/hajkmap/hajk` (full Docker) | **Ja** | Välj image med rättad Node.js/OS-bas, hämta och **återskapa** containern |
| `ghcr.io/hajkmap/hajk-simple` | **Nej**, nginx och statiska filer | Ingen motsvarande Node.js-runtime finns i denna driftavbild |

[Hajk releaseguide](https://github.com/hajkmap/Hajk/blob/master/docs/releases.md) · [Hajks säkerhetspolicy](https://github.com/hajkmap/Hajk/blob/master/SECURITY.md).

**Officiell Docker:** En vanlig release-image byggs **en gång** och blir inte automatiskt säker när grundimagen uppdateras. `nightly-master` byggs om regelbundet från aktuell release och nyare grundimage; det innebär inte automatiskt att alla npm-beroenden uppdateras, eftersom låsfilerna styr dem. Hajk beskriver även hur en release-image byggs om från rätt tagg. Kontrollera **image-digest och versioner i körande container**, inte bara taggnamnet i en `docker pull`-logg.

**Node.js:** För de uppströms Node-grenar som berörs finns första rättade versionerna 22.23.4, 24.22.1 och 26.12.1 (24.22.2 eller senare rekommenderas framför regressionspåverkade 24.22.1). Äldre osupporterade Node-grenar får inte dessa rättningar från Node-projektet. [Node.js säkerhetsrelease](https://nodejs.org/en/blog/vulnerability/october-2026-security-releases).

**Hajks supportsnitt:** Enligt den granskade policyn får bara senaste release säkerhetsrättningar; äldre versionsserier behöver uppgraderas till stödd release. Det finns ingen grund att dra slutsatsen att ett godtyckligt äldre ZIP- eller Docker-paket automatiskt innehåller uppdaterad Node.js.

## Slutbedömningar för Hajk-distributionerna

**Hajk med aktiv Express-backend**

**Slutbedömning av denna riskkedja (exempel, ingen verifierad lokal installation):**

| Fält | Bedömning |
| --- | --- |
| **Påverkan** | Källkodsstödd möjlig Brotli-väg via Express/Node; ingen verifierad exploiterbar Hajk-endpoint. |
| **Exponering** | Kräver full backend med berörd Node och angriparpåverkad `br`-begäran som når parsern. |
| **Konsekvens** | Möjligt serveravbrott; RCE eller dataläckage i Hajk inte visat. |
| **Prioritet** | **Rätta aktiv nåbar backend skyndsamt** om Node-kopian är berörd; kontrollera offentlig och intern åtkomst. |
| **Rättning** | Rättad Node-runtime i ZIP-driftsättningen eller ombyggd/full Docker-image med rättad runtime. |
| **Kvarvarande osäkerhet** | Faktiskt körande image/Node-version, proxy, parser och eventuell separat hämtning. |

**Statiska `hajk-simple`-distributioner**

**Slutbedömning av denna riskkedja (exempel, ingen verifierad lokal installation):**

| Fält | Bedömning |
| --- | --- |
| **Påverkan** | Ingen Node-backend i den beskrivna simple-distributionen; det utesluter inte separata servrar/tillägg. |
| **Exponering** | Ingen visad **backend-Node**-väg genom själva statiska paketet. |
| **Konsekvens** | Ingen visad CVE-följd via just denna backend-väg. |
| **Prioritet** | **Lägre akut prioritet för denna Node-väg**, men kontrollera webbserver, browserklient och fristående tjänster separat. |
| **Rättning** | Ingen Node-patch i själva simple-paketet för denna väg; uppdatera eventuellt berörda externa komponenter. |
| **Kvarvarande osäkerhet** | Vilka externa proxy-/serverkomponenter hör till installationen? |

**Öppna frågor:** Vilken faktisk Node.js-version ingår i den körande fulla avbilden? Accepterar reverse proxy och backend aktuell typ av komprimerade HTTP-kroppar? Finns andra externa svarsvägar genom klient- och backendkomponenter?

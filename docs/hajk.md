# Hajk – kontrollera underliggande Node.js/Brotli i backenddrift

**Källkontroll: 8 november 2026**

**Var ligger sårbarheten?** CVE-2026-111497 gäller **Googles Brotli-avkodare**, som har ingått i berörda versioner av **Node.js**. Det är inte ett Hajk-specifikt fel som här har påvisats i projektets egen applikationskod. Hajk tas upp eftersom vissa driftsättningar använder Node.js i backend – precis som många andra tjänster, även utanför GIS.

**Vad vet vi?** Hajks Express-backend registrerar parsers med en **möjlig** väg till Brotli-avkodning i den körande Node.js-miljön. Hajks backenddistributioner och statiska `hajk-simple` har **olika runtimeförutsättningar**.

**Vad vet vi inte?** Ingen Hajk-specifik CVE-utlösning eller RCE är styrkt här. Vilken väg som faktiskt är nåbar beror på installerad distribution, Node-version, proxy och parserkonfiguration. Statiska paket kan dessutom omges av andra servrar eller tillägg.

**Vad bör göras ändå?** Identifiera om en aktiv Node-backend körs och **vilken Node.js-runtime den faktiskt använder**. För ZIP-installation: uppdatera driftmiljöns Node.js. För full officiell Docker: hämta en image med rättad Node-kod, **återskapa den körande containern** och verifiera versionen; nattliga byggen installerar inte sig själva. Avvakta inte Hajk-specifik exploiteringsreproduktion. För `hajk-simple` undersöks eventuella separata serverkomponenter i stället. [Daterad rättningsstatus](patchkedjan.md).

**Tekniska belägg och vidare läsning:** [README](../README.md) · [HTTP-avkodning](http-angreppsvagar.md) · [Patchkedja](patchkedjan.md)

## Varför kan en Node.js-backend behöva kontrolleras?

```text
Inkommande HTTP-begäran med Content-Encoding: br
  → proxy/ingång till Hajk Backend
  → Express.json()/urlencoded()/text(), om parsern matchar
  → Node.js zlib → medföljande eller distribuerad Brotli
```

Hajks [backend-serverkod](https://github.com/hajkmap/Hajk/blob/master/apps/backend/server/common/server.js) registrerar `Express.json()`, `Express.urlencoded()` och `Express.text()` i middlewarekedjan. [Express-projektet](https://expressjs.com/en/blog/2026-10-31-security-releases/#a-note-on-brotli-decompression-cve-2026-111497) dokumenterar att de kan behandla Brotli-komprimerade begärandekroppar. Den verkliga exponeringen påverkas av `Content-Type`, body-parsers konfiguration, reverse proxy och den runtime som faktiskt körs.

Det här visar hur **en generell Express/Node.js-kodväg** kan bli relevant i vissa Hajk-backends. Det bevisar **inte** ett fel i Hajks applikationskod, CVE-utlösning i ett visst Hajk-bygge, en exploaterbar endpoint eller RCE. Även eventuella externa HTTP-anrop från backend måste analyseras **separat**: att middleware avkodar inkommande begäranden betyder inte att en annan HTTP-klient avkodar hämtade svar med samma inställning.

## Uppdatera Node.js där den faktiskt körs

| Hajk-distribution | Kör Node.js-backend? | Vad behöver rättas? |
| --- | --- | --- |
| `hajk-vX.Y.Z-nodejs.zip` | **Ja**, om backend startas | Node.js-runtime installeras **separat** och måste uppdateras i driftmiljön |
| `hajk-vX.Y.Z-simple.zip` | **Nej**, statiska klientfiler | Ingen Node.js-backend följer med denna ZIP; externa webbservrar och tillägg bedöms separat |
| `ghcr.io/hajkmap/hajk` (full Docker) | **Ja** | Välj image med rättad Node.js/OS-bas, hämta och **återskapa** containern |
| `ghcr.io/hajkmap/hajk-simple` | **Nej**, nginx och statiska filer | Node.js används vid byggning, men **inte i den levererade driftavbilden**, som kör nginx och statiska klientfiler |

[Hajk releaseguide](https://github.com/hajkmap/Hajk/blob/master/docs/releases.md) · [Hajks säkerhetspolicy](https://github.com/hajkmap/Hajk/blob/master/SECURITY.md).

**Officiell Docker – automatiskt bygge är inte automatisk driftpatch:** Hajks `nightly-master` byggs **varje natt** från senaste release med en aktuell Node.js-/OS-grundavbild. Hajk-projektet väljer denna bas och publicerar avbilden, men **uppdaterar inte organisationens redan körande containrar**. Driftansvarig behöver hämta en ny image (exempelvis `docker pull ghcr.io/hajkmap/hajk:nightly-master`) och återskapa containern, manuellt eller med ett övervakat uppdateringsflöde. En ordinarie release-image byggs en gång och får inte ny Node-/OS-bas mellan releaser. En kort versionstagg (som `4.3`) flyttas vid en *patchrelease*, inte vid varje basuppdatering. [Hajks releaseguide](https://github.com/hajkmap/Hajk/blob/master/docs/releases.md#docker-images) · [Hajks Docker-instruktioner](https://github.com/hajkmap/Hajk/blob/master/Docker/README.md#keeping-the-base-image-up-to-date) · [nattligt byggflöde](https://github.com/hajkmap/Hajk/blob/master/.github/workflows/nightly-master.yml).

**Låsta npm-beroenden:** Nattliga ombyggen tar in uppdaterad bas, **inte automatiskt nya npm-paket**. De installeras enligt Hajks låsfiler och rättas genom projektets egna beroende- och releasearbete. Som alternativ till `nightly-master` beskriver Hajk hur driftansvarig kan **bygga en egen image från vald releasetagg**, med `docker build --pull --no-cache`, och sedan ersätta containern. Varken taggnamn eller en lyckad `docker pull` verifierar i sig patchstatus: kontrollera **image-digest och faktiskt körande Node-version**. [Säkerhetspolicy – officiella avbilder](https://github.com/hajkmap/Hajk/blob/master/SECURITY.md#official-docker-images) · [Docker-instruktioner](https://github.com/hajkmap/Hajk/blob/master/Docker/README.md#building-the-images-yourself).

**Node.js:** För de uppströms Node-grenar som berörs finns första rättade versionerna 22.23.4, 24.22.1 och 26.12.1 (24.22.2 eller senare rekommenderas framför regressionspåverkade 24.22.1). Äldre osupporterade Node-grenar får inte dessa rättningar från Node-projektet. [Node.js säkerhetsrelease](https://nodejs.org/en/blog/vulnerability/october-2026-security-releases).

**Hajks supportsnitt:** Enligt [Hajks säkerhetspolicy](https://github.com/hajkmap/Hajk/blob/master/SECURITY.md#supported-versions) får bara den senaste releasen produktens säkerhetsrättningar; projektet kan publicera patchreleaser för den senaste versionsserien. Äldre versioner behöver uppgraderas. **Detta ska inte blandas ihop med Node.js-/OS-basens nattliga uppdateringar:** en äldre ZIP- eller Docker-installation får inte dem automatiskt.

## Samlad slutsats – olika driftsättningar, samma underliggande CVE

**Hajk med aktiv Node.js-backend:** Om den körande Node-runtimen innehåller berörd Brotli-kod behöver den rättas. En inkommande `br`-begäran som når en Express-parser är en *möjlig* exponeringsväg. Hur nåbar vägen är, och vilka eventuella följder den har i en viss driftsättning, är inte verifierat här. **Egen CVE-reproduktion krävs inte före en relevant säkerhetsuppdatering.**

**Statiska `hajk-simple`:** Den beskrivna statiska distributionen innehåller **inte denna Node-backend**. Därmed finns inte just denna backendbaserade Node-väg i paketet. Det är inte ett generellt säkerhetsintyg för eventuella proxyer, webbservrar, klientbibliotek eller separata tillägg.

**Kvar att kontrollera:** Faktiskt körande Node-version eller Docker-image, eventuell proxy och parsers, samt om det finns andra komponenter som avkodar inkommande begäranden eller hämtade svar. Det är **Node.js/Brotli i driftkedjan**, inte Hajks produktnamn, som avgör patchbehovet för denna väg.

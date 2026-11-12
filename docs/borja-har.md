# Från första GIS-frågan till rätt ansvarig

**CVE-2026-111497 · Källunderlag: 8 november 2026.** Se [README](../README.md) för produktstatus och rättningar. Här finns stöd för att avgränsa frågan och lämna den till GIS, IT eller leverantören.

## När räcker en vanlig användarkontroll?

Den som öppnar kartor, kontrollerar attribut eller arbetar med egna shapefiler behöver normalt inte undersöka DLL:er eller försöka utlösa CVE:n. Använd godkänd, uppdaterad programvara och följ rutinerna för filer och datakällor. Om programmet hanteras centralt: lämna uppdateringsfrågan till **GIS/IT**.

Lokalt arbete kan minska vissa exponeringsvägar, men även en fil som ligger på den egna datorn kan ha hämtats från en annan källa. Att inga automatiska externa hämtningar har hittats bevisar inte att installationen är säker. Följ fortfarande leverantörens uppdateringsråd. [QGIS](qgis.md) · [Filer](fil-angreppsvagar.md).

## När behöver frågan lämnas vidare?

| Situation | Det räcker att identifiera | Lämna vidare till |
| --- | --- | --- |
| Du tar över äldre GIS, exempelvis **ArcMap**, eller en blandad filsamling | Vilka importer, tjänster och jobb som fortfarande **körs**, inte varje äldre fil. | GIS-förvaltare och produktleverantör. |
| Du använder Origo och vet inte vad som körs bakom kartan | Finns **Origo Server** med Node.js som backend, utöver webbläsarklienten? Vem ansvarar för körande Node-runtime och eventuell proxy? | GIS-förvaltare och IT/drift. |
| Du har ett automatiserat **FME**- eller integrationsjobb | Vilken arbetsyta, datakälla, Engine och tidsplan? | Integrationsansvarig och IT/leverantör. |
| En publik backend tar emot HTTP eller hämtar externa data | Vilken tjänst, vilken begäran eller filtyp och vem som driftar den? | IT/säkerhet och leverantör. |

**Om du saknar driftansvar:** Försök inte utlösa CVE:n på servern. Förmedla vad du vet och vad som fortfarande är okänt. Beskriv problemet som en **möjlig eller bekräftad påverkan via berörd Brotli-kod**, med angiven bevisnivå, inte som att leverantören orsakat felet eller att produkten är garanterat säker eller exploaterbar. Känsliga uppgifter om interna system, tokens och personuppgifter hör inte hemma i publika ärenden.

## Kort fråga till IT eller leverantören

> Vi använder **[produkt och version]** för **[aktuell funktion/import/tjänst]**. Vi behöver kontrollera både data som systemet tar emot och data som det självt hämtar. Är CVE-2026-111497 relevant för avkodaren i denna version? Vilken rättning rekommenderas för vår plattform, och hur verifierar vi den efter uppdatering? Om påverkan är okänd, vad behöver undersökas?

För en förvaltare som behöver kartlägga mer: följ **data från källa till faktisk avkodare** i [dataflöde till patch](dataflode-till-patch.md). För teknisk prioritering av process och behörigheter: [riskkedja](riskkedja.md). Vid okänd produkt: [inventeringsmetod](ny-programvara.md).

## Frivilligt inventeringsstöd för tekniskt ansvariga

Om den ansvarige behöver undersöka paketerad kod kan [Trail of Bits Brotli Checker](https://github.com/trailofbits/brotli-checker) användas som ett **icke-exploaterande inventeringsstöd** för källkod, beroenden och vissa binärer. Verktyget kan inte analysera alla filformat eller paketeringar, och vissa filer kan vara otillgängliga. En signaturträff visar inte att avkodaren är nåbar eller att CVE:n kan utlösas. En skanning utan träffar är inte ett säkerhetsintyg. Täckningsluckor är inte ytterligare sårbarheter.

Vanliga GIS-användare behöver inte köra skannern. Följ leverantörens uppdateringar och byt inte ut programmets DLL-filer manuellt.

## Avsluta i en handlingsbar bedömning

- **Rätta skyndsamt:** aktiv, berörd och nåbar avkodare har identifierats; ange uppdateringsansvarig och verifiering.
- **Undersök först:** den viktiga frågan är fortfarande vilken process, avkodare eller version som används; ange ägare och vad som återstår.
- **Ordinarie uppdatering:** ingen aktiv riskkedja har påvisats *för det avgränsade arbetssättet*; det är ingen garanti för allmän säkerhet.
- **Verifiera rättning:** patch finns eller har installerats men faktisk driftversion är ännu inte kontrollerad.

**En rekommenderad rättning ska inte invänta en egen CVE-reproduktion.** Bedömningen avgör arbetsordning, kompletterande skydd och nästa fråga, inte om berörd programvara bör uppdateras. För versionsbundna besked och fullständig helhetsbild, gå till [README](../README.md).

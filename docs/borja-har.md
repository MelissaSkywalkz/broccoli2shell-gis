# Från första GIS-frågan till rätt ansvarig

**CVE-2026-111497 · Källunderlag: 8 november 2026.** Den **sammanhållna vägledningen, produktstatusen och patchöversikten finns i [README](../README.md)**. Här finns bara kompletterande hjälp för den som behöver avgränsa en fråga eller lämna den vidare till GIS, IT eller leverantör.

## När räcker en vanlig användarkontroll?

En GIS-användare som öppnar kartor, kontrollerar attribut eller arbetar med en egen **Shapefil** behöver normalt **inte** undersöka DLL:er, avkodare eller CVE-utlösning. Använd godkänd, uppdaterad programvara och följ rutinerna för filer och datakällor. Om programmet hanteras centralt: lämna uppdateringsfrågan till **GIS/IT**.

Lokalt arbete kan minska vissa exponeringsvägar, men även en fil som ligger på den egna datorn kan ha hämtats från en annan källa. Ingen dokumenterad automatisk extern hämtning betyder **inte** ”garanterat säker”. En rekommenderad rättning är fortfarande relevant. [QGIS](qgis.md) · [Filer](fil-angreppsvagar.md).

## När behöver frågan lämnas vidare?

| Situation | Det räcker att identifiera | Lämna vidare till |
| --- | --- | --- |
| Du tar över äldre GIS, exempelvis **ArcMap**, eller en blandad filsamling | Vilka importer, tjänster och jobb som fortfarande **körs** – inte varje äldre fil. | GIS-förvaltare och produktleverantör. |
| Du använder Origo och vet inte vad som körs bakom kartan | Finns **Origo Server** med Node.js som backend, utöver webbläsarklienten? Vem ansvarar för körande Node-runtime och eventuell proxy? | GIS-förvaltare och IT/drift. |
| Du har ett automatiserat **FME**- eller integrationsjobb | Vilken arbetsyta, datakälla, Engine och tidsplan? | Integrationsansvarig och IT/leverantör. |
| En publik backend tar emot HTTP eller hämtar externa data | Vilken tjänst, vilken begäran eller filtyp och vem som driftar den? | IT/säkerhet och leverantör. |

**Stanna när ansvaret tar slut.** Om du inte har behörighet eller insyn i servern behöver du inte själv försöka framkalla CVE:n. Förmedla vad du vet och vad som fortfarande är **okänt**. Beskriv problemet som en **möjlig eller bekräftad påverkan via berörd Brotli-kod**, med angiven bevisnivå – inte som att leverantören orsakat felet eller att produkten är garanterat säker eller exploaterbar. Känsliga uppgifter om interna system, tokens och personuppgifter hör inte hemma i publika ärenden.

## Kort fråga till IT eller leverantören

> Vi använder **[produkt och version]** för **[aktuell funktion/import/tjänst]**. Är CVE-2026-111497 relevant för den avkodare som används i denna version? Vilken rättning rekommenderas för vår plattform, och hur kontrollerar vi att den används efter uppdatering? Om påverkan är okänd: vilken uppgift behövs för att bedöma den?

För en förvaltare som behöver kartlägga mer: följ **data från källa till faktisk avkodare** i [dataflöde till patch](dataflode-till-patch.md). För teknisk prioritering av process och behörigheter: [riskkedja](riskkedja.md). Vid okänd produkt: [inventeringsmetod](ny-programvara.md).

## Avsluta i en handlingsbar bedömning

- **Rätta skyndsamt:** aktiv, berörd och nåbar avkodare har identifierats; ange uppdateringsansvarig och verifiering.
- **Undersök först:** den viktiga frågan är fortfarande vilken process, avkodare eller version som används; ange ägare och vad som återstår.
- **Ordinarie uppdatering:** ingen aktiv riskkedja har påvisats *för det avgränsade arbetssättet*; det är ingen garanti för allmän säkerhet.
- **Verifiera rättning:** patch finns eller har installerats men faktisk driftversion är ännu inte kontrollerad.

**En rekommenderad rättning ska inte invänta en egen CVE-reproduktion.** Bedömningen avgör arbetsordning, kompletterande skydd och nästa fråga – inte om berörd programvara bör uppdateras. För versionsbundna besked och fullständig helhetsbild, gå till [README](../README.md).

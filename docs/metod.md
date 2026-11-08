# Metod – börja i CVE-felet och följ angriparstyrd indata till faktisk avkodare

**CVE-2026-111497 · Metod och bevisgränser · 8 november 2026**  
[Till CVE-översikten](../README.md) · [Felmekanismen](cve-mekanism.md) · [Nytt program](ny-programvara.md)

## Analysordningen är CVE → triggerkrav → ingress → beroenden → program → patch

Börja med sårbarhetsmekanismen och vilka bytes som **kan** nå den. Först därefter identifierar vi vilka produktfunktioner som exponerar en sådan avkodningsväg. Produktnamn är en slutsats av grafen, inte utgångspunkten. **Tillskriv grundfelet rätt komponent:** ett projekt kan ha en *påverkad produkt genom ett beroende* utan att dess utvecklare har orsakat CVE-felet. Återge leverantörsbekräftad påverkan tydligt, men hitta inte på produktspecifika kodbrister eller exploiterbarhet. Håll samtidigt fast vid rekommenderad patchning. Analysera också [angreppsvägar via HTTP](http-angreppsvagar.md), [filingångar](fil-angreppsvagar.md), [konsekvenser](konsekvenser.md) och [försvarslager](skydd.md). Skilj en sårbar avkodningsimplementation från attackerbar dataingång, och en sårbarhetsutlösning från visad exploatering.

## Bedöm produkten, vägen och bibliotekskopian separat

För varje väg redovisas samma åtta frågor:

1. **Indata och förtroendegräns:** Vem kan påverka den komprimerade filen, HTTP-kroppen eller svaret? Måste en GIS-användare öppna något, eller behandlas det automatiskt?
2. **Komponentkedja:** Vilka parser-, nätverks- och runtime-komponenter hanterar indatan innan den når Brotli? Hänvisa till versionslåst kod, dokumentation eller test.
3. **Berörd avkodare:** Är det Google Brotli, en inbyggd kopia, Brotli4j eller en annan implementation? Exakt bygg, plattform och dynamisk/statisk länkning spelar roll.
4. **Nödvändiga villkor:** Krävs aktiverad avkodning, rätt `Content-Type`, en Brotli-komprimerad Parquet-sida, en viss container eller annan konfiguration?
5. **Vad är belagt?** Separera förekomst, laddat bibliotek, avkodning, CVE-specifik utlösning och säkerhetseffekt.
6. **Konsekvens och skydd:** Vad händer om angriparen får en OOB-skrivning? Vad är enbart tänkbart, vad är visat? Vilka lager begränsar inträde eller följdskada?
7. **Patch:** Var finns rättningen, vilken version eller produktpatch gäller och når den den faktiskt körande avkodaren?
8. **Öppna frågor:** Vilken enskild länk i kedjan är ännu inte styrkt? Okänt betyder inte säkert eller sårbart.

## Riskkedja: vem kan nå processen, och vad kan processen nå?

Komplettera beroendegrafen och dataflödet med en **villkorad konsekvensmodell**: [angreppsväg → påverkad/vid RCE komprometterad process → åtkomliga resurser → möjlig skada → nödvändiga skydd](riskkedja.md). Processkrasch kräver inte övertagande; informationsröjande och RCE kräver separat stöd. En personuppgiftsintegration eller tillgänglig databas innebär därför inte automatiskt att något har läckt. Där faktisk processåtkomst är okänd ska påverkan anges som *okänd*, inte upphöjas till bevisad skada.

## Bevisnivåer

| Nivå | Minsta stöd | Slutsats som **inte** följer automatiskt |
| --- | --- | --- |
| **Leverantörsbekräftad påverkan** | Leverantörsbesked för angiven produkt och version | Att alla möjliga indataingångar eller RCE är visade |
| **Dokumenterad beroendekedja** | Byggskript, SBOM, källkod eller paketmetadata | Att biblioteket laddas eller avkodar i just den körande installationen |
| **Verifierad avkodningsväg** | Daterad körning eller oberoende produktdokumentation som visar data till den namngivna avkodaren | Att just CVE-felet kan utlösas |
| **CVE-specifik utlösning** | Avgränsad, verifierad felaktig indata når det berörda felvillkoret | Att minneskorruption eller godtycklig kodkörning demonstrerats i en oförändrad produkt |
| **Demonstrerad säkerhetseffekt** | Observerad krasch, informationsläcka, minneskorruption eller kodkörning i en specificerad byggning | Att samma effekt gäller i andra byggen, med annan konfiguration eller andra skydd |
| **Outredd väg** | Relevant dokumenterat stöd för en möjlig ingång men otillräckligt stöd för hela kedjan | Varken “ej påverkad” eller “bekräftat exploaterbar” |

En produkt kan ha **leverantörsbekräftad påverkan och samtidigt okänd ingång**. Den kan också innehålla berörd kod medan en viss dataväg inte avkodar Brotli. Blanda inte dessa påståenden.

**Hur slutsatser ska läsas, citeras och sammanfattas:** Avsaknad av en *identifierad* angreppsväg är inte ett bevis att programmet är generellt säkert. En berörd avkodare är inte bevis för att produkten kan exploateras, än mindre för RCE. En arbetsmiljö med lägre exponering ska ändå följa relevanta leverantörsrättningar. Om version, avkodare eller angreppsväg är okänd ska det stå **okänd – kontroll behövs**, inte ”inte påverkad”. Dessa gränser gäller också om någon bara läser en tabellrad eller återger texten via en AI-tjänst.

## Välj nästa undersökning – inte bara nästa patch

**Skyddsprioritet** handlar om vilken aktiv riskkedja som bör begränsas eller rättas först. **Undersökningsprioritet** handlar om vilken *obesvarad fråga* som är mest värdefull att kontrollera härnäst. De kan peka mot olika system. En redan rättad server kan vara relevant för forskning i ett separat labb utan att kräva en akut driftåtgärd; en känslig integration kan behöva inventeras och uppdateras även när CVE-utlösning inte har demonstrerats i produkten.

Utgå från tre frågor för varje tänkbar undersökning:

1. **Vad vet vi redan?** Ange högsta belagda nivå ovan och om påverkan, exponering och rättningsstatus är kända eller okända.
2. **Vilken avgörande fråga återstår?** Välj den *första saknade länken* som kan ändra bedömningen – exempelvis om indata når en sårbar avkodare, om rättad kod faktiskt körs eller vad processen kan komma åt.
3. **Vilken säker kontroll kan besvara den?** Välj först dokumentation, SBOM, paket-/runtimeinventering, konfiguration eller legitim testindata i ett isolerat, behörigt labb. Ett test som visar avkodning bevisar inte CVE-utlösning. Testa inte felutlösande indata mot publika eller produktionssatta system.

| Exempel | Vad är nästa meningsfulla kontroll? | Vad ska **inte** antas? |
| --- | --- | --- |
| **QGIS:** berörd avkodare och en avkodningsväg har dokumenterats | Avgör vilken bevislänk som fortfarande saknas och om frågan bör tas vidare konfidentiellt med underhållare | Att nåbar avkodning bevisar RCE i QGIS |
| **Origo Server:** källkod visar en möjlig Express/Node-väg | Kontrollera med giltig Brotli-indata om den egna isolerade labbversionen faktiskt använder avkodaren | Att alla Origo-installationer är nåbara eller opatchade |
| **FME:** leverantören har rapporterat påverkan | Inventera aktiva hämtningar, faktisk produkt-/runtimeversion och processens behörigheter; verifiera rättningsbesked | Att en viss integration har läckt data eller att varje jobb avkodar Brotli |
| **GeoServer eller annat okänt program:** ingen generell direkt påverkan styrkt | Börja med faktisk komponent- och beroendeinventering innan någon avkodnings- eller exploateringskontroll | Att frånvaro från produktlistor betyder säkert eller sårbart |

**Avsluta med ett nästa steg, inte ett påhittat slutbesked.** Om ingen relevant avkodare kan beläggas: dokumentera vad som granskats och vad som är okänt. Om leverantören rekommenderar en rättning för den berörda produktversionen: genomför den utan krav på egen reproduktion. Om en aktiv sårbar väg dessutom bekräftas: höj prioriteten för skydd och verifiering före mer forskning. Håll privata testresultat och känsliga miljöuppgifter utanför det publika repot. [Riskkedja och verksamhetskonsekvenser](riskkedja.md) · [Prioritering i drift](skydd.md#prioritera-aktiva-riskkedjor).

## Patchstatus är en separat dimension

- **Rättning tillgänglig i upstream:** källkod eller versionsutgåva finns, men nedströmsbygget kan fortfarande innehålla gammal kod.
- **Rättning tillgänglig hos leverantör/distribution:** rätt version, produktpatch eller image finns att installera; kontrollera plattform/version.
- **Under arbete / oklart:** publicerat besked saknas, motsägelse finns eller rättat byggnummer är inte verifierat.
- **Verifierat i drift:** bara när den faktiska körmiljön, processens laddade bibliotek eller den körande containeravbilden har kontrollerats **efter** uppdatering. Repoets författare kan inte intyga detta för andras installationer.

**Leverantörsrekommenderad patch kräver inte egen CVE-reproduktion.** Kontroll av nåbarhet styr hur brådskande en viss riskkedja ska undersökas och vilka extra skydd som behövs; använd inte avsaknad av ett produktspecifikt exploitbevis som skäl att ignorera ett säkerhetsbesked för en berörd produktversion.

En ändrad SBOM eller ett uppdaterat paket på en annan maskin bevisar inte rättning i den körande processen. Särskilt viktigt är system-Brotli kontra medföljande/embedded Brotli.

## Källhierarki och skilda riskbegrepp

1. Officiellt säkerhetsbesked för aktuell produkt och version.
2. Versionslåst källkod, byggrecept, paket eller dokumenterad runtime.
3. Avgränsade och reproducerbara undersökningar där bevisgränsen framgår.
4. Sekundära analyser som ger sammanhang, inte ersätter produktspecifik evidens.

**SBOM** anger vad som finns i en produkt; en **beroendegraf** anger vem som använder vad; en **dataflödesmodell/hotmodell** anger hur angriparpåverkad indata skulle kunna nå en funktion. **VEX** används för produktrelaterade påverkansbedömningar. Inget av dessa bevisar i sig CVE-utlösning.

Detta repo beskriver primärt **spridning av ett känt fel genom legitima programvaruberoenden**, inte ett bevisat intrång i paketbyggnad eller leveranskedja. [OWASP threat modeling](https://cheatsheetseries.owasp.org/cheatsheets/Threat_Modeling_Cheat_Sheet.html) · [OWASP SBOM/beroendegraf](https://cheatsheetseries.owasp.org/cheatsheets/Dependency_Graph_SBOM_Cheat_Sheet.html) · [CISA VEX](https://www.cisa.gov/sites/default/files/2023-04/minimum-requirements-for-vex-508c.pdf).

## Publiceringsgräns

Publicera **vad användaren behöver veta för att bedöma och rätta sin installation**, men inte nya, ännu osamordnade reproduktionsdetaljer, felutlösande bytes, exploiteringsskript, minnesadresser eller råloggar från privata produkttester. Rapportera nya produktspecifika säkerhetsfynd konfidentiellt enligt underhållarens policy. Källor och produktstatus uppdateras när nya leverantörsbesked gör skillnad.

## Skilj fyra sorters angrepp från varandra

1. **Angriparstyrd fil eller datatjänst:** datakällans innehåll är kontrollerat eller komprometterat; CVE:n kan bara bli relevant om rätt avkodning verkligen sker.
2. **Nätverksangrepp mot okrypterad HTTP:** en mellanhand med trafikåtkomst kan ändra innehåll före avkodning; korrekt verifierad TLS bryter normalt *denna* väg.
3. **Angrepp mot en publikt åtkomlig parser:** en klient kan lämna komprimerad indata direkt, om tjänsten accepterar och avkodar den innan andra skydd stoppar den.
4. **Social engineering:** någon förmås lägga till en tjänst eller öppna en fil; det är hur data levereras, inte ett separat bevis för att CVE:n utlöses.

Även ett legitimt program kan innehålla sårbar kod utan att leveranskedjan har manipulerats. "Supply-chain attack" i betydelsen illvilligt ändrat paket ska inte användas som synonym för att en sårbar transitive dependency distribueras. [OWASP Supply Chain Security](https://cheatsheetseries.owasp.org/cheatsheets/Software_Supply_Chain_Security_Cheat_Sheet.html).

## Bedöm skydd och verifiering separat

Att källan är betrodd, TLS-anslutningen giltig, filen skannad eller en WAF installerad är **en kontroll**, inte ett generellt bevis att en sårbar avkodare är onåbar. Beskriv vilken dataingång kontrollen faktiskt bryter. Uppdaterad upstream-kod, uppdaterat distributionspaket och **faktisk rättad drift** redovisas var för sig.

Ett dokumenttest som kontrollerar att "HTTP" förekommer i texten är inte en säkerhetstest av en tjänst. En osamordnad produktutlösning i ett privat labb bör inte publiceras med felutlösande indata, råloggar eller instruktioner innan ansvariga kontaktats.

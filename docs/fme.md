# FME: synthetic test fixture
**Källkontroll: Safes CVE-specifika artikel läst den 7 och 9 november 2026. Den 9 november granskades artikeln via inloggad webbläsare. Vid anonym kontroll den 8 november krävdes inloggning. Övriga nedladdnings- och changelogsidor kontrollerades den 8 november.**
**Vad vet vi?** Safe bekräftar påverkan i FME Form, Engine, Flow och Hosted.
Safe anger att rättade byggen för FME 2026.1 till 2026.3 förbereds. **Inget CVE-rättat FME-byggnummer har bekräftats i det kontrollerade underlaget.** Det är ett daterat besked, inte en garanti om senare versioner.
**Källa:** [Safes CVE-specifika artikel](https://support.safe.com/hc/en-us/articles/50040725669517-Brotli-Decoder-CVE-2026-111497-Is-FME-Impacted) ger produktbedömningen och rekommendationerna ovan. Den öppna novemberartikeln om andra säkerhetsbrister styrker inte ett visst CVE-rättat Brotli-byggnummer.
**Bevis:** Vägarna återges ur Safes besked. Det är leverantörsinformation, inte en egen reproduktion eller demonstration av RCE i FME.
**Statusen nedan gäller Safes daterade besked, inte vilken version som körs i en viss installation.**
| **Windows / macOS** | Uppdaterad FME-utgåva som ersätter medföljande Brotli | Safe anger att rättade 2026.1 till 2026.3-byggen förbereds. Inget rättat byggnummer bekräftat här. |
| **DuckDB i FME** | DuckDB-rättningen måste ingå i den FME-utgåva som levererar biblioteket | Safe anger separat arbete med DuckDB. Att en kodändring är sammanslagen innebär inte att den nått en FME-release. |
| **Flow Hosted** | Plattformens Ubuntu-säkerhetsuppdatering och relevant omstart; DuckDB separat | Safe anger automatiska säkerhetsuppdateringar för system-Brotli, men en omstart krävs för att använda rättad kod. DuckDB påverkas inte av systemuppdateringen. |
**Versionsskillnad i Safes artikel:** Sammanfattningstabellen anger påverkan till och med **FME 2026.3**, medan detaljavsnittet för Windows/macOS anger medföljande Brotli `1.1.0` till och med **FME 2026.4**. Skillnaden är inte förklarad i artikeln. Utgå inte från att 2026.4 är rättad eller opåverkad enbart genom att läsa tabellen.
**Öppna frågor:** Vilken FME-byggning rättar den *medföljande* avkodaren?

# Brotli2Shell | Origo Server
CVE-2026-111497 | Kontrollerad 2026-11-04

Kontrollera vilken Node.js-version Origo Server-tjänsten faktiskt använder. `node -v` visar versionen i aktuell terminal, inte nödvändigtvis tjänstens.

Korrigerade Node.js-versioner är 22.23.4, 24.22.1 och 26.12.1 inom respektive versionsgren.

Planera uppgradering av berörd Node.js-miljö efter kompatibilitetskontroll och testa tjänsten efter omstart. Origo-specifik exploaterbarhet är inte verifierad.

Källa: https://www.cisecurity.org/advisory/a-vulnerability-in-brotli-library-could-allow-for-remote-code-execution_2026-120

# Filer, GeoParquet och social engineering: när når importdata avkodaren?

**Kontrollerat: 8 november 2026.** [Översikt](../README.md) · [Skydd](skydd.md)

## Grundmekanismen

```text
Extern fil / delad arbetsyta / bilaga / uppladdning / fjärrfil
  → GIS-import eller automatiskt jobb
  → formatläsare med komprimerad del
  → Arrow, DuckDB, libjxl, annan native-parser
  → berörd Brotli-avkodare
```

En fil behöver **inte** ha filändelsen `.br`. Brotli kan ligga inne i andra format, till exempel som kompression av datasidor i Parquet eller som metadata i JPEG XL. Yttre filstruktur, metadata eller en lyckad filtypskontroll garanterar inte att en inbäddad komprimerad del är säker att avkoda. [GDAL Parquet](https://gdal.org/en/stable/drivers/vector/parquet.html) · [JFrog om libjxl](https://research.jfrog.com/post/brotli2shell-beyond-node2shell-and-http2shell/).

## Varför kan en korrupt eller manipulerad GeoParquet-fil spela roll?

Parquet låter kolumndatasidor lagras med olika codec. En sida kan använda **BROTLI**, medan andra sidor använder andra codec. När en relevant läsare behöver läsa den sidan överlämnas dess komprimerade byte till Brotli. CVE:n kan vara relevant **först om den sårbara avkodaren används och den särskilda felaktiga strömmen når felvillkoret**. Att öppna godtycklig GeoParquet eller att välja en fil med en annan codec är därför inte automatiskt en angreppsväg.

Den dokumenterade kedjan i vissa QGIS-/OSGeo4W-byggen är **GDAL/OGR → Arrow C++/Parquet → system-Brotli**. Det beror på byggkonfiguration. Ett annat Arrow-/PyArrow-paket kan i stället leverera en inbyggd kopia. [OSGeo4W Arrow-recept](https://github.com/jef-n/OSGeo4W/blob/c8514a953de4b50a6dd4887fb02a874c8d74eec9/src/arrow-cpp/osgeo4w/package.sh) · [Arrow 26.0.1](https://arrow.apache.org/blog/2026/11/03/26.0.1-release/). [QGIS-tillämpning](qgis.md).

## Hur kan en sådan fil nå en organisation?

- **Social engineering:** en GIS-handläggare eller analytiker uppmanas öppna en dataleverans, bilaga eller nedladdad geodatamängd. Legitima filtyper kan bära farligt innehåll.
- **Betrodd dataleverantör:** leverantörens källa eller konto kan vara komprometterat, eller deras egna importerade data kan komma från en okontrollerad tredje part. En känd avsändare är ingen garanti för att avkodaren inte får felaktig input.
- **Automatiska importflöden:** FME, Python, batchjobb eller GIS-server importerar data enligt ett schema, utan att en människa granskar filen.
- **Uppladdning till tjänst:** en extern användare kan skicka data till ett system som behandlar filens komprimerade delar på servern; exponering beror på om filtypen accepteras och *avkodningen faktiskt sker*.

Inget av dessa exempel är bevis för att en viss mottagare har exploaterats. De visar möjliga **leveransvägar till avkodningen**.

## Vilka försvar hjälper?

**Patcha parserns faktiska Brotli-kopia** i första hand. Begränsa automatiska importer från obekräftade källor, validera rimlig filtyp och storlek, använd AV/sandbox när lämpligt, separera karantän och bearbetning, kör med låg behörighet och övervaka krascher. **Filändelse, magic bytes, virusskanning och "filen gick att öppna" är inte garantier mot en ny parserbugg.** [OWASP File Upload](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html).

En fil som ligger lokalt kan fortfarande vara **angriparpåverkad**. Det är relevant att skilja mellan ett program som bara visar metadata och ett som faktiskt dekomprimerar datasidor. Samma system kan ha flera avkodare och därmed flera patchkedjor. [Skydd](skydd.md) · [Hur man utreder en ny produkt](ny-programvara.md).

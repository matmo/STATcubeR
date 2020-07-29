# STATcubeR

R Interface für die STATcube API

## Installation

Falls die Portal-Credentials am R Server hinterlegt sind, kann über 
`install_statbucket()` installiert werden.

``` r
rinstSTAT::install_statbucket("METH/STATcubeR")
library(STATcubeR)
```

Sonst ist auch eine Installation mit `git clone`/`R CMD INSTALL` möglich

```
git clone ssh://git@gitrepo:7999/METH/STATcubeR.git
R CMD INSTALL STATcubeR
```

## Verwendung

Zur Verwendung diese Paketes sind folgende Schritte notwendig

* (einmalig) Hinzufügen des API Keys zum authSTAT-Vault
* Herunterladen einer API Abfrage von STATcube. (JSON-Format)
* Absenden der Abfrage über den R Server
* Konvertieren der API-Response in ein typisches R Datenformat

### Hinzufügen des API Keys

Zum verwenden dieses Pakets muss einmalig ein API-Schlüssel für STATcube
auf den R Server geladen werden. Rufen Sie die Funktion
`sc_browse_preferences()` auf. Nun öffnet sich ein Browserfenster in
dem der Schlüssel sichtbar ist. Eventuell muss davor Windows-User und
Windows-Passwort eingegeben werden. Kopieren Sie den Schlüssel in die
Zwischenablage.

```r
sc_browse_preferences()
```

Rufen Sie nun die Funktion `statcube_token_set()` auf. Ersetzen Sie dabei
`"XXXX"` durch den vorhin kopierten Token.

```r
sc_token_set("XXXX")
#> STATcube Key wurde erfolgreich getestet und im Vault hinterlegt
```

### Herunterladen der API Abfrage

Laden sie eine "Open Data API Abfrage" über die STATcube GUI herunter. Erstellen
Sie hierzu eine STATcube Tabelle und wählen Sie "API Abfrage" als
Download-Option.

![](man/figures/download_json.png)

Nun wird eine json-Datei auf dem Windows-System abgelegt

### Absenden der Abfrage

Geben Sie den Pfad zu dem heruntergeladenen JSON-File in
der Funktion `sc_get_response()` ein, um die API Abfrage auszuführen.

``` r
my_response <- sc_get_response("pfad/zu/api_abfrage.json")
```

Alternativ kann mit der Funktion `upload_json()` die json-Datei über einen
Upload-Dialog angegeben werden.

```r
my_response <- upload_json()
```

Das Objekt `my_response` beinhaltet die "rohe" API-Response. Die Print-Methode
gibt einen Einblick in die Abfrage.

```r
json_pfad <- sc_example("bev_seit_1982.json")
my_response <- sc_get_response(json_pfad)
my_response
#> Objekt der Klasse STATcube_response
#>
#> Datenbank:       Bevölkerung zu Jahresbeginn ab 1982 
#> Werte:           Fallzahl 
#> Dimensionen:     Jahr, Bundesland, Geburtsland 
#>
#> Abfrage:         2020-07-29 11:40:43 
#> STATcubeR:       0.1.0
```

### Umwandeln in typische R-Datenformate

Um die importierten Daten in R zu nutzen kann `as.data.frame()` oder
`as.array()` verwendet werden.

```r
as.array(my_response)
as.data.frame(my_response)
```

Im Falle von `as.data.frame()` wird ein tidy Datensatz zurückgegeben, der jede
Dimension des Cubes als Spalte enthält. Außerdem gibt es eine Spalte pro
Wert. Beispiel:

```r
library(dplyr)
set.seed(1234)
as.data.frame(my_response) %>% 
  filter(Fallzahl != 0) %>% 
  sample_n(10)
#>    Jahr        Bundesland Geburtsland Fallzahl
#> 1  2018   Salzburg <AT32>     Ausland   104206
#> 2  2006 Steiermark <AT22>  Österreich  1096408
#> 3  2020    Kärnten <AT21>    Zusammen   561293
#> 4  2003 Steiermark <AT22>  Österreich  1088798
#> 5  2008 Steiermark <AT22>  Österreich  1094940
#> 6  2003 Vorarlberg <AT34>     Ausland    59655
#> 7  2003   Salzburg <AT32>  Österreich   436825
#> 8  2004   Salzburg <AT32>     Ausland    79142
#> 9  2002 Burgenland <AT11>    Zusammen   276673
#> 10 2013          Zusammen  Österreich  7087089
```

### Sonstiges

Um den Inhalt der Response zu erhalten, kann `httr::content()` verwendet werden.

```r
my_content <- httr::content(my_response$response)
my_content$measures
#> [[1]]
#> [[1]]$uri
#> [1] "str:statfn:debevstandjb:F-BEVSTANDJB:F-ISIS-1:SUM"
#> 
#> [[1]]$label
#> [1] "Fallzahl"
#> 
#> [[1]]$measure
#> [1] "str:measure:debevstandjb:F-BEVSTANDJB:F-ISIS-1"
#> 
#> [[1]]$`function`
#> [1] "SUM"
```

Die Endpoints `/info`, `/schema` und `/rate_limit` sind testweise angebunden
aber noch nicht exportiert.

```r
STATcubeR:::sc_get_info() %>% httr::content()
STATcubeR:::sc_get_schema() %>% httr::content()
STATcubeR:::sc_get_rate_limit() %>% httr::content()
```

STATcube verfügt über einen Cache. Wenn die selbe Abfrage mehrmals abgeschickt
wird, so wird das Rate-Limit (1000 Anfragen pro Stunde) nicht belastet.

## TODO

* Erklären der Begriffe Datenbank, Werte und Dimensionen
* Verwenden von `my_content$cubes${value_key}$annotations`
    * Hilfsspalten in `as.data.frame()` Version für jede Werte-Spalte
* Verwenden der URIs, um Variablen auch als Codes (nicht Labels) anzubieten.
  Könnte sein, dass hierzu der `/schema`-Endpoint notwendig ist.
    * `my_content$fields[[i]]$items[[j]]$uris[[1]]`
    * `my_content$fields[[i]]$uri`
    * `my_content$database$uri`
    * `my_content$measures[[i]]$uri`
* Fehlermeldung, falls `my_response$status_code != 200`

## API Dokumentation

* `/table` Endpoint: https://docs.wingarc.com.au/superstar/latest/open-data-api/open-data-api-reference/table-endpoint
* `/schema` Endpoint: https://docs.wingarc.com.au/superstar/latest/open-data-api/open-data-api-reference/schema-endpoint

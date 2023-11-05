---
title: "Atomkraftwerke in Europa"
date: 2021-09-08
format: hugo
slug: atomkraft-europa
categories: 
  - R
  - Kartierung
  - Webscraping
summary: "Eine datengetriebene Übersicht über die Entwicklung der Kernernergie in Europa."
---
<script src="{{< blogdown/postref >}}index_files/kePrint/kePrint.js"></script>
<script src="{{< blogdown/postref >}}index_files/kePrint/kePrint.js"></script>
<script src="{{< blogdown/postref >}}index_files/kePrint/kePrint.js"></script>




Benötigte R-Pakete:



```r
library(tidyverse)
library(ggmap)
library(gganimate)
library(rvest)
```


## **Datenbeschaffung**

Die Internationale Atomenergie-Organisation (*IAEA*) unterhält auf ihrer Webseite eine Datenbank, in der sich detallierte Angaben zu allen Atomreaktoren weltweit befinden - das [Power Reactor Information System (PRIS)](https://pris.iaea.org/PRIS/home.aspx). Auf GitHub habe ich einen [Auszug](https://github.com/cristianst85/GeoNuclearData) aus dieser Datenbank (Stand 21.06.2021) gefunden, den ich für die Auswertungen in diesem Artikel verwende. Die Daten befinden sich im .CSV Format und müssen nur in R eingelesen werden.

Der Datenbestand umfasst allerdings nur allgemeine Angaben zu den Reaktoren und ich möchte diese um die ins Netz eingespeiste Energiemenge erweitern. Diese Angaben findet man ebenfalls für jeden Reaktor im *PRIS* unter der Rubrik [Operating History](https://pris.iaea.org/PRIS/CountryStatistics/ReactorDetails.aspx?current=85), welche jährlich vom Betreiber gepflegt wird (sofern dieser Daten an die IAEA übermittelt). Mittels Webscraping extrahiere ich dabei die Spalte `Electricity Supplied [GW.h]` von jedem Reaktor. Jeder Reaktor besitzt in der Datenbank eine ID, welche auch auf der Webseite verwendet wird. Unten findet man den Code für die Extraktion der Daten. Die Liste der relevanten Reaktor-IDs ergibt sich aus dem ersten Datenbestand von GitHub. Ich verwende die Funktion `tryCatch()` um mögliche Fehlermeldungen zu überspringen, falls eine Reaktor-ID nicht mehr auf der Webseite verfügbar ist.



```r
scraper_pris <- function(url){
  
  # Vectors
  reactor_name <- vector(mode = "character")
  year <- vector(mode = "double")
  electricity <- vector(mode = "double")
  
  # Name
  tryCatch(
    reactor_name <-
      url %>%
      read_html() %>%
      html_nodes("#MainContent_MainContent_lblReactorName") %>%
      html_text(),
    error = function(e){NA})  
    
  # Year
  tryCatch(
    year <- 
      url %>%
      read_html() %>%
      html_nodes(".active td:nth-child(1)") %>%
      html_text() %>%
      str_replace_all("\r\n","") %>%
      str_trim() %>%
      as.numeric(),
    error = function(e){NA})  
    
  # Electricity supplied
  tryCatch(
    electricity <- 
      url %>%
      read_html() %>%
      html_nodes(".active td:nth-child(2)") %>%
      html_text() %>%
      str_replace_all("\r\n","") %>%
      str_trim() %>%
      as.numeric(),
    error = function(e){NA}) 
    
  # Data
    scraped_data <- tibble(
      reactor_name, year, electricity
    )

  return(scraped_data)
}
```



## **Datenaufbereitung**

Im Datenbankauszug mit den allgemeinen Reaktorangaben gibt es 3 .CSV Dateien, die Schlüsseltabellen für die Länderkürzel, den Reaktorstatus sowie den Reaktortyp enthalten.



```r
data_countries <-
  read_delim(
    file = "data/1-countries.csv", 
    delim = ","
    ) %>%
  rename("CountryCode" = Code, "CountryName" = Name)

data_status <- 
  read_delim(
    file = "data/2-nuclear_power_plant_status_type.csv", 
    delim = ","
    ) %>%
  rename("StatusId" = Id, "StatusType" = Type)

data_reactors <- 
  read_delim(
    file = "data/3-nuclear_reactor_type.csv", 
    delim = ","
    ) %>%
  rename("ReactorTypeId" = Id, "ReactorType" = Type)
```


Die 4. .CSV Datei enthält die Grunddaten für die einzelnen Atomreaktoren. Jede Zeile dieses Datensatzes repräsentiert einen Reaktor. Am Standort eines Atomkraftwerks kann es mehrere Reaktoren geben. Die Grunddaten der Reaktoren verknüpfe ich mit den drei zuvor eingelesenen Schlüsseltabellen.



```r
data_raw <- 
  read_delim(
    file = "data/4-nuclear_power_plants.csv", 
    delim = ",")

data_wide <-  
  data_raw %>%
  left_join(data_status,  by = "StatusId") %>%
  left_join(data_reactors,  by = "ReactorTypeId") %>%
  left_join(data_countries,  by = "CountryCode") %>%
  select(-c(StatusId, ReactorTypeId, Id)) 
```


Die Daten sehen so aus:

<br>


<div style="border: 1px solid #ddd; padding: 0px; overflow-y: scroll; height:300px; overflow-x: scroll; width:100%; "><table class="table table-responsive" style="font-size: 10px; margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;"> Name </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;"> Latitude </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;"> Longitude </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;"> CountryCode </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;"> ReactorModel </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;"> ConstructionStartAt </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;"> OperationalFrom </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;"> OperationalTo </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;"> Capacity </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;"> Source </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;"> LastUpdatedAt </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;"> IAEAId </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;"> StatusType </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;"> ReactorType </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;"> Description </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;"> CountryName </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> Ågesta </td>
   <td style="text-align:right;"> 59.20600 </td>
   <td style="text-align:right;"> 18.08290 </td>
   <td style="text-align:left;"> SE </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> 1957-12-01 </td>
   <td style="text-align:left;"> 1964-05-01 </td>
   <td style="text-align:left;"> 1974-06-02 </td>
   <td style="text-align:right;"> 9 </td>
   <td style="text-align:left;"> WNA/IAEA </td>
   <td style="text-align:left;"> 2015-05-24 01:51:37 </td>
   <td style="text-align:right;"> 528 </td>
   <td style="text-align:left;"> Shutdown </td>
   <td style="text-align:left;"> PHWR </td>
   <td style="text-align:left;"> Pressurized Heavy Water Reactor </td>
   <td style="text-align:left;"> Sweden </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Akademik Lomonosov-1 </td>
   <td style="text-align:right;"> 69.70958 </td>
   <td style="text-align:right;"> 170.30625 </td>
   <td style="text-align:left;"> RU </td>
   <td style="text-align:left;"> KLT-40S 'Floating' </td>
   <td style="text-align:left;"> 2007-04-15 </td>
   <td style="text-align:left;"> 2020-05-22 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> 30 </td>
   <td style="text-align:left;"> WNA/IAEA/Google Maps </td>
   <td style="text-align:left;"> 2021-05-31 00:00:00 </td>
   <td style="text-align:right;"> 895 </td>
   <td style="text-align:left;"> Operational </td>
   <td style="text-align:left;"> PWR </td>
   <td style="text-align:left;"> Pressurized Water Reactor </td>
   <td style="text-align:left;"> Russia </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Akademik Lomonosov-2 </td>
   <td style="text-align:right;"> 69.70958 </td>
   <td style="text-align:right;"> 170.30625 </td>
   <td style="text-align:left;"> RU </td>
   <td style="text-align:left;"> KLT-40S 'Floating' </td>
   <td style="text-align:left;"> 2007-04-15 </td>
   <td style="text-align:left;"> 2020-05-22 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> 30 </td>
   <td style="text-align:left;"> WNA/IAEA/Google Maps </td>
   <td style="text-align:left;"> 2021-05-31 00:00:00 </td>
   <td style="text-align:right;"> 896 </td>
   <td style="text-align:left;"> Operational </td>
   <td style="text-align:left;"> PWR </td>
   <td style="text-align:left;"> Pressurized Water Reactor </td>
   <td style="text-align:left;"> Russia </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Akhvaz-1 </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:left;"> IR </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:left;"> WNA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:left;"> Planned </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> Iran </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Akhvaz-2 </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:left;"> IR </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:left;"> WNA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:left;"> Planned </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> Iran </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Akkuyu-1 </td>
   <td style="text-align:right;"> 36.14444 </td>
   <td style="text-align:right;"> 33.54111 </td>
   <td style="text-align:left;"> TR </td>
   <td style="text-align:left;"> VVER V-509 </td>
   <td style="text-align:left;"> 2018-04-03 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> 1114 </td>
   <td style="text-align:left;"> WNA/Wikipedia/IAEA </td>
   <td style="text-align:left;"> 2018-06-30 22:21:08 </td>
   <td style="text-align:right;"> 553 </td>
   <td style="text-align:left;"> Under Construction </td>
   <td style="text-align:left;"> PWR </td>
   <td style="text-align:left;"> Pressurized Water Reactor </td>
   <td style="text-align:left;"> Turkey </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Akkuyu-2 </td>
   <td style="text-align:right;"> 36.14444 </td>
   <td style="text-align:right;"> 33.54111 </td>
   <td style="text-align:left;"> TR </td>
   <td style="text-align:left;"> VVER V-509 </td>
   <td style="text-align:left;"> 2020-04-08 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> 1114 </td>
   <td style="text-align:left;"> Wikipedia/IAEA </td>
   <td style="text-align:left;"> 2020-08-30 22:39:56 </td>
   <td style="text-align:right;"> 1080 </td>
   <td style="text-align:left;"> Under Construction </td>
   <td style="text-align:left;"> PWR </td>
   <td style="text-align:left;"> Pressurized Water Reactor </td>
   <td style="text-align:left;"> Turkey </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Akkuyu-3 </td>
   <td style="text-align:right;"> 36.14444 </td>
   <td style="text-align:right;"> 33.54111 </td>
   <td style="text-align:left;"> TR </td>
   <td style="text-align:left;"> VVER V-509 </td>
   <td style="text-align:left;"> 2021-03-10 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> 1114 </td>
   <td style="text-align:left;"> Wikipedia/IAEA </td>
   <td style="text-align:left;"> 2021-04-19 23:47:37 </td>
   <td style="text-align:right;"> 1081 </td>
   <td style="text-align:left;"> Under Construction </td>
   <td style="text-align:left;"> PWR </td>
   <td style="text-align:left;"> Pressurized Water Reactor </td>
   <td style="text-align:left;"> Turkey </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Akkuyu-4 </td>
   <td style="text-align:right;"> 36.14444 </td>
   <td style="text-align:right;"> 33.54111 </td>
   <td style="text-align:left;"> TR </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:left;"> Wikipedia </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:left;"> Planned </td>
   <td style="text-align:left;"> PWR </td>
   <td style="text-align:left;"> Pressurized Water Reactor </td>
   <td style="text-align:left;"> Turkey </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Aktau (Shevchenko) </td>
   <td style="text-align:right;"> 43.60700 </td>
   <td style="text-align:right;"> 51.28300 </td>
   <td style="text-align:left;"> KZ </td>
   <td style="text-align:left;"> BN-350 </td>
   <td style="text-align:left;"> 1964-10-01 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> 1999-04-22 </td>
   <td style="text-align:right;"> 52 </td>
   <td style="text-align:left;"> WNA/IAEA </td>
   <td style="text-align:left;"> 2021-04-19 23:47:37 </td>
   <td style="text-align:right;"> 414 </td>
   <td style="text-align:left;"> Shutdown </td>
   <td style="text-align:left;"> FBR </td>
   <td style="text-align:left;"> Fast Breeder Reactor </td>
   <td style="text-align:left;"> Kazakhstan </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Almaraz-1 </td>
   <td style="text-align:right;"> 39.80700 </td>
   <td style="text-align:right;"> -5.69800 </td>
   <td style="text-align:left;"> ES </td>
   <td style="text-align:left;"> WH 3LP </td>
   <td style="text-align:left;"> 1973-07-03 </td>
   <td style="text-align:left;"> 1983-09-01 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> 900 </td>
   <td style="text-align:left;"> WNA/IAEA </td>
   <td style="text-align:left;"> 2017-02-10 21:56:15 </td>
   <td style="text-align:right;"> 153 </td>
   <td style="text-align:left;"> Operational </td>
   <td style="text-align:left;"> PWR </td>
   <td style="text-align:left;"> Pressurized Water Reactor </td>
   <td style="text-align:left;"> Spain </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Almaraz-2 </td>
   <td style="text-align:right;"> 39.80700 </td>
   <td style="text-align:right;"> -5.69800 </td>
   <td style="text-align:left;"> ES </td>
   <td style="text-align:left;"> WH 3LP </td>
   <td style="text-align:left;"> 1973-07-03 </td>
   <td style="text-align:left;"> 1984-07-01 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> 900 </td>
   <td style="text-align:left;"> WNA/IAEA </td>
   <td style="text-align:left;"> 2019-06-02 17:17:55 </td>
   <td style="text-align:right;"> 154 </td>
   <td style="text-align:left;"> Operational </td>
   <td style="text-align:left;"> PWR </td>
   <td style="text-align:left;"> Pressurized Water Reactor </td>
   <td style="text-align:left;"> Spain </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Angra-1 </td>
   <td style="text-align:right;"> -23.00800 </td>
   <td style="text-align:right;"> -44.45700 </td>
   <td style="text-align:left;"> BR </td>
   <td style="text-align:left;"> WH 2LP </td>
   <td style="text-align:left;"> 1971-05-01 </td>
   <td style="text-align:left;"> 1985-01-01 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> 609 </td>
   <td style="text-align:left;"> WNA/IAEA </td>
   <td style="text-align:left;"> 2021-02-14 01:47:53 </td>
   <td style="text-align:right;"> 24 </td>
   <td style="text-align:left;"> Operational </td>
   <td style="text-align:left;"> PWR </td>
   <td style="text-align:left;"> Pressurized Water Reactor </td>
   <td style="text-align:left;"> Brazil </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Angra-2 </td>
   <td style="text-align:right;"> -23.00800 </td>
   <td style="text-align:right;"> -44.45700 </td>
   <td style="text-align:left;"> BR </td>
   <td style="text-align:left;"> PRE KONVOI </td>
   <td style="text-align:left;"> 1976-01-01 </td>
   <td style="text-align:left;"> 2001-02-01 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> 1245 </td>
   <td style="text-align:left;"> WNA/IAEA </td>
   <td style="text-align:left;"> 2015-05-24 01:50:19 </td>
   <td style="text-align:right;"> 25 </td>
   <td style="text-align:left;"> Operational </td>
   <td style="text-align:left;"> PWR </td>
   <td style="text-align:left;"> Pressurized Water Reactor </td>
   <td style="text-align:left;"> Brazil </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Angra-3 </td>
   <td style="text-align:right;"> -23.01000 </td>
   <td style="text-align:right;"> -44.47000 </td>
   <td style="text-align:left;"> BR </td>
   <td style="text-align:left;"> PRE KONVOI </td>
   <td style="text-align:left;"> 2010-06-01 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> 1340 </td>
   <td style="text-align:left;"> WNA/IAEA </td>
   <td style="text-align:left;"> 2018-06-30 22:21:29 </td>
   <td style="text-align:right;"> 26 </td>
   <td style="text-align:left;"> Under Construction </td>
   <td style="text-align:left;"> PWR </td>
   <td style="text-align:left;"> Pressurized Water Reactor </td>
   <td style="text-align:left;"> Brazil </td>
  </tr>
  <tr>
   <td style="text-align:left;"> APS-1 Obninsk </td>
   <td style="text-align:right;"> 55.08400 </td>
   <td style="text-align:right;"> 36.57000 </td>
   <td style="text-align:left;"> RU </td>
   <td style="text-align:left;"> AM-1 </td>
   <td style="text-align:left;"> 1951-01-01 </td>
   <td style="text-align:left;"> 1954-12-01 </td>
   <td style="text-align:left;"> 2002-04-29 </td>
   <td style="text-align:right;"> 5 </td>
   <td style="text-align:left;"> WNA/IAEA </td>
   <td style="text-align:left;"> 2015-05-24 01:51:32 </td>
   <td style="text-align:right;"> 447 </td>
   <td style="text-align:left;"> Shutdown </td>
   <td style="text-align:left;"> LWGR </td>
   <td style="text-align:left;"> Light Water Graphite Reactor </td>
   <td style="text-align:left;"> Russia </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Arkansas Nuclear One-1 (ANO-1) </td>
   <td style="text-align:right;"> 35.31000 </td>
   <td style="text-align:right;"> -93.23000 </td>
   <td style="text-align:left;"> US </td>
   <td style="text-align:left;"> B&amp;W LLP (DRYAMB) </td>
   <td style="text-align:left;"> 1968-10-01 </td>
   <td style="text-align:left;"> 1974-12-19 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> 850 </td>
   <td style="text-align:left;"> WNA/IAEA </td>
   <td style="text-align:left;"> 2017-02-10 21:58:30 </td>
   <td style="text-align:right;"> 652 </td>
   <td style="text-align:left;"> Operational </td>
   <td style="text-align:left;"> PWR </td>
   <td style="text-align:left;"> Pressurized Water Reactor </td>
   <td style="text-align:left;"> United States </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Arkansas Nuclear One-2 (ANO-2) </td>
   <td style="text-align:right;"> 35.31000 </td>
   <td style="text-align:right;"> -93.22900 </td>
   <td style="text-align:left;"> US </td>
   <td style="text-align:left;"> CE 2LP (DRYAMB) </td>
   <td style="text-align:left;"> 1968-12-06 </td>
   <td style="text-align:left;"> 1980-03-26 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> 912 </td>
   <td style="text-align:left;"> WNA/IAEA </td>
   <td style="text-align:left;"> 2017-02-10 21:58:53 </td>
   <td style="text-align:right;"> 689 </td>
   <td style="text-align:left;"> Operational </td>
   <td style="text-align:left;"> PWR </td>
   <td style="text-align:left;"> Pressurized Water Reactor </td>
   <td style="text-align:left;"> United States </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Armenian-1 </td>
   <td style="text-align:right;"> 40.18200 </td>
   <td style="text-align:right;"> 44.14700 </td>
   <td style="text-align:left;"> AM </td>
   <td style="text-align:left;"> VVER V-270 </td>
   <td style="text-align:left;"> 1969-07-01 </td>
   <td style="text-align:left;"> 1977-10-06 </td>
   <td style="text-align:left;"> 1989-02-25 </td>
   <td style="text-align:right;"> 376 </td>
   <td style="text-align:left;"> WNA/IAEA </td>
   <td style="text-align:left;"> 2021-06-07 00:00:00 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:left;"> Shutdown </td>
   <td style="text-align:left;"> PWR </td>
   <td style="text-align:left;"> Pressurized Water Reactor </td>
   <td style="text-align:left;"> Armenia </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Armenian-2 </td>
   <td style="text-align:right;"> 40.18200 </td>
   <td style="text-align:right;"> 44.14700 </td>
   <td style="text-align:left;"> AM </td>
   <td style="text-align:left;"> VVER V-270 </td>
   <td style="text-align:left;"> 1975-07-01 </td>
   <td style="text-align:left;"> 1980-05-03 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> 375 </td>
   <td style="text-align:left;"> WNA/IAEA </td>
   <td style="text-align:left;"> 2021-06-28 21:12:17 </td>
   <td style="text-align:right;"> 2 </td>
   <td style="text-align:left;"> Operational </td>
   <td style="text-align:left;"> PWR </td>
   <td style="text-align:left;"> Pressurized Water Reactor </td>
   <td style="text-align:left;"> Armenia </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Armenian-3 </td>
   <td style="text-align:right;"> 40.18080 </td>
   <td style="text-align:right;"> 44.14720 </td>
   <td style="text-align:left;"> AM </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:left;"> WNA </td>
   <td style="text-align:left;"> 2021-06-07 00:00:00 </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:left;"> Planned </td>
   <td style="text-align:left;"> PWR </td>
   <td style="text-align:left;"> Pressurized Water Reactor </td>
   <td style="text-align:left;"> Armenia </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Asco-1 </td>
   <td style="text-align:right;"> 41.20200 </td>
   <td style="text-align:right;"> 0.57100 </td>
   <td style="text-align:left;"> ES </td>
   <td style="text-align:left;"> WH 3LP </td>
   <td style="text-align:left;"> 1974-05-16 </td>
   <td style="text-align:left;"> 1984-12-10 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> 995 </td>
   <td style="text-align:left;"> WNA/IAEA </td>
   <td style="text-align:left;"> 2021-01-05 23:19:47 </td>
   <td style="text-align:right;"> 155 </td>
   <td style="text-align:left;"> Operational </td>
   <td style="text-align:left;"> PWR </td>
   <td style="text-align:left;"> Pressurized Water Reactor </td>
   <td style="text-align:left;"> Spain </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Asco-2 </td>
   <td style="text-align:right;"> 41.20200 </td>
   <td style="text-align:right;"> 0.57100 </td>
   <td style="text-align:left;"> ES </td>
   <td style="text-align:left;"> WH 3LP </td>
   <td style="text-align:left;"> 1975-03-07 </td>
   <td style="text-align:left;"> 1986-03-31 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> 997 </td>
   <td style="text-align:left;"> WNA/IAEA </td>
   <td style="text-align:left;"> 2021-04-19 23:47:37 </td>
   <td style="text-align:right;"> 156 </td>
   <td style="text-align:left;"> Operational </td>
   <td style="text-align:left;"> PWR </td>
   <td style="text-align:left;"> Pressurized Water Reactor </td>
   <td style="text-align:left;"> Spain </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Atucha-1 </td>
   <td style="text-align:right;"> -33.96700 </td>
   <td style="text-align:right;"> -59.20900 </td>
   <td style="text-align:left;"> AR </td>
   <td style="text-align:left;"> PHWR KWU </td>
   <td style="text-align:left;"> 1968-06-01 </td>
   <td style="text-align:left;"> 1974-06-24 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> 340 </td>
   <td style="text-align:left;"> WNA/IAEA </td>
   <td style="text-align:left;"> 2021-02-14 01:47:53 </td>
   <td style="text-align:right;"> 3 </td>
   <td style="text-align:left;"> Operational </td>
   <td style="text-align:left;"> PHWR </td>
   <td style="text-align:left;"> Pressurized Heavy Water Reactor </td>
   <td style="text-align:left;"> Argentina </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Atucha-2 </td>
   <td style="text-align:right;"> -33.96700 </td>
   <td style="text-align:right;"> -59.20900 </td>
   <td style="text-align:left;"> AR </td>
   <td style="text-align:left;"> PHWR KWU </td>
   <td style="text-align:left;"> 1981-07-14 </td>
   <td style="text-align:left;"> 2016-05-26 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> 692 </td>
   <td style="text-align:left;"> WNA/IAEA </td>
   <td style="text-align:left;"> 2017-09-25 00:19:13 </td>
   <td style="text-align:right;"> 5 </td>
   <td style="text-align:left;"> Operational </td>
   <td style="text-align:left;"> PHWR </td>
   <td style="text-align:left;"> Pressurized Heavy Water Reactor </td>
   <td style="text-align:left;"> Argentina </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Balakovo-1 </td>
   <td style="text-align:right;"> 52.09200 </td>
   <td style="text-align:right;"> 47.95200 </td>
   <td style="text-align:left;"> RU </td>
   <td style="text-align:left;"> VVER V-320 </td>
   <td style="text-align:left;"> 1980-12-01 </td>
   <td style="text-align:left;"> 1986-05-23 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> 950 </td>
   <td style="text-align:left;"> WNA/IAEA </td>
   <td style="text-align:left;"> 2015-05-24 01:51:36 </td>
   <td style="text-align:right;"> 524 </td>
   <td style="text-align:left;"> Operational </td>
   <td style="text-align:left;"> PWR </td>
   <td style="text-align:left;"> Pressurized Water Reactor </td>
   <td style="text-align:left;"> Russia </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Balakovo-2 </td>
   <td style="text-align:right;"> 52.09200 </td>
   <td style="text-align:right;"> 47.95200 </td>
   <td style="text-align:left;"> RU </td>
   <td style="text-align:left;"> VVER V-320 </td>
   <td style="text-align:left;"> 1981-08-01 </td>
   <td style="text-align:left;"> 1988-01-18 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> 950 </td>
   <td style="text-align:left;"> WNA/IAEA </td>
   <td style="text-align:left;"> 2015-05-24 01:51:36 </td>
   <td style="text-align:right;"> 525 </td>
   <td style="text-align:left;"> Operational </td>
   <td style="text-align:left;"> PWR </td>
   <td style="text-align:left;"> Pressurized Water Reactor </td>
   <td style="text-align:left;"> Russia </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Balakovo-3 </td>
   <td style="text-align:right;"> 52.09200 </td>
   <td style="text-align:right;"> 47.95200 </td>
   <td style="text-align:left;"> RU </td>
   <td style="text-align:left;"> VVER V-320 </td>
   <td style="text-align:left;"> 1982-11-01 </td>
   <td style="text-align:left;"> 1989-04-08 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> 950 </td>
   <td style="text-align:left;"> WNA/IAEA </td>
   <td style="text-align:left;"> 2015-05-24 01:51:36 </td>
   <td style="text-align:right;"> 526 </td>
   <td style="text-align:left;"> Operational </td>
   <td style="text-align:left;"> PWR </td>
   <td style="text-align:left;"> Pressurized Water Reactor </td>
   <td style="text-align:left;"> Russia </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Balakovo-4 </td>
   <td style="text-align:right;"> 52.09200 </td>
   <td style="text-align:right;"> 47.95200 </td>
   <td style="text-align:left;"> RU </td>
   <td style="text-align:left;"> VVER V-320 </td>
   <td style="text-align:left;"> 1984-04-01 </td>
   <td style="text-align:left;"> 1993-12-22 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> 950 </td>
   <td style="text-align:left;"> WNA/IAEA </td>
   <td style="text-align:left;"> 2015-05-24 01:51:36 </td>
   <td style="text-align:right;"> 527 </td>
   <td style="text-align:left;"> Operational </td>
   <td style="text-align:left;"> PWR </td>
   <td style="text-align:left;"> Pressurized Water Reactor </td>
   <td style="text-align:left;"> Russia </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Baltic-1 </td>
   <td style="text-align:right;"> 54.93900 </td>
   <td style="text-align:right;"> 22.16200 </td>
   <td style="text-align:left;"> RU </td>
   <td style="text-align:left;"> VVER V-491 </td>
   <td style="text-align:left;"> 2012-02-22 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> 1109 </td>
   <td style="text-align:left;"> WNA/IAEA </td>
   <td style="text-align:left;"> 2015-05-24 01:51:57 </td>
   <td style="text-align:right;"> 968 </td>
   <td style="text-align:left;"> Under Construction </td>
   <td style="text-align:left;"> PWR </td>
   <td style="text-align:left;"> Pressurized Water Reactor </td>
   <td style="text-align:left;"> Russia </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Barakah-1 (Braka) </td>
   <td style="text-align:right;"> 23.95275 </td>
   <td style="text-align:right;"> 52.19330 </td>
   <td style="text-align:left;"> AE </td>
   <td style="text-align:left;"> APR-1400 </td>
   <td style="text-align:left;"> 2012-07-19 </td>
   <td style="text-align:left;"> 2021-04-01 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> 1345 </td>
   <td style="text-align:left;"> WNA/IAEA </td>
   <td style="text-align:left;"> 2021-05-30 14:38:32 </td>
   <td style="text-align:right;"> 1050 </td>
   <td style="text-align:left;"> Operational </td>
   <td style="text-align:left;"> PWR </td>
   <td style="text-align:left;"> Pressurized Water Reactor </td>
   <td style="text-align:left;"> United Arab Emirates </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Barakah-2 (Braka) </td>
   <td style="text-align:right;"> 23.95275 </td>
   <td style="text-align:right;"> 52.19330 </td>
   <td style="text-align:left;"> AE </td>
   <td style="text-align:left;"> APR-1400 </td>
   <td style="text-align:left;"> 2013-04-16 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> 1345 </td>
   <td style="text-align:left;"> WNA/IAEA </td>
   <td style="text-align:left;"> 2015-12-27 15:05:52 </td>
   <td style="text-align:right;"> 1051 </td>
   <td style="text-align:left;"> Under Construction </td>
   <td style="text-align:left;"> PWR </td>
   <td style="text-align:left;"> Pressurized Water Reactor </td>
   <td style="text-align:left;"> United Arab Emirates </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Barakah-3 (Braka) </td>
   <td style="text-align:right;"> 23.95275 </td>
   <td style="text-align:right;"> 52.20330 </td>
   <td style="text-align:left;"> AE </td>
   <td style="text-align:left;"> APR-1400 </td>
   <td style="text-align:left;"> 2014-09-24 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> 1345 </td>
   <td style="text-align:left;"> WNA/IAEA </td>
   <td style="text-align:left;"> 2015-05-24 01:51:58 </td>
   <td style="text-align:right;"> 1052 </td>
   <td style="text-align:left;"> Under Construction </td>
   <td style="text-align:left;"> PWR </td>
   <td style="text-align:left;"> Pressurized Water Reactor </td>
   <td style="text-align:left;"> United Arab Emirates </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Chutka-1 </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:left;"> IN </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:left;"> WNA </td>
   <td style="text-align:left;"> 2021-06-07 00:00:00 </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:left;"> Planned </td>
   <td style="text-align:left;"> PHWR </td>
   <td style="text-align:left;"> Pressurized Heavy Water Reactor </td>
   <td style="text-align:left;"> India </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Chutka-2 </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:left;"> IN </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:left;"> WNA </td>
   <td style="text-align:left;"> 2021-06-07 00:00:00 </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:left;"> Planned </td>
   <td style="text-align:left;"> PHWR </td>
   <td style="text-align:left;"> Pressurized Heavy Water Reactor </td>
   <td style="text-align:left;"> India </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Barseback-1 </td>
   <td style="text-align:right;"> 55.74500 </td>
   <td style="text-align:right;"> 12.92600 </td>
   <td style="text-align:left;"> SE </td>
   <td style="text-align:left;"> AA-II </td>
   <td style="text-align:left;"> 1971-02-01 </td>
   <td style="text-align:left;"> 1975-07-01 </td>
   <td style="text-align:left;"> 1999-11-30 </td>
   <td style="text-align:right;"> 570 </td>
   <td style="text-align:left;"> WNA/IAEA </td>
   <td style="text-align:left;"> 2018-03-10 12:52:00 </td>
   <td style="text-align:right;"> 538 </td>
   <td style="text-align:left;"> Shutdown </td>
   <td style="text-align:left;"> BWR </td>
   <td style="text-align:left;"> Boiling Water Reactor </td>
   <td style="text-align:left;"> Sweden </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Barseback-2 </td>
   <td style="text-align:right;"> 55.74500 </td>
   <td style="text-align:right;"> 12.92600 </td>
   <td style="text-align:left;"> SE </td>
   <td style="text-align:left;"> AA-II </td>
   <td style="text-align:left;"> 1973-01-01 </td>
   <td style="text-align:left;"> 1977-07-01 </td>
   <td style="text-align:left;"> 2005-05-31 </td>
   <td style="text-align:right;"> 570 </td>
   <td style="text-align:left;"> WNA/IAEA </td>
   <td style="text-align:left;"> 2018-03-10 12:52:02 </td>
   <td style="text-align:right;"> 540 </td>
   <td style="text-align:left;"> Shutdown </td>
   <td style="text-align:left;"> BWR </td>
   <td style="text-align:left;"> Boiling Water Reactor </td>
   <td style="text-align:left;"> Sweden </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Beaver Valley-1 </td>
   <td style="text-align:right;"> 40.62400 </td>
   <td style="text-align:right;"> -80.43200 </td>
   <td style="text-align:left;"> US </td>
   <td style="text-align:left;"> WH 3LP (DRYSUB) </td>
   <td style="text-align:left;"> 1970-06-26 </td>
   <td style="text-align:left;"> 1976-10-01 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> 835 </td>
   <td style="text-align:left;"> WNA/IAEA </td>
   <td style="text-align:left;"> 2017-02-10 21:58:44 </td>
   <td style="text-align:right;"> 669 </td>
   <td style="text-align:left;"> Operational </td>
   <td style="text-align:left;"> PWR </td>
   <td style="text-align:left;"> Pressurized Water Reactor </td>
   <td style="text-align:left;"> United States </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Beaver Valley-2 </td>
   <td style="text-align:right;"> 40.62400 </td>
   <td style="text-align:right;"> -80.43200 </td>
   <td style="text-align:left;"> US </td>
   <td style="text-align:left;"> WH 3LP (DRYSUB) </td>
   <td style="text-align:left;"> 1974-05-03 </td>
   <td style="text-align:left;"> 1987-11-17 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> 836 </td>
   <td style="text-align:left;"> WNA/IAEA </td>
   <td style="text-align:left;"> 2017-02-10 21:58:59 </td>
   <td style="text-align:right;"> 712 </td>
   <td style="text-align:left;"> Operational </td>
   <td style="text-align:left;"> PWR </td>
   <td style="text-align:left;"> Pressurized Water Reactor </td>
   <td style="text-align:left;"> United States </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Belarusian-1 </td>
   <td style="text-align:right;"> 54.76667 </td>
   <td style="text-align:right;"> 26.11667 </td>
   <td style="text-align:left;"> BY </td>
   <td style="text-align:left;"> VVER V-491 </td>
   <td style="text-align:left;"> 2013-11-08 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> 1110 </td>
   <td style="text-align:left;"> IAEA </td>
   <td style="text-align:left;"> 2021-01-05 23:19:47 </td>
   <td style="text-align:right;"> 1056 </td>
   <td style="text-align:left;"> Operational </td>
   <td style="text-align:left;"> PWR </td>
   <td style="text-align:left;"> Pressurized Water Reactor </td>
   <td style="text-align:left;"> Belarus </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Belarusian-2 </td>
   <td style="text-align:right;"> 54.76667 </td>
   <td style="text-align:right;"> 26.11667 </td>
   <td style="text-align:left;"> BY </td>
   <td style="text-align:left;"> VVER V-491 </td>
   <td style="text-align:left;"> 2014-04-27 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> 1110 </td>
   <td style="text-align:left;"> IAEA </td>
   <td style="text-align:left;"> 2018-03-10 12:54:39 </td>
   <td style="text-align:right;"> 1061 </td>
   <td style="text-align:left;"> Under Construction </td>
   <td style="text-align:left;"> PWR </td>
   <td style="text-align:left;"> Pressurized Water Reactor </td>
   <td style="text-align:left;"> Belarus </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Belene-1 </td>
   <td style="text-align:right;"> 43.62453 </td>
   <td style="text-align:right;"> 25.18675 </td>
   <td style="text-align:left;"> BG </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> 1987-01-01 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> 953 </td>
   <td style="text-align:left;"> WNA/IAEA </td>
   <td style="text-align:left;"> 2021-02-13 22:57:00 </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:left;"> Cancelled Construction </td>
   <td style="text-align:left;"> PWR </td>
   <td style="text-align:left;"> Pressurized Water Reactor </td>
   <td style="text-align:left;"> Bulgaria </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Belene-2 </td>
   <td style="text-align:right;"> 43.62453 </td>
   <td style="text-align:right;"> 25.18650 </td>
   <td style="text-align:left;"> BG </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> 1987-03-31 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> 953 </td>
   <td style="text-align:left;"> WNA/IAEA </td>
   <td style="text-align:left;"> 2021-02-13 22:57:00 </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:left;"> Cancelled Construction </td>
   <td style="text-align:left;"> PWR </td>
   <td style="text-align:left;"> Pressurized Water Reactor </td>
   <td style="text-align:left;"> Bulgaria </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Belleville-1 </td>
   <td style="text-align:right;"> 47.51100 </td>
   <td style="text-align:right;"> 2.87100 </td>
   <td style="text-align:left;"> FR </td>
   <td style="text-align:left;"> P4 REP 1300 </td>
   <td style="text-align:left;"> 1980-05-01 </td>
   <td style="text-align:left;"> 1988-06-01 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> 1310 </td>
   <td style="text-align:left;"> WNA/IAEA </td>
   <td style="text-align:left;"> 2015-05-24 01:51:02 </td>
   <td style="text-align:right;"> 211 </td>
   <td style="text-align:left;"> Operational </td>
   <td style="text-align:left;"> PWR </td>
   <td style="text-align:left;"> Pressurized Water Reactor </td>
   <td style="text-align:left;"> France </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Belleville-2 </td>
   <td style="text-align:right;"> 47.51100 </td>
   <td style="text-align:right;"> 2.87100 </td>
   <td style="text-align:left;"> FR </td>
   <td style="text-align:left;"> P4 REP 1300 </td>
   <td style="text-align:left;"> 1980-08-01 </td>
   <td style="text-align:left;"> 1989-01-01 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> 1310 </td>
   <td style="text-align:left;"> WNA/IAEA </td>
   <td style="text-align:left;"> 2015-05-24 01:51:02 </td>
   <td style="text-align:right;"> 212 </td>
   <td style="text-align:left;"> Operational </td>
   <td style="text-align:left;"> PWR </td>
   <td style="text-align:left;"> Pressurized Water Reactor </td>
   <td style="text-align:left;"> France </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Beloyarsk-1 </td>
   <td style="text-align:right;"> 56.84200 </td>
   <td style="text-align:right;"> 61.32100 </td>
   <td style="text-align:left;"> RU </td>
   <td style="text-align:left;"> AMB-100 </td>
   <td style="text-align:left;"> 1958-06-01 </td>
   <td style="text-align:left;"> 1964-04-26 </td>
   <td style="text-align:left;"> 1983-01-01 </td>
   <td style="text-align:right;"> 102 </td>
   <td style="text-align:left;"> WNA/IAEA </td>
   <td style="text-align:left;"> 2015-05-24 01:51:34 </td>
   <td style="text-align:right;"> 488 </td>
   <td style="text-align:left;"> Shutdown </td>
   <td style="text-align:left;"> LWGR </td>
   <td style="text-align:left;"> Light Water Graphite Reactor </td>
   <td style="text-align:left;"> Russia </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Beloyarsk-2 </td>
   <td style="text-align:right;"> 56.84200 </td>
   <td style="text-align:right;"> 61.32100 </td>
   <td style="text-align:left;"> RU </td>
   <td style="text-align:left;"> AMB-200 </td>
   <td style="text-align:left;"> 1962-01-01 </td>
   <td style="text-align:left;"> 1969-12-01 </td>
   <td style="text-align:left;"> 1990-01-01 </td>
   <td style="text-align:right;"> 146 </td>
   <td style="text-align:left;"> WNA/IAEA </td>
   <td style="text-align:left;"> 2015-05-24 01:51:35 </td>
   <td style="text-align:right;"> 503 </td>
   <td style="text-align:left;"> Shutdown </td>
   <td style="text-align:left;"> LWGR </td>
   <td style="text-align:left;"> Light Water Graphite Reactor </td>
   <td style="text-align:left;"> Russia </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Beloyarsk-3 </td>
   <td style="text-align:right;"> 56.84200 </td>
   <td style="text-align:right;"> 61.32100 </td>
   <td style="text-align:left;"> RU </td>
   <td style="text-align:left;"> BN-600 </td>
   <td style="text-align:left;"> 1969-01-01 </td>
   <td style="text-align:left;"> 1981-11-01 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> 560 </td>
   <td style="text-align:left;"> WNA/IAEA </td>
   <td style="text-align:left;"> 2015-05-24 01:51:34 </td>
   <td style="text-align:right;"> 484 </td>
   <td style="text-align:left;"> Operational </td>
   <td style="text-align:left;"> FBR </td>
   <td style="text-align:left;"> Fast Breeder Reactor </td>
   <td style="text-align:left;"> Russia </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Beloyarsk-4 </td>
   <td style="text-align:right;"> 56.84200 </td>
   <td style="text-align:right;"> 61.32100 </td>
   <td style="text-align:left;"> RU </td>
   <td style="text-align:left;"> BN-800 </td>
   <td style="text-align:left;"> 2006-07-18 </td>
   <td style="text-align:left;"> 2016-10-31 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> 820 </td>
   <td style="text-align:left;"> WNA/IAEA </td>
   <td style="text-align:left;"> 2017-09-25 00:19:53 </td>
   <td style="text-align:right;"> 451 </td>
   <td style="text-align:left;"> Operational </td>
   <td style="text-align:left;"> FBR </td>
   <td style="text-align:left;"> Fast Breeder Reactor </td>
   <td style="text-align:left;"> Russia </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Beloyarsk-5 </td>
   <td style="text-align:right;"> 56.84200 </td>
   <td style="text-align:right;"> 61.32100 </td>
   <td style="text-align:left;"> RU </td>
   <td style="text-align:left;"> BN-1200 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:left;"> WNA/Wikipedia </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:left;"> Planned </td>
   <td style="text-align:left;"> FBR </td>
   <td style="text-align:left;"> Fast Breeder Reactor </td>
   <td style="text-align:left;"> Russia </td>
  </tr>
</tbody>
</table></div>




In diesem Datensatz existieren die Spalten `ConstructionStartAt`, `OperationalFrom` und `OperationalTo`, welche den Zeitpunkt markieren an dem der Bau, der Betrieb und die Stilllegung des Reaktors stattfand. Später möchte ich eine animierte Karte erstellen, bei der sich diese fixen Datumsangaben wie eine Zeitreihe verhalten - d.h. ab dem Zeitpunkt an dem der Bau beginnt, befindet sich das AKW solange im Bau bis sich der Status ändert (bspw. wenn ein Reaktor in Betrieb gesetzt wird). Hierfür transformiere ich den Datensatz und fasse das Ereignis, das jede Spalte repräsentiert, in der Spalte (`TimeEvent`) zusammen und hinterlege das jeweilige Datum in der Spalte (`Date`). Ist ein Ereignis für einen Reaktor noch nicht eingetreten so wird es entfernt.



```r
data_long <-
  data_wide %>%
  pivot_longer(cols = c("ConstructionStartAt", "OperationalFrom", "OperationalTo"), 
               names_to = "TimeEvent", 
               values_to = "Date") %>%
  mutate(Year = lubridate::year(Date),
         TimeEvent_Animation = case_when(
           TimeEvent == "ConstructionStartAt" ~ "under construction",
           TimeEvent == "OperationalFrom" ~ "active",
           TimeEvent == "OperationalTo" ~ "decommissioned"
         )) %>%
  filter(!is.na(Date))
```


Die transformierten Daten (ausgewählte Spalten) sehen so aus:

<br>


<div style="border: 1px solid #ddd; padding: 0px; overflow-y: scroll; height:300px; overflow-x: scroll; width:100%; "><table class="table table-responsive" style="font-size: 10px; margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;"> Name </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;"> TimeEvent </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;"> Date </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;"> TimeEvent_Animation </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> Ågesta </td>
   <td style="text-align:left;"> ConstructionStartAt </td>
   <td style="text-align:left;"> 1957-12-01 </td>
   <td style="text-align:left;"> under construction </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Ågesta </td>
   <td style="text-align:left;"> OperationalFrom </td>
   <td style="text-align:left;"> 1964-05-01 </td>
   <td style="text-align:left;"> active </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Ågesta </td>
   <td style="text-align:left;"> OperationalTo </td>
   <td style="text-align:left;"> 1974-06-02 </td>
   <td style="text-align:left;"> decommissioned </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Akademik Lomonosov-1 </td>
   <td style="text-align:left;"> ConstructionStartAt </td>
   <td style="text-align:left;"> 2007-04-15 </td>
   <td style="text-align:left;"> under construction </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Akademik Lomonosov-1 </td>
   <td style="text-align:left;"> OperationalFrom </td>
   <td style="text-align:left;"> 2020-05-22 </td>
   <td style="text-align:left;"> active </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Akademik Lomonosov-2 </td>
   <td style="text-align:left;"> ConstructionStartAt </td>
   <td style="text-align:left;"> 2007-04-15 </td>
   <td style="text-align:left;"> under construction </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Akademik Lomonosov-2 </td>
   <td style="text-align:left;"> OperationalFrom </td>
   <td style="text-align:left;"> 2020-05-22 </td>
   <td style="text-align:left;"> active </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Akkuyu-1 </td>
   <td style="text-align:left;"> ConstructionStartAt </td>
   <td style="text-align:left;"> 2018-04-03 </td>
   <td style="text-align:left;"> under construction </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Akkuyu-2 </td>
   <td style="text-align:left;"> ConstructionStartAt </td>
   <td style="text-align:left;"> 2020-04-08 </td>
   <td style="text-align:left;"> under construction </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Akkuyu-3 </td>
   <td style="text-align:left;"> ConstructionStartAt </td>
   <td style="text-align:left;"> 2021-03-10 </td>
   <td style="text-align:left;"> under construction </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Aktau (Shevchenko) </td>
   <td style="text-align:left;"> ConstructionStartAt </td>
   <td style="text-align:left;"> 1964-10-01 </td>
   <td style="text-align:left;"> under construction </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Aktau (Shevchenko) </td>
   <td style="text-align:left;"> OperationalTo </td>
   <td style="text-align:left;"> 1999-04-22 </td>
   <td style="text-align:left;"> decommissioned </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Almaraz-1 </td>
   <td style="text-align:left;"> ConstructionStartAt </td>
   <td style="text-align:left;"> 1973-07-03 </td>
   <td style="text-align:left;"> under construction </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Almaraz-1 </td>
   <td style="text-align:left;"> OperationalFrom </td>
   <td style="text-align:left;"> 1983-09-01 </td>
   <td style="text-align:left;"> active </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Almaraz-2 </td>
   <td style="text-align:left;"> ConstructionStartAt </td>
   <td style="text-align:left;"> 1973-07-03 </td>
   <td style="text-align:left;"> under construction </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Almaraz-2 </td>
   <td style="text-align:left;"> OperationalFrom </td>
   <td style="text-align:left;"> 1984-07-01 </td>
   <td style="text-align:left;"> active </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Angra-1 </td>
   <td style="text-align:left;"> ConstructionStartAt </td>
   <td style="text-align:left;"> 1971-05-01 </td>
   <td style="text-align:left;"> under construction </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Angra-1 </td>
   <td style="text-align:left;"> OperationalFrom </td>
   <td style="text-align:left;"> 1985-01-01 </td>
   <td style="text-align:left;"> active </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Angra-2 </td>
   <td style="text-align:left;"> ConstructionStartAt </td>
   <td style="text-align:left;"> 1976-01-01 </td>
   <td style="text-align:left;"> under construction </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Angra-2 </td>
   <td style="text-align:left;"> OperationalFrom </td>
   <td style="text-align:left;"> 2001-02-01 </td>
   <td style="text-align:left;"> active </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Angra-3 </td>
   <td style="text-align:left;"> ConstructionStartAt </td>
   <td style="text-align:left;"> 2010-06-01 </td>
   <td style="text-align:left;"> under construction </td>
  </tr>
  <tr>
   <td style="text-align:left;"> APS-1 Obninsk </td>
   <td style="text-align:left;"> ConstructionStartAt </td>
   <td style="text-align:left;"> 1951-01-01 </td>
   <td style="text-align:left;"> under construction </td>
  </tr>
  <tr>
   <td style="text-align:left;"> APS-1 Obninsk </td>
   <td style="text-align:left;"> OperationalFrom </td>
   <td style="text-align:left;"> 1954-12-01 </td>
   <td style="text-align:left;"> active </td>
  </tr>
  <tr>
   <td style="text-align:left;"> APS-1 Obninsk </td>
   <td style="text-align:left;"> OperationalTo </td>
   <td style="text-align:left;"> 2002-04-29 </td>
   <td style="text-align:left;"> decommissioned </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Arkansas Nuclear One-1 (ANO-1) </td>
   <td style="text-align:left;"> ConstructionStartAt </td>
   <td style="text-align:left;"> 1968-10-01 </td>
   <td style="text-align:left;"> under construction </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Arkansas Nuclear One-1 (ANO-1) </td>
   <td style="text-align:left;"> OperationalFrom </td>
   <td style="text-align:left;"> 1974-12-19 </td>
   <td style="text-align:left;"> active </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Arkansas Nuclear One-2 (ANO-2) </td>
   <td style="text-align:left;"> ConstructionStartAt </td>
   <td style="text-align:left;"> 1968-12-06 </td>
   <td style="text-align:left;"> under construction </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Arkansas Nuclear One-2 (ANO-2) </td>
   <td style="text-align:left;"> OperationalFrom </td>
   <td style="text-align:left;"> 1980-03-26 </td>
   <td style="text-align:left;"> active </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Armenian-1 </td>
   <td style="text-align:left;"> ConstructionStartAt </td>
   <td style="text-align:left;"> 1969-07-01 </td>
   <td style="text-align:left;"> under construction </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Armenian-1 </td>
   <td style="text-align:left;"> OperationalFrom </td>
   <td style="text-align:left;"> 1977-10-06 </td>
   <td style="text-align:left;"> active </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Armenian-1 </td>
   <td style="text-align:left;"> OperationalTo </td>
   <td style="text-align:left;"> 1989-02-25 </td>
   <td style="text-align:left;"> decommissioned </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Armenian-2 </td>
   <td style="text-align:left;"> ConstructionStartAt </td>
   <td style="text-align:left;"> 1975-07-01 </td>
   <td style="text-align:left;"> under construction </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Armenian-2 </td>
   <td style="text-align:left;"> OperationalFrom </td>
   <td style="text-align:left;"> 1980-05-03 </td>
   <td style="text-align:left;"> active </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Asco-1 </td>
   <td style="text-align:left;"> ConstructionStartAt </td>
   <td style="text-align:left;"> 1974-05-16 </td>
   <td style="text-align:left;"> under construction </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Asco-1 </td>
   <td style="text-align:left;"> OperationalFrom </td>
   <td style="text-align:left;"> 1984-12-10 </td>
   <td style="text-align:left;"> active </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Asco-2 </td>
   <td style="text-align:left;"> ConstructionStartAt </td>
   <td style="text-align:left;"> 1975-03-07 </td>
   <td style="text-align:left;"> under construction </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Asco-2 </td>
   <td style="text-align:left;"> OperationalFrom </td>
   <td style="text-align:left;"> 1986-03-31 </td>
   <td style="text-align:left;"> active </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Atucha-1 </td>
   <td style="text-align:left;"> ConstructionStartAt </td>
   <td style="text-align:left;"> 1968-06-01 </td>
   <td style="text-align:left;"> under construction </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Atucha-1 </td>
   <td style="text-align:left;"> OperationalFrom </td>
   <td style="text-align:left;"> 1974-06-24 </td>
   <td style="text-align:left;"> active </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Atucha-2 </td>
   <td style="text-align:left;"> ConstructionStartAt </td>
   <td style="text-align:left;"> 1981-07-14 </td>
   <td style="text-align:left;"> under construction </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Atucha-2 </td>
   <td style="text-align:left;"> OperationalFrom </td>
   <td style="text-align:left;"> 2016-05-26 </td>
   <td style="text-align:left;"> active </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Balakovo-1 </td>
   <td style="text-align:left;"> ConstructionStartAt </td>
   <td style="text-align:left;"> 1980-12-01 </td>
   <td style="text-align:left;"> under construction </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Balakovo-1 </td>
   <td style="text-align:left;"> OperationalFrom </td>
   <td style="text-align:left;"> 1986-05-23 </td>
   <td style="text-align:left;"> active </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Balakovo-2 </td>
   <td style="text-align:left;"> ConstructionStartAt </td>
   <td style="text-align:left;"> 1981-08-01 </td>
   <td style="text-align:left;"> under construction </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Balakovo-2 </td>
   <td style="text-align:left;"> OperationalFrom </td>
   <td style="text-align:left;"> 1988-01-18 </td>
   <td style="text-align:left;"> active </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Balakovo-3 </td>
   <td style="text-align:left;"> ConstructionStartAt </td>
   <td style="text-align:left;"> 1982-11-01 </td>
   <td style="text-align:left;"> under construction </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Balakovo-3 </td>
   <td style="text-align:left;"> OperationalFrom </td>
   <td style="text-align:left;"> 1989-04-08 </td>
   <td style="text-align:left;"> active </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Balakovo-4 </td>
   <td style="text-align:left;"> ConstructionStartAt </td>
   <td style="text-align:left;"> 1984-04-01 </td>
   <td style="text-align:left;"> under construction </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Balakovo-4 </td>
   <td style="text-align:left;"> OperationalFrom </td>
   <td style="text-align:left;"> 1993-12-22 </td>
   <td style="text-align:left;"> active </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Baltic-1 </td>
   <td style="text-align:left;"> ConstructionStartAt </td>
   <td style="text-align:left;"> 2012-02-22 </td>
   <td style="text-align:left;"> under construction </td>
  </tr>
</tbody>
</table></div>




Zum Schluss erweitere ich noch die Grunddaten um die eingespeiste Energiemenge. Zunächst erstelle ich einen Vektor mit den relevanten Reaktor-IDs. Diese werden jeweils an die URL der PRIS-Datenbank angehangen, sodass der jeweilige Reaktor aufgerufen wird. Anschließend fasse ich alle extrahierten Daten im Tibble `operatingHistory` zusammen.



```r
# relevante Reaktoren (IEAD ID)
reactor_ids <- 
  data_wide %>%
  select(IAEAId) %>%
  filter(!is.na(IAEAId)) %>%
  arrange(IAEAId) %>%
  pull()

pages <- tibble(
  IAEAId = reactor_ids,
  url = str_c("https://pris.iaea.org/PRIS/CountryStatistics/ReactorDetails.aspx?current=",
              reactor_ids),
  running_id = 1:length(reactor_ids)
  )

operatingHistory <- purrr::map_dfr(.x = pages$url, 
                                   .f = ~scraper_pris(url = .x), 
                                   .id = "IAEAId") 

operatingHistory <- operatingHistory %>%
  mutate(running_id = as.numeric(running_id)) %>%
  left_join(pages, by="running_id")
```


Die extrahierten Daten werden dann mit den Basisdaten der Reaktoren verknüpft und sehen so aus (ausgewählte Spalten):



```r
data_oH <-
  data_wide %>%
  left_join(operatingHistory, by = "IAEAId")
```


<br>


<div style="border: 1px solid #ddd; padding: 0px; overflow-y: scroll; height:300px; overflow-x: scroll; width:100%; "><table class="table table-responsive" style="font-size: 10px; margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;"> Name </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;"> Latitude </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;"> Longitude </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;"> Capacity </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;"> year </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;"> electricity </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> Ågesta </td>
   <td style="text-align:right;"> 59.20600 </td>
   <td style="text-align:right;"> 18.0829 </td>
   <td style="text-align:right;"> 9 </td>
   <td style="text-align:right;"> 1971 </td>
   <td style="text-align:right;"> 30.00 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Ågesta </td>
   <td style="text-align:right;"> 59.20600 </td>
   <td style="text-align:right;"> 18.0829 </td>
   <td style="text-align:right;"> 9 </td>
   <td style="text-align:right;"> 1972 </td>
   <td style="text-align:right;"> 49.00 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Ågesta </td>
   <td style="text-align:right;"> 59.20600 </td>
   <td style="text-align:right;"> 18.0829 </td>
   <td style="text-align:right;"> 9 </td>
   <td style="text-align:right;"> 1973 </td>
   <td style="text-align:right;"> 43.50 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Ågesta </td>
   <td style="text-align:right;"> 59.20600 </td>
   <td style="text-align:right;"> 18.0829 </td>
   <td style="text-align:right;"> 9 </td>
   <td style="text-align:right;"> 1974 </td>
   <td style="text-align:right;"> 18.10 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Akademik Lomonosov-1 </td>
   <td style="text-align:right;"> 69.70958 </td>
   <td style="text-align:right;"> 170.3063 </td>
   <td style="text-align:right;"> 30 </td>
   <td style="text-align:right;"> 2019 </td>
   <td style="text-align:right;"> 0.68 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Akademik Lomonosov-1 </td>
   <td style="text-align:right;"> 69.70958 </td>
   <td style="text-align:right;"> 170.3063 </td>
   <td style="text-align:right;"> 30 </td>
   <td style="text-align:right;"> 2020 </td>
   <td style="text-align:right;"> 64.93 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Akademik Lomonosov-2 </td>
   <td style="text-align:right;"> 69.70958 </td>
   <td style="text-align:right;"> 170.3063 </td>
   <td style="text-align:right;"> 30 </td>
   <td style="text-align:right;"> 2019 </td>
   <td style="text-align:right;"> 2.12 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Akademik Lomonosov-2 </td>
   <td style="text-align:right;"> 69.70958 </td>
   <td style="text-align:right;"> 170.3063 </td>
   <td style="text-align:right;"> 30 </td>
   <td style="text-align:right;"> 2020 </td>
   <td style="text-align:right;"> 51.44 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Aktau (Shevchenko) </td>
   <td style="text-align:right;"> 43.60700 </td>
   <td style="text-align:right;"> 51.2830 </td>
   <td style="text-align:right;"> 52 </td>
   <td style="text-align:right;"> 1992 </td>
   <td style="text-align:right;"> 463.94 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Aktau (Shevchenko) </td>
   <td style="text-align:right;"> 43.60700 </td>
   <td style="text-align:right;"> 51.2830 </td>
   <td style="text-align:right;"> 52 </td>
   <td style="text-align:right;"> 1993 </td>
   <td style="text-align:right;"> 444.11 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Aktau (Shevchenko) </td>
   <td style="text-align:right;"> 43.60700 </td>
   <td style="text-align:right;"> 51.2830 </td>
   <td style="text-align:right;"> 52 </td>
   <td style="text-align:right;"> 1994 </td>
   <td style="text-align:right;"> 378.07 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Aktau (Shevchenko) </td>
   <td style="text-align:right;"> 43.60700 </td>
   <td style="text-align:right;"> 51.2830 </td>
   <td style="text-align:right;"> 52 </td>
   <td style="text-align:right;"> 1995 </td>
   <td style="text-align:right;"> 83.19 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Aktau (Shevchenko) </td>
   <td style="text-align:right;"> 43.60700 </td>
   <td style="text-align:right;"> 51.2830 </td>
   <td style="text-align:right;"> 52 </td>
   <td style="text-align:right;"> 1996 </td>
   <td style="text-align:right;"> 89.58 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Aktau (Shevchenko) </td>
   <td style="text-align:right;"> 43.60700 </td>
   <td style="text-align:right;"> 51.2830 </td>
   <td style="text-align:right;"> 52 </td>
   <td style="text-align:right;"> 1997 </td>
   <td style="text-align:right;"> 302.75 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Aktau (Shevchenko) </td>
   <td style="text-align:right;"> 43.60700 </td>
   <td style="text-align:right;"> 51.2830 </td>
   <td style="text-align:right;"> 52 </td>
   <td style="text-align:right;"> 1998 </td>
   <td style="text-align:right;"> 91.16 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Aktau (Shevchenko) </td>
   <td style="text-align:right;"> 43.60700 </td>
   <td style="text-align:right;"> 51.2830 </td>
   <td style="text-align:right;"> 52 </td>
   <td style="text-align:right;"> 1999 </td>
   <td style="text-align:right;"> 0.00 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Almaraz-1 </td>
   <td style="text-align:right;"> 39.80700 </td>
   <td style="text-align:right;"> -5.6980 </td>
   <td style="text-align:right;"> 900 </td>
   <td style="text-align:right;"> 1981 </td>
   <td style="text-align:right;"> 1888.90 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Almaraz-1 </td>
   <td style="text-align:right;"> 39.80700 </td>
   <td style="text-align:right;"> -5.6980 </td>
   <td style="text-align:right;"> 900 </td>
   <td style="text-align:right;"> 1982 </td>
   <td style="text-align:right;"> 2322.60 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Almaraz-1 </td>
   <td style="text-align:right;"> 39.80700 </td>
   <td style="text-align:right;"> -5.6980 </td>
   <td style="text-align:right;"> 900 </td>
   <td style="text-align:right;"> 1983 </td>
   <td style="text-align:right;"> 3713.90 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Almaraz-1 </td>
   <td style="text-align:right;"> 39.80700 </td>
   <td style="text-align:right;"> -5.6980 </td>
   <td style="text-align:right;"> 900 </td>
   <td style="text-align:right;"> 1984 </td>
   <td style="text-align:right;"> 4820.50 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Almaraz-1 </td>
   <td style="text-align:right;"> 39.80700 </td>
   <td style="text-align:right;"> -5.6980 </td>
   <td style="text-align:right;"> 900 </td>
   <td style="text-align:right;"> 1985 </td>
   <td style="text-align:right;"> 4825.17 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Almaraz-1 </td>
   <td style="text-align:right;"> 39.80700 </td>
   <td style="text-align:right;"> -5.6980 </td>
   <td style="text-align:right;"> 900 </td>
   <td style="text-align:right;"> 1986 </td>
   <td style="text-align:right;"> 5425.02 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Almaraz-1 </td>
   <td style="text-align:right;"> 39.80700 </td>
   <td style="text-align:right;"> -5.6980 </td>
   <td style="text-align:right;"> 900 </td>
   <td style="text-align:right;"> 1987 </td>
   <td style="text-align:right;"> 7193.69 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Almaraz-1 </td>
   <td style="text-align:right;"> 39.80700 </td>
   <td style="text-align:right;"> -5.6980 </td>
   <td style="text-align:right;"> 900 </td>
   <td style="text-align:right;"> 1988 </td>
   <td style="text-align:right;"> 5879.59 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Almaraz-1 </td>
   <td style="text-align:right;"> 39.80700 </td>
   <td style="text-align:right;"> -5.6980 </td>
   <td style="text-align:right;"> 900 </td>
   <td style="text-align:right;"> 1989 </td>
   <td style="text-align:right;"> 6562.18 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Almaraz-1 </td>
   <td style="text-align:right;"> 39.80700 </td>
   <td style="text-align:right;"> -5.6980 </td>
   <td style="text-align:right;"> 900 </td>
   <td style="text-align:right;"> 1990 </td>
   <td style="text-align:right;"> 6460.66 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Almaraz-1 </td>
   <td style="text-align:right;"> 39.80700 </td>
   <td style="text-align:right;"> -5.6980 </td>
   <td style="text-align:right;"> 900 </td>
   <td style="text-align:right;"> 1991 </td>
   <td style="text-align:right;"> 7481.71 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Almaraz-1 </td>
   <td style="text-align:right;"> 39.80700 </td>
   <td style="text-align:right;"> -5.6980 </td>
   <td style="text-align:right;"> 900 </td>
   <td style="text-align:right;"> 1992 </td>
   <td style="text-align:right;"> 6379.06 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Almaraz-1 </td>
   <td style="text-align:right;"> 39.80700 </td>
   <td style="text-align:right;"> -5.6980 </td>
   <td style="text-align:right;"> 900 </td>
   <td style="text-align:right;"> 1993 </td>
   <td style="text-align:right;"> 6530.85 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Almaraz-1 </td>
   <td style="text-align:right;"> 39.80700 </td>
   <td style="text-align:right;"> -5.6980 </td>
   <td style="text-align:right;"> 900 </td>
   <td style="text-align:right;"> 1994 </td>
   <td style="text-align:right;"> 7448.60 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Almaraz-1 </td>
   <td style="text-align:right;"> 39.80700 </td>
   <td style="text-align:right;"> -5.6980 </td>
   <td style="text-align:right;"> 900 </td>
   <td style="text-align:right;"> 1995 </td>
   <td style="text-align:right;"> 6588.46 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Almaraz-1 </td>
   <td style="text-align:right;"> 39.80700 </td>
   <td style="text-align:right;"> -5.6980 </td>
   <td style="text-align:right;"> 900 </td>
   <td style="text-align:right;"> 1996 </td>
   <td style="text-align:right;"> 5904.30 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Almaraz-1 </td>
   <td style="text-align:right;"> 39.80700 </td>
   <td style="text-align:right;"> -5.6980 </td>
   <td style="text-align:right;"> 900 </td>
   <td style="text-align:right;"> 1997 </td>
   <td style="text-align:right;"> 6642.83 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Almaraz-1 </td>
   <td style="text-align:right;"> 39.80700 </td>
   <td style="text-align:right;"> -5.6980 </td>
   <td style="text-align:right;"> 900 </td>
   <td style="text-align:right;"> 1998 </td>
   <td style="text-align:right;"> 8032.46 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Almaraz-1 </td>
   <td style="text-align:right;"> 39.80700 </td>
   <td style="text-align:right;"> -5.6980 </td>
   <td style="text-align:right;"> 900 </td>
   <td style="text-align:right;"> 1999 </td>
   <td style="text-align:right;"> 6988.63 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Almaraz-1 </td>
   <td style="text-align:right;"> 39.80700 </td>
   <td style="text-align:right;"> -5.6980 </td>
   <td style="text-align:right;"> 900 </td>
   <td style="text-align:right;"> 2000 </td>
   <td style="text-align:right;"> 7471.57 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Almaraz-1 </td>
   <td style="text-align:right;"> 39.80700 </td>
   <td style="text-align:right;"> -5.6980 </td>
   <td style="text-align:right;"> 900 </td>
   <td style="text-align:right;"> 2001 </td>
   <td style="text-align:right;"> 8151.39 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Almaraz-1 </td>
   <td style="text-align:right;"> 39.80700 </td>
   <td style="text-align:right;"> -5.6980 </td>
   <td style="text-align:right;"> 900 </td>
   <td style="text-align:right;"> 2002 </td>
   <td style="text-align:right;"> 7427.99 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Almaraz-1 </td>
   <td style="text-align:right;"> 39.80700 </td>
   <td style="text-align:right;"> -5.6980 </td>
   <td style="text-align:right;"> 900 </td>
   <td style="text-align:right;"> 2003 </td>
   <td style="text-align:right;"> 7499.11 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Almaraz-1 </td>
   <td style="text-align:right;"> 39.80700 </td>
   <td style="text-align:right;"> -5.6980 </td>
   <td style="text-align:right;"> 900 </td>
   <td style="text-align:right;"> 2004 </td>
   <td style="text-align:right;"> 8185.69 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Almaraz-1 </td>
   <td style="text-align:right;"> 39.80700 </td>
   <td style="text-align:right;"> -5.6980 </td>
   <td style="text-align:right;"> 900 </td>
   <td style="text-align:right;"> 2005 </td>
   <td style="text-align:right;"> 7519.43 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Almaraz-1 </td>
   <td style="text-align:right;"> 39.80700 </td>
   <td style="text-align:right;"> -5.6980 </td>
   <td style="text-align:right;"> 900 </td>
   <td style="text-align:right;"> 2006 </td>
   <td style="text-align:right;"> 7152.42 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Almaraz-1 </td>
   <td style="text-align:right;"> 39.80700 </td>
   <td style="text-align:right;"> -5.6980 </td>
   <td style="text-align:right;"> 900 </td>
   <td style="text-align:right;"> 2007 </td>
   <td style="text-align:right;"> 8189.80 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Almaraz-1 </td>
   <td style="text-align:right;"> 39.80700 </td>
   <td style="text-align:right;"> -5.6980 </td>
   <td style="text-align:right;"> 900 </td>
   <td style="text-align:right;"> 2008 </td>
   <td style="text-align:right;"> 7190.76 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Almaraz-1 </td>
   <td style="text-align:right;"> 39.80700 </td>
   <td style="text-align:right;"> -5.6980 </td>
   <td style="text-align:right;"> 900 </td>
   <td style="text-align:right;"> 2009 </td>
   <td style="text-align:right;"> 6880.10 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Almaraz-1 </td>
   <td style="text-align:right;"> 39.80700 </td>
   <td style="text-align:right;"> -5.6980 </td>
   <td style="text-align:right;"> 900 </td>
   <td style="text-align:right;"> 2010 </td>
   <td style="text-align:right;"> 7884.25 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Almaraz-1 </td>
   <td style="text-align:right;"> 39.80700 </td>
   <td style="text-align:right;"> -5.6980 </td>
   <td style="text-align:right;"> 900 </td>
   <td style="text-align:right;"> 2011 </td>
   <td style="text-align:right;"> 7519.49 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Almaraz-1 </td>
   <td style="text-align:right;"> 39.80700 </td>
   <td style="text-align:right;"> -5.6980 </td>
   <td style="text-align:right;"> 900 </td>
   <td style="text-align:right;"> 2012 </td>
   <td style="text-align:right;"> 7346.07 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Almaraz-1 </td>
   <td style="text-align:right;"> 39.80700 </td>
   <td style="text-align:right;"> -5.6980 </td>
   <td style="text-align:right;"> 900 </td>
   <td style="text-align:right;"> 2013 </td>
   <td style="text-align:right;"> 7695.84 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Almaraz-1 </td>
   <td style="text-align:right;"> 39.80700 </td>
   <td style="text-align:right;"> -5.6980 </td>
   <td style="text-align:right;"> 900 </td>
   <td style="text-align:right;"> 2014 </td>
   <td style="text-align:right;"> 7252.45 </td>
  </tr>
</tbody>
</table></div>




Für die Datenanalyse möchte ich ausschließlich Reaktoren berücksichtigen, die sich in Europa befinden und reduziere die Datensätze auf die relevanten Länder.



```r
data_wide_eu <-
  data_wide %>%
  filter(CountryName %in% c(
    "Austria","Belarus","Belgium","Bulgaria",
    "Czechia","Finland","France","Germany","Hungary",
    "Italy","Lithunia","Netherlands","Romania",
    "Slovakia","Slovenia","Spain","Sweden",
    "Switzerland","Turkey","Ukraine","United Kingdom")
    ) %>%
  mutate(StatusType = ifelse(
    StatusType %in% c("Cancelled Construction",
                      "Never Commissioned",
                      "Suspended Construction",
                      "Unknown"), 
    "Not Built", StatusType),
    StatusType = fct_relevel(StatusType, 
                             "Not Built", "Planned",
                             "Under Construction",
                             "Operational", "Shutdown")
    )

data_long_eu <-
  data_long %>%
  filter(CountryName %in% c(
    "Austria","Belarus","Belgium","Bulgaria",
    "Czechia","Finland","France","Germany","Hungary",
    "Italy","Lithunia","Netherlands","Romania",
    "Slovakia","Slovenia","Spain","Sweden",
    "Switzerland","Turkey","Ukraine","United Kingdom")
    )

data_oH_eu <- 
  data_oH %>%
  filter(CountryName %in% c(
    "Austria","Belarus","Belgium","Bulgaria",
    "Czechia","Finland","France","Germany","Hungary",
    "Italy","Lithunia","Netherlands","Romania",
    "Slovakia","Slovenia","Spain","Sweden",
    "Switzerland","Turkey","Ukraine","United Kingdom")
    )
```


## **Datenanalyse**

Unten findet man eine Übersicht der europäischen Atomreaktoren. 103 Reaktoren wurden bereits stillgelegt und 7 Reaktoren sind nie in Betrieb gegangen bzw. haben das Planungsstadium verlassen. Dennoch befinden sich noch 141 Reaktoren in Betrieb und sogar 12 Reaktoren in Bau bzw. 9 in Planung.



```r
data_wide_eu %>%
  group_by(StatusType) %>%
  summarise(anzahl = n()) %>%
  mutate(anteil = round(anzahl / sum(anzahl), digits = 2)) %>%
  ggplot(data = ., aes(x = "", y = anteil, fill = StatusType)) +
  geom_col(color = "black") +
  geom_text(aes(label = anzahl),
            position = position_stack(vjust = 0.5)) +
  coord_polar(theta = "y") +
  scale_fill_manual(values = c("grey", "#3399FF","#FFCC00","#00CC00","#CC0000")) +
  labs(fill = "") +
  theme_void()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-12-1.png" width="672" style="display: block; margin: auto;" />


***

Für jedes europäische Land ist die geplante bzw. realisierte Leistung aller Reaktoren nachfolgend dargestellt. Besonders in Deutschland wurden bereits viele Reaktoren vom Netz genommen, während sich in Frankreich der Großteil der Reaktoren noch im Betrieb befindet.



```r
data_wide_eu %>%
  group_by(CountryName) %>%
  summarise(tot_capa_country = sum(Capacity, na.rm = TRUE)/1000) %>%
  left_join(data_wide_eu %>%
              group_by(CountryName, StatusType) %>%
              summarise(tot_capa_status = sum(Capacity, na.rm = TRUE)/1000), 
            by = "CountryName") %>%
  ggplot(data = ., aes(x = reorder(CountryName, tot_capa_country), 
                       y = tot_capa_status, 
                       fill =StatusType)) +
  geom_bar(stat = "identity", color = "black") +
  coord_flip() +
  scale_y_continuous(breaks = seq(0,70,10)) +
  scale_fill_manual(values = c("grey", "#3399FF","#FFCC00","#00CC00","#CC0000")) +
  labs(x = "", y = "Total capacity (gigawatt)", fill = "")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-13-1.png" width="672" style="display: block; margin: auto;" />


***

Als Nächstes betrachte ich die ins Netz eingespeiste Energiemenge der europäischen Atommeiler. Auf der x-Achse habe ich die Reaktorunfälle von Tschernobyl im Jahr 1986 und Fukushima im Jahr 2011 in rot hervorgehoben. Nur Italien ist im Anchschluss an Tschernobyl vollständig aus der Kernergie ausgestiegen. Bei den anderen europäischen Ländern ist kein Rückgang der Nutzung der Atomkraft zu erkennen - in Frankreich ist diese sogar deutlich gestiegen. In Deutschland ist ein starker Rückgang der eingespeisten Energiemenge nach dem Reaktorunglück von Fukushima zu erkennen.



```r
data_oH_eu %>%
  group_by(CountryName, year) %>%
  summarise(sum_ele_europe = sum(electricity, na.rm = T)/1000) %>%
  ggplot(data =., aes(x= year, y = sum_ele_europe, group = CountryName)) +
  geom_line() +
  geom_vline(xintercept=1986, color = "red") +
  geom_vline(xintercept = 2011, color = "red") +
  facet_wrap(. ~ CountryName) +
  scale_x_continuous(limits = c(1950, 2022),
                     breaks = seq(1960,2020,10)) +
  labs(x = "", y = "Electricity supplied (TW/h)") +
  theme(axis.text.x = element_text(angle = 90))
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-14-1.png" width="672" style="display: block; margin: auto;" />



***

Zum Schluss möchte ich mir einen Überblick verschaffen, wie sich der Ausbau der Kernergie in Europa im Zeitverlauf verändert hat. Dazu nutze ich den Datensatz `datal_long_eu`, in welchem ich die Reaktordaten so transfomiert hatte, dass sich der Zeitpunkt des Baus, der Inbetriebesetzung oder der Stilllegung wie eine Zeitreihe verhält. Unten ist dargestellt bei wie vielen Reaktoren in einem Jahr mit dem Bau begonnen wurde bzw. wie viele Reaktoren in Betrieb gesetzt oder stillgelegt wurden. Zwischen 1970 und 1990 wurden die meisten Reaktoren gebaut bzw. in Betrieb genommen. Ab 1990 wurden zunehmend Reaktoren stillgelegt.



```r
data_long_eu %>%
  ggplot(aes(x = Year, fill = TimeEvent_Animation)) +
  geom_bar(color = "black") +
  scale_x_continuous(limits = c(1950, 2022),
                     breaks = seq(1950,2022,10)) +
  scale_fill_manual(name = "", values = c("#00CC00","#CC0000","#FFCC00")) +
  labs(x = "", y = "Number of reactors")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-15-1.png" width="672" style="display: block; margin: auto;" />


***

Unten habe ich für Deutschland die Entwicklung der Atomreaktoren mit einer Leistung von mindestens 300 Megawatt auf einem Zeitstrahl dargestellt. Dafür nutze ich die Funktion `geom_segment()` mit der eine gerade Linie zwischen zwei Punkten gezeichnet werden kann. Das Ende des jeweiligen Ereignisses auf dem Zeitstrahl begrenze ich mithilfe der Funktion `lead()` auf das nächstfolgende Ereignis. Aktuell sind in Deutschland noch 7 Reaktoren am Netz. Besonders interessant finde ich das Kernkraftwerk Mülheim-Kärlich, welches nach seiner Inbetriebnahme nur etwa 1 Jahr am Netz war. Auch die nach dem Reaktorunglück von Fukushima (2011) stillgelegten Reaktoren können dieser Darstellung entnommen werden.



```r
data_long_eu %>%
  group_by(Name) %>%
  mutate(End = lead(Year),
         End = ifelse(is.na(End),2021,End)) %>%
  filter(CountryName =="Germany" & Capacity >= 300) %>%
  ggplot() +
  geom_segment(aes(x = Year, xend = End, 
                   y = Name, yend = Name, 
                   col = TimeEvent_Animation), 
               lineend = "round", size = 2) +
  scale_color_manual(name = "", values = c("#00CC00","#CC0000","#FFCC00")) +
  scale_x_continuous(limits = c(1965, 2021),
                     breaks = seq(1965,2020,5)) +
  labs(x = "", y = "") +
  theme(axis.text.x = element_text(angle = 90))
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-16-1.png" width="672" style="display: block; margin: auto;" />


***

Mit einer animierten Karte möchte ich diese Informationen für ganz Europa darstellen. Mit der Funktion `shadow_mark()` aus dem R-Paket `ggmaps` kann man steuern, dass Punkte, die ein mal auf einer Karte hinterlegt wurden, auch im Zeitverlauf auf der Karte verbleiben. So bleibt der Status eines Reaktors so lange gültig bis ein anderer Status eintritt. Da es am Standort eines Atomkraftwerks meist mehrere Reaktoren gibt, ist diese Darstellung nicht vollständig präzise, da die Reaktoren nicht unbedingt zur selben Zeit gebaut oder in Betrieb genommen werden. So kann ein zweiter (dritter, vierter, ...) Reaktor an einem Standort den Status des ersten Reaktors überscheiben.



```r
Get_Map_Country <- get_map(location = "Europe", zoom = 4, 
                           maptype = "stamen_toner_background", 
                           filename = "ggmapTemp", color = "bw", source = "stadia")

karte_europa <- ggmap(ggmap = Get_Map_Country)

karte_animation <- karte_europa + 
  geom_point(data = data_long_eu, 
             aes(x = Longitude, 
                 y = Latitude, 
                 fill = TimeEvent_Animation,
                 group = Year), 
             shape = 21, size = 3, alpha = 1.0) +
  transition_time(Year) +
  scale_fill_manual(values = c("#00FF00","#FF0000","#FFFF00")) +
  labs(title = "Reaktorstandorte in Europa im Jahr = {as.integer(frame_time)}",
       x = "Längengrad", 
       y = "Breitengrad",
       fill = "") +
  shadow_mark(past = TRUE) 

animate(karte_animation, nframes = 100, fps = 2,
        width = 800,
        height = 600)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-17-1.gif" style="display: block; margin: auto;" />


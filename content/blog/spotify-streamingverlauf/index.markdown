---
title: "Spotify Streaming Verlauf"
date: 2021-06-11
format: hugo
slug: spotify-streamingverlauf
categories: 
  - R
  - Freizeit
summary: "Eine Auswertung der Streaming-Daten, die Spotify sammelt."
---
<script src="{{< blogdown/postref >}}index_files/kePrint/kePrint.js"></script>
<script src="{{< blogdown/postref >}}index_files/kePrint/kePrint.js"></script>




Benötigte R-Pakete:



```r
library(tidyverse)
library(jsonlite)
library(lubridate)
library(gghighlight)
```


## **Datenbeschaffung**

Vor kurzem hat meine Freundin beim Musikstreaminganbieter *Spotify* ihre Nutzerdaten beantragt und heruntergeladen. Eine Schritt-für-Schritt-Anleitung wie das funktioniert, findet man in den Account-Einstellungen unter [Datenschutz](https://www.spotify.com/de/account/privacy/). Laut Spotify umfasst der Downlad:

> [...] eine Kopie deiner Playlists, deiner Suchanfragen, deines Streaming-Verlaufs des letzten Jahres, eine Liste der in deiner Bibliothek gespeicherten Elemente, die Zahl deiner Follower, die Zahl und die Namen der anderen Nutzer\*innen und Künstler\*innen, denen du folgst, sowie deine Zahlungs- und Abodaten.

Von Spotify bekommt man nach etwa 30 Tagen einen Link zu einer *.zip* Datei zur Verfügung gestellt, die ein Readme im PDF-Format sowie mehrere *.json* Dateien enthält. Für die meisten Spotify-Nutzer\*innen ist das wahrscheinlich nicht das geläufigste Dateiformat. Die Dateien können zwar mit einem Texteditor betrachtet werden, jedoch lässt sich bspw. der *Streaming-Verlauf* aufgrund seiner Größe nicht gerade leicht erfassen.

| ![Eintrag im Streaming-Verlauf](/img/spotify.jpg) |
|:--:| 
| *Eintrag im Streaming-Verlauf* |

Aus diesem Grund habe ich ein R-Skript erstellt, mit welchem die von Spotify zur Verfügung gestellten Dateien ausgewertet werden können.

## **Datenaufbereitung**

Zunächst lese ich den Streaming-Verlauf mit der Funktion `fromJSON()` aus dem R-Paket `jsonlite` ein. Abhängig von den eigenen Hörgewohnheiten kann es eine oder mehrere Dateien für den Streaming-Verlauf geben. Nach dem Einlesen der Dateien kombiniere ich die Verläufe zu einem einzigen Datenbestand.



```r
streamVerlauf1 <- fromJSON("data/StreamingHistory0.json", flatten = TRUE)
streamVerlauf2 <- fromJSON("data/StreamingHistory1.json", flatten = TRUE)
streamVerlauf3 <- fromJSON("data/StreamingHistory2.json", flatten = TRUE)

streamVerlauf_spotify <- bind_rows(streamVerlauf1,
                                   streamVerlauf2,
                                   streamVerlauf3)
```


Unten ist ein Ausschnitt der Daten dargestellt. Jede Zeile in diesem Datensatz repräsentiert dabei einen Zeitpunkt (`endTime`) an dem ein Musiktitel (`trackName`) eines Interpreten (`artistName`) abgespielt wurde. Auch die Abspieldauer des jeweiligen Titels ist vorhanden (`msPlayed`).


<div style="border: 1px solid #ddd; padding: 0px; overflow-y: scroll; height:300px; overflow-x: scroll; width:100%; "><table class="table table-responsive" style="font-size: 10px; margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;"> endTime </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;"> artistName </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;"> trackName </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;"> msPlayed </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> 2021-01-15 17:40 </td>
   <td style="text-align:left;"> Kate Bush </td>
   <td style="text-align:left;"> Wuthering Heights </td>
   <td style="text-align:right;"> 269066 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-15 17:44 </td>
   <td style="text-align:left;"> Kate Nash </td>
   <td style="text-align:left;"> Mariella </td>
   <td style="text-align:right;"> 255546 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-15 17:47 </td>
   <td style="text-align:left;"> Maisie Peters </td>
   <td style="text-align:left;"> In My Head </td>
   <td style="text-align:right;"> 187827 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-15 17:51 </td>
   <td style="text-align:left;"> Yeah Yeah Yeahs </td>
   <td style="text-align:left;"> Heads Will Roll </td>
   <td style="text-align:right;"> 221000 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-15 17:54 </td>
   <td style="text-align:left;"> SUNMI </td>
   <td style="text-align:left;"> Siren </td>
   <td style="text-align:right;"> 199133 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-15 17:58 </td>
   <td style="text-align:left;"> Stromae </td>
   <td style="text-align:left;"> sommeil </td>
   <td style="text-align:right;"> 218653 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-15 18:03 </td>
   <td style="text-align:left;"> Wir Sind Helden </td>
   <td style="text-align:left;"> Ein Elefant für dich </td>
   <td style="text-align:right;"> 282653 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-15 18:05 </td>
   <td style="text-align:left;"> St. Vincent </td>
   <td style="text-align:left;"> New York </td>
   <td style="text-align:right;"> 123035 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 10:55 </td>
   <td style="text-align:left;"> Lorde </td>
   <td style="text-align:left;"> Hard Feelings/Loveless </td>
   <td style="text-align:right;"> 367391 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 10:58 </td>
   <td style="text-align:left;"> David Bowie </td>
   <td style="text-align:left;"> Life on Mars? - 2015 Remaster </td>
   <td style="text-align:right;"> 133913 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 11:14 </td>
   <td style="text-align:left;"> David Bowie </td>
   <td style="text-align:left;"> Life on Mars? - 2015 Remaster </td>
   <td style="text-align:right;"> 235986 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 11:17 </td>
   <td style="text-align:left;"> Ezra Furman </td>
   <td style="text-align:left;"> Love You So Bad </td>
   <td style="text-align:right;"> 219000 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 11:21 </td>
   <td style="text-align:left;"> Frog </td>
   <td style="text-align:left;"> Judy Garland </td>
   <td style="text-align:right;"> 217177 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 11:24 </td>
   <td style="text-align:left;"> Wings </td>
   <td style="text-align:left;"> Live And Let Die </td>
   <td style="text-align:right;"> 194613 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 11:29 </td>
   <td style="text-align:left;"> Chappell Roan </td>
   <td style="text-align:left;"> Pink Pony Club </td>
   <td style="text-align:right;"> 258034 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 11:32 </td>
   <td style="text-align:left;"> Kate Bush </td>
   <td style="text-align:left;"> Babooshka - 2018 Remaster </td>
   <td style="text-align:right;"> 199226 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 11:33 </td>
   <td style="text-align:left;"> Dominic Fike </td>
   <td style="text-align:left;"> 10x Stronger </td>
   <td style="text-align:right;"> 75585 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 11:37 </td>
   <td style="text-align:left;"> Lorde </td>
   <td style="text-align:left;"> Writer In The Dark </td>
   <td style="text-align:right;"> 216610 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 11:40 </td>
   <td style="text-align:left;"> BTS </td>
   <td style="text-align:left;"> Fly To My Room </td>
   <td style="text-align:right;"> 222242 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 11:44 </td>
   <td style="text-align:left;"> Halsey </td>
   <td style="text-align:left;"> I HATE EVERYBODY </td>
   <td style="text-align:right;"> 171015 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 11:48 </td>
   <td style="text-align:left;"> Kate Bush </td>
   <td style="text-align:left;"> Wuthering Heights </td>
   <td style="text-align:right;"> 269066 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 11:52 </td>
   <td style="text-align:left;"> Kate Nash </td>
   <td style="text-align:left;"> Mariella </td>
   <td style="text-align:right;"> 255546 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 11:56 </td>
   <td style="text-align:left;"> Maisie Peters </td>
   <td style="text-align:left;"> In My Head </td>
   <td style="text-align:right;"> 187827 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 11:59 </td>
   <td style="text-align:left;"> Yeah Yeah Yeahs </td>
   <td style="text-align:left;"> Heads Will Roll </td>
   <td style="text-align:right;"> 221000 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 12:03 </td>
   <td style="text-align:left;"> SUNMI </td>
   <td style="text-align:left;"> Siren </td>
   <td style="text-align:right;"> 199133 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 12:06 </td>
   <td style="text-align:left;"> Stromae </td>
   <td style="text-align:left;"> sommeil </td>
   <td style="text-align:right;"> 218653 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 12:07 </td>
   <td style="text-align:left;"> Wir Sind Helden </td>
   <td style="text-align:left;"> Ein Elefant für dich </td>
   <td style="text-align:right;"> 50533 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 12:11 </td>
   <td style="text-align:left;"> Wir Sind Helden </td>
   <td style="text-align:left;"> Wenn es passiert </td>
   <td style="text-align:right;"> 212360 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 12:15 </td>
   <td style="text-align:left;"> Wir Sind Helden </td>
   <td style="text-align:left;"> Echolot </td>
   <td style="text-align:right;"> 271386 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 12:19 </td>
   <td style="text-align:left;"> Wir Sind Helden </td>
   <td style="text-align:left;"> Von hier an blind </td>
   <td style="text-align:right;"> 210666 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 12:22 </td>
   <td style="text-align:left;"> Wir Sind Helden </td>
   <td style="text-align:left;"> Zuhälter </td>
   <td style="text-align:right;"> 210306 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 12:27 </td>
   <td style="text-align:left;"> Wir Sind Helden </td>
   <td style="text-align:left;"> Ein Elefant für dich </td>
   <td style="text-align:right;"> 282653 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 12:30 </td>
   <td style="text-align:left;"> Wir Sind Helden </td>
   <td style="text-align:left;"> Darf ich das behalten </td>
   <td style="text-align:right;"> 198466 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 12:36 </td>
   <td style="text-align:left;"> Wir Sind Helden </td>
   <td style="text-align:left;"> Wütend genug </td>
   <td style="text-align:right;"> 269386 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 12:37 </td>
   <td style="text-align:left;"> Wir Sind Helden </td>
   <td style="text-align:left;"> Geht auseinander </td>
   <td style="text-align:right;"> 26645 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 12:40 </td>
   <td style="text-align:left;"> Wir Sind Helden </td>
   <td style="text-align:left;"> Gekommen um zu bleiben </td>
   <td style="text-align:right;"> 190120 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 12:44 </td>
   <td style="text-align:left;"> Wir Sind Helden </td>
   <td style="text-align:left;"> Nur ein Wort </td>
   <td style="text-align:right;"> 236200 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 12:47 </td>
   <td style="text-align:left;"> Wir Sind Helden </td>
   <td style="text-align:left;"> Ich werde ein Leben lang üben, dich so zu lieben, wie ich dich lieben will, wenn du gehst </td>
   <td style="text-align:right;"> 172240 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 12:51 </td>
   <td style="text-align:left;"> Wir Sind Helden </td>
   <td style="text-align:left;"> Bist du nicht müde </td>
   <td style="text-align:right;"> 233653 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 13:02 </td>
   <td style="text-align:left;"> BTS </td>
   <td style="text-align:left;"> Trivia &lt;U+8F49&gt; : Seesaw </td>
   <td style="text-align:right;"> 246334 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 13:05 </td>
   <td style="text-align:left;"> j-hope </td>
   <td style="text-align:left;"> P.O.P (Piece Of Peace) Pt. 1 </td>
   <td style="text-align:right;"> 181113 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 13:08 </td>
   <td style="text-align:left;"> Agust D </td>
   <td style="text-align:left;"> People </td>
   <td style="text-align:right;"> 197000 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 13:12 </td>
   <td style="text-align:left;"> BTS </td>
   <td style="text-align:left;"> Trivia &lt;U+627F&gt; : Love </td>
   <td style="text-align:right;"> 225697 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 13:16 </td>
   <td style="text-align:left;"> BTS </td>
   <td style="text-align:left;"> 134340 </td>
   <td style="text-align:right;"> 230063 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 13:17 </td>
   <td style="text-align:left;"> j-hope </td>
   <td style="text-align:left;"> Blue Side (Outro) </td>
   <td style="text-align:right;"> 90539 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 13:21 </td>
   <td style="text-align:left;"> RM </td>
   <td style="text-align:left;"> everythingoes </td>
   <td style="text-align:right;"> 222493 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 13:25 </td>
   <td style="text-align:left;"> BTS </td>
   <td style="text-align:left;"> Go Go </td>
   <td style="text-align:right;"> 235779 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 13:28 </td>
   <td style="text-align:left;"> BTS </td>
   <td style="text-align:left;"> Moon </td>
   <td style="text-align:right;"> 191346 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 14:01 </td>
   <td style="text-align:left;"> Taylor Swift </td>
   <td style="text-align:left;"> the 1 </td>
   <td style="text-align:right;"> 101263 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 12:18 </td>
   <td style="text-align:left;"> The Ronettes </td>
   <td style="text-align:left;"> Be My Baby </td>
   <td style="text-align:right;"> 160906 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 12:22 </td>
   <td style="text-align:left;"> Eurythmics </td>
   <td style="text-align:left;"> Sweet Dreams (Are Made of This) - Remastered </td>
   <td style="text-align:right;"> 216933 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 12:26 </td>
   <td style="text-align:left;"> Blondie </td>
   <td style="text-align:left;"> Heart Of Glass </td>
   <td style="text-align:right;"> 252186 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 12:29 </td>
   <td style="text-align:left;"> The Beach Boys </td>
   <td style="text-align:left;"> God Only Knows - Remastered 1997 / Mono </td>
   <td style="text-align:right;"> 173040 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 12:31 </td>
   <td style="text-align:left;"> Bill Withers </td>
   <td style="text-align:left;"> Ain't No Sunshine </td>
   <td style="text-align:right;"> 125093 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 12:36 </td>
   <td style="text-align:left;"> Paul Simon </td>
   <td style="text-align:left;"> You Can Call Me Al </td>
   <td style="text-align:right;"> 280000 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 12:40 </td>
   <td style="text-align:left;"> Iggy Pop </td>
   <td style="text-align:left;"> The Passenger </td>
   <td style="text-align:right;"> 283360 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 12:44 </td>
   <td style="text-align:left;"> Earth, Wind &amp; Fire </td>
   <td style="text-align:left;"> September </td>
   <td style="text-align:right;"> 215093 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 12:48 </td>
   <td style="text-align:left;"> Jimi Hendrix </td>
   <td style="text-align:left;"> All Along the Watchtower </td>
   <td style="text-align:right;"> 240800 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 12:51 </td>
   <td style="text-align:left;"> Solomon Burke </td>
   <td style="text-align:left;"> Cry to Me </td>
   <td style="text-align:right;"> 154906 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 12:53 </td>
   <td style="text-align:left;"> The Monkees </td>
   <td style="text-align:left;"> I'm a Believer - 2006 Remaster </td>
   <td style="text-align:right;"> 136933 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 12:54 </td>
   <td style="text-align:left;"> The Monkees </td>
   <td style="text-align:left;"> I'm a Believer - 2006 Remaster </td>
   <td style="text-align:right;"> 30458 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 12:56 </td>
   <td style="text-align:left;"> The Beach Boys </td>
   <td style="text-align:left;"> Wouldn't It Be Nice - Stereo / Remastered </td>
   <td style="text-align:right;"> 153205 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 12:59 </td>
   <td style="text-align:left;"> Van Morrison </td>
   <td style="text-align:left;"> Brown Eyed Girl </td>
   <td style="text-align:right;"> 183506 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 13:02 </td>
   <td style="text-align:left;"> The Mamas &amp; The Papas </td>
   <td style="text-align:left;"> California Dreamin' - Single Version </td>
   <td style="text-align:right;"> 162373 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 13:05 </td>
   <td style="text-align:left;"> The Monkees </td>
   <td style="text-align:left;"> Daydream Believer </td>
   <td style="text-align:right;"> 179613 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 13:07 </td>
   <td style="text-align:left;"> Little Peggy March </td>
   <td style="text-align:left;"> I Will Follow Him </td>
   <td style="text-align:right;"> 148160 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 13:11 </td>
   <td style="text-align:left;"> Sonny &amp; Cher </td>
   <td style="text-align:left;"> I Got You Babe </td>
   <td style="text-align:right;"> 190080 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 13:14 </td>
   <td style="text-align:left;"> Steve Miller Band </td>
   <td style="text-align:left;"> The Joker </td>
   <td style="text-align:right;"> 213480 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 13:17 </td>
   <td style="text-align:left;"> Bruce Springsteen </td>
   <td style="text-align:left;"> Hungry Heart </td>
   <td style="text-align:right;"> 198973 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 13:20 </td>
   <td style="text-align:left;"> Roy Orbison </td>
   <td style="text-align:left;"> Oh, Pretty Woman </td>
   <td style="text-align:right;"> 176840 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 13:23 </td>
   <td style="text-align:left;"> The Archies </td>
   <td style="text-align:left;"> Sugar Sugar </td>
   <td style="text-align:right;"> 167213 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 13:27 </td>
   <td style="text-align:left;"> The Who </td>
   <td style="text-align:left;"> My Generation - Stereo Version </td>
   <td style="text-align:right;"> 198706 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 13:31 </td>
   <td style="text-align:left;"> John Cale </td>
   <td style="text-align:left;"> Paris 1919 </td>
   <td style="text-align:right;"> 246799 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 13:34 </td>
   <td style="text-align:left;"> B.J. Thomas </td>
   <td style="text-align:left;"> Raindrops Keep Fallin' on My Head - Rerecorded </td>
   <td style="text-align:right;"> 178146 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 13:37 </td>
   <td style="text-align:left;"> The Monkees </td>
   <td style="text-align:left;"> Daydream Believer </td>
   <td style="text-align:right;"> 179613 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 13:45 </td>
   <td style="text-align:left;"> Don McLean </td>
   <td style="text-align:left;"> American Pie </td>
   <td style="text-align:right;"> 516893 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 13:48 </td>
   <td style="text-align:left;"> Johnny Nash </td>
   <td style="text-align:left;"> I Can See Clearly Now - Edit </td>
   <td style="text-align:right;"> 164733 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-18 05:02 </td>
   <td style="text-align:left;"> BTS </td>
   <td style="text-align:left;"> Life Goes On </td>
   <td style="text-align:right;"> 207481 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-18 05:06 </td>
   <td style="text-align:left;"> BTS </td>
   <td style="text-align:left;"> Fly To My Room </td>
   <td style="text-align:right;"> 222242 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-18 05:10 </td>
   <td style="text-align:left;"> BTS </td>
   <td style="text-align:left;"> Blue &amp; Grey </td>
   <td style="text-align:right;"> 254950 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-18 05:14 </td>
   <td style="text-align:left;"> BTS </td>
   <td style="text-align:left;"> Telepathy </td>
   <td style="text-align:right;"> 202313 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-18 05:18 </td>
   <td style="text-align:left;"> BTS </td>
   <td style="text-align:left;"> Dis-ease </td>
   <td style="text-align:right;"> 239722 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-18 05:21 </td>
   <td style="text-align:left;"> BTS </td>
   <td style="text-align:left;"> Stay </td>
   <td style="text-align:right;"> 204800 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-18 05:24 </td>
   <td style="text-align:left;"> BTS </td>
   <td style="text-align:left;"> Dynamite </td>
   <td style="text-align:right;"> 199053 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-18 05:36 </td>
   <td style="text-align:left;"> BTS </td>
   <td style="text-align:left;"> Life Goes On </td>
   <td style="text-align:right;"> 207481 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-18 05:40 </td>
   <td style="text-align:left;"> BTS </td>
   <td style="text-align:left;"> Fly To My Room </td>
   <td style="text-align:right;"> 222242 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-18 05:44 </td>
   <td style="text-align:left;"> BTS </td>
   <td style="text-align:left;"> Blue &amp; Grey </td>
   <td style="text-align:right;"> 254950 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-18 05:47 </td>
   <td style="text-align:left;"> BTS </td>
   <td style="text-align:left;"> Telepathy </td>
   <td style="text-align:right;"> 155572 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-18 05:47 </td>
   <td style="text-align:left;"> BTS </td>
   <td style="text-align:left;"> Telepathy </td>
   <td style="text-align:right;"> 46756 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-18 05:51 </td>
   <td style="text-align:left;"> BTS </td>
   <td style="text-align:left;"> Stay </td>
   <td style="text-align:right;"> 204800 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-18 05:56 </td>
   <td style="text-align:left;"> BTS </td>
   <td style="text-align:left;"> Blue &amp; Grey </td>
   <td style="text-align:right;"> 254950 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-18 05:59 </td>
   <td style="text-align:left;"> BTS </td>
   <td style="text-align:left;"> Life Goes On </td>
   <td style="text-align:right;"> 207481 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-18 06:02 </td>
   <td style="text-align:left;"> BTS </td>
   <td style="text-align:left;"> Fly To My Room </td>
   <td style="text-align:right;"> 159413 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-18 15:20 </td>
   <td style="text-align:left;"> BTS </td>
   <td style="text-align:left;"> Fly To My Room </td>
   <td style="text-align:right;"> 62845 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-18 15:20 </td>
   <td style="text-align:left;"> BTS </td>
   <td style="text-align:left;"> Telepathy </td>
   <td style="text-align:right;"> 36469 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-18 15:25 </td>
   <td style="text-align:left;"> Epik High </td>
   <td style="text-align:left;"> Rosario </td>
   <td style="text-align:right;"> 302323 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-18 15:30 </td>
   <td style="text-align:left;"> Son Lux </td>
   <td style="text-align:left;"> Easy (Switch Screens) [feat. Lorde] </td>
   <td style="text-align:right;"> 262521 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-18 15:30 </td>
   <td style="text-align:left;"> Ezra Furman </td>
   <td style="text-align:left;"> Restless Year </td>
   <td style="text-align:right;"> 37458 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-18 15:31 </td>
   <td style="text-align:left;"> Troye Sivan </td>
   <td style="text-align:left;"> Easy (with Kacey Musgraves feat. Mark Ronson) </td>
   <td style="text-align:right;"> 19904 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-18 15:31 </td>
   <td style="text-align:left;"> JACKBOYS </td>
   <td style="text-align:left;"> GATTI </td>
   <td style="text-align:right;"> 34534 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-18 15:34 </td>
   <td style="text-align:left;"> Maisie Peters </td>
   <td style="text-align:left;"> Maybe Don't (feat. JP Saxe) - HONNE Remix </td>
   <td style="text-align:right;"> 175997 </td>
  </tr>
</tbody>
</table></div>




Bevor ich mit der Auswertung beginne, müssen noch die Datums- und Zeitangaben angepasst werden. Zum einen soll die Spalte `endTime` getrennt werden, sodass ich je eine Spalte für das Datum und die Uhrzeit erhalte. Hierfür nutze ich die Funktion `ymd_hm()` aus dem R-Paket `lubridate`. Die Uhrzeit ist in den Rohdaten in der Zeitzone *UTC+0* codiert (@Spotify: das wäre eine nützliche Angabe im Readme gewesen), sodass ich noch eine Stunde für die Umwandlung in die *Mitteleuropäische Zeitzone (UTC+1)* addieren muss. Mit der Funktion `floor_date()` wird dann die Datums-/Zeitangabe auf den jeweiligen Tag gerundet. Die Abspieldauer wird zudem in Minuten umgewandelt (`mPlayed`).



```r
streamVerlauf_spotify <- 
  streamVerlauf_spotify %>% 
  as_tibble() %>% 
  mutate(
    endTime_mez = ymd_hm(endTime) + hours(1),
    date = as_date(floor_date(endTime_mez, "day")), 
    mPlayed = round(msPlayed / (1000*60), digits = 2)
    )
```


Die fertig aufbereiteten Daten sehen so aus:


<div style="border: 1px solid #ddd; padding: 0px; overflow-y: scroll; height:300px; overflow-x: scroll; width:100%; "><table class="table table-responsive" style="font-size: 10px; margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;"> endTime </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;"> artistName </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;"> trackName </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;"> msPlayed </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;"> endTime_mez </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;"> date </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;"> mPlayed </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> 2021-01-15 17:40 </td>
   <td style="text-align:left;"> Kate Bush </td>
   <td style="text-align:left;"> Wuthering Heights </td>
   <td style="text-align:right;"> 269066 </td>
   <td style="text-align:left;"> 2021-01-15 18:40:00 </td>
   <td style="text-align:left;"> 2021-01-15 </td>
   <td style="text-align:right;"> 4.48 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-15 17:44 </td>
   <td style="text-align:left;"> Kate Nash </td>
   <td style="text-align:left;"> Mariella </td>
   <td style="text-align:right;"> 255546 </td>
   <td style="text-align:left;"> 2021-01-15 18:44:00 </td>
   <td style="text-align:left;"> 2021-01-15 </td>
   <td style="text-align:right;"> 4.26 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-15 17:47 </td>
   <td style="text-align:left;"> Maisie Peters </td>
   <td style="text-align:left;"> In My Head </td>
   <td style="text-align:right;"> 187827 </td>
   <td style="text-align:left;"> 2021-01-15 18:47:00 </td>
   <td style="text-align:left;"> 2021-01-15 </td>
   <td style="text-align:right;"> 3.13 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-15 17:51 </td>
   <td style="text-align:left;"> Yeah Yeah Yeahs </td>
   <td style="text-align:left;"> Heads Will Roll </td>
   <td style="text-align:right;"> 221000 </td>
   <td style="text-align:left;"> 2021-01-15 18:51:00 </td>
   <td style="text-align:left;"> 2021-01-15 </td>
   <td style="text-align:right;"> 3.68 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-15 17:54 </td>
   <td style="text-align:left;"> SUNMI </td>
   <td style="text-align:left;"> Siren </td>
   <td style="text-align:right;"> 199133 </td>
   <td style="text-align:left;"> 2021-01-15 18:54:00 </td>
   <td style="text-align:left;"> 2021-01-15 </td>
   <td style="text-align:right;"> 3.32 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-15 17:58 </td>
   <td style="text-align:left;"> Stromae </td>
   <td style="text-align:left;"> sommeil </td>
   <td style="text-align:right;"> 218653 </td>
   <td style="text-align:left;"> 2021-01-15 18:58:00 </td>
   <td style="text-align:left;"> 2021-01-15 </td>
   <td style="text-align:right;"> 3.64 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-15 18:03 </td>
   <td style="text-align:left;"> Wir Sind Helden </td>
   <td style="text-align:left;"> Ein Elefant für dich </td>
   <td style="text-align:right;"> 282653 </td>
   <td style="text-align:left;"> 2021-01-15 19:03:00 </td>
   <td style="text-align:left;"> 2021-01-15 </td>
   <td style="text-align:right;"> 4.71 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-15 18:05 </td>
   <td style="text-align:left;"> St. Vincent </td>
   <td style="text-align:left;"> New York </td>
   <td style="text-align:right;"> 123035 </td>
   <td style="text-align:left;"> 2021-01-15 19:05:00 </td>
   <td style="text-align:left;"> 2021-01-15 </td>
   <td style="text-align:right;"> 2.05 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 10:55 </td>
   <td style="text-align:left;"> Lorde </td>
   <td style="text-align:left;"> Hard Feelings/Loveless </td>
   <td style="text-align:right;"> 367391 </td>
   <td style="text-align:left;"> 2021-01-16 11:55:00 </td>
   <td style="text-align:left;"> 2021-01-16 </td>
   <td style="text-align:right;"> 6.12 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 10:58 </td>
   <td style="text-align:left;"> David Bowie </td>
   <td style="text-align:left;"> Life on Mars? - 2015 Remaster </td>
   <td style="text-align:right;"> 133913 </td>
   <td style="text-align:left;"> 2021-01-16 11:58:00 </td>
   <td style="text-align:left;"> 2021-01-16 </td>
   <td style="text-align:right;"> 2.23 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 11:14 </td>
   <td style="text-align:left;"> David Bowie </td>
   <td style="text-align:left;"> Life on Mars? - 2015 Remaster </td>
   <td style="text-align:right;"> 235986 </td>
   <td style="text-align:left;"> 2021-01-16 12:14:00 </td>
   <td style="text-align:left;"> 2021-01-16 </td>
   <td style="text-align:right;"> 3.93 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 11:17 </td>
   <td style="text-align:left;"> Ezra Furman </td>
   <td style="text-align:left;"> Love You So Bad </td>
   <td style="text-align:right;"> 219000 </td>
   <td style="text-align:left;"> 2021-01-16 12:17:00 </td>
   <td style="text-align:left;"> 2021-01-16 </td>
   <td style="text-align:right;"> 3.65 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 11:21 </td>
   <td style="text-align:left;"> Frog </td>
   <td style="text-align:left;"> Judy Garland </td>
   <td style="text-align:right;"> 217177 </td>
   <td style="text-align:left;"> 2021-01-16 12:21:00 </td>
   <td style="text-align:left;"> 2021-01-16 </td>
   <td style="text-align:right;"> 3.62 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 11:24 </td>
   <td style="text-align:left;"> Wings </td>
   <td style="text-align:left;"> Live And Let Die </td>
   <td style="text-align:right;"> 194613 </td>
   <td style="text-align:left;"> 2021-01-16 12:24:00 </td>
   <td style="text-align:left;"> 2021-01-16 </td>
   <td style="text-align:right;"> 3.24 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 11:29 </td>
   <td style="text-align:left;"> Chappell Roan </td>
   <td style="text-align:left;"> Pink Pony Club </td>
   <td style="text-align:right;"> 258034 </td>
   <td style="text-align:left;"> 2021-01-16 12:29:00 </td>
   <td style="text-align:left;"> 2021-01-16 </td>
   <td style="text-align:right;"> 4.30 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 11:32 </td>
   <td style="text-align:left;"> Kate Bush </td>
   <td style="text-align:left;"> Babooshka - 2018 Remaster </td>
   <td style="text-align:right;"> 199226 </td>
   <td style="text-align:left;"> 2021-01-16 12:32:00 </td>
   <td style="text-align:left;"> 2021-01-16 </td>
   <td style="text-align:right;"> 3.32 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 11:33 </td>
   <td style="text-align:left;"> Dominic Fike </td>
   <td style="text-align:left;"> 10x Stronger </td>
   <td style="text-align:right;"> 75585 </td>
   <td style="text-align:left;"> 2021-01-16 12:33:00 </td>
   <td style="text-align:left;"> 2021-01-16 </td>
   <td style="text-align:right;"> 1.26 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 11:37 </td>
   <td style="text-align:left;"> Lorde </td>
   <td style="text-align:left;"> Writer In The Dark </td>
   <td style="text-align:right;"> 216610 </td>
   <td style="text-align:left;"> 2021-01-16 12:37:00 </td>
   <td style="text-align:left;"> 2021-01-16 </td>
   <td style="text-align:right;"> 3.61 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 11:40 </td>
   <td style="text-align:left;"> BTS </td>
   <td style="text-align:left;"> Fly To My Room </td>
   <td style="text-align:right;"> 222242 </td>
   <td style="text-align:left;"> 2021-01-16 12:40:00 </td>
   <td style="text-align:left;"> 2021-01-16 </td>
   <td style="text-align:right;"> 3.70 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 11:44 </td>
   <td style="text-align:left;"> Halsey </td>
   <td style="text-align:left;"> I HATE EVERYBODY </td>
   <td style="text-align:right;"> 171015 </td>
   <td style="text-align:left;"> 2021-01-16 12:44:00 </td>
   <td style="text-align:left;"> 2021-01-16 </td>
   <td style="text-align:right;"> 2.85 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 11:48 </td>
   <td style="text-align:left;"> Kate Bush </td>
   <td style="text-align:left;"> Wuthering Heights </td>
   <td style="text-align:right;"> 269066 </td>
   <td style="text-align:left;"> 2021-01-16 12:48:00 </td>
   <td style="text-align:left;"> 2021-01-16 </td>
   <td style="text-align:right;"> 4.48 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 11:52 </td>
   <td style="text-align:left;"> Kate Nash </td>
   <td style="text-align:left;"> Mariella </td>
   <td style="text-align:right;"> 255546 </td>
   <td style="text-align:left;"> 2021-01-16 12:52:00 </td>
   <td style="text-align:left;"> 2021-01-16 </td>
   <td style="text-align:right;"> 4.26 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 11:56 </td>
   <td style="text-align:left;"> Maisie Peters </td>
   <td style="text-align:left;"> In My Head </td>
   <td style="text-align:right;"> 187827 </td>
   <td style="text-align:left;"> 2021-01-16 12:56:00 </td>
   <td style="text-align:left;"> 2021-01-16 </td>
   <td style="text-align:right;"> 3.13 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 11:59 </td>
   <td style="text-align:left;"> Yeah Yeah Yeahs </td>
   <td style="text-align:left;"> Heads Will Roll </td>
   <td style="text-align:right;"> 221000 </td>
   <td style="text-align:left;"> 2021-01-16 12:59:00 </td>
   <td style="text-align:left;"> 2021-01-16 </td>
   <td style="text-align:right;"> 3.68 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 12:03 </td>
   <td style="text-align:left;"> SUNMI </td>
   <td style="text-align:left;"> Siren </td>
   <td style="text-align:right;"> 199133 </td>
   <td style="text-align:left;"> 2021-01-16 13:03:00 </td>
   <td style="text-align:left;"> 2021-01-16 </td>
   <td style="text-align:right;"> 3.32 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 12:06 </td>
   <td style="text-align:left;"> Stromae </td>
   <td style="text-align:left;"> sommeil </td>
   <td style="text-align:right;"> 218653 </td>
   <td style="text-align:left;"> 2021-01-16 13:06:00 </td>
   <td style="text-align:left;"> 2021-01-16 </td>
   <td style="text-align:right;"> 3.64 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 12:07 </td>
   <td style="text-align:left;"> Wir Sind Helden </td>
   <td style="text-align:left;"> Ein Elefant für dich </td>
   <td style="text-align:right;"> 50533 </td>
   <td style="text-align:left;"> 2021-01-16 13:07:00 </td>
   <td style="text-align:left;"> 2021-01-16 </td>
   <td style="text-align:right;"> 0.84 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 12:11 </td>
   <td style="text-align:left;"> Wir Sind Helden </td>
   <td style="text-align:left;"> Wenn es passiert </td>
   <td style="text-align:right;"> 212360 </td>
   <td style="text-align:left;"> 2021-01-16 13:11:00 </td>
   <td style="text-align:left;"> 2021-01-16 </td>
   <td style="text-align:right;"> 3.54 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 12:15 </td>
   <td style="text-align:left;"> Wir Sind Helden </td>
   <td style="text-align:left;"> Echolot </td>
   <td style="text-align:right;"> 271386 </td>
   <td style="text-align:left;"> 2021-01-16 13:15:00 </td>
   <td style="text-align:left;"> 2021-01-16 </td>
   <td style="text-align:right;"> 4.52 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 12:19 </td>
   <td style="text-align:left;"> Wir Sind Helden </td>
   <td style="text-align:left;"> Von hier an blind </td>
   <td style="text-align:right;"> 210666 </td>
   <td style="text-align:left;"> 2021-01-16 13:19:00 </td>
   <td style="text-align:left;"> 2021-01-16 </td>
   <td style="text-align:right;"> 3.51 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 12:22 </td>
   <td style="text-align:left;"> Wir Sind Helden </td>
   <td style="text-align:left;"> Zuhälter </td>
   <td style="text-align:right;"> 210306 </td>
   <td style="text-align:left;"> 2021-01-16 13:22:00 </td>
   <td style="text-align:left;"> 2021-01-16 </td>
   <td style="text-align:right;"> 3.51 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 12:27 </td>
   <td style="text-align:left;"> Wir Sind Helden </td>
   <td style="text-align:left;"> Ein Elefant für dich </td>
   <td style="text-align:right;"> 282653 </td>
   <td style="text-align:left;"> 2021-01-16 13:27:00 </td>
   <td style="text-align:left;"> 2021-01-16 </td>
   <td style="text-align:right;"> 4.71 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 12:30 </td>
   <td style="text-align:left;"> Wir Sind Helden </td>
   <td style="text-align:left;"> Darf ich das behalten </td>
   <td style="text-align:right;"> 198466 </td>
   <td style="text-align:left;"> 2021-01-16 13:30:00 </td>
   <td style="text-align:left;"> 2021-01-16 </td>
   <td style="text-align:right;"> 3.31 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 12:36 </td>
   <td style="text-align:left;"> Wir Sind Helden </td>
   <td style="text-align:left;"> Wütend genug </td>
   <td style="text-align:right;"> 269386 </td>
   <td style="text-align:left;"> 2021-01-16 13:36:00 </td>
   <td style="text-align:left;"> 2021-01-16 </td>
   <td style="text-align:right;"> 4.49 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 12:37 </td>
   <td style="text-align:left;"> Wir Sind Helden </td>
   <td style="text-align:left;"> Geht auseinander </td>
   <td style="text-align:right;"> 26645 </td>
   <td style="text-align:left;"> 2021-01-16 13:37:00 </td>
   <td style="text-align:left;"> 2021-01-16 </td>
   <td style="text-align:right;"> 0.44 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 12:40 </td>
   <td style="text-align:left;"> Wir Sind Helden </td>
   <td style="text-align:left;"> Gekommen um zu bleiben </td>
   <td style="text-align:right;"> 190120 </td>
   <td style="text-align:left;"> 2021-01-16 13:40:00 </td>
   <td style="text-align:left;"> 2021-01-16 </td>
   <td style="text-align:right;"> 3.17 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 12:44 </td>
   <td style="text-align:left;"> Wir Sind Helden </td>
   <td style="text-align:left;"> Nur ein Wort </td>
   <td style="text-align:right;"> 236200 </td>
   <td style="text-align:left;"> 2021-01-16 13:44:00 </td>
   <td style="text-align:left;"> 2021-01-16 </td>
   <td style="text-align:right;"> 3.94 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 12:47 </td>
   <td style="text-align:left;"> Wir Sind Helden </td>
   <td style="text-align:left;"> Ich werde ein Leben lang üben, dich so zu lieben, wie ich dich lieben will, wenn du gehst </td>
   <td style="text-align:right;"> 172240 </td>
   <td style="text-align:left;"> 2021-01-16 13:47:00 </td>
   <td style="text-align:left;"> 2021-01-16 </td>
   <td style="text-align:right;"> 2.87 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 12:51 </td>
   <td style="text-align:left;"> Wir Sind Helden </td>
   <td style="text-align:left;"> Bist du nicht müde </td>
   <td style="text-align:right;"> 233653 </td>
   <td style="text-align:left;"> 2021-01-16 13:51:00 </td>
   <td style="text-align:left;"> 2021-01-16 </td>
   <td style="text-align:right;"> 3.89 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 13:02 </td>
   <td style="text-align:left;"> BTS </td>
   <td style="text-align:left;"> Trivia &lt;U+8F49&gt; : Seesaw </td>
   <td style="text-align:right;"> 246334 </td>
   <td style="text-align:left;"> 2021-01-16 14:02:00 </td>
   <td style="text-align:left;"> 2021-01-16 </td>
   <td style="text-align:right;"> 4.11 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 13:05 </td>
   <td style="text-align:left;"> j-hope </td>
   <td style="text-align:left;"> P.O.P (Piece Of Peace) Pt. 1 </td>
   <td style="text-align:right;"> 181113 </td>
   <td style="text-align:left;"> 2021-01-16 14:05:00 </td>
   <td style="text-align:left;"> 2021-01-16 </td>
   <td style="text-align:right;"> 3.02 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 13:08 </td>
   <td style="text-align:left;"> Agust D </td>
   <td style="text-align:left;"> People </td>
   <td style="text-align:right;"> 197000 </td>
   <td style="text-align:left;"> 2021-01-16 14:08:00 </td>
   <td style="text-align:left;"> 2021-01-16 </td>
   <td style="text-align:right;"> 3.28 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 13:12 </td>
   <td style="text-align:left;"> BTS </td>
   <td style="text-align:left;"> Trivia &lt;U+627F&gt; : Love </td>
   <td style="text-align:right;"> 225697 </td>
   <td style="text-align:left;"> 2021-01-16 14:12:00 </td>
   <td style="text-align:left;"> 2021-01-16 </td>
   <td style="text-align:right;"> 3.76 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 13:16 </td>
   <td style="text-align:left;"> BTS </td>
   <td style="text-align:left;"> 134340 </td>
   <td style="text-align:right;"> 230063 </td>
   <td style="text-align:left;"> 2021-01-16 14:16:00 </td>
   <td style="text-align:left;"> 2021-01-16 </td>
   <td style="text-align:right;"> 3.83 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 13:17 </td>
   <td style="text-align:left;"> j-hope </td>
   <td style="text-align:left;"> Blue Side (Outro) </td>
   <td style="text-align:right;"> 90539 </td>
   <td style="text-align:left;"> 2021-01-16 14:17:00 </td>
   <td style="text-align:left;"> 2021-01-16 </td>
   <td style="text-align:right;"> 1.51 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 13:21 </td>
   <td style="text-align:left;"> RM </td>
   <td style="text-align:left;"> everythingoes </td>
   <td style="text-align:right;"> 222493 </td>
   <td style="text-align:left;"> 2021-01-16 14:21:00 </td>
   <td style="text-align:left;"> 2021-01-16 </td>
   <td style="text-align:right;"> 3.71 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 13:25 </td>
   <td style="text-align:left;"> BTS </td>
   <td style="text-align:left;"> Go Go </td>
   <td style="text-align:right;"> 235779 </td>
   <td style="text-align:left;"> 2021-01-16 14:25:00 </td>
   <td style="text-align:left;"> 2021-01-16 </td>
   <td style="text-align:right;"> 3.93 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 13:28 </td>
   <td style="text-align:left;"> BTS </td>
   <td style="text-align:left;"> Moon </td>
   <td style="text-align:right;"> 191346 </td>
   <td style="text-align:left;"> 2021-01-16 14:28:00 </td>
   <td style="text-align:left;"> 2021-01-16 </td>
   <td style="text-align:right;"> 3.19 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-16 14:01 </td>
   <td style="text-align:left;"> Taylor Swift </td>
   <td style="text-align:left;"> the 1 </td>
   <td style="text-align:right;"> 101263 </td>
   <td style="text-align:left;"> 2021-01-16 15:01:00 </td>
   <td style="text-align:left;"> 2021-01-16 </td>
   <td style="text-align:right;"> 1.69 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 12:18 </td>
   <td style="text-align:left;"> The Ronettes </td>
   <td style="text-align:left;"> Be My Baby </td>
   <td style="text-align:right;"> 160906 </td>
   <td style="text-align:left;"> 2021-01-17 13:18:00 </td>
   <td style="text-align:left;"> 2021-01-17 </td>
   <td style="text-align:right;"> 2.68 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 12:22 </td>
   <td style="text-align:left;"> Eurythmics </td>
   <td style="text-align:left;"> Sweet Dreams (Are Made of This) - Remastered </td>
   <td style="text-align:right;"> 216933 </td>
   <td style="text-align:left;"> 2021-01-17 13:22:00 </td>
   <td style="text-align:left;"> 2021-01-17 </td>
   <td style="text-align:right;"> 3.62 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 12:26 </td>
   <td style="text-align:left;"> Blondie </td>
   <td style="text-align:left;"> Heart Of Glass </td>
   <td style="text-align:right;"> 252186 </td>
   <td style="text-align:left;"> 2021-01-17 13:26:00 </td>
   <td style="text-align:left;"> 2021-01-17 </td>
   <td style="text-align:right;"> 4.20 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 12:29 </td>
   <td style="text-align:left;"> The Beach Boys </td>
   <td style="text-align:left;"> God Only Knows - Remastered 1997 / Mono </td>
   <td style="text-align:right;"> 173040 </td>
   <td style="text-align:left;"> 2021-01-17 13:29:00 </td>
   <td style="text-align:left;"> 2021-01-17 </td>
   <td style="text-align:right;"> 2.88 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 12:31 </td>
   <td style="text-align:left;"> Bill Withers </td>
   <td style="text-align:left;"> Ain't No Sunshine </td>
   <td style="text-align:right;"> 125093 </td>
   <td style="text-align:left;"> 2021-01-17 13:31:00 </td>
   <td style="text-align:left;"> 2021-01-17 </td>
   <td style="text-align:right;"> 2.08 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 12:36 </td>
   <td style="text-align:left;"> Paul Simon </td>
   <td style="text-align:left;"> You Can Call Me Al </td>
   <td style="text-align:right;"> 280000 </td>
   <td style="text-align:left;"> 2021-01-17 13:36:00 </td>
   <td style="text-align:left;"> 2021-01-17 </td>
   <td style="text-align:right;"> 4.67 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 12:40 </td>
   <td style="text-align:left;"> Iggy Pop </td>
   <td style="text-align:left;"> The Passenger </td>
   <td style="text-align:right;"> 283360 </td>
   <td style="text-align:left;"> 2021-01-17 13:40:00 </td>
   <td style="text-align:left;"> 2021-01-17 </td>
   <td style="text-align:right;"> 4.72 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 12:44 </td>
   <td style="text-align:left;"> Earth, Wind &amp; Fire </td>
   <td style="text-align:left;"> September </td>
   <td style="text-align:right;"> 215093 </td>
   <td style="text-align:left;"> 2021-01-17 13:44:00 </td>
   <td style="text-align:left;"> 2021-01-17 </td>
   <td style="text-align:right;"> 3.58 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 12:48 </td>
   <td style="text-align:left;"> Jimi Hendrix </td>
   <td style="text-align:left;"> All Along the Watchtower </td>
   <td style="text-align:right;"> 240800 </td>
   <td style="text-align:left;"> 2021-01-17 13:48:00 </td>
   <td style="text-align:left;"> 2021-01-17 </td>
   <td style="text-align:right;"> 4.01 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 12:51 </td>
   <td style="text-align:left;"> Solomon Burke </td>
   <td style="text-align:left;"> Cry to Me </td>
   <td style="text-align:right;"> 154906 </td>
   <td style="text-align:left;"> 2021-01-17 13:51:00 </td>
   <td style="text-align:left;"> 2021-01-17 </td>
   <td style="text-align:right;"> 2.58 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 12:53 </td>
   <td style="text-align:left;"> The Monkees </td>
   <td style="text-align:left;"> I'm a Believer - 2006 Remaster </td>
   <td style="text-align:right;"> 136933 </td>
   <td style="text-align:left;"> 2021-01-17 13:53:00 </td>
   <td style="text-align:left;"> 2021-01-17 </td>
   <td style="text-align:right;"> 2.28 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 12:54 </td>
   <td style="text-align:left;"> The Monkees </td>
   <td style="text-align:left;"> I'm a Believer - 2006 Remaster </td>
   <td style="text-align:right;"> 30458 </td>
   <td style="text-align:left;"> 2021-01-17 13:54:00 </td>
   <td style="text-align:left;"> 2021-01-17 </td>
   <td style="text-align:right;"> 0.51 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 12:56 </td>
   <td style="text-align:left;"> The Beach Boys </td>
   <td style="text-align:left;"> Wouldn't It Be Nice - Stereo / Remastered </td>
   <td style="text-align:right;"> 153205 </td>
   <td style="text-align:left;"> 2021-01-17 13:56:00 </td>
   <td style="text-align:left;"> 2021-01-17 </td>
   <td style="text-align:right;"> 2.55 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 12:59 </td>
   <td style="text-align:left;"> Van Morrison </td>
   <td style="text-align:left;"> Brown Eyed Girl </td>
   <td style="text-align:right;"> 183506 </td>
   <td style="text-align:left;"> 2021-01-17 13:59:00 </td>
   <td style="text-align:left;"> 2021-01-17 </td>
   <td style="text-align:right;"> 3.06 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 13:02 </td>
   <td style="text-align:left;"> The Mamas &amp; The Papas </td>
   <td style="text-align:left;"> California Dreamin' - Single Version </td>
   <td style="text-align:right;"> 162373 </td>
   <td style="text-align:left;"> 2021-01-17 14:02:00 </td>
   <td style="text-align:left;"> 2021-01-17 </td>
   <td style="text-align:right;"> 2.71 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 13:05 </td>
   <td style="text-align:left;"> The Monkees </td>
   <td style="text-align:left;"> Daydream Believer </td>
   <td style="text-align:right;"> 179613 </td>
   <td style="text-align:left;"> 2021-01-17 14:05:00 </td>
   <td style="text-align:left;"> 2021-01-17 </td>
   <td style="text-align:right;"> 2.99 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 13:07 </td>
   <td style="text-align:left;"> Little Peggy March </td>
   <td style="text-align:left;"> I Will Follow Him </td>
   <td style="text-align:right;"> 148160 </td>
   <td style="text-align:left;"> 2021-01-17 14:07:00 </td>
   <td style="text-align:left;"> 2021-01-17 </td>
   <td style="text-align:right;"> 2.47 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 13:11 </td>
   <td style="text-align:left;"> Sonny &amp; Cher </td>
   <td style="text-align:left;"> I Got You Babe </td>
   <td style="text-align:right;"> 190080 </td>
   <td style="text-align:left;"> 2021-01-17 14:11:00 </td>
   <td style="text-align:left;"> 2021-01-17 </td>
   <td style="text-align:right;"> 3.17 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 13:14 </td>
   <td style="text-align:left;"> Steve Miller Band </td>
   <td style="text-align:left;"> The Joker </td>
   <td style="text-align:right;"> 213480 </td>
   <td style="text-align:left;"> 2021-01-17 14:14:00 </td>
   <td style="text-align:left;"> 2021-01-17 </td>
   <td style="text-align:right;"> 3.56 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 13:17 </td>
   <td style="text-align:left;"> Bruce Springsteen </td>
   <td style="text-align:left;"> Hungry Heart </td>
   <td style="text-align:right;"> 198973 </td>
   <td style="text-align:left;"> 2021-01-17 14:17:00 </td>
   <td style="text-align:left;"> 2021-01-17 </td>
   <td style="text-align:right;"> 3.32 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 13:20 </td>
   <td style="text-align:left;"> Roy Orbison </td>
   <td style="text-align:left;"> Oh, Pretty Woman </td>
   <td style="text-align:right;"> 176840 </td>
   <td style="text-align:left;"> 2021-01-17 14:20:00 </td>
   <td style="text-align:left;"> 2021-01-17 </td>
   <td style="text-align:right;"> 2.95 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 13:23 </td>
   <td style="text-align:left;"> The Archies </td>
   <td style="text-align:left;"> Sugar Sugar </td>
   <td style="text-align:right;"> 167213 </td>
   <td style="text-align:left;"> 2021-01-17 14:23:00 </td>
   <td style="text-align:left;"> 2021-01-17 </td>
   <td style="text-align:right;"> 2.79 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 13:27 </td>
   <td style="text-align:left;"> The Who </td>
   <td style="text-align:left;"> My Generation - Stereo Version </td>
   <td style="text-align:right;"> 198706 </td>
   <td style="text-align:left;"> 2021-01-17 14:27:00 </td>
   <td style="text-align:left;"> 2021-01-17 </td>
   <td style="text-align:right;"> 3.31 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 13:31 </td>
   <td style="text-align:left;"> John Cale </td>
   <td style="text-align:left;"> Paris 1919 </td>
   <td style="text-align:right;"> 246799 </td>
   <td style="text-align:left;"> 2021-01-17 14:31:00 </td>
   <td style="text-align:left;"> 2021-01-17 </td>
   <td style="text-align:right;"> 4.11 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 13:34 </td>
   <td style="text-align:left;"> B.J. Thomas </td>
   <td style="text-align:left;"> Raindrops Keep Fallin' on My Head - Rerecorded </td>
   <td style="text-align:right;"> 178146 </td>
   <td style="text-align:left;"> 2021-01-17 14:34:00 </td>
   <td style="text-align:left;"> 2021-01-17 </td>
   <td style="text-align:right;"> 2.97 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 13:37 </td>
   <td style="text-align:left;"> The Monkees </td>
   <td style="text-align:left;"> Daydream Believer </td>
   <td style="text-align:right;"> 179613 </td>
   <td style="text-align:left;"> 2021-01-17 14:37:00 </td>
   <td style="text-align:left;"> 2021-01-17 </td>
   <td style="text-align:right;"> 2.99 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 13:45 </td>
   <td style="text-align:left;"> Don McLean </td>
   <td style="text-align:left;"> American Pie </td>
   <td style="text-align:right;"> 516893 </td>
   <td style="text-align:left;"> 2021-01-17 14:45:00 </td>
   <td style="text-align:left;"> 2021-01-17 </td>
   <td style="text-align:right;"> 8.61 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-17 13:48 </td>
   <td style="text-align:left;"> Johnny Nash </td>
   <td style="text-align:left;"> I Can See Clearly Now - Edit </td>
   <td style="text-align:right;"> 164733 </td>
   <td style="text-align:left;"> 2021-01-17 14:48:00 </td>
   <td style="text-align:left;"> 2021-01-17 </td>
   <td style="text-align:right;"> 2.75 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-18 05:02 </td>
   <td style="text-align:left;"> BTS </td>
   <td style="text-align:left;"> Life Goes On </td>
   <td style="text-align:right;"> 207481 </td>
   <td style="text-align:left;"> 2021-01-18 06:02:00 </td>
   <td style="text-align:left;"> 2021-01-18 </td>
   <td style="text-align:right;"> 3.46 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-18 05:06 </td>
   <td style="text-align:left;"> BTS </td>
   <td style="text-align:left;"> Fly To My Room </td>
   <td style="text-align:right;"> 222242 </td>
   <td style="text-align:left;"> 2021-01-18 06:06:00 </td>
   <td style="text-align:left;"> 2021-01-18 </td>
   <td style="text-align:right;"> 3.70 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-18 05:10 </td>
   <td style="text-align:left;"> BTS </td>
   <td style="text-align:left;"> Blue &amp; Grey </td>
   <td style="text-align:right;"> 254950 </td>
   <td style="text-align:left;"> 2021-01-18 06:10:00 </td>
   <td style="text-align:left;"> 2021-01-18 </td>
   <td style="text-align:right;"> 4.25 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-18 05:14 </td>
   <td style="text-align:left;"> BTS </td>
   <td style="text-align:left;"> Telepathy </td>
   <td style="text-align:right;"> 202313 </td>
   <td style="text-align:left;"> 2021-01-18 06:14:00 </td>
   <td style="text-align:left;"> 2021-01-18 </td>
   <td style="text-align:right;"> 3.37 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-18 05:18 </td>
   <td style="text-align:left;"> BTS </td>
   <td style="text-align:left;"> Dis-ease </td>
   <td style="text-align:right;"> 239722 </td>
   <td style="text-align:left;"> 2021-01-18 06:18:00 </td>
   <td style="text-align:left;"> 2021-01-18 </td>
   <td style="text-align:right;"> 4.00 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-18 05:21 </td>
   <td style="text-align:left;"> BTS </td>
   <td style="text-align:left;"> Stay </td>
   <td style="text-align:right;"> 204800 </td>
   <td style="text-align:left;"> 2021-01-18 06:21:00 </td>
   <td style="text-align:left;"> 2021-01-18 </td>
   <td style="text-align:right;"> 3.41 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-18 05:24 </td>
   <td style="text-align:left;"> BTS </td>
   <td style="text-align:left;"> Dynamite </td>
   <td style="text-align:right;"> 199053 </td>
   <td style="text-align:left;"> 2021-01-18 06:24:00 </td>
   <td style="text-align:left;"> 2021-01-18 </td>
   <td style="text-align:right;"> 3.32 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-18 05:36 </td>
   <td style="text-align:left;"> BTS </td>
   <td style="text-align:left;"> Life Goes On </td>
   <td style="text-align:right;"> 207481 </td>
   <td style="text-align:left;"> 2021-01-18 06:36:00 </td>
   <td style="text-align:left;"> 2021-01-18 </td>
   <td style="text-align:right;"> 3.46 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-18 05:40 </td>
   <td style="text-align:left;"> BTS </td>
   <td style="text-align:left;"> Fly To My Room </td>
   <td style="text-align:right;"> 222242 </td>
   <td style="text-align:left;"> 2021-01-18 06:40:00 </td>
   <td style="text-align:left;"> 2021-01-18 </td>
   <td style="text-align:right;"> 3.70 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-18 05:44 </td>
   <td style="text-align:left;"> BTS </td>
   <td style="text-align:left;"> Blue &amp; Grey </td>
   <td style="text-align:right;"> 254950 </td>
   <td style="text-align:left;"> 2021-01-18 06:44:00 </td>
   <td style="text-align:left;"> 2021-01-18 </td>
   <td style="text-align:right;"> 4.25 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-18 05:47 </td>
   <td style="text-align:left;"> BTS </td>
   <td style="text-align:left;"> Telepathy </td>
   <td style="text-align:right;"> 155572 </td>
   <td style="text-align:left;"> 2021-01-18 06:47:00 </td>
   <td style="text-align:left;"> 2021-01-18 </td>
   <td style="text-align:right;"> 2.59 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-18 05:47 </td>
   <td style="text-align:left;"> BTS </td>
   <td style="text-align:left;"> Telepathy </td>
   <td style="text-align:right;"> 46756 </td>
   <td style="text-align:left;"> 2021-01-18 06:47:00 </td>
   <td style="text-align:left;"> 2021-01-18 </td>
   <td style="text-align:right;"> 0.78 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-18 05:51 </td>
   <td style="text-align:left;"> BTS </td>
   <td style="text-align:left;"> Stay </td>
   <td style="text-align:right;"> 204800 </td>
   <td style="text-align:left;"> 2021-01-18 06:51:00 </td>
   <td style="text-align:left;"> 2021-01-18 </td>
   <td style="text-align:right;"> 3.41 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-18 05:56 </td>
   <td style="text-align:left;"> BTS </td>
   <td style="text-align:left;"> Blue &amp; Grey </td>
   <td style="text-align:right;"> 254950 </td>
   <td style="text-align:left;"> 2021-01-18 06:56:00 </td>
   <td style="text-align:left;"> 2021-01-18 </td>
   <td style="text-align:right;"> 4.25 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-18 05:59 </td>
   <td style="text-align:left;"> BTS </td>
   <td style="text-align:left;"> Life Goes On </td>
   <td style="text-align:right;"> 207481 </td>
   <td style="text-align:left;"> 2021-01-18 06:59:00 </td>
   <td style="text-align:left;"> 2021-01-18 </td>
   <td style="text-align:right;"> 3.46 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-18 06:02 </td>
   <td style="text-align:left;"> BTS </td>
   <td style="text-align:left;"> Fly To My Room </td>
   <td style="text-align:right;"> 159413 </td>
   <td style="text-align:left;"> 2021-01-18 07:02:00 </td>
   <td style="text-align:left;"> 2021-01-18 </td>
   <td style="text-align:right;"> 2.66 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-18 15:20 </td>
   <td style="text-align:left;"> BTS </td>
   <td style="text-align:left;"> Fly To My Room </td>
   <td style="text-align:right;"> 62845 </td>
   <td style="text-align:left;"> 2021-01-18 16:20:00 </td>
   <td style="text-align:left;"> 2021-01-18 </td>
   <td style="text-align:right;"> 1.05 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-18 15:20 </td>
   <td style="text-align:left;"> BTS </td>
   <td style="text-align:left;"> Telepathy </td>
   <td style="text-align:right;"> 36469 </td>
   <td style="text-align:left;"> 2021-01-18 16:20:00 </td>
   <td style="text-align:left;"> 2021-01-18 </td>
   <td style="text-align:right;"> 0.61 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-18 15:25 </td>
   <td style="text-align:left;"> Epik High </td>
   <td style="text-align:left;"> Rosario </td>
   <td style="text-align:right;"> 302323 </td>
   <td style="text-align:left;"> 2021-01-18 16:25:00 </td>
   <td style="text-align:left;"> 2021-01-18 </td>
   <td style="text-align:right;"> 5.04 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-18 15:30 </td>
   <td style="text-align:left;"> Son Lux </td>
   <td style="text-align:left;"> Easy (Switch Screens) [feat. Lorde] </td>
   <td style="text-align:right;"> 262521 </td>
   <td style="text-align:left;"> 2021-01-18 16:30:00 </td>
   <td style="text-align:left;"> 2021-01-18 </td>
   <td style="text-align:right;"> 4.38 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-18 15:30 </td>
   <td style="text-align:left;"> Ezra Furman </td>
   <td style="text-align:left;"> Restless Year </td>
   <td style="text-align:right;"> 37458 </td>
   <td style="text-align:left;"> 2021-01-18 16:30:00 </td>
   <td style="text-align:left;"> 2021-01-18 </td>
   <td style="text-align:right;"> 0.62 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-18 15:31 </td>
   <td style="text-align:left;"> Troye Sivan </td>
   <td style="text-align:left;"> Easy (with Kacey Musgraves feat. Mark Ronson) </td>
   <td style="text-align:right;"> 19904 </td>
   <td style="text-align:left;"> 2021-01-18 16:31:00 </td>
   <td style="text-align:left;"> 2021-01-18 </td>
   <td style="text-align:right;"> 0.33 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-18 15:31 </td>
   <td style="text-align:left;"> JACKBOYS </td>
   <td style="text-align:left;"> GATTI </td>
   <td style="text-align:right;"> 34534 </td>
   <td style="text-align:left;"> 2021-01-18 16:31:00 </td>
   <td style="text-align:left;"> 2021-01-18 </td>
   <td style="text-align:right;"> 0.58 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-18 15:34 </td>
   <td style="text-align:left;"> Maisie Peters </td>
   <td style="text-align:left;"> Maybe Don't (feat. JP Saxe) - HONNE Remix </td>
   <td style="text-align:right;"> 175997 </td>
   <td style="text-align:left;"> 2021-01-18 16:34:00 </td>
   <td style="text-align:left;"> 2021-01-18 </td>
   <td style="text-align:right;"> 2.93 </td>
  </tr>
</tbody>
</table></div>






## **Datenanalyse**

Zuerst möchte ich mir den Streaming-Verlauf über den gesamten Zeitraum anschauen. Mittels `scale_fill_gradient()` kann man eine Farbverlaufsskala erstellen, wobei rot eine relativ hohe und gelb eine relativ niedrige Streaming-Aktivität darstellt. 



```r
streamVerlauf_spotify %>% 
  group_by(date) %>%
  summarize(hours = sum(mPlayed) / 60) %>% 
  arrange(date) %>% 
  ggplot(aes(x = date, y = hours)) + 
  geom_col(aes(fill = hours)) +
  scale_fill_gradient(low = "yellow", high = "red") + 
  scale_x_date(date_breaks = "4 weeks", date_labels = "%b") +
  labs(
    x = "", 
    y = "Streamingaktivität (in Stunden)", 
    fill = "Stunden gestreamt"
    )
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-7-1.png" width="672" style="display: block; margin: auto;" />


Besonders interessant finde ich den Höchstwert im Juli. Ich vermute, dass dieser auf meine Geburtstagsfeier zurückgeht, da meine Freundin an diesem Abend als Spotify-DJ engagiert war. Unten schränke ich den Auswertungszeitraum auf den Monat Juli ein und ändere die Skalierung der x-Achse, sodass die einzelnen Tage besser erkennbar sind.



```r
streamVerlauf_spotify %>% 
  filter(date >= "2020-07-01" & date <= "2020-07-31") %>%
  group_by(date) %>%
  summarize(hours = sum(mPlayed) / 60) %>% 
  arrange(date) %>% 
  ggplot(aes(x = date, y = hours)) + 
  geom_col(aes(fill = hours)) +
  scale_fill_gradient(low = "yellow", high = "red") + 
  scale_x_date(date_breaks = "2 days", date_labels = "%d") +
  labs(
    x = "Juli", 
    y = "Streamingaktivität (in Stunden)", 
    fill = "Stunden gestreamt"
    )
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-8-1.png" width="672" style="display: block; margin: auto;" />


Der 25. Juli war ein Samstag und hier fand meine Geburtstagsfeier statt - danke für die Erinnerung Spotify. Lässt sich mit den Daten auch rekonstruieren wie lange die Feier ging? Hierfür schränke ich die Daten noch enger auf den 25. Juli 16 Uhr bis zum 26. Juli 9 Uhr ein und summiere die Streaming-Aktivität für jede Stunde auf.



```r
streamVerlauf_spotify %>% 
  filter(endTime_mez >= "2020-07-25 15:00:00" & 
         endTime_mez <= "2020-07-26 09:00:00") %>%
  group_by(date, hour = hour(endTime_mez)) %>%
  summarize(hours = sum(mPlayed) / 60) %>% 
  ggplot(aes(x = hour, y = hours)) + 
  geom_col(fill = "white", color = "black") +
  facet_grid(. ~ date) +
  scale_x_continuous(breaks = seq(0,24,2)) +
  scale_y_continuous(breaks = seq(0,1,0.5)) +
  labs(x = "Uhr", y = "Streamingaktivität (in Stunden)")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-9-1.png" width="672" style="display: block; margin: auto;" />


***

Meine Freundin war besonders daran interessiert zu sehen, welche Künstler\*innen und Lieder sie am häufigsten auf Spotify angehört hat. Hierzu summiere ich für jeden Künstler\*in (`artistName`) die gestreamten Minuten (`mPlayed`) im gesamten Zeitraum auf und werte die Top-15 aus. 



```r
streamVerlauf_spotify %>%
  group_by(artistName) %>%
  summarise(hours = sum(mPlayed, na.rm = T)/60) %>%
  arrange(desc(hours)) %>%
  slice(1:15) %>%
  ggplot(data = .,
         aes(x = reorder(artistName, hours), 
             y = hours)) +
  geom_bar(stat ="identity", fill = "white", color = "black") +
  coord_flip() +
  scale_y_continuous(breaks = seq(0,110,10)) +
  labs(x = "", y = "Streamingaktivität (in Stunden)") +
  theme(legend.position = "none")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-10-1.png" width="672" style="display: block; margin: auto;" />


Die K-Pop Gruppe *BTS* hat es ihr besonders angetan und ihnen wurden umgerechnet über 4 Tage durchgehender Musikkonsum gewidmet. Welcher Fan bietet mehr? 

Als nächstes schaue ich mir die Top 15 der gestreamten Musiktitel an. Damit man als Pop-Musik-Laie die Titel dem jeweiligen Interpreten zuordnen kann, erstelle ich mit der Funktion `str_c()` eine kombinierte Spalte aus Titel-/ und Künstlername.



```r
streamVerlauf_spotify %>%
  mutate(track.artistName = str_c(trackName, " {",artistName,"}")) %>%
  group_by(track.artistName) %>%
  summarise(hours = sum(mPlayed, na.rm = T)/60) %>%
  arrange(desc(hours)) %>%
  slice(1:15) %>%
  ggplot(aes(x = reorder(track.artistName, hours), 
             y = hours)) +
  geom_bar(stat ="identity", fill = "white", color = "black") +
  coord_flip() +
  scale_y_continuous(breaks = seq(0,7,1)) +
  labs(x = "", y = "Streamingaktivität (in Stunden)") +
  theme(legend.position = "none")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-11-1.png" width="672" style="display: block; margin: auto;" />


Auch hier sind einige Titel der Gruppe *BTS* mit über 6 Stunden Abspielzeit im betrachteten Zeitraum an der Chartspitze vertreten. 

***

Wie intensiv hört man einen Interpreten im Zeitverlauf? Mit dem R-Paket `gghighlight` kann man in einem Liniendiagramm bestimmte Gruppen (`artistName`) hervorheben. Oft wechselt sich dabei ein Liebglingsinterpret mit dem anderen ab.



```r
streamVerlauf_spotify %>% 
  group_by(artistName, date = floor_date(date, "month")) %>% 
  summarize(hours = sum(mPlayed) / 60) %>% 
  ggplot(aes(x = date, y = hours, 
             group = artistName, 
             color = artistName)) + 
  geom_line() +
  gghighlight(artistName %in% c("BTS","Taylor Swift",
                                "Halsey", "Agust D",
                                "Lorde", "Domoinic Fike")
              ) +
  scale_x_date(date_breaks = "1 month", date_labels = "%b") +
  labs(x = "", 
       y = "Streamingaktivität (in Stunden)")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-12-1.png" width="672" style="display: block; margin: auto;" />


***

Zu welchen Tageszeiten hört meine Freundin am häufigsten Musik? Auch das lässt sich mit einer geeigneten Darstellungsmethode leicht beantworten. Mit der Funktion `geom_tile()` aus dem R-Paket `ggplot2` erstelle ich Rechtecke, welche die gesamte Streamingaktivität für jede Stunde innerhalb eines Tages repräsentieren. 



```r
streamVerlauf_spotify %>% 
  group_by(date, hour = hour(endTime_mez)) %>% 
  summarize(minutesListened = sum(mPlayed)) %>% 
  ggplot(aes(x = hour, y = date, fill = minutesListened)) + 
  geom_tile() + 
  scale_fill_gradient(low = "yellow", high = "red") + 
  scale_x_continuous(breaks = seq(0,23,1)) +
  scale_y_date(date_breaks = "1 month", date_labels = "%b %Y") +
  labs(x = "Uhr", y = "", fill = "Minuten gestreamt") 
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-13-1.png" width="672" style="display: block; margin: auto;" />


Kopfhörer auf und ab in die Bahn! Besonders gut zu erkennen sind die täglichen Pendelzeiten zwischen Wohnung und Arbeitsstelle. Diese liegen von Juni bis Ende Oktober am Morgen meist zwischen 5 und 6 Uhr und am Nachmittag zwischen 15 und 16 Uhr. Zwischen November und April verschiebt sich die Streamingaktivität am Morgen und am Nachmittag erkennbar um eine Stunde nach vorne (in der dunklen Jahreszeit kommt man einfach etwas schwerer aus dem Bett). 

Interessant aber auch etwas gruselig, was sich aus den von Spotify gesammelten Nutzerdaten so alles erkennen lässt!


---
## Configure page content in wide column
title:
number_featured: 0 # pulling from mainSections in config.toml
use_featured: false # if false, use most recent by date
number_categories: 0 # set to zero to exclude
show_intro: true
show_outro: false
intro: |

  <style>
  #shinyapps-big .shinyapp {
  	display: inline-block;
  	vertical-align: top;
  	width: 25rem;
  	color: #3d3852;
  	margin: 0 1.125rem 2.5rem;
  	background: #fff;
  	transition: all 250ms;
  	text-align: left;
  	box-shadow: 0 0 1.25rem #ccc;
  	border: 1px solid #eee;
  	float: left;
  }
  #shinyapps-big .applink {
    color: inherit;
    display: block;
    text-decoration: none !important;
  }
  div {
    display: block;
  }
  </style>
  <div id="portfolio">
      <div id="shinyapps-big">
        <div class="shinyapp">
              <a class="applink" href="https://philippleppert.shinyapps.io/Disney/">
                <img class="appimg" src="/img/disney_dice.jpg">
                <div class="apptitle"></div>
                <div class="appdesc"></div>
              </a>
          </div>
        <div class="shinyapp">
              <a class="applink" href="https://philippleppert.shinyapps.io/Bundestagswahlergebnisse-Thueringen/">
                <img class="appimg" src="/img/bundestagswahl_th.jpg">
                <div class="apptitle"></div>
                <div class="appdesc"></div>
              </a>
          </div>
      </div>
  </div>
---

** index doesn't contain a body, just front matter above.
See about/list.html in the layouts folder **

# Showing correlation and geography at the same time.
## This is how you make a pretty and informative bivariate choropleth map with [`tidycensus`](https://cran.r-project.org/web/packages/tidycensus/index.html) and [`biscale`](https://cran.r-project.org/web/packages/biscale/index.html). 

Scatterplots show the relationship between two variables. Choropleth maps show the relative concentration of a variable across geographies. If you want to show the relationship between two variables across geographies, things get a little bit tricky. This is where bivariate choropleth maps come in! 

My map shows how racial composition and educational attainment vary across census tracts in Hennepin County, Minnesota. I noticed a few things after making the map:
* The concentration of non-white people without college degrees in north Minneapolis, northwestern suburbs, and southern suburbs
* The concentration of white people with college degrees along the chain of lakes and Minnehaha Creek
* The absence of red tracts (mostly white without college degrees) in Minneapolis proper

One cosmetic problem I ran into: the `biscale` legend uses unicode arrows like this: →. If you want to use a different typeface &mdash; and don't want to dig through the code for the `bi_legend()` function and replace the arrows &mdash; you'll need to pick a font that supports unicode, like [Noto Sans](https://fonts.google.com/noto/specimen/Noto+Sans). Roboto and Lato won't cut it.

```
library(tidycensus)
library(sf)
library(tidyverse)
library(biscale)
library(cowplot)
library(showtext)
library(ggtext)

#run showtext for cool fonts, match the dpi to system graphics (96)
showtext_opts(dpi = 96)
showtext_auto(enable = TRUE)

#add Noto from google fonts
font_add_google("Noto Sans", "noto")

#check out the 2020 ACS variables
v20 <- load_variables(2020, "acs5", cache = TRUE)
View(v20)

#define variables on race and educational attainment
vars <- c("white" = "B01001H_001",
          "total" = "B01003_001",
          "bach" = "B16010_041",
          "over25" = "B16010_001")

#get ACS data for Census tracts in Hennepin County
hc_sf <- get_acs(geography = "tract",
                 variables = vars,
                 state = "MN",
                 year = 2020,
                 county = "Hennepin",
                 output = "wide",
                 geometry = TRUE)

#calculate the percent white and percent with a college degree, apply the bivariate bins
data <- hc_sf %>% 
  mutate(pctWhite = whiteE/totalE,
         pctBach = bachE/over25E) %>% 
  bi_class(x = pctWhite, y = pctBach, style = "quantile", dim = 3)

#make a legend
legend <- bi_legend(pal = "DkViolet",
                    dim = 3,
                    xlab = "Higher % white",
                    ylab = "Higher % college grads",
                    size = 8) +
  theme(text = element_text(family = "noto"))

#define the plot
map <- ggplot() +
  geom_sf(data = data, mapping = aes(fill = bi_class), color = "white",
          size = 0.1, show.legend = FALSE) +
  bi_scale_fill(pal = "DkViolet", dim = 3) + 
  labs(
    title = "Race and educational attainment in Hennepin County, MN",
    caption = "Scott Shaffer | April 2022 | 2016-2020 ACS 5-year estimates | tidycensus, ggplot, and biscale packages") +
  bi_theme(base_size = 12) +
  theme(text = element_text(family = "noto"),
        plot.title = element_text(family = "noto", face = "bold"),
        plot.subtitle = element_text(family = "noto", color = "#333333"),
        plot.caption = element_text(face = "italic"))

#Add the legend to the plot
finalPlot <- ggdraw() +
  draw_plot(map, 0, 0, 1, 1) +
  draw_plot(legend, .73, .65, 0.25, 0.25)

#draw the final plot
finalPlot

#switch showtext to 300 dpi if you want to ggsave at 300 dpi
showtext_opts(dpi = 300)
ggsave(
  "bivariate_map.png",
  height = 6.4,
  width = 6.4,
  units = "in",
  bg = "#ffffff",
  dpi = 300
)
```
[Bivariate choropleth map of Hennepin County](https://github.com/scottdshaffer/bivariate-choropleth/blob/main/bivariate_map.png?raw=true)
<img src = "bivariate_map.png?raw=true">

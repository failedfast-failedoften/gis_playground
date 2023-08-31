Design an art map with R and OpenStreetMap
================

This is an [R Markdown](http://rmarkdown.rstudio.com) Notebook. When you
execute code within the notebook, the results appear beneath the code.

Try executing this chunk by clicking the *Run* button within the chunk
or by placing your cursor inside it and pressing *Ctrl+Shift+Enter*.

``` r
library(osmdata)
```

    ## Warning: package 'osmdata' was built under R version 4.3.1

    ## Data (c) OpenStreetMap contributors, ODbL 1.0. https://www.openstreetmap.org/copyright

``` r
library(sf)
```

    ## Warning: package 'sf' was built under R version 4.3.1

    ## Linking to GEOS 3.11.2, GDAL 3.6.2, PROJ 9.2.0; sf_use_s2() is TRUE

``` r
library(lwgeom)
```

    ## Warning: package 'lwgeom' was built under R version 4.3.1

    ## Linking to liblwgeom 3.0.0beta1 r16016, GEOS 3.11.2, PROJ 9.2.0

``` r
library(ggplot2)
```

Add a new chunk by clicking the *Insert Chunk* button on the toolbar or
by pressing *Ctrl+Alt+I*.

When you save the notebook, an HTML file containing the code and output
will be saved alongside it (click the *Preview* button or press
*Ctrl+Shift+K* to preview the HTML file).

The preview shows you a rendered HTML copy of the contents of the
editor. Consequently, unlike *Knit*, *Preview* does not run any R code
chunks. Instead, the output of the chunk when it was last run in the
editor is displayed.

``` r
min_lon <- 44.3587; max_lon <- 44.7054
min_lat <- 48.6193; max_lat <- 48.8462

vog <- rbind(x=c(min_lon,max_lon),y=c(min_lat,max_lat))
colnames(vog) <- c("min","max")

min_lon <- 37.5138; max_lon <- 37.7260
min_lat <- 55.6976; max_lat <- 55.8003
msk <- rbind(x=c(min_lon,max_lon),y=c(min_lat,max_lat))
colnames(msk) <- c("min","max")

min_lon <- 20.9324; max_lon <- 21.0742
min_lat <- 52.1972; max_lat <- 52.2862
war <- rbind(x=c(min_lon,max_lon),y=c(min_lat,max_lat))
colnames(war) <- c("min","max")

min_lon <- -1.6723; max_lon <- -1.5010
min_lat <- 54.9400; max_lat <- 55.0242
ncl <- rbind(x=c(min_lon,max_lon),y=c(min_lat,max_lat))
colnames(ncl) <- c("min","max")



min_lon <- -97.8140; max_lon <- -97.6592
min_lat <- 30.2277; max_lat <- 30.3628
aus <- rbind(x=c(min_lon,max_lon),y=c(min_lat,max_lat))
colnames(aus) <- c("min","max")
```

Load Street data for Seattle:

``` r
## DEFINE BOUNDING BOX
min_lon <- -122.3269; max_lon <- -122.1384
min_lat <- 47.4998; max_lat <- 47.6302
sea <- rbind(x=c(min_lon,max_lon),y=c(min_lat,max_lat))
colnames(sea) <- c("min","max")


highways <- sea %>%
  opq()%>%
  add_osm_feature(key = "highway", 
                  value=c("motorway", "trunk",
                          "primary","secondary", 
                          "tertiary",
                          "motorway_link",
                          "trunk_link","primary_link",
                          "secondary_link"
                          ,"tertiary_link"
                          )) %>%
  osmdata_sf()

streets <- sea %>%
  opq()%>%
  add_osm_feature(key = "highway", 
                  value = c("residential", "living_street",
                            "service",##"unclassified",
                            "pedestrian", "footway",
                            "track","path")) %>%
  osmdata_sf()

water <- sea %>%
  opq()%>%
  add_osm_feature(key = "natural", 
                  value = c("water")
                    ) %>%
  osmdata_sf()


land <- sea %>%
  opq()%>%
  add_osm_feature(key = "natural", 
                  value = c("fell",'grassland','wood','beach', 'hill', 'sand', 'scree','rock','valley')
                    ) %>%
  osmdata_sf()

ggplot() +
  geom_sf(data = highways$osm_lines,
          aes(color=highway),
          size = .4,
          alpha = .7)+
  geom_sf(data = streets$osm_lines,
          aes(color=highway),
          size = .3,
          alpha = .65)+
  theme_void()
```

![](tests_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

``` r
ggplot() +
  geom_sf(data = water$osm_multipolygons,
          color = NA,
          fill = 'steelblue')
```

![](tests_files/figure-gfm/unnamed-chunk-3-2.png)<!-- -->

``` r
land <- sea %>%
  opq()%>%
  add_osm_feature(key = "boundary", 
                  #value = c("fell",'grassland','wood','beach', 'hill', 'sand', 'scree','rock','valley')
                    ) %>%
  osmdata_sf()

ggplot() +
  geom_sf(data = land$osm_multipolygons,
          #aes(fill=natural),
          )+
  theme_void()+
  theme(panel.background = element_rect(fill = 'lightblue'),
        panel.grid.major = element_blank(),
        panel.grid.minor = element_blank()
        )
```

![](tests_files/figure-gfm/unnamed-chunk-3-3.png)<!-- -->

``` r
color_roads <- '#469B90'
fill_land <- '#202222'
fill_water <-'#231F24'
accent <- '#73B5B3'

ggplot() +
 
   #geom_sf(data = land$osm_multipolygons,
       #   fill = fill_land,
       #   col = NA)+
   geom_sf(data = water$osm_multipolygons,
          fill = fill_water,
          col = NA)+
   geom_sf(data = streets$osm_lines,
          col = color_roads,
          size = .2,
          alpha = .7) +
   geom_sf(data = highways$osm_lines,
          col = color_roads,
          size = .4,
          alpha = .9)+
   coord_sf(xlim = c(min_lon,max_lon),
           ylim = c(min_lat,max_lat),
         expand = FALSE)+
   ##home
   geom_point(aes(x = -122.21856, y = 47.57999),
             col = accent, 
             size = 4,
             alpha = 0.8)+
   ##Alisa
   geom_point(aes(x = -122.18758, y = 47.62041),
             col = accent, 
             size = 4,
             alpha = 0.8)+
  
  #labs(title = 'EVERGREEN')+
   theme_void()+

   theme(legend.position = 'none',
        plot.caption.position = 'panel',
        plot.background = element_rect(fill = '#394141'),
        #panel.background = element_rect(fill = '#394141'),
        #panel.grid.major = element_blank(),
        #panel.grid.minor = element_blank()
        ) 
```

![](tests_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

``` r
streets <- vog %>%
  opq()%>%
  add_osm_feature(key = "highway", 
                  value = c("residential", "living_street",
                            "service",##"unclassified",
                            "pedestrian", "footway",
                            "track","path")) %>%
  osmdata_sf()

ggplot() +
  geom_sf(data = streets$osm_lines,
          aes(color=highway),
          size = .4,
          alpha = .65)+
  theme_void()
```

![](tests_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

``` r
volga <- vog %>%
  opq()%>%
  add_osm_feature(key = "natural", 
                  value = c("water")##c("river", "oxbow","canal",
                            ##"ditch", "lock","fish_pass","lake","basin",
                           ##"reservoir", "pond", "lagoon")
                    ) %>%
  osmdata_sf()

ggplot() +
  geom_sf(data = volga$osm_multipolygons,
          aes(),
          color= 'steelblue',
          fill = 'steelblue')+
  theme_void()
```

![](tests_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

``` r
color_roads <- rgb(0.42,0.449,0.488)
ggplot() +
  geom_sf(data = streets$osm_lines,
          col = color_roads,
          size = .3,
          alpha = .6) +
  geom_sf(data = highways$osm_lines,
          col = color_roads,
          size = .4,
          alpha = .85)+
  coord_sf(xlim = c(min_lon,max_lon),
           ylim = c(min_lat,max_lat),
         expand = FALSE)+
  theme(legend.position = F) + theme_void()
```

![](tests_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

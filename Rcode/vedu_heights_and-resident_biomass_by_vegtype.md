ventenata heights and resident biomass by vegetation type
================
Claire Tortorelli
August 18, 2021

Read in hieght data

``` r
vdheights <- read_csv(here("data", "2020_vd_heights_by_vegtype.csv"))

#set vegtype to a factor
vdheights$vegtype <- factor(vdheights$vegtype, c("ARRI", "ARAR", "SEEP"))
```

read in biomass data

``` r
bio <- read_csv(here("data", "biomass_data_2019_2020.csv"))
```

### Organize data for analysis

``` r
#add col for plot number, plot, and treatment: C = community, uncleared; N = no neighbors, cleared (apologies for the confusing codes)

bio$plotno <- factor(substr(bio$plot_quad,5,6))
bio$plot <- factor(substr(bio$plot_quad,1,6))

#convert veg type to factor
bio$vegtype <- factor(substr(bio$plot_quad,1,4), levels = c("ARRI", "ARAR" , "SEEP"))

#remove cleared subplots since these do not represent naturally occurring resident biomass in 2020
bio_uncleared <- bio %>% filter(grepl('C', plot_quad))
```

### model ventenata height response to vegetation type

``` r
#model ventenata height response to vegtype
mheights <- lme(log(Height.cm) ~ vegtype, random = ~ 1|plotno/Plot, data = vdheights)

plot(mheights)
summary(mheights)
```

Compare means with emmeans

``` r
(em <- emmeans(mheights, specs = revpairwise ~ vegtype, type = "response"))
emdf <- data.frame(em$emmeans)

emdf$vegtype <- factor(emdf$vegtype, labels = c("ARRI"="scab-flat", "ARAR"="low sage-steppe", "SEEP" = "wet meadow"))
```

### model resident biomass response to vegetation type

``` r
#random effects for plot and block (plot no)
mbio <- lme(log(resident20_g) ~ vegtype, random = ~ 1|plotno/plot, data = bio_uncleared)
plot(mbio)
summary(mbio)
```

Extract means with emmeans for plotting

``` r
#use emmeans to reduce 
(embio <- emmeans(mbio, specs = revpairwise ~ vegtype, type = "response", adjust= "none"))
confint(embio)
embio_df <- data.frame(embio$emmeans) 
embio_df$vegtype <- factor(embio_df$vegtype, labels = c("ARRI"="scab-flat", "ARAR"="low sage-steppe", "SEEP" = "wet meadow"))
```

### Plot results

plot vedu mean height and confidence intervals at each vegetation tpe

``` r
pal <- c("#A6761D", "#E6AB02" , "#666666")

( g1 = ggplot(emdf, aes(y = response, x = vegtype, color = vegtype)) +
# Define stock as group this week as well as set x and y axes
geom_point(position = position_dodge(width = .75), size = 3 ) + # Add points, dodge by group
geom_errorbar(aes(ymin = lower.CL, ymax = upper.CL, width = 0.5), size = 1,
position = position_dodge(width = .75) ) + # Add errorbars, dodge by group
theme_bw(base_size = 14) +
labs(y = expression("mean"~italic("V. dubia")~" height"))+
scale_color_manual(values = pal))+
    guides(color = FALSE)+
theme(
panel.grid.minor = element_blank(),
axis.title.x=element_blank(),# Remove gridlines
panel.grid.major.x = element_blank())
```

Plot mean resident biomass and 95% CIs at each vegetation type

``` r
( g2 = ggplot(embio_df, aes(y = response, x = vegtype, color = vegtype)) +
# Define stock as group this week as well as set x and y axes
geom_point(position = position_dodge(width = .75), size = 3 ) + # Add points, dodge by group
geom_errorbar(aes(ymin = lower.CL, ymax = upper.CL, width = 0.5), size = 1,
position = position_dodge(width = .75) ) + # Add errorbars, dodge by group
theme_bw(base_size = 14) +
  scale_y_continuous(breaks = c(4,8,12))+
labs(y = "mean resident biomass (g/subplot)")+
scale_color_manual(values = pal))+
    guides(color = FALSE)+
theme(
panel.grid.minor = element_blank(),
axis.title.x=element_blank(),# Remove gridlines
panel.grid.major.x = element_blank())
```

combine figures with cowplot

``` r
library(cowplot)

plot_grid(g2, g1, labels = c("(a)", "(b)"))
```

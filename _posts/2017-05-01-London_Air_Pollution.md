---
title: "Modeling $$PM_{2.5}$$ in Greater London Area Using Remote Sensing of Aerosol Optical Depth "
date: 2017-05-11
tags: [air pollution, geo-analysis, London]
header:
  image: "/images/ParsePermits/Boston.jpg"
excerpt: "Data Wrangling, Data Science, Messy Data"
mathjax: "true"
---

## Intro

### $$PM_{2.5}$$
Since the health effects of particulate matter (PM) pronounced by the groundbreaking six cities study whose pioneer work linked air pollution to elevated mortality rate[^1], a large body of literature has found exposure to particulate matter associated to a wide range of deleterious health outcomes , such as asthma , cardiovascular disease, lung cancer, and cognitive decline . Fine particulate matter, defined as particulate matters with aerodynamic diameter less than 2.5 μm, was the foci of most of these studies, since they travel deeper into alveolus and impose higher health risk. A recent in vitro study has also found that low dose of PM2.5 exposure for only 24 hours can induce acute oxidative stress, inflammation and pulmonary impairment in healthy mice[^2]. Therefore, highly resolved-spatial and temporal evaluation of ambient PM levels becomes particularly relevant in furthering scientific basis for next-step air pollution regulation.


### AOD
Aerosol optical depth (AOD) measures light extinction by aerosol in the atmospheric column above the earth’s surface. It is measured by NASA’s Moderate Resolution Imaging Spectroradiometer (MODIS) daily globally. Thanks to its wide spatial availability, AOD compliments the sparsity of ground level PM monitors readings in works using spatial interpolation to better profile individual’s exposure level.

### Other factors
PM-AOD models suffer from two limitations; first, factors correlated to both AOD and ambient PM concentration, such as planetary boundary height and meteorological oscillation, may introduce high instability to the model; second, the association between AOD and PM has tendency to fluctuate on a daily basis, predominated by certain temporal and spatial correlation patterns. Both issues call for more sophisticated model calibration than simple AOD and PM2.5 empirical regressions. Some investigations tackled the first issue by controlling for local meteorological information[^3], land use information  or both; incorporating whether the aerosol is confined to the surface planetary boundary layer (PBL) or aloft can also engender a better agreement with surface PM level ; those findings suggested endogenizing those information in the predictive model, although a reliable ex ante model specification could not be provided. For the second issue, sophisticated statistical methods such as linear mixed effect model  and neural network  are employed to deliver a more stable parameter estimation, advocating data-driven model selection method in this category of predictive models.

The aim of this study is to calibrate an AOD-land use-PM2.5 model using a data-driven model selection to interpolate PM2.5 concentrations spatially and temporally in Greater London area.

## Data

### Study area
Our study is located in the Greater London Area, a rectangular region ranging from -0.561° to 0.372° in longitude and 51.217° N to 51.798° N in latitude. The study region comprised a mixture of urban and rural areas and a wide variation of population density, which ranges from the most populated borough of Islington (14,517 people/$$km^2$$) in the greater London area to the least populated neighborhood Sevenoaks in county of Kent (317 people/ $$km^2$$).

<img src="{{ site.url }}{{ site.baseurl }}/images/LondonFog/picture1.png",title = 'Figure 1. the Greater London Area'>

### AOD
AOD data can be downloaded from [NASA MODIS](https://neo.sci.gsfc.nasa.gov/view.php?datasetId=MODAL2_M_AER_OD), although those data is very chunky and sparse, requiring significant amount of storage and computational power to process (Thanks to my colleague [Qian, Di](https://www.linkedin.com/in/qiandi/) for the initial acquisition). NASA developed an algorithms (multi-angle implementation of atmospheric correction algorithm[^4]) to deduce AOD from MODIS remote-sensing data, which have a theoretical precision of ±0.05τ . NASA provides AOD readings at a precision of 1 km × 1 km grid cells. The study region was covered by an orthogonal array that consisted of 22,878 such grid cells.

### Meteorological data and air pollution data
Spatial-Temporal Exposure Assessment Methods (STEAM) project led by King’s College, London provided $$PM_{2.5}$$ daily concentration at 29 ground-level monitors, as well as meteorological factors comprising of barometric pressure (BP), temperature (TMP), cloudiness (CLD), dew point temperature (DEWA), wind speed (WDSP), wind direction (WDIR) and planetary boundary layer height (PBLH). (These are proprietary data.) In preparation, we spatially joined each day’s meteorological data to AOD readings, using an algorithm that allowed for adjacency allocation varying daily. Number of households and population density was extracted from the 2011 census and merged to the AOD grid cells.

<img src="{{ site.url }}{{ site.baseurl }}/images/LondonFog/picture2.png",title = 'Figure 2. Average AOD readings from 2004 to 2014 in the study area and locations of PM2.5 monitors. '>


## Model training
### The $ex ante$ model
$$PM_{2.5_{ij}}=β_0+b_{1j}+(β_{1}+b_{2j})×AOD_{ij}+β_3×((AOD_{ij},Time_j,BP_{ij},TMP_{ij},CLD_{ij},DWEA_{ij},WDSP,PBLH_j))^2   +β_4×WDIR_{ij}+ϵ_{ij} , (b_1j,b_2j )~ N(0,Σ) $$

$$AOD_{ij}$$ notifies AOD readings on day j at site i. $$b_1$$ and $$b_2$$ stand for the random intercept and random slope of AOD through time.

### LASSO

The $ex ante$ model allows for many interaction terms in the $$β_3$$ term, which may result in overfitting. For model selection, we use east absolute shrinkage and selection operator (LASSO) to identiy the optimal predition model. At the optimal penalty $$\lambda$$


We then used the model specification generated from LASSO to predict the spatial and temporal distribution of PM2.5 in Greater London area. All model training and prediction was done using R version 3.3.0 (2016-05-03).
[^1]: Dockery, Douglas W., et al. "An association between air pollution and mortality in six US cities." New England journal of medicine 329.24 (1993): 1753-1759.
[^2]: Riva, D. R., et al. "Low dose of fine particulate matter (PM2. 5) can induce acute oxidative stress, inflammation and pulmonary impairment in healthy mice." Inhalation toxicology 23.5 (2011): 257-267.
[^3]: Schaap, M., et al. "Exploring the relation between aerosol optical depth and PM 2.5 at Cabauw, the Netherlands." Atmospheric Chemistry and Physics 9.3 (2009): 909-925.
[^4]:   Lyapustin, A., et al. "Multiangle implementation of atmospheric correction (MAIAC): 2. Aerosol algorithm." Journal of Geophysical Research: Atmospheres 116.D3 (2011).

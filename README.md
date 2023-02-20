# line-to-point-spatial-conversion-raster-extraction

# Crate SpatialPointsDataframe from the SpatialLinesDataFrame by distance


The objective of the process is to execute R programming code that produces results equivalent to the QGIS SAGA "Profiles from lines" function. 
Additionally, this code has the capability to specify the distance from each point.

## Load library
```
library(rgeos)
library(rgdal)
library(raster)
library(sp)
library(tidyverse)
```

## Set Work directory
```
setwd("")
```

## Read SpatialLinesDataFrame data
```
SeenlandRoute = rgdal::readOGR("Bicycle_Routes/13_Seenland-Route/13_Seenland-Route_tracks_geom.shp")
```

## Read rater DEM
```
DEM <- raster("DEM/Lusatia_DEM_UTM.tif")
```

## Convert to UTM
```
utm_proj <- CRS("+init=epsg:32633")
SeenlandRoute_UTM <- sp::spTransform(SeenlandRoute, utm_proj)
```

## Calculate the length of the line
```
route_length <- rgeos::gLength(SeenlandRoute_UTM)
```

## Specify the desire distance of each points
```
distance = 10 # m 
point_distances <- seq(0, route_length, by = distance)
```

## Create points from the line
```
points_matrix <- rgeos::gInterpolate(SeenlandRoute_UTM, point_distances)
```

## Convert SpatialPoints to SpatialPointsDataframe
```
points_matrix = as(points_matrix,"SpatialPointsDataFrame")
```

## Create data frame table 
```
points_matrix@data = data.frame(matrix(nrow = length(point_distances), ncol = 6))
colnames(points_matrix@data) = c("LINE_ID", "ID", "DIST", "X", "Y", "Z")
```

## Insert information to the data frame table
```
points_matrix@data$ID = 1:length(point_distances)
points_matrix@data$X = points_matrix@coords[,1]
points_matrix@data$Y = points_matrix@coords[,2]
points_matrix@data$DIST = seq(0, route_length, by = distance)
points_matrix@data$Z = raster::extract(DEM, points_matrix)
```

## Plot the extracted value over the distance
```
ggplot(points_matrix@data, aes(x = DIST, y = Z)) +
  geom_line()
```

## Export the SpatialPointsDataframe as shapefile
```
writeOGR(obj=points_matrix, dsn="Bicycle_Routes/13_Seenland-Route", layer="13_Seenland-Route_tracks_points_UTM", driver="ESRI Shapefile") 
```






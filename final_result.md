---
title: "Qualitative Risk Analysis _ Inha"
author: "Kyungtak Kim"
date: '2020 3 26 '
output:
 html_document:
   keep_md: TRUE
  
   
 
---


```r
# 패키지 설치
#install.packages("sf")
#install.packages("tmap")
#install.packages("dplyr")


#원본 데이터 읽기
hazard <- read.csv('output/hazard_result.csv')
exposure <- read.csv('output/exposure_result.csv')
vulnerability <- read.csv('output/vulnerability_result.csv')
capacity <- read.csv('output/capacity_result.csv')


#데이터 결합
DB <- cbind(hazard, c(exposure[,4:5],vulnerability[,4:5],capacity[,4:6]))



# 홍수피해위험지수 표준화 함수 설정
standard <- function(x){
  return((x-min(x))/(max(x)-min(x)))
}


# 16년~17년 홍수피해위험지수 산정
result_index_16 <- as.data.frame((rowSums(DB[,c("X16_hazard","X16_exposure","X16_vulnerability","X16_capacity")]))/4)
colnames(result_index_16) <- c("X16_result_index")
result_index_17 <- as.data.frame((rowSums(DB[,c("X17_hazard","X17_exposure","X17_vulnerability","X17_capacity")]))/4)
colnames(result_index_17) <- c("X17_result_index")
result_index <- cbind(DB[,1:3], c(result_index_16,result_index_17))


# 연도별 데이터 프레임에 표준화 적용
result <- as.data.frame(lapply(result_index[,4:5],standard))
colnames(result) <- c("X16_result", "X17_result")
result <- cbind(DB[,1:3], result)


# 시군 shp 파일 불러오기
library(sf)
```

```
## Linking to GEOS 3.6.1, GDAL 2.2.3, PROJ 4.9.3
```

```r
analysis <- st_read("input/analysis.shp")
```

```
## Reading layer `analysis' from data source `C:\00_R\0_Git\KRM_inha\input\analysis.shp' using driver `ESRI Shapefile'
## Simple feature collection with 161 features and 3 fields
## geometry type:  MULTIPOLYGON
## dimension:      XY
## bbox:           xmin: 746109.3 ymin: 1458771 xmax: 1387956 ymax: 2068444
## epsg (SRID):    NA
## proj4string:    +proj=tmerc +lat_0=38 +lon_0=127.5 +k=0.9996 +x_0=1000000 +y_0=2000000 +ellps=GRS80 +units=m +no_defs
```

```r
# 폴리곤 에러 체크(기본 파일을 에러 수정한 파일로 변경하였음)
#st_is_valid(analysis)
#library(lwgeom)
#analysis <- st_make_valid(analysis)
st_is_valid(analysis)
```

```
##   [1] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
##  [16] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
##  [31] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
##  [46] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
##  [61] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
##  [76] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
##  [91] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
## [106] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
## [121] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
## [136] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
## [151] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE
```

```r
# shp파일에 연도별 홍수피해위험지수(표준화 적용) 추가
library(dplyr)
```

```
## Warning: package 'dplyr' was built under R version 3.6.3
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
analysis <- right_join(analysis, result[,3:5])
```

```
## Joining, by = "SGG"
```

```r
# 폴리곤 단순화
library(tmap)
```

```
## Warning: package 'tmap' was built under R version 3.6.3
```

```r
analysis_simp <- st_simplify(analysis, dTolerance = 50)


# 결과 확인
tmap_mode("plot")
```

```
## tmap mode set to plotting
```

```r
breaks = c(0, 0.2, 0.4, 0.6, 0.8, 1)
facets=c("X16_result", "X17_result")
tm_shape(analysis_simp)+
  tm_polygons(facets, breaks=breaks, palette = c("green", "greenyellow", "yellow", "orange", "red"), legend.reverse = TRUE)+
  tm_facets(ncol = 2)+
  tm_layout(legend.position = c("right", "bottom"))+
  tm_compass(type = "rose", position = c("right", "top"), size = 2.0)+
  tm_scale_bar(breaks = c(0, 25, 50, 100, 150, 200), position = c("left", "bottom"))
```

![](final_result_files/figure-html/unnamed-chunk-1-1.png)<!-- -->

```r
# 결과값 저장
write.csv(result, 'output/final_result.csv')


# 열 명칭별 의미

# Name : 161개 시군별 영문명
# NameK : 161개 시군별 한글명
# SGG : 시군구 코드
# X16_hazard : 16년도 hazard 지수(표준화 적용)
# X17_hazard : 17년도 hazard 지수(표준화 적용)
# X18_hazard : 18년도 hazard 지수(표준화 적용)
# X16_exposure : 16년도 Exposure 지수(표준화 적용)
# X17_exposure : 17년도 Exposure 지수(표준화 적용)
# X16_vulnerability : 16년도 Vulnerability 지수(표준화 적용)
# X17_vulnerability : 17년도 Vulnerability 지수(표준화 적용)
# X16_capacity : 16년도 Capacity 지수(표준화 적용)
# X17_capacity : 17년도 Capacity 지수(표준화 적용)
# X18_capacity : 18년도 Capacity 지수(표준화 적용)
# X16_result_index : 16년도 홍수피해위험지수
# X17_result_index : 17년도 홍수피해위험지수
# X16_result : 16년도 홍수피해위험지수(표준화 적용)
# X17_result : 17년도 홍수피해위험지수(표준화 적용)
```


---
title: "final_result.R"
author: "Kyungtak Kim"
date: "2020-03-26"
---


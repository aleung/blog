---
title: 使用OsmAndMapCreator及时更新OsmAnd地图
date: 2017-06-17 15:02:32
tags: [GPS_GIS, Map]
---

使用 [OpenStreetMap](https://www.openstreetmap.org) （简称为OSM）地图数据的手机应用有好些，[OsmAnd](http://osmand.net/) 是其中比较优秀的一个。OsmAnd的地图数据每月更新一次，延后了半个月到一个多月。作为一个OSM mapper，期望能够马上看到和用到最近做出的修改，等一个月实在太久了。

OSM原始地图数据在 [planet.osm](http://wiki.openstreetmap.org/wiki/Planet.osm) 的网站可以下载到，但数据文件是全球的，非常大。从 [GeoFabrik](http://download.geofabrik.de/) 可以下载按国家分割的数据文件。中国区的PBF格式数据是330MB。但每天重新下载一个数据包还是太大了，其实更新的数据不是很多。网站上还提供每天（planet.osm还有每小时）的变更数据包（osc文件），用 `osmupdate` 工具就能自动下载变更并且生成新的完整数据文件。 

有了最新的 osm.pbf 数据文件，就可以使用 `OsmAndMapCreator` 生成 OsmAnd 可用的 OBF 格式离线地图文件了。生成的过程非常耗内存和CPU，全中国的地图在一台 Core i3-4370 CPU 的电脑上用了90分钟，需要分配 10GB 内存才够用（否则会出现 OutOfMemory 错误）。生成出来的 OBF 文件也有845MB之巨。为了缩短时间，我设置了 `osmupdate` 的`-b`参数，过滤出广东中部区域的数据，这样就快很多，几分钟搞定。

处理的流程如下图。实际上安装好软件，修改好配置后，每次更新就执行一个脚本，过程全自动。

```
Previous full map data (osm.pbf) \
                              [osmupdate] --> Latest full map data (osm.pbf)
            Recent changes (osc) /                      |
                                                        v
[OsmAnd] <------ Latest offline map (obf) <------- [OsmAndMapCreator]
```

步骤和脚本放在 gist：https://gist.github.com/aleung/42776348d023b689e8c337b82a3c7cd5，适用于 MacOS 和 Linix，需要自行修改文件中的目录路径。工具在Win平台也可以用，但脚本就要重新写了。
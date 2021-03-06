# Emergency-vehicle-preemption-simulator

## Terminal commands to run simulation

At first, make sure SUMO path value is set.

```
export SUMO_HOME=/usr/share/sumo
```

In second step, cd to directory of your project.
To generate network execute this command:

```
netconvert --node-files=hello.nod.xml --edge-files=hello.edg.xml --output-file=hello.net.xml
```

To run simulation, execute:

```
sumo -c map.sumo.cfg
```

or

```
sumo-gui -c map.sumo.cfg
```

## Prepare OSM map
At first we need to remove all the highways where car ride is not allowed.
In JSOM editor delete nodes, selected by filters:

```
("highway"="service")
("highway"="footway")
("highway"="cycleway")
("highway"="track")
("highway"="path")
("highway"="steps")
("highway"="pedestrian")
```
After deleting the appropriate nodes, save file and close JOSM.

In second step we need to remove all the nodes with tag `"action"="delete"` in previously saved file.
This can be done by executing python file: `./purge.py -i map.osm`


## Import OSM map

Link: http://sumo.dlr.de/wiki/Networks/Import/OpenStreetMap#Importing_the_Road_Network

Convert osm map to sumo format

```
netconvert --osm-files map.osm -o map.net.xml
```

Convert osm map to enable overtaking
```
netconvert --opposites.guess.fix-lengths --opposites.guess true  --osm-files map.osm -o map.net.xml --junctions.join --tls.default-type actuated
```

Recomended options
```
--geometry.remove --roundabouts.guess --ramps.guess --junctions.join --tls.guess-signals --tls.discard-simple --tls.join
```

### Prepare optimized traffic lights phase durations
```
C:\Python27\python.exe "C:\Program Files (x86)\DLR\Sumo\tools\tlsCycleAdaptation.py" -n map.net.xml -r map.rou.xml -o newTLS.add.xml
```

### Importing additional Polygons (Buildings, Water, etc.)

Save xml bellow as typemap.xml

```
<polygonTypes>
    <polygonType id="waterway"                name="water"       color=".71,.82,.82" layer="-4"/>
    <polygonType id="natural"                 name="natural"     color=".55,.77,.42" layer="-4"/>
    <polygonType id="natural.water"           name="water"       color=".71,.82,.82" layer="-4"/>
    <polygonType id="natural.wetland"         name="water"       color=".71,.82,.82" layer="-4"/>
    <polygonType id="natural.wood"            name="forest"      color=".55,.77,.42" layer="-4"/>
    <polygonType id="natural.land"            name="land"        color=".98,.87,.46" layer="-4"/>

    <polygonType id="landuse"                 name="landuse"     color=".76,.76,.51" layer="-3"/>
    <polygonType id="landuse.forest"          name="forest"      color=".55,.77,.42" layer="-3"/>
    <polygonType id="landuse.park"            name="park"        color=".81,.96,.79" layer="-3"/>
    <polygonType id="landuse.residential"     name="residential" color=".92,.92,.89" layer="-3"/>
    <polygonType id="landuse.commercial"      name="commercial"  color=".82,.82,.80" layer="-3"/>
    <polygonType id="landuse.industrial"      name="industrial"  color=".82,.82,.80" layer="-3"/>
    <polygonType id="landuse.military"        name="military"    color=".60,.60,.36" layer="-3"/>
    <polygonType id="landuse.farm"            name="farm"        color=".95,.95,.80" layer="-3"/>
    <polygonType id="landuse.greenfield"      name="farm"        color=".95,.95,.80" layer="-3"/>
    <polygonType id="landuse.village_green"   name="farm"        color=".95,.95,.80" layer="-3"/>

    <polygonType id="tourism"                 name="tourism"     color=".81,.96,.79" layer="-2"/>
    <polygonType id="military"                name="military"    color=".60,.60,.36" layer="-2"/>
    <polygonType id="sport"                   name="sport"       color=".31,.90,.49" layer="-2"/>
    <polygonType id="leisure"                 name="leisure"     color=".81,.96,.79" layer="-2"/>
    <polygonType id="leisure.park"            name="tourism"     color=".81,.96,.79" layer="-2"/>
    <polygonType id="aeroway"                 name="aeroway"     color=".50,.50,.50" layer="-2"/>
    <polygonType id="aerialway"               name="aerialway"   color=".20,.20,.20" layer="-2"/>

    <polygonType id="shop"                    name="shop"        color=".93,.78,1.0" layer="-1"/>
    <polygonType id="historic"                name="historic"    color=".50,1.0,.50" layer="-1"/>
    <polygonType id="man_made"                name="building"    color="1.0,.90,.90" layer="-1"/>
    <polygonType id="building"                name="building"    color="1.0,.90,.90" layer="-1"/>
    <polygonType id="amenity"                 name="amenity"     color=".93,.78,.78" layer="-1"/>
    <polygonType id="amenity.parking"         name="parking"     color=".72,.72,.70" layer="-1"/>
    <polygonType id="power"                   name="power"       color=".10,.10,.30" layer="-1" discard="true"/>
    <polygonType id="highway"                 name="highway"     color=".10,.10,.10" layer="-1" discard="true"/>

    <polygonType id="boundary" name="boundary"    color="1.0,.33,.33" layer="0" fill="false" discard="true"/>
    <polygonType id="admin_level" name="admin_level"    color="1.0,.33,.33" layer="0" fill="false" discard="true"/>
</polygonTypes>
```

Convert poly map with command:
```
polyconvert --net-file map.net.xml --osm-files map.osm --type-file typemap.xml -o map.poly.xml
```


### Generate routes and trips
"randomTrips.py" generates a set of random trips for a given network (option -n)
The trips are distributed evenly in an interval defined by begin (option -b, default 0) and end time (option -e, default 3600) in seconds. 
The number of trips is defined by the repetition rate (option -p, default 1) in seconds.

First command generate .rou.xml file, which is used as an input to second command, where trips are generated.
```
python $SUMO_HOME/tools/randomTrips.py -n map.net.xml -e 100 -l

python $SUMO_HOME/tools/randomTrips.py -n map.net.xml -r map.rou.xml -e 100 -l
```


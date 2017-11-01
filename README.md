# osmtogtfs
Extracts partial GTFS feed from OSM data.

OpenStreeMaps data contain information about bus, tram, train and other public transport means.
This information is not enought for providing a complete routing service, most importantly because
it lacks timing data. However, it still contains routes, stop positions and some other useful data.

This tool takes an OSM file or URI and thanks to [osmium](http://osmcode.org/) library converts it to a partial 
[GTFS](https://developers.google.com/transit/gtfs/reference/) feed. GTFS is the de facto standard 
for sharing public transport information and there are many tools around it. The resulting feed would
not validate if you check it, because it is of course partial. Nevertheless, it is yet valuable to us.

# Installation
This tool uses osmium which is a C++ library built using boost, so one should install that first.
The best way would be using the package manager of your OS and installing [pyosmium](https://github.com/osmcode/pyosmium).
Afterwards clone the repo and install it:

    $ git clone https://github.com/hiposfer/osmtogtfs & cd osmtogtfs
    $ python setup.py install


This will install `osmtogtfs.py` executable on your OS. You can also directly run the script found
in the source code. Make sure to run it with python 3.

# Usage
Run the tool over your OSM data source (or whatever osmium accepts):

    python osmtogtfs.py <osmfile>

After a while, depending on the file size, a file named `gtfs.zip` will be produced inside the working directory.

# Implementation Notes
In this section we describe important aspects of the implementation in order to help understand how the program works.

## Field Mapping
GTFS feeds could contain up to thirteen different CSV files with `.txt` extension. Six of these files are required for a valid
field, including _agency.txt_, _stops.txt_, _routes.txt_, _trips.txt_, _stop_tiles.txt_ and _calendar.txt_. 
Each file contains a set of comumns. Some columns are required and some are optional. 
Most importantly, not all the fields necessary to build a GTFS feed are available in OSM data. 
Therefore we have to generate some fileds ourselves or leave them blank.
Below we cover how the values for each column of the files that we produce at the moment are produced.

### agency.txt
We use _operator_ tag on OSM relations which are tagged as `relation=route` to extract agency information. 
However, there are some routes without operator tags. In such cases we use a dummy agency:

    {'agency_id': -1, 'agency_name': 'Unkown agency', 'agency_timezone': ''}

#### agency_id
We use the _operator_ value to produce _agency_id_:

    agency_id = abs(hash(operator_name))

#### agency_name
Is the value of _operator_ tag.

#### agency_timezone
Since there is no explicit operator timezone tag, we try to guess it based on the coordinate of the elements in a relation.

### stops.txt

 - stop_id: is the node id from OSM
 - stop_name: value of _name_ tag or _Unknown_
 - stop_lon: longitute of the node
 - stop_lat: latitute of the node

### routes.txt

 - route_id: id of the OSM relation element
 - route_short_name: value of _name_ or _ref_ tag of the relation
 - route_long_name: a combination of _from_ and _to_ tags on the relation otherwise empty
 - route_type: we map OSM route types to GTFS
 - route_url: link to the relation on openstreetmaps.org
 - route_color: value of the _color_ tag if present otherwise empty
 - agency_id: ID of the agency otherwise -1

 #### OSM to GTFS Route Type Mapping
 Below is the mapping that we use, the left column is the OSM value and the right column is the 
 corresponding value from GTFS specification (make sure the see the code for any changes):

    tram: 		0
    light_rail: 0
    subway: 	1
    rail: 		2
    railway: 	2
    train: 		2
    bus: 		3
    ex-bus: 	3
    ferry: 		4
    cableCar: 	5
    gondola: 	6
    funicular: 	7


# Lincense
MIT

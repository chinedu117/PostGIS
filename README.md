# PostGIS
Git Repo for the "Intro to PostGIS" playlist


### https://gis.stackexchange.com/questions/283445/geoalchemy-get-closest-point-to-another-point

For performance reasons, it is better to use distance_centroid rather than ST_Distance in your query:

from geoalchemy2.comparator import Comparator
q = db.session.query(cities1000.c.name, 
                    cities1000.c.stateRegion, 
                    cities1000.c.country).\
    order_by(
    Comparator.distance_centroid(cities1000.c.geom,
                 func.Geometry(func.ST_GeographyFromText(
                     'POINT({} {})'.format(lon, lat))))).limit(1).first()
ST_Distance forces the database to calculate the distance between the query Point and every city in your table of candidate features, then sort them all and take the first result. Whereas the PostGIS <-> operator that translates into distance_centroid in geoalchemy2 will do an index based Nearest Neighbour (NN) search. An index search partitions the space into multiple boxes (an R-Tree data structure) which considerably speeds up the nearest neighbor search for large databases.

To better understand how R-Tree and K-NN search work, you can read this great post on the Mapbox blog.
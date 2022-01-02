# RSP-QL --> FlinkSQL query rewritings (made by OntopStream)

## Example #1 (window query)

RSP-QL:
```
PREFIX : <http://www.semanticweb.org/TaxiRides#>

SELECT *
FROM NAMED WINDOW :wind1 ON :trips [RANGE PT1M STEP PT1M]
WHERE {
       ?x a :Ride.
       ?x :hasTaxi ?y.
}
```

FlinkSQL:
```
SELECT DISTINCT `v1`.`rideId` AS `rideId1m5`, `v1`.`taxiId` AS `taxiId1m5`
FROM `Rides` `v1`
WHERE (`v1`.`rideId` IS NOT NULL AND `v1`.`taxiId` IS NOT NULL)
GROUP BY `v1`.`rideId`,`v1`.`taxiId`,TUMBLE(`rideTime`, INTERVAL '1' MINUTE)
```

## Example #2 (real-time query)

RSP-QL:
```
PREFIX : <http://www.semanticweb.org/TaxiRides#>

SELECT ?ride ?taxi ?driver
WHERE {
      ?ride a :Ride.
      ?ride :hasTaxi ?taxi.
      ?driver a :Driver; :hasShift [:uses ?taxi].
}
```

FlinkSQL:
```
SELECT `v5`.`driverId1m4` AS `driverId1m4`, `v5`.`rideId1m5` AS `rideId1m5`, `v5`.`taxiId1m5` AS `taxiId1m5`
FROM LATERAL (SELECT DISTINCT `v3`.`driverId` AS `driverId1m4`, `v1`.`rideId` AS `rideId1m5`, `v1`.`taxiId` AS `taxiId1m5`, ('http://www.semanticweb.org/TaxiRides/Shift#' || CAST(`v2`.`driverId` AS STRING) || '_' || CAST(`v1`.`taxiId` AS STRING)) AS `v0`
FROM `Rides` `v1`, `DriverChanges` `v2`, `DriverChanges` `v3`
WHERE (('http://www.semanticweb.org/TaxiRides/Shift#' || CAST(`v2`.`driverId` AS STRING) || '_' || CAST(`v1`.`taxiId` AS STRING)) = ('http://www.semanticweb.org/TaxiRides/Shift#' || CAST(`v3`.`driverId` AS STRING) || '_' || CAST(`v3`.`taxiId` AS STRING)) AND `v1`.`rideId` IS NOT NULL AND `v2`.`driverId` IS NOT NULL AND `v3`.`driverId` IS NOT NULL AND `v3`.`taxiId` IS NOT NULL AND `v1`.`taxiId` = `v2`.`taxiId`)
) `v5`
```

## Example #3 (real-time query)

RSP-QL:
```
PREFIX : <http://www.semanticweb.org/TaxiRides#>

SELECT ?ride ?taxi ?drivingShift ?tip
WHERE {
      ?ride a :Ride.
      ?ride :hasTaxi ?taxi.
      ?drivingShift :uses ?taxi.
      ?fare :isReferredTo ?ride; :tip ?tip.
}
```

FlinkSQL:
```
SELECT DISTINCT `v1`.`rideId` AS `rideId1m5`, `v1`.`taxiId` AS `taxiId1m5`, `v3`.`tip` AS `tip1m11`, ('http://www.semanticweb.org/TaxiRides/Shift#' || CAST(`v2`.`driverId` AS STRING) || '_' || CAST(`v1`.`taxiId` AS STRING)) AS `v0`
FROM `Rides` `v1`, `DriverChanges` `v2`, `Fares` `v3`
WHERE (`v2`.`driverId` IS NOT NULL AND `v3`.`tip` IS NOT NULL AND `v1`.`taxiId` = `v2`.`taxiId` AND `v1`.`rideId` = `v3`.`rideId`)

```

## Example #4 (inference query (a :Taxi is also a :Car))

RSP-QL:
```
PREFIX : <http://www.semanticweb.org/TaxiRides#>

SELECT ?x ?y ?type
FROM NAMED WINDOW :wind1 ON :trips [RANGE PT1M STEP PT1M]
WHERE {
       ?x a :Ride.
       ?x :hasTaxi ?y.
       ?y a ?type.
}
```

FlinkSQL:
```
SELECT DISTINCT `v19`.`rideId1m5` AS `rideId1m5`, `v19`.`taxiId1m5` AS `taxiId1m5`, `v19`.`v9` AS `v9`
FROM (SELECT `v1`.`rideId` AS `rideId1m5`, `v8`.`taxiId1m5` AS `taxiId1m5`, 'http://www.semanticweb.org/TaxiRides#Car' AS `v9`
FROM `Rides` `v1`, LATERAL (SELECT DISTINCT `v6`.`taxiId1m5` AS `taxiId1m5`
FROM (SELECT `v2`.`taxiId` AS `taxiId1m5`
FROM `Rides` `v2`
WHERE `v2`.`taxiId` IS NOT NULL
GROUP BY `v2`.`taxiId`,TUMBLE(`rideTime`, INTERVAL '1' MINUTE)
UNION ALL
SELECT `v4`.`taxiId` AS `taxiId1m5`
FROM `DriverChanges` `v4`
WHERE (`v4`.`driverId` IS NOT NULL AND `v4`.`taxiId` IS NOT NULL)
GROUP BY `v4`.`taxiId`,`v4`.`driverId`,TUMBLE(`usageStartTime`, INTERVAL '1' MINUTE)
) `v6`
) `v8`
WHERE (`v1`.`rideId` IS NOT NULL AND `v8`.`taxiId1m5` = `v1`.`taxiId`)
UNION ALL
SELECT `v10`.`rideId` AS `rideId1m5`, `v17`.`taxiId1m5` AS `taxiId1m5`, 'http://www.semanticweb.org/TaxiRides#Taxi' AS `v9`
FROM `Rides` `v10`, LATERAL (SELECT DISTINCT `v15`.`taxiId1m5` AS `taxiId1m5`
FROM (SELECT `v11`.`taxiId` AS `taxiId1m5`
FROM `Rides` `v11`
WHERE `v11`.`taxiId` IS NOT NULL
GROUP BY `v11`.`taxiId`,TUMBLE(`rideTime`, INTERVAL '1' MINUTE)
UNION ALL
SELECT `v13`.`taxiId` AS `taxiId1m5`
FROM `DriverChanges` `v13`
WHERE (`v13`.`driverId` IS NOT NULL AND `v13`.`taxiId` IS NOT NULL)
GROUP BY `v13`.`taxiId`,`v13`.`driverId`,TUMBLE(`usageStartTime`, INTERVAL '1' MINUTE)
) `v15`
) `v17`
WHERE (`v10`.`rideId` IS NOT NULL AND `v17`.`taxiId1m5` = `v10`.`taxiId`)
) `v19`
```

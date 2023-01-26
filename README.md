# data-engineer

SELECT COUNT(*)
FROM green_taxi_data
WHERE 1 = 1
	AND lpep_pickup_datetime::DATE = '2019-01-15'
	AND lpep_dropoff_datetime::DATE = '2019-01-15';
-- LIMIT 10;

WITH data_clean AS (
	SELECT lpep_pickup_datetime::DATE as pickup
		, trip_distance
	FROM green_taxi_data
), max_by_day AS (
	SELECT pickup
		, MAX(trip_distance) AS max_distance
	FROM data_clean
	GROUP BY pickup
)
SELECT * FROM max_by_day ORDER BY max_distance DESC;

SELECT passenger_count
	, COUNT(*) total_trips
FROM green_taxi_data
WHERE 1 = 1
	AND lpep_pickup_datetime::DATE = '2019-01-01'
	-- AND lpep_dropoff_datetime::DATE = '2019-01-01'
	AND passenger_count BETWEEN 2 AND 3
GROUP BY passenger_count


WITH zone_astoria AS (
	SELECT "LocationID"
		, "Zone"
	FROM zones
	WHERE "Zone" = 'Astoria'
), filter_trips AS (
	SELECT gtd.lpep_pickup_datetime
		, gtd."PULocationID" AS pu_location_id
		, gtd."DOLocationID" AS do_location_id
		, gtd.extra
		, gtd.tip_amount
		, za."Zone" AS zone_name
	FROM green_taxi_data AS gtd
	INNER JOIN zone_astoria AS za
	ON gtd."PULocationID" = za."LocationID"
	WHERE tip_amount > 0
)
SELECT *
FROM filter_trips AS fp
INNER JOIN zones as z
ON fp.do_location_id = z."LocationID"
ORDER BY tip_amount DESC

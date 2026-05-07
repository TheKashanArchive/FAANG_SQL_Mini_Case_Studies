# Project Overview

This notebook contains SQL queries used to analyze operational and business data from a simulated Uber ride-sharing platform containing drivers, riders, vehicles, trips, promotions, earnings, and ratings.

The queries progress from basic SQL queries to advanced analytical queries, demonstrating practical techniques used in data analysis.

# Table of Contents

1. Count Active Drivers by City
2. Find the Total Count of Completed Trips
3. Retrieve All Drivers with a High Rating
4. Find the Latest Trip for Each Rider
5. List All Trips Where Surge Pricing Was Applied
6. Calculate Total Fare Collected by City
7. Analyze Driver Performance Metrics by City
8. Calculate the Percentage of Drivers by Vehicle Type
9. Analyze Trip Volume by Hour to Identify Peak Periods
10. Analyze Rider Retention
11. Analyze Promotion Effectiveness

---

# 1. Count Active Drivers by City

## Question

Count active drivers by city.

## SQL Query

```sql
SELECT c.city_name AS city_name,
COUNT(d.driver_id) AS active_driver_count
FROM drivers d
INNER JOIN cities c
ON d.city_id = c.city_id
WHERE d.status = 'active'
GROUP BY c.city_name
ORDER BY active_driver_count DESC;
```

## Answer / Result

Returns the total number of active drivers operating in each city.

## Insight

This query uses the COUNT() aggregate function with GROUP BY to analyze driver distribution across different cities.

---

# 2. Find the Total Count of Completed Trips

## Question

Find the total count of completed trips.

## SQL Query

```sql
SELECT COUNT(t.trip_id) AS total_completed_trips
FROM trips t
WHERE t.trip_status = 'completed';
```

## Answer / Result

Returns the total number of trips that were successfully completed.

## Insight

This query measures completed ride activity by filtering only trips with a completed status.

---

# 3. Retrieve All Drivers with a High Rating

## Question

Retrieve all drivers with a high rating.

## SQL Query

```sql
SELECT d.first_name || ' ' || d.last_name AS driver_name,
d.rating AS rating,
d.total_trips AS total_trips,
c.city_name AS city_name
FROM drivers d
INNER JOIN cities c
ON d.city_id = c.city_id
WHERE d.rating >= 4.5
ORDER BY d.rating DESC;
```

## Answer / Result

Returns all drivers with ratings greater than or equal to 4.5.

## Insight

This query helps identify top-performing drivers based on customer ratings.

---

# 4. Find the Latest Trip for Each Rider

## Question

Find the latest trip for each rider.

## SQL Query

```sql
SELECT r.first_name || ' ' || r.last_name AS rider_name,
t.pickup_datetime AS latest_trip_date,
t.pickup_location AS pickup_location,
t.fare_amount AS fare_amount
FROM riders r
INNER JOIN trips t
ON r.rider_id = t.rider_id
WHERE t.pickup_datetime = (
SELECT MAX(t2.pickup_datetime)
FROM trips t2
WHERE t2.rider_id = r.rider_id
)
ORDER BY latest_trip_date DESC;
```

## Answer / Result

Returns the most recent completed trip information for each rider.

## Insight

This query uses a correlated subquery with MAX() to retrieve the latest trip activity for every rider.

---

# 5. List All Trips Where Surge Pricing Was Applied

## Question

List all trips where surge pricing was applied.

## SQL Query

```sql
SELECT t.trip_id AS trip_id,
r.first_name || ' ' || r.last_name AS rider_name,
d.first_name || ' ' || d.last_name AS driver_name,
t.surge_multiplier AS surge_multiplier,
t.fare_amount AS fare_amount
FROM trips t
INNER JOIN riders r
ON t.rider_id = r.rider_id
INNER JOIN drivers d
ON t.driver_id = d.driver_id
WHERE t.trip_status = 'completed'
AND t.surge_multiplier > 1.0
ORDER BY t.surge_multiplier DESC;
```

## Answer / Result

Returns all completed trips where surge pricing was applied.

## Insight

This query identifies high-demand rides where fare prices increased due to surge pricing.

---

# 6. Calculate Total Fare Collected by City

## Question

Calculate total fare collected by city.

## SQL Query

```sql
SELECT c.city_name AS city_name,
SUM(t.fare_amount) AS total_fare
FROM cities c
INNER JOIN trips t
ON c.city_id = t.city_id
WHERE t.trip_status = 'completed'
GROUP BY c.city_id, c.city_name
ORDER BY total_fare DESC;
```

## Answer / Result

Returns the total fare revenue generated in each city.

## Insight

This query uses the SUM() aggregate function to analyze revenue performance across cities.

---

# 7. Analyze Driver Performance Metrics by City

## Question

Analyze driver performance metrics by city.

## SQL Query

```sql
SELECT c.city_name AS city_name,
COUNT(DISTINCT d.driver_id) AS driver_count,
ROUND(AVG(d.rating), 2) AS avg_rating,
SUM(d.total_trips) AS total_trips,
ROUND(COALESCE(SUM(de.total_earning), 0), 2) AS total_earnings
FROM cities c
INNER JOIN drivers d
ON c.city_id = d.city_id
LEFT JOIN driver_earnings de
ON d.driver_id = de.driver_id
GROUP BY c.city_id, c.city_name
ORDER BY total_earnings DESC;
```

## Answer / Result

Returns driver statistics including ratings, trips, and earnings grouped by city.

## Insight

This query combines multiple aggregate functions to evaluate operational driver performance across different locations.

---

# 8. Calculate the Percentage of Drivers by Vehicle Type

## Question

Calculate the percentage of drivers by vehicle type.

## SQL Query

```sql
SELECT v.vehicle_type AS vehicle_type,
COUNT(d.driver_id) AS driver_count,
ROUND(
COUNT(d.driver_id) * 100.0 /
(
SELECT COUNT(*)
FROM drivers
WHERE status = 'active'
), 2
) AS percentage
FROM vehicles v
INNER JOIN drivers d
ON v.vehicle_id = d.vehicle_id
WHERE d.status = 'active'
GROUP BY v.vehicle_type
ORDER BY percentage DESC;
```

## Answer / Result

Returns the percentage distribution of active drivers across vehicle types.

## Insight

This query helps analyze the popularity and operational share of each vehicle category.

---

# 9. Analyze Trip Volume by Hour to Identify Peak Periods

## Question

Analyze trip volume by hour to identify peak periods.

## SQL Query

```sql
SELECT strftime('%H', pickup_datetime) AS hour_of_day,
COUNT(trip_id) AS trip_count,
ROUND(AVG(fare_amount), 2) AS avg_fare
FROM trips
WHERE trip_status = 'completed'
GROUP BY strftime('%H', pickup_datetime)
ORDER BY trip_count DESC;
```

## Answer / Result

Returns trip counts and average fares grouped by pickup hour.

## Insight

This query helps identify peak ride demand periods throughout the day.

---

# 10. Analyze Rider Retention

## Question

Analyze rider retention by finding riders who have taken more than one trip.

## SQL Query

```sql
WITH trip_gaps AS (
SELECT rider_id,
pickup_datetime,
LAG(pickup_datetime) OVER (
PARTITION BY rider_id
ORDER BY pickup_datetime
) AS prev_trip_date,
julianday(pickup_datetime) -
julianday(
LAG(pickup_datetime) OVER (
PARTITION BY rider_id
ORDER BY pickup_datetime
)
) AS gap_days
FROM trips
WHERE trip_status = 'completed'
),

rider_gap_stats AS (
SELECT rider_id,
COUNT(*) + 1 AS total_trips,
ROUND(AVG(gap_days), 1) AS avg_gap_days
FROM trip_gaps
WHERE prev_trip_date IS NOT NULL
GROUP BY rider_id
)

SELECT r.first_name || ' ' || r.last_name AS rider_name,
rgs.total_trips AS total_trips,
rgs.avg_gap_days AS avg_gap_days
FROM rider_gap_stats rgs
INNER JOIN riders r
ON rgs.rider_id = r.rider_id
WHERE rgs.total_trips >= 2
ORDER BY rgs.avg_gap_days ASC;
```

## Answer / Result

Returns riders with multiple completed trips along with their average trip gap.

## Insight

This query uses window functions and CTEs to analyze rider retention and repeat ride behavior.

---

# 11. Analyze Promotion Effectiveness

## Question

Analyze promotion effectiveness by comparing promotional ride usage.

## SQL Query

```sql
SELECT p.promotion_code AS promotion_code,
p.description AS description,
COUNT(rp.rider_promotion_id) AS times_used,
ROUND(SUM(rp.discount_applied), 2) AS total_discount,
ROUND(AVG(rp.discount_applied), 2) AS avg_discount_per_use
FROM promotions p
INNER JOIN rider_promotions rp
ON p.promotion_id = rp.promotion_id
GROUP BY p.promotion_id,
p.promotion_code,
p.description
ORDER BY total_discount DESC;
```

## Answer / Result

Returns promotion usage statistics including total discounts and average discount value.

## Insight

This query evaluates how promotional campaigns influenced rider activity and discount utilization.

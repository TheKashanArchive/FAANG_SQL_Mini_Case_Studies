# Project Overview

This notebook contains SQL queries used to analyze operational and business data from a simulated Uber ride-sharing platform containing drivers, riders, vehicles, trips, promotions, earnings, and ratings.

The queries progress from basic SQL queries to advanced analytical queries, demonstrating practical techniques used in data analysis.

The focus is on:
- driver performance analysis
- rider behavior tracking
- trip and revenue analysis
- surge pricing evaluation
- promotion effectiveness
- operational efficiency metrics

---

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

Returns the number of active drivers operating in each city.

## Insight

This query uses **INNER JOIN with GROUP BY aggregation and COUNT() function** to measure driver distribution across cities and evaluate supply availability per region.

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

Returns total number of successfully completed trips.

## Insight

This query uses a **COUNT aggregation with filtering condition** to measure total ride completion volume on the platform.

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

Returns drivers with ratings 4.5 and above.

## Insight

This query uses an **INNER JOIN with WHERE filtering and ORDER BY ranking** to identify top-performing drivers based on customer feedback.

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

Returns the most recent trip per rider.

## Insight

This query uses a **correlated subquery with MAX() aggregation** to extract the latest transactional activity for each rider.

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

Returns completed trips affected by surge pricing.

## Insight

This query uses **multi-table INNER JOINs with conditional filtering** to analyze high-demand pricing behavior in the ride system.

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

Returns total revenue generated per city.

## Insight

This query uses **SUM() aggregation with GROUP BY city dimension** to evaluate revenue distribution across geographic locations.

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

Returns city-level driver performance metrics.

## Insight

This query uses **multiple aggregations with INNER JOIN and LEFT JOIN** to evaluate operational efficiency and driver performance across cities.

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

Returns distribution of active drivers by vehicle type.

## Insight

This query uses **GROUP BY with COUNT() and a scalar subquery** to compute proportional distribution of driver vehicle categories.

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

Returns trip demand patterns by hour.

## Insight

This query uses **time-based grouping with aggregation functions** to identify peak demand hours and pricing behavior trends.

---

# 10. Analyze Rider Retention

## Question

Analyze rider retention.

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

Returns riders with repeated trips and average time gaps.

## Insight

This query uses **CTEs and window functions (LAG)** to analyze behavioral retention patterns and repeat usage frequency.

---

# 11. Analyze Promotion Effectiveness

## Question

Analyze promotion effectiveness.

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
GROUP BY p.promotion_id, p.promotion_code, p.description
ORDER BY total_discount DESC;
```

## Answer / Result

Returns promotion usage and discount impact metrics.

## Insight

This query uses **aggregation with GROUP BY across promotional tables** to evaluate marketing effectiveness and discount utilization patterns.

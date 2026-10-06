# GoAsia Benchmark — Test Questions

## About GoAsia

GoAsia is a fictional APJC super-app operating across 10 markets (Singapore, Malaysia, Thailand, Philippines, Vietnam, Indonesia, India, Japan, Australia, South Korea). It runs two business lines:

- **Rides**: Ride-hailing with 50M trips, 500K drivers, 10M riders, surge pricing, promos, and driver earnings
- **Logistics**: Parcel delivery with 50M shipments, 200K couriers, 100K shippers, 500 warehouses, SLA tracking, route legs, and invoicing

Shared dimensions connect both domains via 10 countries, 102 cities, and 1,295 zones. Additionally, 38K operational documents (incident reports, rider complaints, driver surveys, safety audits, support chats, regulatory filings, news articles, market research) are indexed for search.

**Use case**: You are a GoAsia operations analyst. Ask both your Snowflake Cortex Agent and Databricks Genie Supervisor Agent the same questions and compare the accuracy of their responses.

---

## Pattern 1: Entity Disambiguation

*Confusing entity names that differ by 1-2 characters. Does the agent silently pick one or surface both?*

**Q1: Compare GoFlash's delivery rate — are they meeting expectations?**
Expected: Should identify 2 companies — GoFlash Logistics Pte Ltd (79.59%) and GoFlash Logistix Corp (81.40%). Separate rates for each.

**Q2: How many shipments did SwiftLine handled totally?**
Expected: SwiftLine Express (649 shipments). Should flag SwiftLane Express (621) as a near-match.

**Q3: What is Pacific Cargo's total invoiced amount?**
Expected: Should flag Pacific Cargo Holdings ($0, no invoices) separately from Pacifica Cargo Services ($188,874, 29 invoices).

**Q4: What is the throughput at Jakarta Central warehouse?**
Expected: Should identify Jakarta Hub Central (WH-0001) and flag Jakarta Hub Centre (WH-0002) as a different facility.

**Q5: Show me delivery stats for Singapore North hub**
Expected: Singapore North Hub (WH-0003, MICRO) — ~100K shipments, ~87% delivery rate. Should not conflate with Singapore Northpoint Hub (WH-0004).

---

## Pattern 2: Wrong Column / Filter Selection

*Multiple columns or filters could answer the same question but give different results.*

**Q6: What is the total fare revenue from actual completed rides?**
Expected: ~$686.5M using `fare` column with `status='COMPLETED'`. Not $710M (is_completed trap) or $800M (total_fare_amount).

**Q7: What is the total fare amount collected in Singapore?**
Expected: ~$28.1M using `fare` for completed rides with pickup in Singapore.

**Q8: How many packages were successfully delivered in total?**
Expected: **40,997,968** using `status='DELIVERED'`. Not 43.5M (is_delivered includes RETURNED).

**Q9: What is our delivery success rate?**
Expected: **82.0%** (status='DELIVERED' / total). Not 87.0% (is_delivered trap).

**Q10: What is the ride completion rate by country?**
Expected: ~88% across all countries. Not ~91% (is_completed trap).

**Q11: What is GoAsia's total ride revenue?**
Expected: ~$686.5M (fare, COMPLETED only). Not $780M (all statuses) or $800M (total_fare_amount).

**Q12: What is our SLA compliance rate?**
Expected: **97.6%** (SLA met / delivered shipments). Not 80.0% (SLA met / all shipments — wrong denominator).

**Q13: What is GoAsia's total combined revenue across rides and logistics?**
Expected: ~$939.5M (rides $686.5M + logistics $253.0M). Requires combining both domains with correct filters.

**Q14: What is GoAsia's total workforce cost — combining driver earnings and courier earnings?**
Expected: ~$471.1M (drivers $353.5M net + couriers $117.6M net). Not $594M (gross earnings).

**Q15: How many total successful operations (completed rides + delivered shipments) did GoAsia handle?**
Expected: ~85.0M (44.0M completed rides + 41.0M delivered shipments). Not 89.0M (trap booleans).

**Q16: Calculate our delivery success rate, then check our operational docs for any reported reasons behind delivery failures**
Expected: 82.0% delivery rate + document insights on failure reasons (weather, traffic, lost parcels, etc.).

**Q17: What is our ride completion rate, and what do incident reports say about the most common reasons rides don't complete?**
Expected: 88.0% completion rate + doc themes (driver cancellations, long waits, route deviations).

**Q18: What is our SLA compliance rate, and what do news articles report about GoAsia's delivery performance in the region?**
Expected: 97.6% SLA compliance + news/market research insights on delivery performance.

---

## Pattern 3: Structural Gap / LEFT JOIN Detection

*Entities with zero rows in fact tables. Can the agent detect "never" / "zero" entities?*

**Q19: How many registered drivers have never completed any ride?**
Expected: **166,229** out of 500,000 drivers (33.25%).

**Q20: How many registered shippers have never sent a single shipment?**
Expected: **20,139** out of 100,000 shippers (20.14%).

**Q21: How many riders signed up but never took a ride and never redeemed a promo?**
Expected: **2,008,974** fully dormant riders.

**Q22: How many couriers never delivered a package AND never earned any money?**
Expected: **66,369** out of 200,000 couriers.

**Q23: How many of GoAsia's total workforce (drivers + couriers) have never generated any revenue?**
Expected: **232,598** (166,229 idle drivers + 66,369 idle couriers out of 700,000 total workforce).

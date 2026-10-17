# B08-RailPorter
This is effective Rail Porter database system,mostly worked on Database managment
**23AIE303 · DATABASE MANAGEMENT SYSTEMS** 
# **RailPort** 

## Railway Station Porter Booking System 

A PostgreSQL-backed platform that lets passengers book luggage porters at railway stations, lets porters accept and complete jobs, and gives administrators a complete view of stations, bookings, payments and ratings. 

##### **PROJECT REPORT** 

|Team number|Team 8|
|---|---|
|Team members|**Suraj B**– CB.SC.U4AIE24105<br>**Sai Karthik C**– CB.SC.U4AIE24107<br>**Virinchi Sai Ch**– CB.SC.U4AIE24158<br>**Jatin Chandra Gupta K**– CB.SC.U4AIE24162|
|Course|23AIE303 – Database Management Systems|
|Technology|PostgreSQL 16 · Node.js / Express · React (Vite)|
|Date|16 October 2026|



RailPort – Railway Station Porter Booking System 

23AIE303 DBMS · Project Report 

### **Contents** 

|**2. Team Members**|**3**|
|---|---|
|**3. Introduction and Problem Statement**|**4**|
|**4. Conceptual and Logical Database Design**|**6**|
|**5. Normalization and Functional Dependencies**|**8**|
|**6. Advanced SQL Implementation and Query Design**|**11**|
|**7. ER Diagram and UML Diagram**|**17**|
|**8. Relational Schema**|**20**|
|**9. Database Tables**|**22**|
|**10. Conclusion and Future Scope**|**29**|
|**Appendix A – Test cases and validation**|**30**|
|**Appendix B – REST API summary**|**31**|
|**Appendix C – How to reproduce the database**|**31**|
|**Appendix D – Complete DDL (01_schema.sql)**|**32**|



Page 2 

RailPort – Railway Station Porter Booking System 

23AIE303 DBMS · Project Report 

### **2. Team Members** 

This project was designed, implemented and documented by **Team 8** of the course 23AIE303 – Database Management Systems. 

|**#**|**Name**|**Roll No.**|
|---|---|---|
|1|Suraj B|CB.SC.U4AIE24105|
|2|Sai Karthik C|CB.SC.U4AIE24107|
|3|Virinchi Sai Ch|CB.SC.U4AIE24158|
|4|Jatin Chandra Gupta K|CB.SC.U4AIE24162|




Page 3 

RailPort – Railway Station Porter Booking System 

23AIE303 DBMS · Project Report 

### **3. Introduction and Problem Statement** 

#### **3.1 Background** 

Porters (coolies) are a familiar part of Indian railway stations, yet finding one is still a manual, unstructured experience. A passenger arriving with heavy luggage has to look around the platform, negotiate a price on the spot and hope the porter can be found again at the right coach. Porters, on the other hand, have no reliable way of knowing who needs help, where, and on which train. 

#### **3.2 Problem statement** 

**In short:** there is no common system that connects passengers who need luggage help with porters who are available, records what was agreed and paid, and measures service quality. 

There is no common digital system that connects passengers who need luggage help with porters who are available at that station, records what was agreed and paid, and measures service quality. 

- **Hard to find a porter** – passengers search for porters physically, often with little time before the train leaves. 

- **No fixed price or record** – the fare is negotiated every time; nothing proves what was agreed or paid. 

- **No coordination** – porters do not know which train, coach or platform a passenger is waiting at; free porters and waiting passengers are never matched. 

- **No accountability** – there is no rating or complaint history, so good porters are not rewarded and poor service is not visible. 

- **No management view** – station authorities cannot see demand, revenue or porter performance. 

#### **3.3 Objectives** 

- Store passengers, porters, stations, platforms, bookings, payments and ratings in a normalized relational database with strong integrity rules. 

- Let a passenger raise a porter request with train number, coach, luggage count, pickup platform and destination. 

- Let porters see open requests at their own station, accept them and update the booking status until completion. 

- Record payments (Cash/UPI/Card) and passenger ratings, and keep every porter's average rating up to date automatically. 

- Give the admin full CRUD control and analytical queries (revenue per station, porter leaderboard, unpaid jobs, demand vs supply). 

#### **3.4 Users and scope** 

|**Actor**|**What they can do**|
|---|---|
|Passenger|Register/login, request a porter, track status, pay, rate, cancel.|
|Porter|Login, set availability, view open requests at their station, accept, update status, see ratings and earnings.|
|Admin|Manage users, stations, platforms, porters; monitor bookings, payments and ratings.|



#### **3.5 Technology and architecture** 

RailPort is a three-tier application. The React front end talks JSON over HTTP to an Express (Node.js) REST API; the API reaches PostgreSQL through a _pg_ connection pool and uses only parameterized queries ($1, $2, …), which prevents SQL injection. Passwords are hashed with bcrypt before they are stored. 

Page 4 

RailPort – Railway Station Porter Booking System 

23AIE303 DBMS · Project Report 



_Figure 3.1 – Three-tier architecture of RailPort_ 

Page 5 

RailPort – Railway Station Porter Booking System 

23AIE303 DBMS · Project Report 

### **4. Conceptual and Logical Database Design** 

#### **4.1 Entities** 

Seven entities were identified from the requirements. The ER diagram is in Section 7 (Figure 7.1). 

|**Entity**|**Purpose**|
|---|---|
|**users**|Every person who logs in: passengers who book, porters who serve and admins who manage. One row per<br>account; the role column decides which dashboard the person sees.|
|**stations**|Railway stations where the service operates (name, short code, city, platform count).|
|**platforms**|Individual platforms of a station. A platform cannot exist without its station.|
|**porters**|Extra details for users whose role is PORTER: the station they are posted at, badge number, live availability, the<br>platform they are standing on and their average rating.|
|**bookings**|The core transaction: a passenger asks for help with luggage on a given train/coach; a porter is assigned later<br>and the booking moves through its status life-cycle.|
|**payments**|The payment made for a booking (cash, UPI or card) with its status, transaction reference and time. At most one<br>payment record per booking.|
|**ratings**|Passenger feedback (1-5 stars and a review) about the porter who served a completed booking.|



#### **4.2 Relationships and cardinality** 

|**Entities**|**Relationship**|**Cardinality**|**Meaning**|**Implemented by**|
|---|---|---|---|---|
|users – porters|registers as|1 : 0..1|A PORTER user has exactly one porter profile;<br>passengers/admins have none.|porters.user_id (UNIQUE,<br>FK)|
|stations – platforms|has|1 : N|A station has many platforms; a platform<br>belongs to one station.|platforms.station_id|
|stations – porters|posts|1 : N|Many porters are posted at a station; a porter<br>works at one station.|porters.station_id|
|platforms – porters|stands on|0..1 : N|A porter may currently stand on one platform<br>(optional).|porters.current_platform<br>_id|
|users – bookings|requests|1 : N|A passenger makes many bookings.|bookings.passenger_id|
|porters – bookings|handles|0..1 : N|A booking has at most one porter (none until<br>accepted); a porter handles many bookings.|bookings.porter_id<br>(nullable)|
|stations – bookings|held at|1 : N|Every booking happens at exactly one station.|bookings.station_id|
|platforms – bookings|pickup from|0..1 : N|A booking may name the pickup platform.|bookings.pickup_platform<br>_id (nullable)|
|bookings – payments|paid by|1 : 0..1|A booking has at most one payment record (its<br>status moves PENDING → SUCCESS/FAILED).|payments.booking_id<br>(UNIQUE)|
|bookings – ratings|rated as|1 : 0..1|A completed booking may be rated once.|ratings.booking_id<br>(UNIQUE)|
|users – ratings|gives|1 : N|A passenger can give many ratings.|ratings.passenger_id|
|porters – ratings|receives|1 : N|A porter receives many ratings.|ratings.porter_id|



#### **4.3 Design decisions** 

- **Single users table with a role.** All three kinds of actors need the same login data. A PORTER additionally has one row in _porters_ (generalization implemented as a one-to-one table with a UNIQUE foreign key). 

- **Bookings are separate from payments and ratings.** A booking can have several payment attempts (FAILED, then SUCCESS) and at most one rating, so these are separate tables instead of columns of bookings. 

- **porter_id is nullable in bookings.** A request exists before any porter accepts it; the NULL value is how "open requests" are found ( _porter_id IS NULL_ ). 

- **One payment per booking.** payments.booking_id is UNIQUE, so a booking has at most one payment record whose status moves from PENDING to SUCCESS or FAILED. 

Page 6 

RailPort – Railway Station Porter Booking System 

23AIE303 DBMS · Project Report 

- **Delete behaviour is chosen per relationship.** ON DELETE CASCADE is used for dependents that make no sense alone (platforms of a station, the porter profile of a deleted user, payments and ratings of a deleted booking). Bookings, however, keep plain foreign keys to users, porters and stations: such rows cannot be deleted while bookings still refer to them, so passenger history is never lost by accident. 

- **Status columns use CHECK constraints** instead of free text, giving enumerated domains (role, availability, booking status, payment method/status). 

#### **4.4 Logical design: ER-to-relational mapping rules applied** 

|**Rule**|**Where it was applied**|
|---|---|
|Strong entity → table with its key as primary key|users, stations, porters, bookings, payments, ratings|
|1 : N relationship → foreign key on the N side|platforms.station_id, porters.station_id, bookings.passenger_id / station_id,<br>payments.booking_id, ratings.passenger_id / porter_id|
|1 : 1 (or 1 : 0..1) → foreign key with UNIQUE|porters.user_id, ratings.booking_id|
|Optional participation → nullable foreign key|bookings.porter_id, bookings.pickup_platform_id, porters.current_platform_id|
|Enumerated attribute → CHECK domain|users.role, bookings.status, payments.payment_method …|
|Derived attribute kept deliberately → stored value|porters.rating (average of ratings.stars)|



Page 7 

RailPort – Railway Station Porter Booking System 

23AIE303 DBMS · Project Report 

### **5. Normalization and Functional Dependencies** 

**Result in one line:** all seven relations are in 1NF, 2NF and 3NF and, apart from one documented optional dependency in _bookings_ (Note 1), in BCNF. 

Every relation was checked against the four normal forms. Six relations are in **First** , **Second** , **Third** and **Boyce-Codd** normal form; _bookings_ is too, apart from one deliberate, documented relaxation (Note 1). 

#### **5.1 First normal form (1NF)** 

All attributes are atomic (no repeating groups or arrays). Each table has a primary key, and each column holds one kind of value – for example the passenger, the porter and the platform are three separate columns of bookings, never a combined text. 

#### **5.2 Second normal form (2NF)** 

2NF is violated only when a non-key attribute depends on _part_ of a composite key. Every table uses a single-column surrogate primary key (the only composite key, (station_id, platform_number), is a candidate key whose only dependent is platform_id). Hence no partial dependency exists. 

#### **5.3 Third normal form and BCNF** 

For every non-trivial functional dependency X → Y in a relation, X must be a (super)key. The dependencies below were derived from the business rules and the constraints in the schema. 

###### **users   candidate keys: user_id; phone; email** 

- › user_id → name, phone, email, password, role, created_at 

- › phone → user_id (and all other attributes) 

- › email → user_id (and all other attributes) 

**Result:** Every determinant is a candidate key → BCNF. 

###### **stations   candidate keys: station_id; station_code** 

- › station_id → station_name, station_code, city, number_of_platforms 

- › station_code → station_id (and all others) 

**Result:** Every determinant is a candidate key → BCNF. 

###### **platforms   candidate keys: platform_id; (station_id, platform_number)** 

- › platform_id → station_id, platform_number 

- › (station_id, platform_number) → platform_id 

**Result:** Both determinants are candidate keys → BCNF. 

###### **porters   candidate keys: porter_id; user_id; badge_number** 

- › porter_id → user_id, station_id, badge_number, availability_status, current_platform_id, rating 

- › user_id → porter_id (and all others) 

- › badge_number → porter_id (and all others) 

**Result:** Every determinant is a candidate key → BCNF. (rating is a stored aggregate, see note 3.) 

Page 8 

RailPort – Railway Station Porter Booking System 

23AIE303 DBMS · Project Report 

###### **bookings   candidate keys: booking_id** 

- › booking_id → passenger_id, porter_id, station_id, pickup_platform_id, train_number, coach_number, luggage_count, destination, fare, status, created_at, completed_at 

- › pickup_platform_id → station_id (only when pickup_platform_id is not NULL – see note 1) 

**Result:** booking_id is the only key. 3NF/BCNF hold except for the optional pickup_platform_id → station_id dependency, discussed in note 1. 

###### **payments   candidate keys: payment_id; booking_id** 

- › payment_id → booking_id, amount, payment_method, payment_status, transaction_id, payment_time 

- › booking_id → payment_id (and all others) (booking_id is UNIQUE) 

**Result:** Both determinants are candidate keys → BCNF. 

###### **ratings   candidate keys: rating_id; booking_id** 

- › rating_id → booking_id, passenger_id, porter_id, stars, review, created_at 

- › booking_id → rating_id, passenger_id, porter_id, stars, review, created_at (booking_id is UNIQUE) 

   - **Result:** Every determinant is a candidate key → BCNF. (passenger/porter repeated from the booking, see note 2.) 

#### **5.4 Deliberate redundancies and how they are controlled** 

**Note 1 – bookings.station_id and pickup_platform_id.** A platform belongs to exactly one station, so whenever pickup_platform_id is present it determines station_id. Strictly, booking_id → pickup_platform_id → station_id is a transitive dependency and station_id is a non-prime attribute, so this is the one place where bookings is not perfectly 3NF. The designers kept station_id on purpose: pickup_platform_id is optional (NULL when the passenger does not know the platform), and a booking must still say _at which station_ the service happens. Using the dependency to drop the column would lose that information. The risk is an inconsistent pair (platform of station 2 stored with station 1); it can be removed completely with a composite foreign key (pickup_platform_id, station_id) → platforms(platform_id, station_id), listed under future improvements. The same reasoning applies to porters.current_platform_id and porters.station_id. 

**Note 2 – ratings.passenger_id and ratings.porter_id.** Both can be read from the booking. They are kept so that "ratings of a porter" and "ratings given by a passenger" need no extra join. Because booking_id is UNIQUE in ratings it is itself a candidate key, so the dependency booking_id → passenger_id, porter_id does not violate 3NF/BCNF. Optional trigger T2 (04_enhancements.sql) rejects rows whose passenger/porter differs from the booking. 

**Note 3 – porters.rating and stations.number_of_platforms.** These are derived values (AVG of ratings; COUNT of platforms) stored for speed. porters.rating is a cached aggregate: the API updates it, and optional trigger T3 can keep it in sync automatically. number_of_platforms is the real total of the station, while the platforms table only stores the platforms that are registered in the system, so it is not derivable from it. 

**Note 4 – payments is 1 : 1 with bookings.** Because payments.booking_id is UNIQUE, payments could in principle be merged into bookings. It was kept separate because a payment has its own life-cycle (status, method, transaction id, time), is only created after the job, and keeping it apart avoids NULL-filled columns in bookings for unpaid or cancelled jobs. 

#### **5.5 Summary** 

|**Relation**|**1NF**|**2NF**|**3NF**|**BCNF**|
|---|---|---|---|---|
|users|`✓`|`✓`|`✓`|`✓`|
|stations|`✓`|`✓`|`✓`|`✓`|
|platforms|`✓`|`✓`|`✓`|`✓`|



Page 9 

RailPort – Railway Station Porter Booking System 

23AIE303 DBMS · Project Report 

|**Relation**|**1NF**|**2NF**|**3NF**|**BCNF**|
|---|---|---|---|---|
|porters|`✓`|`✓`|`✓`|`✓`|
|bookings|`✓`|`✓`|`✓*`|`✓*`|
|payments|`✓`|`✓`|`✓`|`✓`|
|ratings|`✓`|`✓`|`✓`|`✓`|



* bookings: holds except for the optional pickup_platform_id → station_id dependency (Note 1). No decomposition was needed: the design obtained by mapping the ER diagram is already in BCNF everywhere else. 

Page 10 

RailPort – Railway Station Porter Booking System 

23AIE303 DBMS · Project Report 

### **6. Advanced SQL Implementation and Query Design** 

All scripts are PostgreSQL 16 and are kept in the _database/_ folder: **01_schema.sql** (DDL and indexes – the project's own schema), **02_seed.sql** (the project's own sample data), **03_demo_data.sql** (extra rows so that analytical queries return meaningful results), **04_enhancements.sql** (optional triggers and view) and **05_queries.sql** (the queries below). Every output shown was produced by running these scripts, in this order, on PostgreSQL 16. 

#### **6.1 Constraints used** 

|**Constraint type**|**Examples in RailPort**|
|---|---|
|PRIMARY KEY|SERIAL surrogate key in every table|
|FOREIGN KEY … ON DELETE CASCADE|platforms→stations, porters→users, payments→bookings, ratings→bookings|
|FOREIGN KEY (default NO ACTION)|porters→stations/platforms; bookings→users/porters/stations/platforms;<br>ratings→users/porters – referenced rows cannot be deleted while children exist|
|UNIQUE|users.phone / email, stations.station_code, porters.user_id / badge_number, (station_id,<br>platform_number), payments.booking_id, ratings.booking_id|
|CHECK|role, availability_status, status, payment_method, payment_status, stars BETWEEN 1 AND 5,<br>rating 0..5, fare ≥ 0, amount ≥ 0, luggage_count > 0, number_of_platforms > 0,<br>platform_number > 0|
|NOT NULL / DEFAULT|status DEFAULT 'REQUESTED', availability DEFAULT 'OFFLINE', created_at DEFAULT<br>CURRENT_TIMESTAMP, luggage_count DEFAULT 1, fare DEFAULT 0.00|



#### **6.2 Indexes** 

|**Index (from schema.sql)**|**Purpose**|
|---|---|
|idx_bookings_passenger (passenger_id)|"My bookings" screen of a passenger|
|idx_bookings_porter (porter_id)|Jobs assigned to a porter|
|idx_bookings_station (station_id)|Open requests of a station|
|idx_porters_station (station_id)|Porters of a station|
|idx_platforms_station (station_id)|Platforms of a station|



#### **6.3 Optional enhancements: triggers and a view (04_enhancements.sql)** 

The original schema contains no triggers; the rules below are currently applied by the Node.js controllers. They are provided as optional, tested enhancements that move the rules into the database. 

|**Object**|**What it does**|
|---|---|
|trg_booking_completed (BEFORE INSERT/UPDATE on<br>bookings)|T1 – stamps completed_at = NOW() when status becomes COMPLETED.|
|trg_rating_validate (BEFORE INSERT/UPDATE on ratings)|T2 – only COMPLETED bookings can be rated; passenger and porter must<br>match the booking.|
|trg_rating_refresh (AFTER INSERT/UPDATE/DELETE on<br>ratings)|T3 – recomputes porters.rating as the average of stars.|
|v_booking_details (view)|Six-table join giving dashboards a ready-made, readable booking row.|



#### **6.4 Security in the application layer** 

- All queries in the Node.js controllers use **parameterized statements** ($1…$n) – user input is never concatenated into SQL. 

- A single **pg Pool** (config/db.js) is shared by all requests; credentials come from environment variables (.env). 

- Passwords are stored as **bcrypt hashes** ; the UPDATE statements use COALESCE so partial updates never overwrite other columns with NULL. 

Page 11 

RailPort – Railway Station Porter Booking System 

23AIE303 DBMS · Project Report 

#### **6.5 Queries and their results** 

The fourteen queries below cover multi-table joins, GROUP BY with HAVING, scalar / IN / correlated (NOT EXISTS) subqueries, CTEs, window functions, CASE, transactions with row-level locking, triggers and constraint behaviour. 

###### **Q1 – Six-table JOIN: complete booking details** 

Joins bookings with users, stations, platforms, porters and porter's user record (LEFT JOINs keep unassigned bookings). Same query the REST API uses. 

```
SELECT b.booking_id, up.name AS passenger, s.station_code AS stn, pl.platform_number AS plat,
       COALESCE(upr.name,'-- unassigned --') AS porter, b.status, b.fare
FROM bookings b
JOIN users up        ON b.passenger_id = up.user_id
JOIN stations s      ON b.station_id   = s.station_id
LEFT JOIN platforms pl ON b.pickup_platform_id = pl.platform_id
LEFT JOIN porters p  ON b.porter_id = p.porter_id
LEFT JOIN users upr  ON p.user_id   = upr.user_id
ORDER BY b.booking_id
LIMIT 10;
```

|**booking_id**|**passenger**|**stn**|**plat**|**porter**|**status**|**fare**|
|---|---|---|---|---|---|---|
|1|Rahul Kumar|MAS|1|Arun Kumar|COMPLETED|150.00|
|2|Priya Sharma|MAS|2|Selvam R|IN_PROGRESS|120.00|
|3|Rahul Kumar|MAS|3|Arun Kumar|COMPLETED|130.00|
|4|Ananya Iyer|MAS|1|Arun Kumar|COMPLETED|220.00|
|5|Vikram Singh|MAS|2|Selvam R|COMPLETED|150.00|
|6|Meena Nair|MS|1|Murugan K|COMPLETED|100.00|
|7|Karthik S|MS|2|Murugan K|COMPLETED|250.00|
|8|Priya Sharma|CBE|1|Ganesh P|COMPLETED|160.00|
|9|Ananya Iyer|CBE|2|Ganesh P|COMPLETED|110.00|
|10|Vikram Singh|CBE|1|Ravi T|COMPLETED|80.00|



###### **Q2 – GROUP BY + HAVING: revenue per station** 

Total fare, job count and average fare of COMPLETED bookings per station; HAVING keeps stations with more than one job. 

```
SELECT s.station_name, COUNT(*) AS completed_jobs, SUM(b.fare) AS revenue, ROUND(AVG(b.fare),2) AS avg_fare
FROM bookings b JOIN stations s ON s.station_id = b.station_id
WHERE b.status = 'COMPLETED'
GROUP BY s.station_name
HAVING COUNT(*) > 1
ORDER BY revenue DESC;
```

|**station_name**|**completed_jobs**|**revenue**|**avg_fare**|
|---|---|---|---|
|Chennai Central|5|850.00|170.00|
|Chennai Egmore|2|350.00|175.00|
|Coimbatore Junction|3|350.00|116.67|



Page 12 

RailPort – Railway Station Porter Booking System 

23AIE303 DBMS · Project Report 

###### **Q3 – Scalar subquery: above-average porters** 

Lists porters whose rating is higher than the average rating of all porters. 

```
SELECT u.name, p.badge_number, p.rating
FROM porters p JOIN users u ON u.user_id = p.user_id
WHERE p.rating > (SELECT AVG(rating) FROM porters WHERE rating > 0)
ORDER BY p.rating DESC, u.name;
```

|**name**|**badge_number**|**rating**|
|---|---|---|
|Arun Kumar|MAS-P001|4.50|
|Ganesh P|CBE-P001|4.50|
|Murugan K|MS-P001|4.50|
|Selvam R|MAS-P002|4.20|



###### **Q4 – NOT EXISTS (correlated subquery): unpaid completed jobs** 

Completed bookings that have no SUCCESS payment, i.e. revenue still to collect. 

```
SELECT b.booking_id, b.train_number, b.destination, b.fare, COALESCE(pay.payment_status,'NO PAYMENT') AS payment
FROM bookings b
LEFT JOIN payments pay ON pay.booking_id = b.booking_id
WHERE b.status = 'COMPLETED'
```

```
  AND NOT EXISTS (SELECT 1 FROM payments x WHERE x.booking_id = b.booking_id AND x.payment_status = 'SUCCESS');
```

|**booking_id**|**train_number**|**destination**|**fare**|**payment**|
|---|---|---|---|---|
|10|12680|Main Exit|80.00|FAILED|
|16|12295|Main Exit|200.00|PENDING|



###### **Q5 – Window function RANK(): porter leaderboard per station** 

Ranks porters inside their own station by earnings from completed jobs (PARTITION BY station). 

```
SELECT station_code, porter, jobs, earnings,
       RANK() OVER (PARTITION BY station_code ORDER BY earnings DESC) AS rank_in_station
FROM (
```

```
    SELECT s.station_code, u.name AS porter, COUNT(b.booking_id) AS jobs, COALESCE(SUM(b.fare),0) AS earnings
    FROM porters p
    JOIN users u    ON u.user_id = p.user_id
    JOIN stations s ON s.station_id = p.station_id
    LEFT JOIN bookings b ON b.porter_id = p.porter_id AND b.status = 'COMPLETED'
    GROUP BY s.station_code, u.name
) t
ORDER BY station_code, rank_in_station;
```

|**station_code**|**porter**|**jobs**|**earnings**|**rank_in_station**|
|---|---|---|---|---|
|CBE|Ganesh P|2|270.00|1|
|CBE|Ravi T|1|80.00|2|
|MAS|Arun Kumar|4|700.00|1|
|MAS|Selvam R|1|150.00|2|
|MS|Murugan K|2|350.00|1|



Page 13 

RailPort – Railway Station Porter Booking System 

23AIE303 DBMS · Project Report 

###### **Q6 – CTE (WITH): passenger spending and loyalty tier** 

A common table expression totals each passenger's successful payments, then CASE classifies them. 

```
WITH spend AS (
    SELECT u.user_id, u.name, COUNT(DISTINCT b.booking_id) AS trips, COALESCE(SUM(pay.amount),0) AS total_paid
    FROM users u
    JOIN bookings b      ON b.passenger_id = u.user_id
    LEFT JOIN payments pay ON pay.booking_id = b.booking_id AND pay.payment_status = 'SUCCESS'
    WHERE u.role = 'PASSENGER'
    GROUP BY u.user_id, u.name
)
SELECT name, trips, total_paid,
```

```
       CASE WHEN total_paid >= 400 THEN 'GOLD' WHEN total_paid >= 200 THEN 'SILVER' ELSE 'BRONZE' END AS tier
FROM spend ORDER BY total_paid DESC;
```

|**name**|**trips**|**total_paid**|**tier**|
|---|---|---|---|
|Ananya Iyer|3|330.00|SILVER|
|Rahul Kumar|3|280.00|SILVER|
|Karthik S|2|250.00|SILVER|
|Priya Sharma|3|160.00|BRONZE|
|Vikram Singh|3|150.00|BRONZE|
|Meena Nair|2|100.00|BRONZE|



###### **Q7 – Payment-method share with window SUM** 

Percentage share of each payment method among SUCCESS payments, using SUM() OVER (). 

```
SELECT payment_method, COUNT(*) AS txns, SUM(amount) AS amount,
       ROUND(100.0 * SUM(amount) / SUM(SUM(amount)) OVER (), 1) AS pct_of_revenue
FROM payments WHERE payment_status = 'SUCCESS'
GROUP BY payment_method ORDER BY amount DESC;
```

|**payment_method**|**txns**|**amount**|**pct_of_revenue**|
|---|---|---|---|
|UPI|4|540.00|42.5|
|CARD|2|470.00|37.0|
|CASH|2|260.00|20.5|



###### **Q8 – Open requests vs idle porters per station** 

Joins two aggregated derived tables to show demand (REQUESTED, unassigned) against supply (AVAILABLE porters) at each station. 

```
SELECT s.station_code, COALESCE(d.open_requests,0) AS open_requests, COALESCE(a.available_porters,0) AS
      available_porters
FROM stations s
LEFT JOIN (SELECT station_id, COUNT(*) AS open_requests FROM bookings
           WHERE status='REQUESTED' AND porter_id IS NULL GROUP BY station_id) d ON d.station_id = s.station_id
LEFT JOIN (SELECT station_id, COUNT(*) AS available_porters FROM porters
           WHERE availability_status='AVAILABLE' GROUP BY station_id) a ON a.station_id = s.station_id
ORDER BY s.station_id;
```

|**station_code**|**open_requests**|**available_porters**|
|---|---|---|
|MAS|1|1|
|MS|1|1|
|CBE|1|1|



Page 14 

RailPort – Railway Station Porter Booking System 

23AIE303 DBMS · Project Report 

###### **Q9 – IN-subquery: passengers who rated 5 stars** 

Passengers who have given at least one 5-star rating. 

```
SELECT user_id, name FROM users
WHERE user_id IN (SELECT passenger_id FROM ratings WHERE stars = 5)
ORDER BY name;
```

|**user_id**|**name**|
|---|---|
|6|Ananya Iyer|
|8|Meena Nair|
|2|Priya Sharma|
|1|Rahul Kumar|



###### **Q10 – Busiest platforms (multi-join + aggregate + LIMIT)** 

Platforms ranked by number of pickups together with total luggage carried. 

```
SELECT s.station_code, pl.platform_number, COUNT(*) AS pickups, SUM(b.luggage_count) AS bags
FROM bookings b
JOIN platforms pl ON pl.platform_id = b.pickup_platform_id
JOIN stations  s  ON s.station_id   = pl.station_id
GROUP BY s.station_code, pl.platform_number
ORDER BY pickups DESC, bags DESC LIMIT 5;
```

|**station_code**|**platform_number**|**pickups**|**bags**|
|---|---|---|---|
|MAS|1|3|10|
|MAS|2|3|9|
|MS|1|2|4|
|CBE|1|2|4|
|MS|2|1|5|



###### **Q11 – Trigger demo (needs 04_enhancements.sql): rating auto-updates porter average** 

Changing a rating fires trg_rating_refresh, so the porter's cached rating is recomputed. (Run inside a transaction and rolled back.) 

```
BEGIN;
SELECT porter_id, rating AS rating_before FROM porters WHERE porter_id = 3;
UPDATE ratings SET stars = 3 WHERE booking_id = 7;
SELECT porter_id, rating AS rating_after FROM porters WHERE porter_id = 3;
ROLLBACK;
porter_id,rating_before
3,4.50
porter_id,rating_after
3,4.00
```

###### **Q12 – Transaction with row lock: porter accepts a request** 

SELECT ... FOR UPDATE SKIP LOCKED guarantees two porters can never accept the same booking. (Rolled back for the demo.) 

```
BEGIN;
WITH picked AS (
    SELECT booking_id FROM bookings
    WHERE status = 'REQUESTED' AND porter_id IS NULL AND station_id = 1
    ORDER BY created_at LIMIT 1 FOR UPDATE SKIP LOCKED
)
UPDATE bookings b SET porter_id = 1, status = 'ACCEPTED'
FROM picked WHERE b.booking_id = picked.booking_id
RETURNING b.booking_id, b.porter_id, b.status;
ROLLBACK;
```

```
booking_id,porter_id,status
13,1,ACCEPTED
```

Page 15 

RailPort – Railway Station Porter Booking System 

23AIE303 DBMS · Project Report 

###### **Q13 – Constraint demo: invalid data is rejected** 

A rating of 9 stars violates CHECK (stars BETWEEN 1 AND 5). 

```
INSERT INTO ratings (booking_id, passenger_id, porter_id, stars) VALUES (1,1,1,9);
ERROR: new row for relation "ratings" violates check constraint "ratings_stars_check"
→ INSERT rejected, no row stored.
```

###### **Q14 – Cascade demo: ON DELETE CASCADE** 

Deleting a booking removes its payment and rating rows (rolled back). 

```
BEGIN;
SELECT (SELECT COUNT(*) FROM payments WHERE booking_id=1) AS payments_before, (SELECT COUNT(*) FROM ratings
      WHERE booking_id=1) AS ratings_before;
DELETE FROM bookings WHERE booking_id = 1;
SELECT (SELECT COUNT(*) FROM payments WHERE booking_id=1) AS payments_after, (SELECT COUNT(*) FROM ratings WHERE
      booking_id=1) AS ratings_after;
ROLLBACK;
payments_before,ratings_before
1,1
payments_after,ratings_after
0,0
```

#### **6.6 Analytical results** 

The same data aggregated for the dashboard: revenue per station (Q2), payment-method mix (Q7), porter ratings (trigger-maintained) and the booking status distribution. 



<!-- Start of picture text -->
Revenue from completed bookings by station<br>seesPo “=<br>Sooeseeinssil =<br>Seton| =<br><!-- End of picture text -->



<!-- Start of picture text -->
‘Average rating per porter<br><!-- End of picture text -->



<!-- Start of picture text -->
‘Successful payments by method<br>“Po me<br>EE =<br>“hz aa<br><!-- End of picture text -->



<!-- Start of picture text -->
Bookings by status<br><!-- End of picture text -->

_Figure 6.1 – Result charts generated from the sample database_ 

Page 16 

RailPort – Railway Station Porter Booking System 

23AIE303 DBMS · Project Report 

### **7. ER Diagram and UML Diagram** 

#### **7.1 Entity–Relationship diagram** 



<!-- Start of picture text -->
PORTERS<br>USERS ‘porter id STATIONS<br>P userid ‘Station id<br>a 4 0.1 —J badge_number station_code<br>‘current_platform_id number_of_platforms<br>rating N.<br>1. 1 O41 <><br>:<br>bookingid<br>pickup_platform_id PLATFORMS<br>fare Reon<br>completed_at 1,<br>0.1<br>N <_ paymentPAYMENTS id<br>N booking_idamount<br>tating idRATINGS ‘transaction_idpayment.time<br>Underlined bold = primary key Italic = foreign key Diamonds = relationships (with cardinality)<br><!-- End of picture text -->

_Figure 7.1 – ER diagram: entities with attributes (bold underlined = primary key, italic = foreign key) and relationships with cardinality_ 

Page 17 

RailPort – Railway Station Porter Booking System 

23AIE303 DBMS · Project Report 

#### **7.2 UML class diagram** 



<!-- Start of picture text -->
requestPorter() manageUsers() availability: Status stationid int<br>cancelBooking() manageStations() rating: float posts| SHtionName: String<br>makePayment() viewReports() stationCode: String<br>7 27 2eceptBookingl) niumberOfPlatforms: int<br>ratePortert) setAvailability() city: String<br>T a - addPlatform()<br>1 1, “ ae<br>userid: int VX handles<br>name: String<br>phone: String<br>password: String . Le<br>osele: Rol 0. Booking |<br>updateProfile() trainNumber: String platformid: int<br>coachNumber: String platformNumber: int<br>luggageCount: int lpickup from]<br>fare: decimal<br>status: BookingStatus<br>createdAt: Date<br>completedAt: Date<br>assign()<br>updateStatus()<br>0.4<br>method: CASH|UPI|CARD_ stars: int (1-5)<br>status: PaymentStatus review: String<br>transactionld: String createdAt: Date<br>paymentTime: Date ‘submit()<br>Dashed line with hollow triangle = generalisation (Passenger, Porter and Admin are roles of User)<br><!-- End of picture text -->

_Figure 7.2 – UML class diagram: Passenger, Porter and Admin specialise User; associations carry multiplicities_ 

Page 18 

RailPort – Railway Station Porter Booking System 

23AIE303 DBMS · Project Report 

#### **7.3 UML use case diagram** 



<!-- Start of picture text -->
RailPort System<br>Request porter<br>(train, coach, luggage)<br>Track booking status<br>CT Pay (Cash / UPI / Card)<br>=<br>So<br>tAE><br>[en C7] EE at my station<br>Io Accept booking<br>(in progressUpdate/ completed)status<br>‘Manage stations<br>& platforms<br>oo<br>Tonto booengs<br>opened<br><!-- End of picture text -->

_Figure 7.3 – Use case diagram for the three actors_ 

Page 19 

RailPort – Railway Station Porter Booking System 

23AIE303 DBMS · Project Report 

### **8. Relational Schema** 

Primary keys are underlined, foreign keys are marked with _FK →_ . Nullable foreign keys are marked (null). 

|**Relation**|**Schema**|
|---|---|
|**users**|user_id,name, phone, email, password, role, created_at|
|**stations**|station_id,station_name, station_code, city, number_of_platforms|
|**platforms**|platform_id,station_id_FK → stations_, platform_number|
|**porters**|porter_id,user_id_FK → users_, station_id_FK → stations_, badge_number, availability_status, current_platform_id_FK_<br>_→ platforms (null)_, rating|
|**bookings**|booking_id,passenger_id_FK → users_, porter_id_FK → porters (null)_, station_id_FK → stations_, pickup_platform_id_FK_<br>_→ platforms (null)_, train_number, coach_number, luggage_count, destination, fare, status, created_at,<br>completed_at|
|**payments**|payment_id,booking_id_FK → bookings_(UNIQUE), amount, payment_method, payment_status, transaction_id,<br>payment_time|
|**ratings**|rating_id,booking_id_FK → bookings_(UNIQUE), passenger_id_FK → users_, porter_id_FK → porters_, stars, review,<br>created_at|



Additional constraints: UNIQUE(users.phone), UNIQUE(users.email), UNIQUE(stations.station_code), UNIQUE(platforms.station_id, platform_number), UNIQUE(porters.user_id), UNIQUE(porters.badge_number), UNIQUE(payments.booking_id), UNIQUE(ratings.booking_id). Because payments.booking_id and ratings.booking_id are UNIQUE, both relationships are one-to-(zero or one). 

Page 20 

RailPort – Railway Station Porter Booking System 

23AIE303 DBMS · Project Report 



<!-- Start of picture text -->
users)<br>nameuserid ae re @seristation_id wtINT FKUOrE stationid tational‘SERIAL PK.<br>phone waseeasiioo) badge_number VARCHAR(SO) UQ station_name VARCHAR(100)<br>email wactentnnVaRHARLIS) weUG current.platform_idavailability_status — VARCHAR(20)'NT CKrK citystation_code \VARCHAR(100)VARCHAR(10) UQ<br>created_at TIMESTAMP / \<br>/ \<br>/ \<br>/ \<br>/pickup.plotform_id INT FK P----\-- == ‘station_id int *«<br>SEE<br><!-- End of picture text -->

_Figure 8.1 – Relational schema diagram (arrows run from foreign key to referenced primary key)_ 

Page 21 

RailPort – Railway Station Porter Booking System 

23AIE303 DBMS · Project Report 

### **9. Database Tables** 

For every table: structure (attributes, data types, keys and constraints) and sample records. Samples below are the project's own seed data (02_seed.sql): users 5, stations 3, platforms 11, porters 2, bookings 2, payments 1, ratings 1 rows. 03_demo_data.sql adds more rows for the queries of Section 6. 

#### **9.1 Table: users** 

Every person who logs in: passengers who book, porters who serve and admins who manage. One row per account; the role column decides which dashboard the person sees. 

|**Attribute**|**Data type**|**Key / constraints**|**Description**|
|---|---|---|---|
|**user_id**|SERIAL|PK|Surrogate primary key, auto-generated.|
|name|VARCHAR(100)|NOT NULL|Full name of the person.|
|phone|VARCHAR(15)|UNIQUE, NOT NULL|Mobile number; unique, used for contact.|
|email|VARCHAR(100)|UNIQUE, NOT NULL|Login e-mail; unique (may be NULL).|
|password|VARCHAR(255)|NOT NULL|bcrypt hash of the password, VARCHAR(255) (never plain text).|
|role|VARCHAR(20)|CHECK (PASSENGER|PORTER|ADMIN)|PASSENGER, PORTER or ADMIN (CHECK).|
|created_at|TIMESTAMP|DEFAULT CURRENT_TIMESTAMP|Account creation time (default NOW()).|



**Sample records** (5 rows in the seed data) 

|**user_id**|**name**|**phone**|**email**|**password**|**role**|**created_at**|
|---|---|---|---|---|---|---|
|1|Rahul Kumar|9876543210|rahul@example.com|$2a$10$HX…|PASSENGER|2026-10-16 11:56|
|2|Priya Sharma|9876543213|priya@example.com|$2a$10$HX…|PASSENGER|2026-10-16 11:56|
|3|Arun Kumar|9876543211|arun@example.com|$2a$10$HX…|PORTER|2026-10-16 11:56|
|4|Selvam R|9876543214|selvam@example.com|$2a$10$HX…|PORTER|2026-10-16 11:56|
|5|Admin User|9876543212|admin@example.com|$2a$10$HX…|ADMIN|2026-10-16 11:56|



Page 22 

RailPort – Railway Station Porter Booking System 

23AIE303 DBMS · Project Report 

#### **9.2 Table: stations** 

Railway stations where the service operates (name, short code, city, platform count). 

|**Attribute**|**Data type**|**Key / constraints**|**Description**|
|---|---|---|---|
|**station_id**|SERIAL|PK|Primary key.|
|station_name|VARCHAR(100)|NOT NULL|Official name, e.g. Coimbatore Junction.|
|station_code|VARCHAR(10)|UNIQUE, NOT NULL|Railway code (CBE, MAS...); unique.|
|city|VARCHAR(100)|NOT NULL|City of the station.|
|number_of_platforms|INT|CHECK > 0|Total platforms of the station; must be > 0.|



**Sample records** (3 rows in the seed data) 

|**station_id**|**station_name**|**station_code**|**city**|**number_of_platforms**|
|---|---|---|---|---|
|1|Chennai Central|MAS|Chennai|12|
|2|Chennai Egmore|MS|Chennai|10|
|3|Coimbatore Junction|CBE|Coimbatore|6|



Page 23 

RailPort – Railway Station Porter Booking System 

23AIE303 DBMS · Project Report 

#### **9.3 Table: platforms** 

Individual platforms of a station. A platform cannot exist without its station. 

|**Attribute**|**Data type**|**Key / constraints**|**Description**|
|---|---|---|---|
|**platform_id**|SERIAL|PK|Primary key.|
|station_id|INT|FK → stations, NOT NULL, CASCADE|FK to stations; deleting a station deletes its platforms (CASCADE).|
|platform_number|INT|NOT NULL, CHECK > 0; UNIQUE with station_id|Platform number printed at the station; unique within a station.|



**Sample records** (11 rows in the seed data) 

|**platform_id**|**station_id**|**platform_number**|
|---|---|---|
|1|1|1|
|2|1|2|
|3|1|3|
|4|1|4|
|5|1|5|
|6|2|1|
|7|2|2|
|8|2|3|



Page 24 

RailPort – Railway Station Porter Booking System 

23AIE303 DBMS · Project Report 

#### **9.4 Table: porters** 

Extra details for users whose role is PORTER: the station they are posted at, badge number, live availability, the platform they are standing on and their average rating. 

|**Attribute**|**Data type**|**Key / constraints**|**Description**|
|---|---|---|---|
|**porter_id**|SERIAL|PK|Primary key.|
|user_id|INT|FK → users, UNIQUE, CASCADE|FK to users; one-to-one (UNIQUE) with the porter's account.|
|station_id|INT|FK → stations, NOT NULL|FK to stations: where the porter is posted (no cascade).|
|badge_number|VARCHAR(50)|UNIQUE, NOT NULL|Official badge; unique.|
|availability_status|VARCHAR(20)|NOT NULL, CHECK, DEFAULT 'OFFLINE'|AVAILABLE, BUSY or OFFLINE; default OFFLINE.|
|current_platform_id|INT|FK → platforms, NULL|FK to platforms (optional): platform the porter is on now.|
|rating|NUMERIC(3,2)|CHECK 0..5, DEFAULT 0.00|Average stars (0-5, default 0.00); updated through the API (PUT /api/porters/:id) or optionally by<br>trigger T3.|



###### **Sample records** (2 rows in the seed data) 

|**porter_id**<br>**user_id**<br>**station_id**|**badge_number**|**availability_status**<br>**current_platform_id**|**rating**|
|---|---|---|---|
|1<br>3<br>1|MAS-P001|AVAILABLE<br>1|4.5|
|2<br>4<br>1|MAS-P002|BUSY<br>2|4.2|



Page 25 

RailPort – Railway Station Porter Booking System 

23AIE303 DBMS · Project Report 

#### **9.5 Table: bookings** 

The core transaction: a passenger asks for help with luggage on a given train/coach; a porter is assigned later and the booking moves through its status life-cycle. 

|**Attribute**|**Data type**|**Key / constraints**|**Description**|
|---|---|---|---|
|**booking_id**|SERIAL|PK|Primary key.|
|passenger_id|INT|FK → users, NOT NULL|FK to users: who asked for help.|
|porter_id|INT|FK → porters, NULL until accepted|FK to porters; NULL until a porter accepts.|
|station_id|INT|FK → stations, NOT NULL|FK to stations: where the service happens.|
|pickup_platform_id|INT|FK → platforms, NULL|FK to platforms (optional).|
|train_number|VARCHAR(20)|NULL allowed|Train the passenger is travelling on.|
|coach_number|VARCHAR(20)|NULL allowed|Coach where the luggage is.|
|luggage_count|INT|NOT NULL, CHECK > 0, DEFAULT 1|Number of bags; must be > 0.|
|destination|VARCHAR(255)|NULL allowed|Where the luggage must be taken / journey destination.|
|fare|NUMERIC(10,2)|NOT NULL, CHECK ≥ 0, DEFAULT 0.00|Agreed fare in rupees; >= 0.|
|status|VARCHAR(20)|NOT NULL, CHECK (5 values), DEFAULT<br>'REQUESTED'|REQUESTED → ACCEPTED → IN_PROGRESS → COMPLETED, or CANCELLED.|
|created_at|TIMESTAMP|DEFAULT CURRENT_TIMESTAMP|Time the request was made.|
|completed_at|TIMESTAMP|NULL|Time the job finished (NULL until COMPLETED).|



###### **Sample records** (2 rows in the seed data) 

|**booking_id**<br>**passenger_id**|**porter_id**<br>**station_id**<br>**pickup_platform_id**|**train_number**|**coach_number**|**luggage_count**|**destination**|**fare**|**status**|**created_at**|**completed_at**|
|---|---|---|---|---|---|---|---|---|---|
|1<br>1|1<br>1<br>1|12621|S5|3|Main Exit|150.0|COMPLETED|2026-10-16 11:56|NULL|
|2<br>2|2<br>1<br>2|12658|A1|2|Parking Lot|120.0|IN_PROGRESS|2026-10-16 11:56|NULL|



Page 26 

RailPort – Railway Station Porter Booking System 

23AIE303 DBMS · Project Report 

#### **9.6 Table: payments** 

The payment made for a booking (cash, UPI or card) with its status, transaction reference and time. At most one payment record per booking. 

|**Attribute**|**Data type**|**Key / constraints**|**Description**|
|---|---|---|---|
|**payment_id**|SERIAL|PK|Primary key.|
|booking_id|INT|FK → bookings, UNIQUE, NOT NULL, CASCADE|FK to bookings, UNIQUE: one payment record per booking; CASCADE on delete.|
|amount|NUMERIC(10,2)|NOT NULL, CHECK ≥ 0|Amount paid; >= 0.|
|payment_method|VARCHAR(20)|NOT NULL, CHECK (CASH|UPI|CARD)|CASH, UPI or CARD.|
|payment_status|VARCHAR(20)|NOT NULL, CHECK, DEFAULT 'PENDING'|PENDING, SUCCESS or FAILED; default PENDING.|
|transaction_id|VARCHAR(100)|NULL for cash|Gateway reference (NULL for cash).|
|payment_time|TIMESTAMP|DEFAULT CURRENT_TIMESTAMP|Time of the payment record (default CURRENT_TIMESTAMP).|



###### **Sample records** (1 rows in the seed data) 

|**payment_id**<br>**booking_id**<br>**amount**|**payment_method**|**payment_status**|**transaction_id**|**payment_time**|
|---|---|---|---|---|
|1<br>1<br>150.0|UPI|SUCCESS|TXN1001|2026-10-16 11:56|



Page 27 

RailPort – Railway Station Porter Booking System 

23AIE303 DBMS · Project Report 

#### **9.7 Table: ratings** 

Passenger feedback (1-5 stars and a review) about the porter who served a completed booking. 

|**Attribute**|**Data type**|**Key / constraints**|**Description**|
|---|---|---|---|
|**rating_id**|SERIAL|PK|Primary key.|
|booking_id|INT|FK → bookings, UNIQUE, NOT NULL, CASCADE|FK to bookings; UNIQUE = at most one rating per booking.|
|passenger_id|INT|FK → users|FK to users who gave the rating.|
|porter_id|INT|FK → porters|FK to porters being rated.|
|stars|INT|NOT NULL, CHECK 1..5|Integer 1 to 5.|
|review|VARCHAR(500)|NULL|Optional comment, up to 500 characters.|
|created_at|TIMESTAMP|DEFAULT CURRENT_TIMESTAMP|Time the rating was given.|



###### **Sample records** (1 rows in the seed data) 

|**rating_id**<br>**booking_id**<br>**passenger_id**<br>**porter_id**|**stars**<br>**review**|**created_at**|
|---|---|---|
|1<br>1<br>1<br>1|5<br>Very helpful and quick s…|2026-10-16 11:56|



Page 28 

RailPort – Railway Station Porter Booking System 

23AIE303 DBMS · Project Report 

### **10. Conclusion and Future Scope** 

#### **10.1 Conclusion** 

RailPort turns the informal process of hiring a railway porter into a structured, auditable service. A seven-table relational schema captures users, stations, platforms, porters, bookings, payments and ratings. Every relation is in BCNF, with one documented relaxation (the optional pickup platform → station dependency in bookings); integrity is enforced inside the database through primary and foreign keys, UNIQUE and CHECK constraints and cascading rules, and optional triggers can move further business rules into the database. Advanced SQL – multi-table joins, subqueries, CTEs, window functions, aggregates and row-locking transactions – answers the operational and management questions of the system, and fifteen test cases confirmed that invalid data is rejected and that cascades and triggers behave as designed. The Node.js REST API uses a connection pool and parameterized queries, and a React front end offers separate passenger, porter and admin dashboards. 

#### **10.2 Future scope** 

- Real-time porter matching and notifications (WebSockets / push) and live location of porters on a platform map. 

- PNR / train-schedule integration so train, coach and platform are filled in automatically. 

- Online payment-gateway integration with refunds, plus dynamic fare calculation based on distance and luggage. 

- Composite foreign keys (pickup_platform_id, station_id) → platforms to make the platform/station dependency impossible to break, and a stored procedure for "accept booking" using row-level locks. 

- Table partitioning and materialised views for reporting as the number of stations grows; role-based access control with JWT. 

- Mobile application, multi-language support and accessibility features for senior citizens and differently-abled passengers. 

- Machine-learning demand forecasting from booking history to position porters before peak trains. 

Page 29 

RailPort – Railway Station Porter Booking System 

23AIE303 DBMS · Project Report 

### **Appendix A – Test cases and validation** 

Each negative test was executed inside a transaction that was rolled back, so the sample data is unchanged. 

|**ID**|**Test**|**Statement**|**Expected**|**Actual database response**|
|---|---|---|---|---|
|TC1|UNIQUE: duplicate e-mail|`INSERT INTO users(name,phone,email,`<br>`password,role) VALUES('X','99999999`<br>`90','rahul@example.com','h','PASSEN`<br>`GER');`|Rejected (unique<br>violation)|duplicate key value violates<br>unique constraint<br>"users_email_key"|
|TC2|CHECK: invalid user role|`INSERT INTO users(name,phone,email,`<br>`password,role) VALUES('X','99999999`<br>`91','x@example.com','h','GUEST');`|Rejected (check<br>violation)|new row for relation "users"<br>violates check constraint<br>"users_role_check"|
|TC3|FOREIGN KEY: unknown<br>passenger|`INSERT INTO bookings(passenger_id,s`<br>`tation_id,train_number,coach_number`<br>`,destination) VALUES(999,1,'12680',`<br>`'S1','Chennai');`|Rejected (FK violation)|insert or update on table<br>"bookings" violates foreign key<br>constraint<br>"bookings_passenger_id_fkey"|
|TC4|UNIQUE: second<br>payment for the same<br>booking|`INSERT INTO payments(booking_id,amo`<br>`unt,payment_method)`<br>`VALUES(1,150,'CASH');`|Rejected<br>(payments.booking_id<br>is UNIQUE)|duplicate key value violates<br>unique constraint<br>"payments_booking_id_key"|
|TC5|CHECK: negative fare|`INSERT INTO bookings(passenger_id,s`<br>`tation_id,train_number,coach_number`<br>`,destination,fare) VALUES(1,1,'1268`<br>`0','S1','Chennai',-10);`|Rejected (check<br>violation)|new row for relation<br>"bookings" violates check<br>constraint<br>"bookings_fare_check"|
|TC6|CHECK: zero luggage|`INSERT INTO bookings(passenger_id,s`<br>`tation_id,train_number,coach_number`<br>`,destination,luggage_count) VALUES(`<br>`1,1,'12680','S1','Chennai',0);`|Rejected (check<br>violation)|new row for relation<br>"bookings" violates check<br>constraint "bookings_luggage_<br>count_check"|
|TC7|CHECK: invalid payment<br>method|`INSERT INTO payments(booking_id,amo`<br>`unt,payment_method)`<br>`VALUES(2,10,'COINS');`|Rejected (check<br>violation)|new row for relation<br>"payments" violates check<br>constraint "payments_paymen<br>t_method_check"|
|TC8|UNIQUE: duplicate<br>platform number in a<br>station|`INSERT INTO platforms(station_id,pl`<br>`atform_number) VALUES(1,1);`|Rejected (unique<br>(station_id,<br>platform_number))|duplicate key value violates<br>unique constraint "platforms_s<br>tation_id_platform_number_k<br>ey"|
|TC9|CHECK: rating outside 1-5|`INSERT INTO ratings(booking_id,pass`<br>`enger_id,porter_id,stars)`<br>`VALUES(1,1,1,9);`|Rejected (check<br>violation)|new row for relation "ratings"<br>violates check constraint<br>"ratings_stars_check"|
|TC10|FOREIGN KEY (no<br>cascade): delete a<br>passenger who has<br>bookings|`DELETE FROM users WHERE user_id=1;`|Rejected - history is<br>protected|update or delete on table<br>"users" violates foreign key<br>constraint<br>"bookings_passenger_id_fkey"<br>on table "bookings"|
|TC11|FOREIGN KEY (no<br>cascade): delete a porter<br>who has bookings|`DELETE FROM porters WHERE`<br>`porter_id=1;`|Rejected - history is<br>protected|update or delete on table<br>"porters" violates foreign key<br>constraint<br>"bookings_porter_id_fkey" on<br>table "bookings"|
|TC12|FOREIGN KEY: delete a<br>station that still has<br>porters|`DELETE FROM stations WHERE`<br>`station_id=1;`|Rejected - porters and<br>bookings still refer to it|update or delete on table<br>"stations" violates foreign key<br>constraint<br>"porters_station_id_fkey" on<br>table "porters"|
|TC13|TRIGGER T2<br>(enhancement): rate a<br>booking that is not<br>completed|`INSERT INTO ratings(booking_id,pass`<br>`enger_id,porter_id,stars)`<br>`VALUES(2,2,2,5);`|Rejected by trigger<br>trg_rating_validate|Only COMPLETED bookings can<br>be rated (booking 2)|
|TC14|ON DELETE CASCADE:<br>delete booking 1|`DELETE FROM bookings WHERE`<br>`booking_id=1;`|Its payment and rating<br>rows are deleted too|left after delete: 0 payments, 0<br>ratings|



Page 30 

RailPort – Railway Station Porter Booking System 

23AIE303 DBMS · Project Report 

|**ID**|**Test**|**Statement**|**Expected**|**Actual database response**|
|---|---|---|---|---|
|TC15|TRIGGER T1|`UPDATE bookings SET`|completed_at filled|completed_at set: yes|
||(enhancement):|`status='COMPLETED' WHERE`|automatically||
||completing a booking<br>stamps completed_at|`booking_id=2;`|||



### **Appendix B – REST API summary** 

|**Methods**|**Endpoint**|**Table**|
|---|---|---|
|POST/GET/GET:id/PUT/DELETE|/api/users|users|
|…|/api/stations|stations|
|…|/api/platforms|platforms|
|…|/api/porters (?user_id, ?station_id, ?availability_status)|porters|
|…|/api/bookings (?passenger_id, ?porter_id, ?station_id, ?status,<br>?unassigned)|bookings|
|…|/api/payments (?booking_id)|payments|
|…|/api/ratings (?porter_id, ?passenger_id, ?booking_id)|ratings|
|POST|/api/auth/… (register / login with bcrypt)|users|



Each resource exposes **POST** (create), **GET** (list, with optional filters), **GET /:id** , **PUT /:id** (update) and **DELETE /:id** . 

### **Appendix C – How to reproduce the database** 

```
createdb railport_db
psql -d railport_db -f database/01_schema.sql        -- tables, constraints, indexes
psql -d railport_db -f database/02_seed.sql          -- project sample data
psql -d railport_db -f database/03_demo_data.sql     -- extra rows for analytics
psql -d railport_db -f database/04_enhancements.sql  -- optional triggers and view
psql -d railport_db -f database/05_queries.sql       -- the 14 advanced queries
cd backend  && cp .env.example .env && npm install && npm start      # http://localhost:5000
cd frontend && npm install && npm run dev
```

GitHub repository: [GitHub repository link – to be added] 

Page 31 

RailPort – Railway Station Porter Booking System 

23AIE303 DBMS · Project Report 

### **Appendix D – Complete DDL (01_schema.sql)** 

```
-- ============================================
-- RailPort Database Schema (PostgreSQL)
-- Run this AFTER: CREATE DATABASE railport_db;
-- Order matters: tables are created in dependency order
-- ============================================
-- 1. USERS (base table for passengers, porters, admins)
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    phone VARCHAR(15) NOT NULL UNIQUE,
    email VARCHAR(100) UNIQUE,
    password VARCHAR(255) NOT NULL,
    role VARCHAR(20) NOT NULL
        CHECK (role IN ('PASSENGER', 'PORTER', 'ADMIN')),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
-- 2. STATIONS
CREATE TABLE stations (
    station_id SERIAL PRIMARY KEY,
    station_name VARCHAR(100) NOT NULL,
    station_code VARCHAR(10) NOT NULL UNIQUE,
    city VARCHAR(100) NOT NULL,
    number_of_platforms INT NOT NULL
        CHECK (number_of_platforms > 0)
);
-- 3. PLATFORMS (depends on stations)
CREATE TABLE platforms (
    platform_id SERIAL PRIMARY KEY,
    station_id INT NOT NULL,
    platform_number INT NOT NULL
        CHECK (platform_number > 0),
    FOREIGN KEY (station_id)
        REFERENCES stations(station_id)
        ON DELETE CASCADE,
    UNIQUE (station_id, platform_number)
);
-- 4. PORTERS (depends on users, stations, platforms)
CREATE TABLE porters (
    porter_id SERIAL PRIMARY KEY,
    user_id INT NOT NULL UNIQUE,
    station_id INT NOT NULL,
    badge_number VARCHAR(50) NOT NULL UNIQUE,
    availability_status VARCHAR(20) NOT NULL DEFAULT 'OFFLINE'
        CHECK (availability_status IN ('AVAILABLE', 'BUSY', 'OFFLINE')),
    current_platform_id INT,
    rating NUMERIC(3,2) DEFAULT 0.00
        CHECK (rating >= 0 AND rating <= 5),
    FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE,
    FOREIGN KEY (station_id) REFERENCES stations(station_id),
    FOREIGN KEY (current_platform_id) REFERENCES platforms(platform_id)
);
-- 5. BOOKINGS (depends on users, porters, stations, platforms)
CREATE TABLE bookings (
    booking_id SERIAL PRIMARY KEY,
    passenger_id INT NOT NULL,
```

Page 32 

RailPort – Railway Station Porter Booking System 

23AIE303 DBMS · Project Report 

```
    porter_id INT,
    station_id INT NOT NULL,
    pickup_platform_id INT,
    train_number VARCHAR(20),
    coach_number VARCHAR(20),
    luggage_count INT NOT NULL DEFAULT 1
        CHECK (luggage_count > 0),
    destination VARCHAR(255),
    fare NUMERIC(10,2) NOT NULL DEFAULT 0.00
        CHECK (fare >= 0),
    status VARCHAR(20) NOT NULL DEFAULT 'REQUESTED'
        CHECK (status IN ('REQUESTED', 'ACCEPTED', 'IN_PROGRESS', 'COMPLETED', 'CANCELLED')),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    completed_at TIMESTAMP,
    FOREIGN KEY (passenger_id) REFERENCES users(user_id),
    FOREIGN KEY (porter_id) REFERENCES porters(porter_id),
    FOREIGN KEY (station_id) REFERENCES stations(station_id),
    FOREIGN KEY (pickup_platform_id) REFERENCES platforms(platform_id)
);
-- 6. PAYMENTS (depends on bookings)
CREATE TABLE payments (
    payment_id SERIAL PRIMARY KEY,
    booking_id INT NOT NULL UNIQUE,
    amount NUMERIC(10,2) NOT NULL
        CHECK (amount >= 0),
    payment_method VARCHAR(20) NOT NULL
        CHECK (payment_method IN ('CASH', 'UPI', 'CARD')),
    payment_status VARCHAR(20) NOT NULL DEFAULT 'PENDING'
        CHECK (payment_status IN ('PENDING', 'SUCCESS', 'FAILED')),
    transaction_id VARCHAR(100),
    payment_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (booking_id) REFERENCES bookings(booking_id) ON DELETE CASCADE
);
-- 7. RATINGS (depends on bookings, users, porters)
CREATE TABLE ratings (
    rating_id SERIAL PRIMARY KEY,
    booking_id INT NOT NULL UNIQUE,
    passenger_id INT NOT NULL,
    porter_id INT NOT NULL,
    stars INT NOT NULL
        CHECK (stars BETWEEN 1 AND 5),
    review VARCHAR(500),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (booking_id) REFERENCES bookings(booking_id) ON DELETE CASCADE,
    FOREIGN KEY (passenger_id) REFERENCES users(user_id),
    FOREIGN KEY (porter_id) REFERENCES porters(porter_id)
);
-- Helpful indexes for common lookups (used heavily once CRUD/API is running)
CREATE INDEX idx_bookings_passenger ON bookings(passenger_id);
CREATE INDEX idx_bookings_porter ON bookings(porter_id);
CREATE INDEX idx_bookings_station ON bookings(station_id);
CREATE INDEX idx_porters_station ON porters(station_id);
CREATE INDEX idx_platforms_station ON platforms(station_id);
```

Page 33 

•	Installation, setup, and execution instructions.
•	Demonstration instructions explaining how to run the project and reproduce the database state

intsall the zip folder as given and extract them,then try to create a database named railport_db,which will be better as all the codes uploaded uses that,then try to run schema.sql,seed.sql,bulk_data.sql in the same order as query in that database.
Then from frontend in ypur cmd after extracting,open cmd in backend folder and  try command 'npm install'(if windows) ,please check for other dependencies based on your system,then try to run that by command 'npm run dev' .Now you should se something like backend running on ~-  https:/local:5000 ,(maybe different localhost).
Do the same process by opening other command window,keeping the last one active and running and do the same process for frontend.Here you will get a local link to open this system.
Then u can access the database and do whatever operations from the local website itself

# ER-diagram

## Data Model (ER Diagram)

All foreign keys were validated against the raw CSVs: **zero orphan records** across every relationship below.

```mermaid
erDiagram
    CUSTOMERS ||--o{ LOADS : "places"
    ROUTES ||--o{ LOADS : "serves"
    LOADS ||--|| TRIPS : "fulfilled by (1:1)"
    LOADS ||--o{ DELIVERY_EVENTS : "has 2 (pickup, delivery)"
    TRIPS ||--o{ DELIVERY_EVENTS : "generates"
    FACILITIES ||--o{ DELIVERY_EVENTS : "hosts"
    DRIVERS |o--o{ TRIPS : "drives"
    TRUCKS |o--o{ TRIPS : "hauls with"
    TRAILERS |o--o{ TRIPS : "pulls"
    TRIPS ||--o{ FUEL_PURCHASES : "consumes"
    TRUCKS |o--o{ FUEL_PURCHASES : "fuelled"
    DRIVERS |o--o{ FUEL_PURCHASES : "bought by"
    TRIPS ||--o{ SAFETY_INCIDENTS : "involves"
    TRUCKS |o--o{ SAFETY_INCIDENTS : "involved in"
    DRIVERS |o--o{ SAFETY_INCIDENTS : "involved in"
    TRUCKS ||--o{ MAINTENANCE_RECORDS : "serviced"
    TRUCKS ||--o{ TRUCK_UTILIZATION_METRICS : "monthly"
    DRIVERS ||--o{ DRIVER_MONTHLY_METRICS : "monthly"

    CUSTOMERS {
        string customer_id PK
        string customer_name
        string customer_type
        int credit_terms_days
        string primary_freight_type
        string account_status
        date contract_start_date
        int annual_revenue_potential
    }
    ROUTES {
        string route_id PK
        string origin_city
        string origin_state
        string destination_city
        string destination_state
        int typical_distance_miles
        float base_rate_per_mile
        float fuel_surcharge_rate
        int typical_transit_days
    }
    LOADS {
        string load_id PK
        string customer_id FK
        string route_id FK
        date load_date
        string load_type
        int weight_lbs
        int pieces
        float revenue
        float fuel_surcharge
        float accessorial_charges
        string load_status
        string booking_type
    }
    TRIPS {
        string trip_id PK
        string load_id FK
        string driver_id FK "nullable"
        string truck_id FK "nullable"
        string trailer_id FK "nullable"
        date dispatch_date
        int actual_distance_miles
        float actual_duration_hours
        float fuel_gallons_used
        float average_mpg
        float idle_time_hours
        string trip_status
    }
    DELIVERY_EVENTS {
        string event_id PK
        string load_id FK
        string trip_id FK
        string event_type
        string facility_id FK
        datetime scheduled_datetime
        datetime actual_datetime
        int detention_minutes
        bool on_time_flag
        string location_city
        string location_state
    }
    FACILITIES {
        string facility_id PK
        string facility_name
        string facility_type
        string city
        string state
        float latitude
        float longitude
        int dock_doors
        string operating_hours
    }
    DRIVERS {
        string driver_id PK
        string first_name
        string last_name
        date hire_date
        date termination_date "nullable"
        string license_number
        string license_state
        date date_of_birth
        string home_terminal
        string employment_status
        string cdl_class
        int years_experience
    }
    TRUCKS {
        string truck_id PK
        int unit_number
        string make
        int model_year
        string vin
        date acquisition_date
        int acquisition_mileage
        string fuel_type
        int tank_capacity_gallons
        string status
        string home_terminal
    }
    TRAILERS {
        string trailer_id PK
        int trailer_number
        string trailer_type
        int length_feet
        int model_year
        string vin
        date acquisition_date
        string status
        string current_location
    }
    FUEL_PURCHASES {
        string fuel_purchase_id PK
        string trip_id FK
        string truck_id FK "nullable"
        string driver_id FK "nullable"
        datetime purchase_date
        string location_city
        string location_state
        float gallons
        float price_per_gallon
        float total_cost
        string fuel_card_number
    }
    SAFETY_INCIDENTS {
        string incident_id PK
        string trip_id FK
        string truck_id FK "nullable"
        string driver_id FK "nullable"
        datetime incident_date
        string incident_type
        string location_city
        string location_state
        bool at_fault_flag
        bool injury_flag
        float vehicle_damage_cost
        float cargo_damage_cost
        float claim_amount
        bool preventable_flag
        string description
    }
    MAINTENANCE_RECORDS {
        string maintenance_id PK
        string truck_id FK
        date maintenance_date
        string maintenance_type
        int odometer_reading
        float labor_hours
        float labor_cost
        float parts_cost
        float total_cost
        string facility_location
        float downtime_hours
        string service_description
    }
    TRUCK_UTILIZATION_METRICS {
        string truck_id PK "composite with month"
        date month PK
        int trips_completed
        int total_miles
        float total_revenue
        float average_mpg
        int maintenance_events
        float maintenance_cost
        float downtime_hours
        float utilization_rate
    }
    DRIVER_MONTHLY_METRICS {
        string driver_id PK "composite with month"
        date month PK
        int trips_completed
        int total_miles
        float total_revenue
        float average_mpg
        float total_fuel_gallons
        float on_time_delivery_rate
        float average_idle_hours
    }
```

### Join notes (read before writing SQL)

- **loads : trips is 1:1.** Every load has exactly one trip, so joining them never inflates revenue.
- **delivery_events has exactly 2 rows per load** (Pickup and Delivery). Joining loads to events duplicates `revenue`. Aggregate events first, or filter to `event_type = 'Delivery'`.
- **fuel_purchases is many per trip** (up to 11), and 8,471 trips have no fuel purchase. Aggregate fuel to trip level before joining.
- **trips.driver_id, truck_id and trailer_id are nullable** (about 1,700 rows each). Use LEFT JOINs, or those trips disappear from your profit totals.
- **Monthly metric tables** (`truck_utilization_metrics`, `driver_monthly_metrics`) are pre-aggregated at truck-month and driver-month grain. Don't join them to trip-level facts, or you will double count.

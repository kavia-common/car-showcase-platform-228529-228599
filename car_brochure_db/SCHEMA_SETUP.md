# Car Brochure DB — Schema & Seed Data (PostgreSQL)

This container uses PostgreSQL. Connection is provided via:

- `db_connection.txt` (contains a `psql postgresql://...` command)

## How schema/seed was applied

Per container rules, SQL was executed **one statement at a time** using:

```bash
$(cat db_connection.txt) -c "SQL_STATEMENT_HERE"
```

No `.sql` migration file was created.

## Tables

### `cars`
Stores the core car models.

Key columns:
- `make`, `model`, `year`, `body_type`
- `msrp_base_cents` (integer cents)
- `description`
- `is_active`
- `created_at`, `updated_at`

### `trims`
Trim levels per car.

Key columns:
- `car_id` → `cars(id)` (cascade delete)
- `name`
- `msrp_cents`
- `is_default`
- Unique constraint: `UNIQUE(car_id, name)`

### `trim_specs`
Structured specs per trim.

Key columns:
- `trim_id` → `trims(id)` (cascade delete)
- `category` (e.g., Powertrain, Efficiency)
- `name` (e.g., Horsepower)
- `value`, `unit`
- `sort_order`
- Unique constraint: `UNIQUE(trim_id, category, name)`

### `trim_features`
Feature bullets per trim.

Key columns:
- `trim_id` → `trims(id)` (cascade delete)
- `feature_group` (e.g., Standard, Safety)
- `description`
- `sort_order`

### `car_images`
Images for a car (optionally associated to a trim).

Key columns:
- `car_id` → `cars(id)` (cascade delete)
- `trim_id` → `trims(id)` (set null on trim delete)
- `url`, `alt_text`
- `kind` (default `'gallery'`)
- `sort_order`
- `is_primary`

### `inquiries`
Lead/contact form submissions.

Key columns:
- `car_id` → `cars(id)` (set null on car delete)
- `trim_id` → `trims(id)` (set null on trim delete)
- `full_name`, `email`, `phone`, `message`
- `preferred_contact_method`
- `status` (default `'new'`)
- `source`
- `created_at`

## Indexes (filtering/search)

Created indexes include:
- `cars`: active, make/model, body_type, year, base price
- trigram search: `pg_trgm` extension + GIN trigram indexes on `cars.make` and `cars.model`
- `trims`: `car_id`, `msrp_cents`
- `trim_specs`: `trim_id`, `category`
- `trim_features`: `trim_id`
- `car_images`: `car_id`, `trim_id`, partial index for primary image per car (`WHERE is_primary = true`)
- `inquiries`: `created_at DESC`, `status`

## Seed/sample data

Seeded:
- 3 cars:
  - RetroMotors Neon Sprint (2025, Hatchback)
  - Sunset Auto Canyon Cruiser (2025, SUV)
  - BlueWave Eclipse GT (2025, Coupe)
- 6 trims across those cars (Base/Sport, Touring/Trail, GT/GT Premium)
- 11 example `trim_specs`
- 1 example `trim_features`

No sample `car_images` or `inquiries` were inserted yet (intentionally left empty for app-driven creation).

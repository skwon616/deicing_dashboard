# deicing_dashboard
# Deicing Queue & Gate Occupancy Analysis

This repository contains a set of Python notebooks for **simulating deicing / ground-handling queues** and **forecasting gate occupancy** at a hub airport.  
The goal is to estimate delay and resource usage for each ground-handling company and to provide data-driven inputs for **capacity planning during snow events**.

> ⚠️ All data used in this repository are sample or anonymized CSV files.  
> Any references to companies (KAS, AAP, SHP, SPI, JAS) represent generic ground-handling agents.

---

##  1. Project Overview

The notebook `deicing.ipynb` performs three main tasks:

1. **Per-handler queue simulation**  
   - Reads flight queues for each ground-handling company (KAS, AAP, SHP, SPI, JAS).  
   - Converts schedule times (SOBT/EOBT) to minutes for calculation.  
   - Assigns service start/end times based on a simple service-time model by aircraft type (e.g., Code C/E/F).  
   - Computes waiting time and service completion time for each flight.

2. **Integrated delay estimation across all handlers**  
   - Merges the processed queues from all five handlers into one consolidated dataframe.  
   - Normalizes column names and fills missing values.  
   - Replaces missing EOBT values with SOBT when necessary.  
   - Produces a unified view of deicing/ground-handling delay for all flights.

3. **Arrival gate-occupied forecast**  
   - Combines arrival data (`arr.csv`) with departure data (`merged_eobt_data.csv`).  
   - Converts EIBT/SOBT/EOBT fields to minutes.  
   - Aligns flight identifiers and link fields (FLIGHT, LKF) to trace turn-around relationships.  
   - Estimates gate occupancy windows for arriving and departing aircraft to support gate capacity assessments.

---

## 2. Methods & Assumptions

- **Time conversion**  
  - All timetable fields (e.g., `SOBT`, `EOBT`, `EIBT`) are converted from `HH:MM` strings (or Excel-style floats) into minutes since midnight for easier arithmetic.
- **Service time model**  
  - A simple mapping is used to model deicing/handling duration by aircraft category:  
    - Code C → 20 minutes  
    - Code E → 25 minutes  
    - Code F → 30 minutes  
  - These values can be adjusted in the helper function `get_worktime()`.

- **Queue logic**  
  - Flights are sorted by scheduled off-block time (`SOBT_MIN`).  
  - For each handler, the next available time is updated as flights are processed.  
  - Waiting time = max(0, service start – scheduled time).  
  - Results can be aggregated by handler, time window, or aircraft type.

---

## 3. Data Inputs

The notebook expects the following CSV files in the working directory:

- `kas_queue.csv`, `aap_queue.csv`, `shp_queue.csv`, `spi_queue.csv`, `jas_queue.csv`  
  - Per-handler flight queues with at least: `FLIGHT`, `STN`, `SOBT`, `EOBT`, aircraft type/code.
- `VTT.csv`  
  - Optional configuration / service time table (vehicle/turn-time information).
- `arr.csv`  
  - Arrival data including `FLIGHT`, `EIBT`, `LKF` and other arrival fields.
- `merged_eobt_data.csv`  
  - Consolidated departure file with `FLIGHT`, `SOBT`, `EOBT`.

> Actual production data are not included due to confidentiality.  
> Users may replace these files with their own datasets following the same schema.

---

## 4. How to Run

1. Clone this repository and move into the project directory:

   ```bash
   git clone https://github.com/<your-id>/deicing-queue-analysis.git
   cd deicing-queue-analysis

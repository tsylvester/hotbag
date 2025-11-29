# **TECHNICAL REQUIREMENTS DOCUMENT (TRD)**

**Pizza Delivery Temperature Calculator**
**Version:** 0.1
**Status:** Draft
**Source:** Translation from PRD (v0.1)

---

# **1. System Overview**

The system provides a web-based tool allowing users to predict the **final temperature of a delivered pizza** based on:

* Delivery route (origin → destination)
* Date/time of delivery
* Real-time or forecasted weather along route
* Pizza cooling model with and without hotbag
* User-specified handoff/wait time (walk-up time)

The system integrates mapping, routing, weather APIs, and a thermal model to compute pizza temperature over time.

---

# **2. Architecture Overview**

The system consists of:

1. **Client Web Application**

   * UI for map selection
   * Input controls for date/time, pizza type, hotbag on/off
   * Displays predicted temperatures and temperature difference

2. **Backend Service Layer**

   * Routing service wrapper
   * Weather service wrapper
   * Thermal modeling engine
   * Caching layer for map/weather results
   * Logging/telemetry

3. **External Integrations**

   * Map provider (e.g., Mapbox, Google Maps, OSM)
   * Weather provider (e.g., OpenWeather, NOAA, Tomorrow.io)
   * Optional calendar/time-zone service

---

# **3. Functional Requirements**

## **3.1 Map Selection**

* User can:

  * Specify **origin** (pizza shop)
  * Specify **destination** (delivery address)
  * Do so by:

    * Searching for addresses
    * Clicking points on the map
* System must:

  * Render base map layers
  * Support pin placement
  * Support drag-to-adjust endpoints

**Data Returned from Map API:**

* Route geometry (polyline)
* Distance
* Estimated travel time
* Optional: speed limits, traffic data

---

## **3.2 Date & Time Selection**

* User selects planned delivery date and time.
* System uses selection to:

  * Retrieve forecasted weather
  * Adjust time zones
  * Estimate predicted traffic if available

---

## **3.3 Weather Integration**

Given route geometry + datetime:

* Query weather API for:

  * Ambient temperature
  * Wind speed and direction
  * Humidity (optional Tier 2)
  * Precipitation (optional Tier 2)

* Weather can be:

  * Current conditions (if delivery is “now”)
  * Forecast conditions (future date)
  * Historical conditions (past queries; Tier 2)

Weather must be sampled:

* At route centroid OR
* At multiple route points (Tier 2 for higher accuracy)

---

## **3.4 Pizza Cooling Calculator**

Backend provides pizza temperature predictions by evaluating:

### Inputs:

* Initial pizza temperature (configurable default)
* Ambient temperature
* Wind speed (affects convection coefficient)
* Time-in-transit
* User-specified “handoff wait time”
* Insulation stack:

  * Pizza → Box → (optional) Bag

### Outputs:

* Temperature with hotbag
* Temperature without hotbag
* Δ Temperature
* Temperature curve (Tier 2 graphing)

### Model Behavior:

* Uses Newtonian cooling with variable heat-transfer coefficient **h(t)**.
* h is function of:

  * Ambient temp
  * Wind speed
  * Insulation layers
  * Vehicle interior conditions (stub)

### Bag/Box Modeling:

* Each insulation layer treated as thermal resistance:

  * R_box
  * R_bag
* Combined as series resistances:
  [
  R_{total} = R_{pizza} + R_{box} + R_{bag}
  ]
* If bag not used:
  [
  R_{total} = R_{pizza} + R_{box}
  ]

### Calibration:

* TRD must allow ingestion of experimental data to refine constants.

---

## **3.5 UI and Output**

UI must display:

* Route map
* Estimated travel duration
* Weather summary
* Final predicted pizza temp (box only)
* Final predicted pizza temp (box + bag)
* Δ Temperature (“how much hotter”)

UI Options:

* Toggle hotbag on/off
* Adjust handoff time

---

# **4. Non-Functional Requirements**

## **4.1 Performance**

* End-to-end calculation (map + weather + thermal model) must return in < 2 sec under normal load.
* Thermal model compute must execute in < 50 ms.

## **4.2 Scalability**

* Must support 10 requests/second (initial target)
* Weather + map results cached for 10 minutes

## **4.3 Accuracy**

* Model must be within ±4°F of physical measurements under controlled test conditions (Tier 2 requirement)

## **4.4 Reliability**

* If map or weather API fails, system must:

  * Return fallback values
  * Notify user gracefully

## **4.5 Security**

* API keys stored server-side
* All traffic HTTPS only

---

# **5. System Components & Interfaces**

## **5.1 Frontend**

### Technologies:

* JS framework of choice (React recommended)
* Map plugin SDK
* Charting library for optional graphs

### Responsibilities:

* Send API requests to backend
* Display data and model results
* Handle validation

---

## **5.2 Backend**

### Services:

#### **Routing Service**

* Input: origin/destination
* Output: route geometry, ETA

#### **Weather Service**

* Input: coordinates + datetime
* Output: weather parameters

#### **Thermal Modeling Engine**

* Input: route time, weather, config
* Output: predicted temperature numbers

---

# **6. Data Model**

## **Core Entities:**

### **DeliveryRequest**

* origin_lat
* origin_lon
* dest_lat
* dest_lon
* datetime_local
* handoff_time_seconds

### **WeatherSample**

* ambient_temp
* wind_speed
* humidity
* precipitation
* timestamp

### **ThermalInputs**

* initial_temp
* R_box
* R_bag
* ambient_temp
* wind_speed
* total_time_secs
* handoff_time_secs

### **ThermalOutput**

* T_final_box
* T_final_bag
* delta_T
* curve (optional)

---

# **7. Algorithms & Calculations**

## **7.1 Newtonian Cooling With Variable Coefficients**

General differential equation:
[
\frac{dT}{dt} = -k(t) (T - T_{amb}(t))
]

Where:
[
k(t) = \frac{1}{R_{total}(t) \cdot C_{pizza}}
]

Wind speed affects convection coefficient:
[
h(t) = h_0 + c_1 \cdot v_{wind}(t)
]

Ambient temp may vary along route (Tier 2 interpolation).

### Integration:

* Euler or RK4 numeric method
* Default step 1 sec

---

# **8. Deployment & DevOps**

* Cloud-hosted backend (AWS, GCP, Azure)
* CI/CD for both frontend and backend
* Automated tests for:

  * Routing adapter
  * Weather adapter
  * Thermal model

---

# **9. Open Questions (Stubs)**

1. How much spatial sampling of weather do we need along the route?
2. Should vehicle interior temperature be modeled?
3. Should pizza type (mass, toppings, cheese moisture) be user-selectable?
4. Should traffic conditions influence delivery time calculation?
5. How do rain/precipitation affect convection coefficient?
6. How do we calibrate R_box and R_bag values from real-world data?
7. Should we support multiple pizza sizes/shapes?
8. Should we expose a temperature curve chart?
9. Should we expose API access for third-party integration?
10. Legal disclaimers: do food safety guidelines require warnings?

---

Next:

* A *visual architecture diagram*
* A *sequence diagram*
* A *database schema*
* The *thermal model specification*
* Or a *roadmap with milestones (MVP → v1.0 → v2.0)*

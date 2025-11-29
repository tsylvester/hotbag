# **📘 PRODUCT REQUIREMENTS DOCUMENT (PRD)**

## **Product: Pizza Heat Saver**

## **Purpose:**

Allow customers to estimate the **final temperature of their delivered pizza**, comparing delivery **with** vs **without** a hotbag, using real map routing, weather conditions, and timing inputs.

---

# **1. Overview**

**Pizza Heat Saver** is a consumer-facing web application that predicts the temperature of a pizza from oven to customer at the time of handoff.
The app uses:

* MAP API → to compute real-world travel time & distance
* WEATHER API → to retrieve ambient temperature, wind, precipitation
* DATE/TIME selector → to get forecast or historical weather
* INSULATION MODEL → to simulate temperature decay through layered materials (pizza + box + bag)

Users choose:

* pickup location (pizzeria)
* delivery destination
* date/time of delivery
* additional wait time (driver walking to door, apartment elevator, etc.)

The system displays predicted pizza temperatures:

* **Without hotbag**
* **With hotbag**
* **Difference at handoff**

---

# **2. Goals & Non-Goals**

### **2.1 Goals**

* Provide accurate, user-friendly heat-loss modeling.
* Show real-world impact of insulated delivery bags.
* Improve consumer understanding of why some pizzas arrive hotter.

### **2.2 Non-Goals**

* Real-time tracking of actual drivers
* Fleet management or operational tools for restaurants
* Predicting pizza texture, moisture, or crispness
* Supporting food types other than pizza (v1)

---

# **3. User Stories**

### **Primary User (Customer)**

1. *As a customer*, I want to see how much hotter my pizza would arrive if the driver uses a hotbag.
2. *As a customer*, I want to visually pick my address and the restaurant on a map.
3. *As a customer*, I want the system to automatically use weather conditions so I don’t have to enter them manually.
4. *As a customer*, I want to change delivery time (past or future order) and see how weather affects temperature.
5. *As a customer*, I want input for additional waiting time at handoff (apartment walk, elevator, gate delays).

### **Secondary User (Restaurant Operators)**

1. *As a restaurant operator*, I want to show customers the benefit of using certified insulated bags.
1. *As a restaurant operator*, I want to show drivers the benefit of using certified insulated bags.
2. *As a restaurant operator*, I want to compare different bag-insulation ratings.
2. *As a restaurant operator*, I want to reduce temperature complaints.

---

# **4. Functional Requirements**

## **4.1 Map Interaction**

* UI displays an interactive map.
* User selects:

  * **Origin** (restaurant)
  * **Destination** (delivery address)
* System calculates **optimal driving route** and **estimated travel time**.

### Functional Requirements

* **FR-1:** Map plugin must support click-to-select and search-by-address.
* **FR-2:** Must return route distance and time via routing API.
* **FR-3:** Must allow user overrides for travel time (traffic, rural roads).

---

## **4.2 Weather Integration**

### Inputs:

* Date & Time selector
* Coordinates from origin/destination

### Required Weather Data:

* Ambient air temperature
* Wind speed & direction
* Precipitation type (rain, snow)
* Humidity (optional)

### Functional Requirements:

* **FR-4:** Weather plugin returns forecast or historical weather for specified date/time.
* **FR-5:** System chooses weather conditions for *the route*, not just start point.
* **FR-6:** Wind chill factor auto-calculated for external exposure.

---

## **4.3 Delivery Timing & User Inputs**

### User Inputs:

* **Handoff delay:** 0–10 minutes
* Optional: **driver waiting at store**, **unusual traffic delay**

### Functional Requirements:

* **FR-7:** UI offers slider or numeric input for Handoff Time (seconds/minutes).
* **FR-8:** Total simulation time = travel time + handoff delay.

---

# **5. Insulation & Temperature Model**

The model must simulate layered heat loss:

### Layers:

1. Pizza
2. Cardboard pizza box
3. Optional: insulated hot delivery bag

### Core Insight:

Heat-loss constant **k is not constant**.
The model must dynamically compute heat flux based on:

* ambient temperature
* wind shear
* vehicle cabin temp (assumed constant but configurable later)
* cumulative insulation R-values
* surface area & geometry of pizza
* convection inside the bag (near-zero airflow)
* conduction through multiple layers

### Functional Requirements:

* **FR-9:** For each layer, compute heat transfer using conduction, convection, and radiation.
* **FR-10:** Box + Bag are cumulative, not either-or.
* **FR-11:** Environmental heat-loss coefficient must vary with weather (wind, rain, temp grad).
* **FR-12:** Model outputs:

  * Final temperature without bag
  * Final temperature with bag
  * ΔT difference

### Temperature Model Algorithm (High-Level)

* Compute thermal mass of pizza.
* Compute effective U-value of:

  * pizza + box
  * pizza + box + bag
* Compute ambient heat flux as a function of:

  * air temp
  * wind speed
  * precipitation
* Run Newtonian cooling / transient heat conduction simulation across time steps (e.g., 1s increments).

---

# **6. UI/UX Requirements**

### Core UI Elements

* Map selector
* Weather summary card
* Input section for:

  * date/time
  * handoff delay
* Results panel with:

  * “With Bag” final temp
  * “Without Bag” final temp
  * Difference
  * Color-coded gauge or thermometer graphic

### Functional Requirements:

* **FR-13:** All results update automatically when inputs change.
* **FR-14:** Provide tooltips explaining assumptions and physics.
* **FR-15:** Mobile & desktop responsiveness.

---

# **7. Non-Functional Requirements**

* **Reliability:** Weather API fallback mechanism
* **Performance:** Temperature simulation completes <500ms
* **Scalability:** Up to 1000 concurrent users
* **Accessibility:** WCAG 2.1 AA compliant
* **Localization:** Support for °C and °F

---

# **8. Data Sources & Integrations**

### Required APIs:

* **Maps:** Google Maps, Mapbox, or OpenStreetMap routing
* **Weather:** NOAA, OpenWeatherMap, Tomorrow.io
* **Time:** Built-in JS Date or external timezone API

---

# **9. Open Questions**

1. Should users be allowed to select different hotbag insulation models?
2. Should the vehicle’s interior temperature be considered (e.g., heated cabin vs scooter delivery)?
3. Should the app support multiple food types (subs, wings) in later versions?
4. Should restaurants be able to embed a white-labeled version on their websites?

---

# **10. Future Enhancements (Post-MVP)**

* Bag comparison mode (cheap vs premium insulation).
* Driver telemetry API for real-time temperature prediction.
* Crowd-sourced local delivery speed & weather accuracy scoring.
* Pizza thickness & toppings profile affecting heat mass.
* Advanced convection modeling (walk-to-door, staircase, breezeways).

---

Next:

* **Wireframes / UI mockups**,
* **The full system architecture**,
* **The data model**, or
* **The actual physics model equations**.

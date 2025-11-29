# **FEATURE SPECIFICATION DOCUMENT**

**Pizza Heat Saver / Pizza Delivery Temperature Calculator**  
**Version:** 0.1  
**Status:** Draft  
**Source:** Compiled from PRD (v0.1) and TRD (v0.1)

---

# **1. Feature Overview**

This document specifies the detailed functional requirements and features for the Pizza Heat Saver application. Features are organized by functional domain and include inputs, outputs, behavior, and acceptance criteria.

---

# **2. Feature: Map Selection & Route Calculation**

## **2.1 Description**

Users can visually select origin (pizzeria) and destination (delivery address) on an interactive map, and the system calculates the optimal driving route with distance and estimated travel time.

## **2.2 Functional Requirements**

* **FR-1:** Map plugin must support click-to-select and search-by-address
* **FR-2:** Must return route distance and time via routing API
* **FR-3:** Must allow user overrides for travel time (traffic, rural roads)

## **2.3 User Stories**

* *As a customer*, I want to visually pick my address and the restaurant on a map.

## **2.4 Inputs**

* **Origin Selection:**
  * User can search for address or click on map
  * Latitude/longitude coordinates
  * Address string (optional display)

* **Destination Selection:**
  * User can search for address or click on map
  * Latitude/longitude coordinates
  * Address string (optional display)

* **Travel Time Override (Optional):**
  * Manual override for estimated travel time
  * Reason: traffic, rural roads, etc.

## **2.5 System Behavior**

* System renders interactive map with base layers
* System supports pin placement at origin and destination
* System supports drag-to-adjust endpoints
* System calculates optimal driving route using routing API
* System displays route polyline on map
* System retrieves route geometry (polyline)
* System retrieves route distance
* System retrieves estimated travel time
* System allows user to override travel time manually

## **2.6 Outputs**

* Route geometry (polyline) for map display
* Route distance (miles/kilometers)
* Estimated travel time (minutes/seconds)
* Origin coordinates (lat, lon)
* Destination coordinates (lat, lon)
* Optional: speed limits, traffic data (if available)

## **2.7 UI Components**

* Interactive map display
* Search input fields for origin/destination
* Map markers/pins for origin and destination
* Route visualization (polyline)
* Route information display (distance, time)
* Manual time override input (optional)

---

# **3. Feature: Weather Integration**

## **3.1 Description**

System automatically retrieves weather conditions (current, forecast, or historical) for the delivery route based on selected date/time and route coordinates. Weather data informs the thermal model calculations.

## **3.2 Functional Requirements**

* **FR-4:** Weather plugin returns forecast or historical weather for specified date/time
* **FR-5:** System chooses weather conditions for *the route*, not just start point
* **FR-6:** Wind chill factor auto-calculated for external exposure

## **3.3 User Stories**

* *As a customer*, I want the system to automatically use weather conditions so I don't have to enter them manually.
* *As a customer*, I want to change delivery time (past or future order) and see how weather affects temperature.

## **3.4 Inputs**

* Date & Time selector (user input)
* Route coordinates (from map selection)
  * Route centroid OR
  * Multiple route points (Tier 2 for higher accuracy)

## **3.5 Required Weather Data**

* **Required:**
  * Ambient air temperature
  * Wind speed
  * Wind direction

* **Optional (Tier 2):**
  * Humidity
  * Precipitation type (rain, snow)

## **3.6 System Behavior**

* System queries weather API using route coordinates + datetime
* System determines weather type based on datetime:
  * Current conditions (if delivery is "now")
  * Forecast conditions (future date)
  * Historical conditions (past queries; Tier 2)
* System samples weather at route centroid (MVP) or multiple route points (Tier 2)
* System calculates wind chill factor for external exposure
* System caches weather results for 10 minutes

## **3.7 Outputs**

* Ambient temperature (°F or °C)
* Wind speed (mph or m/s)
* Wind direction (degrees or cardinal)
* Humidity (optional)
* Precipitation type (optional)
* Weather timestamp
* Wind chill factor (calculated)

## **3.8 UI Components**

* Weather summary card displaying:
  * Current/forecast temperature
  * Wind conditions
  * Precipitation (if applicable)
  * Weather icon/visual indicator

---

# **4. Feature: Date & Time Selection**

## **4.1 Description**

Users select the planned delivery date and time, which determines whether to fetch current, forecast, or historical weather data.

## **4.2 Inputs**

* Date selector (calendar widget)
* Time selector (time picker)
* Timezone (auto-detected or user-selected)

## **4.3 System Behavior**

* System uses selected date/time to:
  * Retrieve appropriate weather data (current/forecast/historical)
  * Adjust time zones automatically
  * Estimate predicted traffic if available (Tier 2)

## **4.4 Outputs**

* Selected datetime (local time)
* Selected datetime (UTC)
* Timezone information

## **4.5 UI Components**

* Date picker
* Time picker
* Timezone display

---

# **5. Feature: Delivery Timing & Handoff Input**

## **5.1 Description**

Users can specify additional waiting time at handoff (walking to door, apartment elevator, gate delays) that is added to the travel time for temperature calculation.

## **5.2 Functional Requirements**

* **FR-7:** UI offers slider or numeric input for Handoff Time (seconds/minutes)
* **FR-8:** Total simulation time = travel time + handoff delay

## **5.3 User Stories**

* *As a customer*, I want input for additional waiting time at handoff (apartment walk, elevator, gate delays).

## **5.4 Inputs**

* **Handoff delay:** 0–10 minutes (user input)
* Optional: driver waiting at store, unusual traffic delay (Tier 2)

## **5.5 System Behavior**

* System accepts handoff delay input (0-10 minutes)
* System calculates total simulation time = travel time + handoff delay
* System applies handoff delay to thermal model calculation

## **5.6 Outputs**

* Handoff delay value (seconds)
* Total simulation time (travel time + handoff delay)

## **5.7 UI Components**

* Slider or numeric input for handoff time
* Label indicating purpose (e.g., "Time at handoff: walking, elevator, gate delays")

---

# **6. Feature: Thermal Modeling Engine**

## **6.1 Description**

Backend service that computes pizza temperature predictions using a physics-based thermal model that accounts for layered insulation (pizza + box + optional bag) and environmental conditions.

## **6.2 Functional Requirements**

* **FR-9:** For each layer, compute heat transfer using conduction, convection, and radiation
* **FR-10:** Box + Bag are cumulative, not either-or
* **FR-11:** Environmental heat-loss coefficient must vary with weather (wind, rain, temp grad)
* **FR-12:** Model outputs: final temperature without bag, final temperature with bag, ΔT difference

## **6.3 Inputs**

* Initial pizza temperature (configurable default, typically ~180°F)
* Ambient temperature (from weather service)
* Wind speed (from weather service, affects convection coefficient)
* Total time-in-transit (travel time + handoff delay)
* User-specified handoff wait time
* Insulation stack:
  * R_pizza (pizza thermal properties)
  * R_box (cardboard box thermal resistance)
  * R_bag (hotbag thermal resistance, optional)
* Vehicle interior conditions (stub for future enhancement)

## **6.4 Thermal Model Algorithm**

### **6.4.1 Model Approach**

* Uses **Newtonian cooling** with variable heat-transfer coefficient h(t)
* Heat-loss constant k is NOT constant - dynamically computed
* Considers:
  * Conduction through multiple layers
  * Convection (affected by wind speed)
  * Radiation (ambient temp differential)
  * Near-zero airflow inside bag (reduced convection)

### **6.4.2 Mathematical Model**

**General differential equation:**
```
dT/dt = -k(t) × (T - T_amb(t))
```

**Where:**
```
k(t) = 1 / (R_total(t) × C_pizza)
```

**Wind speed affects convection coefficient:**
```
h(t) = h_0 + c_1 × v_wind(t)
```

**Total thermal resistance:**
```
With bag:  R_total = R_pizza + R_box + R_bag
Without bag: R_total = R_pizza + R_box
```

### **6.4.3 Calculation Steps**

1. Compute thermal mass of pizza (C_pizza)
2. Compute effective U-value of:
   * pizza + box
   * pizza + box + bag
3. Compute ambient heat flux as a function of:
   * air temperature
   * wind speed
   * precipitation (optional)
4. Run Newtonian cooling / transient heat conduction simulation across time steps (1 second increments)
5. Use Euler or RK4 numeric integration method

### **6.4.4 Dynamic Factors**

* Ambient temperature (may vary along route - Tier 2 interpolation)
* Wind speed (affects convection coefficient h(t))
* Precipitation (affects convection - to be calibrated)
* Cumulative insulation R-values (series resistances)
* Surface area & geometry of pizza
* Convection inside the bag (near-zero airflow)

## **6.5 System Behavior**

* Model calculates temperature decay over time for both scenarios:
  * Scenario 1: Pizza + Box only
  * Scenario 2: Pizza + Box + Bag
* Model uses configurable time step (default: 1 second)
* Model outputs temperature at each time step (for curve generation - Tier 2)
* Model calibrates constants based on experimental data (future capability)

## **6.6 Outputs**

* **T_final_box:** Final temperature without bag (°F or °C)
* **T_final_bag:** Final temperature with bag (°F or °C)
* **delta_T:** Temperature difference (T_final_bag - T_final_box)
* **curve (optional, Tier 2):** Temperature over time array for graphing

## **6.7 Performance Requirements**

* Thermal model computation must execute in < 50 ms (per TRD)

---

# **7. Feature: Temperature Results Display**

## **7.1 Description**

UI displays the predicted pizza temperatures for both scenarios (with and without hotbag) along with the temperature difference and visual indicators.

## **7.2 Functional Requirements**

* **FR-12:** Model outputs displayed clearly
* **FR-13:** All results update automatically when inputs change

## **7.3 User Stories**

* *As a customer*, I want to see how much hotter my pizza would arrive if the driver uses a hotbag.

## **7.4 System Behavior**

* System displays predicted temperatures immediately after calculation
* System updates results automatically when any input changes (FR-13)
* System provides visual feedback during calculation (loading state)

## **7.5 Outputs Displayed**

* "With Bag" final temperature
* "Without Bag" final temperature
* Temperature difference (ΔT)
* Color-coded gauge or thermometer graphic
* Estimated travel duration (from route calculation)
* Weather summary (for context)

## **7.6 UI Components**

* Results panel with:
  * Large temperature display for "With Bag" scenario
  * Large temperature display for "Without Bag" scenario
  * Prominent difference display ("X° hotter with bag")
  * Color-coded gauge/thermometer visualization
  * Optional: Temperature curve chart (Tier 2)

---

# **8. Feature: Hotbag Toggle**

## **8.1 Description**

Users can toggle hotbag on/off to compare scenarios and see the impact of using insulated delivery bags.

## **8.2 Inputs**

* Hotbag on/off toggle (boolean)

## **8.3 System Behavior**

* When toggled, system recalculates temperature predictions
* System displays comparison between both scenarios
* Results update automatically (FR-13)

## **8.4 UI Components**

* Toggle switch or checkbox: "Use hotbag"
* Visual indicator showing current state

---

# **9. Feature: User Interface & Responsiveness**

## **9.1 Description**

Modern, responsive web interface that works on mobile and desktop devices with accessibility features.

## **9.2 Functional Requirements**

* **FR-13:** All results update automatically when inputs change
* **FR-14:** Provide tooltips explaining assumptions and physics
* **FR-15:** Mobile & desktop responsiveness

## **9.3 Core UI Elements**

* Map selector (interactive map)
* Weather summary card
* Input section for:
  * Date/time selector
  * Handoff delay slider/input
* Results panel with:
  * "With Bag" final temp
  * "Without Bag" final temp
  * Difference display
  * Color-coded gauge or thermometer graphic

## **9.4 System Behavior**

* UI updates reactively to all input changes
* Tooltips appear on hover/focus for explanatory text
* Layout adapts to screen size (mobile/tablet/desktop)
* All interactive elements are keyboard navigable
* Screen reader compatible

## **9.5 Accessibility Features**

* WCAG 2.1 AA compliant
* Keyboard navigation support
* Screen reader compatibility
* Color contrast ratios meet standards
* Alternative text for graphics
* Accessible tooltips and help text

---

# **10. Feature: Caching Layer**

## **10.1 Description**

System caches map routing and weather API results to improve performance and reduce API costs.

## **10.2 System Behavior**

* Weather API results cached for 10 minutes
* Map routing results cached for 10 minutes
* Cache invalidation on explicit refresh or staleness
* Cache lookup before external API call

## **10.3 Performance Impact**

* Cache hits return in < 10 ms
* Reduces external API calls by ~90% for repeated queries

---

# **11. Feature: Error Handling & Fallbacks**

## **11.1 Description**

System gracefully handles failures of external APIs and provides fallback values or user notifications.

## **11.2 System Behavior**

### **Weather API Failure:**
* Attempt to use cached weather data if available
* Use default/fallback weather values (average conditions for location)
* Notify user gracefully about degraded accuracy

### **Map API Failure:**
* Return fallback values (estimated distance based on straight-line calculation)
* Allow manual override of travel time (FR-3)
* Notify user gracefully

### **General Error Handling:**
* All errors logged for diagnostics
* User-friendly error messages displayed
* Application continues to function with degraded features

## **11.3 UI Components**

* Error message banners
* Warning indicators for degraded accuracy
* Retry buttons for failed operations

---

# **12. Feature: Temperature Unit Selection**

## **12.1 Description**

Users can select between Celsius (°C) and Fahrenheit (°F) for all temperature displays.

## **12.2 System Behavior**

* User selects preferred unit (°C or °F)
* All temperatures convert and display in selected unit
* Preference persists across session (localStorage)

## **12.3 UI Components**

* Unit selector dropdown/toggle
* All temperature displays update immediately

---

# **13. Feature: Restaurant Operator Features (Secondary User)**

## **13.1 Description**

Features intended for restaurant operators to demonstrate the value of insulated delivery bags to customers and drivers.

## **13.2 User Stories**

* *As a restaurant operator*, I want to show customers the benefit of using certified insulated bags.
* *As a restaurant operator*, I want to show drivers the benefit of using certified insulated bags.
* *As a restaurant operator*, I want to compare different bag-insulation ratings.
* *As a restaurant operator*, I want to reduce temperature complaints.

## **13.3 Features (Tier 2 / Future Enhancements)**

* Bag comparison mode (cheap vs premium insulation)
* Different hotbag insulation model selection
* White-labeled version for restaurant websites
* Driver-facing interface

---

# **14. Data Model**

## **14.1 Core Entities**

### **DeliveryRequest**
* origin_lat (float)
* origin_lon (float)
* dest_lat (float)
* dest_lon (float)
* datetime_local (datetime)
* handoff_time_seconds (integer)

### **WeatherSample**
* ambient_temp (float)
* wind_speed (float)
* humidity (float, optional)
* precipitation (string, optional)
* timestamp (datetime)

### **ThermalInputs**
* initial_temp (float)
* R_box (float)
* R_bag (float, optional)
* ambient_temp (float)
* wind_speed (float)
* total_time_secs (integer)
* handoff_time_secs (integer)

### **ThermalOutput**
* T_final_box (float)
* T_final_bag (float)
* delta_T (float)
* curve (array, optional)

---

# **15. Acceptance Criteria Summary**

## **15.1 Map Selection**
* ✅ User can select origin and destination by clicking map or searching address
* ✅ Route distance and time are calculated and displayed
* ✅ User can manually override travel time

## **15.2 Weather Integration**
* ✅ Weather data retrieved automatically based on route and datetime
* ✅ Weather reflects route conditions (not just start point)
* ✅ Wind chill factor calculated

## **15.3 Thermal Modeling**
* ✅ Temperature calculated for both with/without bag scenarios
* ✅ Model accounts for conduction, convection, and radiation
* ✅ Environmental factors (wind, rain, temp) affect heat loss
* ✅ Results computed in < 50 ms

## **15.4 UI/UX**
* ✅ All results update automatically when inputs change
* ✅ Tooltips explain assumptions and physics
* ✅ Responsive design works on mobile and desktop
* ✅ WCAG 2.1 AA compliant

## **15.5 Performance**
* ✅ End-to-end calculation completes in < 2 seconds
* ✅ Thermal model executes in < 50 ms

---

**Next Steps:**

* Detailed UI wireframes/mockups
* API endpoint specifications
* Database schema design
* Thermal model physics equations specification
* Test case definitions


# **TECHNICAL APPROACH DOCUMENT**

**Pizza Heat Saver / Pizza Delivery Temperature Calculator**  
**Version:** 0.1  
**Status:** Draft  
**Source:** Implementation strategy based on PRD, TRD, NFR, Feature Spec, and Technical Feasibility

---

# **1. Overview**

This document outlines the technical approach for implementing the Pizza Heat Saver application. It describes architecture decisions, development methodology, integration strategies, and how key technical challenges will be addressed.

---

# **2. Architecture Philosophy**

## **2.1 Core Principles**

* **Separation of Concerns:** Clear boundaries between frontend, backend, and external services
* **Adapter Pattern:** Abstraction layers for external APIs to enable provider switching
* **Serverless-First:** Leverage serverless functions for auto-scaling and cost efficiency
* **Type Safety:** TypeScript throughout for compile-time safety
* **Dependency Injection:** Loose coupling through interfaces and dependency injection
* **Testability:** Design for testability with pure functions and mockable dependencies

## **2.2 Design Patterns**

* **Adapter Pattern:** Map and weather API adapters with common interfaces
* **Repository Pattern:** Data access abstraction for future database extensions
* **Strategy Pattern:** Multiple thermal model implementations (basic vs. advanced)
* **Observer Pattern:** Reactive UI updates when inputs change
* **Factory Pattern:** Service factory for creating adapters with configuration

---

# **3. System Architecture**

## **3.1 High-Level Architecture**

```
┌─────────────────────────────────────────────────────────┐
│                    Client Web Application                │
│  (Next.js - Static Site + Server Components)            │
│                                                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │   Map UI     │  │  Input Form  │  │   Results    │  │
│  │  Component   │  │  Components  │  │   Display    │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
└────────────────────────┬────────────────────────────────┘
                         │
                         │ HTTPS API Calls
                         │
┌────────────────────────▼────────────────────────────────┐
│              Backend API Layer                           │
│  (Netlify Edge Functions / Supabase Functions)          │
│                                                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │   Routing    │  │   Weather    │  │   Thermal    │  │
│  │   Service    │  │   Service    │  │   Model      │  │
│  └──────┬───────┘  └──────┬───────┘  └──────────────┘  │
│         │                  │                            │
│  ┌──────▼───────┐  ┌──────▼───────┐                    │
│  │   Routing    │  │   Weather    │                    │
│  │   Adapter    │  │   Adapter    │                    │
│  └──────┬───────┘  └──────┬───────┘                    │
└─────────┼──────────────────┼────────────────────────────┘
          │                  │
          │                  │
┌─────────▼──────────────────▼────────────────────────────┐
│              External Services                           │
│                                                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │  Map API     │  │  Weather API │  │   Cache      │  │
│  │ (Google/     │  │ (OpenWeather │  │  (In-Memory/ │  │
│  │  Mapbox/     │  │  /Tomorrow)  │  │   Redis)     │  │
│  │  OSM)        │  │              │  │              │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
└──────────────────────────────────────────────────────────┘
```

## **3.2 Monorepo Structure**

```
hotbag/
├── apps/
│   ├── frontend/          # Next.js application
│   └── backend/           # API functions (Netlify/Supabase)
├── packages/
│   ├── api/               # API client libraries
│   ├── store/             # State management (Zustand/Redux)
│   ├── types/             # Shared TypeScript types
│   ├── utils/             # Shared utilities
│   ├── thermal-model/     # Thermal modeling engine
│   ├── map-adapter/       # Map API adapter interface
│   ├── weather-adapter/   # Weather API adapter interface
│   └── shared/            # Shared components/logic
├── tools/                 # Build tools, scripts
└── docs/                  # Documentation
```

---

# **4. Implementation Strategy**

## **4.1 Development Methodology**

### **Approach: Iterative MVP Development**

1. **Phase 1: Core Thermal Model (Week 1-2)**
   - Implement basic Newtonian cooling model
   - Unit tests for thermal calculations
   - Validate performance (< 50ms target)
   - Establish R-value constants (initial estimates)

2. **Phase 2: External API Integration (Week 2-3)**
   - Build map adapter interface + Google Maps implementation
   - Build weather adapter interface + OpenWeatherMap implementation
   - Implement caching layer
   - Integration tests

3. **Phase 3: Backend API (Week 3-4)**
   - API endpoints for routing, weather, thermal calculation
   - Error handling and fallbacks
   - Rate limiting and caching

4. **Phase 4: Frontend UI (Week 4-6)**
   - Map selection component
   - Input forms (date/time, handoff delay)
   - Results display
   - Responsive design

5. **Phase 5: Integration & Testing (Week 6-7)**
   - End-to-end integration
   - Performance testing
   - Accessibility audit
   - User acceptance testing

6. **Phase 6: Calibration & Refinement (Week 7-8)**
   - Model calibration against test data
   - Accuracy validation
   - Performance optimization

## **4.2 Incremental Delivery Strategy**

* **MVP Scope:** Core temperature prediction (with/without bag comparison)
* **Tier 1 Enhancements:** Multiple bag models, temperature curve graphs
* **Tier 2 Enhancements:** Historical weather, advanced accuracy, restaurant features

---

# **5. Core Component Implementation**

## **5.1 Thermal Modeling Engine**

### **Architecture:**
```
thermal-model/
├── src/
│   ├── models/
│   │   ├── NewtonianCooling.ts      # Core ODE solver
│   │   ├── ThermalResistance.ts     # R-value calculations
│   │   └── ConvectionCoefficient.ts # Wind/convection effects
│   ├── calculators/
│   │   ├── TemperatureCalculator.ts # Main calculator
│   │   └── ScenarioCalculator.ts    # With/without bag scenarios
│   ├── types/
│   │   └── ThermalTypes.ts          # Type definitions
│   └── constants/
│       └── MaterialConstants.ts     # R-values, thermal properties
└── tests/
    └── unit/                         # Comprehensive unit tests
```

### **Implementation Approach:**

1. **Pure Functions:** All calculations as pure, testable functions
2. **Numerical Integration:** Euler method for MVP, RK4 as enhancement
3. **Configuration-Driven:** R-values and constants externalized for calibration
4. **Performance:** Optimized for < 50ms execution

### **Key Algorithms:**

```typescript
// Pseudo-code structure
class TemperatureCalculator {
  calculate(
    initialTemp: number,
    ambientTemp: number,
    windSpeed: number,
    totalTime: number,
    insulation: InsulationStack
  ): TemperatureResult {
    // 1. Compute thermal resistance
    const R_total = this.computeTotalResistance(insulation);
    
    // 2. Compute convection coefficient
    const h = this.computeConvection(windSpeed);
    
    // 3. Compute cooling coefficient k(t)
    const k = this.computeCoolingCoefficient(R_total, h);
    
    // 4. Integrate temperature over time
    return this.integrate(initialTemp, ambientTemp, k, totalTime);
  }
}
```

## **5.2 Map Integration**

### **Architecture:**
```
map-adapter/
├── src/
│   ├── interfaces/
│   │   └── IMapAdapter.ts           # Common interface
│   ├── implementations/
│   │   ├── GoogleMapsAdapter.ts
│   │   ├── MapboxAdapter.ts
│   │   └── OSMAdapter.ts
│   └── types/
│       └── MapTypes.ts
```

### **Implementation Approach:**

1. **Interface-Based Design:** Common interface for all providers
2. **Adapter Pattern:** Provider-specific implementations
3. **Factory Pattern:** Create adapter based on configuration
4. **Error Handling:** Graceful fallbacks if provider fails

### **Key Interface:**

```typescript
interface IMapAdapter {
  searchAddress(query: string): Promise<Location[]>;
  getRoute(origin: LatLon, destination: LatLon): Promise<Route>;
  renderMap(container: HTMLElement, options: MapOptions): MapInstance;
}
```

## **5.3 Weather Integration**

### **Architecture:**
```
weather-adapter/
├── src/
│   ├── interfaces/
│   │   └── IWeatherAdapter.ts       # Common interface
│   ├── implementations/
│   │   ├── OpenWeatherAdapter.ts
│   │   ├── TomorrowAdapter.ts
│   │   └── NOAAAdapter.ts
│   └── types/
│       └── WeatherTypes.ts
```

### **Implementation Approach:**

1. **Unified Interface:** Common weather data structure
2. **Multiple Providers:** Support switching providers
3. **Caching:** 10-minute TTL for API results
4. **Fallback:** Use cached or default values on failure

### **Key Interface:**

```typescript
interface IWeatherAdapter {
  getCurrentWeather(coords: LatLon): Promise<WeatherData>;
  getForecast(coords: LatLon, datetime: Date): Promise<WeatherData>;
  getHistorical(coords: LatLon, datetime: Date): Promise<WeatherData>;
}
```

## **5.4 Caching Strategy**

### **Implementation:**

1. **Cache Layer:** In-memory cache for MVP, Redis for production
2. **Cache Keys:** Based on route hash, coordinates + datetime
3. **TTL:** 10 minutes for weather/map data
4. **Invalidation:** Time-based expiration, manual refresh option

### **Cache Structure:**
```typescript
interface CacheEntry<T> {
  data: T;
  timestamp: number;
  ttl: number; // milliseconds
}

class CacheService {
  get<T>(key: string): T | null;
  set<T>(key: string, data: T, ttl: number): void;
  invalidate(key: string): void;
}
```

---

# **6. API Design**

## **6.1 RESTful API Endpoints**

### **Structure:**

```
POST /api/calculate
  Request: {
    origin: { lat, lon },
    destination: { lat, lon },
    datetime: ISO8601,
    handoffDelaySeconds: number,
    useHotbag: boolean
  }
  Response: {
    route: { distance, duration, geometry },
    weather: { temp, windSpeed, ... },
    temperatures: {
      withBag: number,
      withoutBag: number,
      difference: number
    }
  }

GET /api/route
  Query: origin, destination
  Response: Route data

GET /api/weather
  Query: lat, lon, datetime
  Response: Weather data
```

### **Implementation Approach:**

1. **Serverless Functions:** Netlify Functions or Supabase Edge Functions
2. **Type Safety:** Shared types between frontend/backend
3. **Error Handling:** Standardized error responses
4. **Rate Limiting:** Per-user rate limits via Supabase

---

# **7. Frontend Architecture**

## **7.1 Component Structure**

```
frontend/
├── app/                    # Next.js App Router
│   ├── page.tsx           # Main page
│   ├── api/               # API routes (if needed)
│   └── layout.tsx         # Root layout
├── components/
│   ├── map/
│   │   ├── MapSelector.tsx
│   │   └── RouteDisplay.tsx
│   ├── forms/
│   │   ├── DateTimePicker.tsx
│   │   └── HandoffInput.tsx
│   ├── results/
│   │   ├── TemperatureDisplay.tsx
│   │   └── ComparisonChart.tsx
│   └── ui/                # Shadcn components
├── lib/
│   ├── api/               # API client
│   ├── hooks/             # Custom React hooks
│   └── utils/             # Utility functions
└── stores/                # State management (Zustand)
```

## **7.2 State Management**

### **Approach: Zustand (Lightweight State Management)**

```typescript
interface AppState {
  // User inputs
  origin: Location | null;
  destination: Location | null;
  datetime: Date;
  handoffDelay: number;
  useHotbag: boolean;
  
  // Results
  route: Route | null;
  weather: WeatherData | null;
  temperatures: TemperatureResult | null;
  
  // UI state
  loading: boolean;
  error: string | null;
  
  // Actions
  setOrigin: (location: Location) => void;
  setDestination: (location: Location) => void;
  calculate: () => Promise<void>;
}
```

## **7.3 Reactive Updates**

### **Implementation:**

1. **React Query / SWR:** For API data fetching and caching
2. **Debounced Calculations:** Prevent excessive API calls on input changes
3. **Optimistic Updates:** Show loading states immediately

---

# **8. Data Flow**

## **8.1 Request Flow**

```
User Input (Map Selection, DateTime, etc.)
    ↓
Frontend State Update (Zustand)
    ↓
Debounced Calculation Trigger
    ↓
API Request to /api/calculate
    ↓
Backend Processing:
    1. Check cache for route/weather
    2. If cache miss: Fetch from external APIs
    3. Store in cache
    4. Run thermal model calculation
    ↓
Response with Results
    ↓
Frontend State Update
    ↓
UI Re-render with Results
```

## **8.2 Error Handling Flow**

```
API Error (Map/Weather/Thermal)
    ↓
Error Handler in Backend
    ↓
Attempt Fallback:
    - Check cache
    - Use default values
    - Calculate straight-line distance
    ↓
Return Partial Results + Error Flag
    ↓
Frontend Error State
    ↓
Display User-Friendly Error Message
    ↓
Allow User to Retry or Override
```

---

# **9. Testing Strategy**

## **9.1 Unit Testing**

* **Thermal Model:** Comprehensive tests for all calculation functions
* **API Adapters:** Mock external APIs, test adapter logic
* **Utilities:** Test all helper functions
* **Components:** Test React components in isolation

### **Tools:**
* Jest or Vitest for unit tests
* React Testing Library for component tests

## **9.2 Integration Testing**

* **API Integration:** Test API endpoints with mocked external services
* **Adapter Integration:** Test adapters with test API keys
* **End-to-End:** Test full flow from user input to results

### **Tools:**
* Playwright or Cypress for E2E tests
* Supertest for API testing

## **9.3 Performance Testing**

* **Thermal Model:** Benchmark to ensure < 50ms
* **End-to-End:** Load testing to validate < 2s response
* **API Rate Limits:** Test caching effectiveness

### **Tools:**
* k6 or Artillery for load testing
* Lighthouse for frontend performance

---

# **10. Deployment Strategy**

## **10.1 Infrastructure**

### **MVP Deployment:**
* **Frontend:** Netlify (automatic deployments from Git)
* **Backend API:** Netlify Functions or Supabase Edge Functions
* **Database:** Supabase PostgreSQL (for cache, user preferences)
* **Cache:** In-memory (MVP) or Supabase Redis (production)

### **Production Considerations:**
* CDN for static assets
* Edge functions for reduced latency
* Database connection pooling
* Monitoring and logging (Sentry, LogRocket)

## **10.2 CI/CD Pipeline**

```
Git Push → GitHub
    ↓
GitHub Actions / Netlify CI
    ↓
Run Tests (Unit + Integration)
    ↓
Build Application
    ↓
Deploy to Staging
    ↓
Run E2E Tests
    ↓
Manual Approval (if needed)
    ↓
Deploy to Production
```

---

# **11. Model Calibration Approach**

## **11.1 Calibration Framework**

1. **Configuration File:** Externalize R-values and constants
2. **Test Harness:** Framework for running calibration experiments
3. **Data Collection:** Instrument real deliveries with temperature sensors
4. **Iterative Refinement:** Compare predictions vs. actual measurements

## **11.2 Calibration Process**

1. **Initial Estimates:** Use literature values for R_box, R_bag
2. **Lab Testing:** Controlled environment testing
3. **Field Testing:** Real-world validation
4. **Parameter Tuning:** Adjust constants to minimize error
5. **Validation:** Verify ±10°F (MVP) or ±4°F (Tier 2) accuracy

---

# **12. Risk Mitigation**

## **12.1 Technical Risks**

| Risk | Mitigation Strategy |
|------|---------------------|
| Model accuracy insufficient | Start with ±10°F target, build calibration framework early |
| External API failures | Caching, fallbacks, multiple provider support |
| Performance not meeting targets | Profile early, optimize thermal model, use edge computing |
| Scalability issues | Serverless architecture, caching, load testing |

## **12.2 Integration Risks**

| Risk | Mitigation Strategy |
|------|---------------------|
| API provider changes | Adapter pattern, interface abstraction |
| Rate limit issues | Aggressive caching, request queuing |
| Cost overruns | Monitor API usage, optimize caching |

---

# **13. Development Best Practices**

## **13.1 Code Quality**

* **TypeScript:** Strict mode enabled
* **ESLint/Prettier:** Consistent code formatting
* **Conventional Commits:** Standardized commit messages
* **Code Reviews:** All PRs reviewed before merge

## **13.2 Documentation**

* **Code Comments:** JSDoc for public APIs
* **README:** Setup and development instructions
* **Architecture Docs:** System design documentation
* **API Docs:** OpenAPI/Swagger documentation

---

# **14. Success Metrics**

## **14.1 Technical Metrics**

* Thermal model execution: < 50ms ✅
* End-to-end response: < 2 seconds ✅
* Model accuracy: ±10°F (MVP), ±4°F (Tier 2) ⚠️
* Uptime: > 99.5%
* Error rate: < 1%

## **14.2 Development Metrics**

* Test coverage: > 80%
* Code review time: < 24 hours
* Build time: < 5 minutes
* Deployment frequency: Daily (or per feature)

---

# **15. Next Steps**

1. **Proof of Concept:** Build thermal model prototype to validate performance
2. **API Research:** Obtain API keys and test integrations
3. **Architecture Setup:** Initialize monorepo structure
4. **Development Environment:** Set up local development with all tools
5. **First Sprint:** Begin Phase 1 (Core Thermal Model)

---

**This technical approach provides a clear roadmap for implementation while remaining flexible for iterative refinement based on learnings during development.**


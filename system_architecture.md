# **SYSTEM ARCHITECTURE DOCUMENT**

**Pizza Heat Saver / Pizza Delivery Temperature Calculator**  
**Version:** 0.1  
**Status:** Draft  
**Source:** Architecture design based on PRD, TRD, Technical Approach, and Tech Stack

---

# **1. Executive Summary**

This document describes the system architecture for the Pizza Heat Saver application. The architecture follows a serverless, microservices-inspired pattern with clear separation between frontend, backend services, and external integrations.

**Key Architectural Principles:**
- Serverless-first for auto-scaling and cost efficiency
- Adapter pattern for external service integration
- Type-safe API boundaries
- Stateless backend services
- Comprehensive caching strategy

---

# **2. Architecture Overview**

## **2.1 System Layers**

```
┌─────────────────────────────────────────────────────────────────┐
│                         PRESENTATION LAYER                       │
│  Next.js Frontend (Static Site + Client-Side React Components)  │
│  - Map UI Component                                             │
│  - Input Forms                                                  │
│  - Results Display                                              │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             │ HTTPS REST API
                             │
┌────────────────────────────▼────────────────────────────────────┐
│                        APPLICATION LAYER                         │
│  Backend API (Netlify Functions / Supabase Edge Functions)      │
│  - Routing Service                                              │
│  - Weather Service                                              │
│  - Thermal Model Service                                        │
│  - Cache Service                                                │
└──────────────┬──────────────────────┬───────────────────────────┘
               │                      │
               │                      │
┌──────────────▼──────────────────────▼───────────────────────────┐
│                        INTEGRATION LAYER                         │
│  - Map Adapter (Google Maps / Mapbox / OSM)                     │
│  - Weather Adapter (OpenWeather / Tomorrow / NOAA)              │
│  - Cache Adapter (In-Memory / Redis)                            │
└──────────────┬──────────────────────┬───────────────────────────┘
               │                      │
               │                      │
┌──────────────▼──────────────────────▼───────────────────────────┐
│                       EXTERNAL SERVICES                          │
│  - Google Maps Platform                                          │
│  - OpenWeatherMap API                                            │
│  - Supabase (Database / Cache)                                   │
└──────────────────────────────────────────────────────────────────┘
```

## **2.2 Component Overview**

* **Frontend:** Next.js application with React components
* **Backend API:** Serverless functions handling business logic
* **Adapters:** Abstraction layer for external services
* **Cache:** Performance optimization layer
* **Database:** Minimal persistence (user preferences, analytics)

---

# **3. Frontend Architecture**

## **3.1 Application Structure**

```
apps/frontend/
├── app/                          # Next.js App Router
│   ├── page.tsx                 # Main landing page
│   ├── layout.tsx               # Root layout with providers
│   └── api/                     # API routes (if needed)
├── components/
│   ├── map/
│   │   ├── MapSelector.tsx      # Interactive map component
│   │   ├── RouteDisplay.tsx     # Route visualization
│   │   └── AddressSearch.tsx    # Address search input
│   ├── forms/
│   │   ├── DateTimePicker.tsx   # Date/time selection
│   │   └── HandoffInput.tsx     # Handoff delay slider
│   ├── results/
│   │   ├── TemperatureDisplay.tsx  # Temperature results
│   │   ├── ComparisonChart.tsx     # Visual comparison
│   │   └── WeatherCard.tsx         # Weather summary
│   └── ui/                      # Shadcn UI components
├── lib/
│   ├── api/
│   │   └── client.ts            # API client with React Query
│   ├── hooks/
│   │   ├── useTemperatureCalculation.ts
│   │   └── useMapInteraction.ts
│   └── utils/
│       ├── temperature.ts       # Temperature conversions
│       └── validation.ts        # Input validation
├── stores/
│   └── appStore.ts              # Zustand global state
└── types/
    └── index.ts                 # Shared TypeScript types
```

## **3.2 State Management**

### **Global State (Zustand)**
```typescript
interface AppState {
  // User Inputs
  origin: Location | null;
  destination: Location | null;
  datetime: Date;
  handoffDelay: number; // seconds
  useHotbag: boolean;
  temperatureUnit: 'C' | 'F';
  
  // Computed Results
  route: Route | null;
  weather: WeatherData | null;
  temperatures: TemperatureResult | null;
  
  // UI State
  loading: boolean;
  error: Error | null;
  mapReady: boolean;
}
```

### **Server State (React Query)**
- API responses cached automatically
- Automatic refetching on window focus
- Optimistic updates for better UX
- Request deduplication

## **3.3 Data Flow**

```
User Action (Map Click, Input Change)
    ↓
Update Zustand Store
    ↓
Debounce (500ms)
    ↓
Trigger React Query Mutation
    ↓
API Call to Backend
    ↓
Update Zustand Store with Results
    ↓
UI Re-renders with New Data
```

---

# **4. Backend Architecture**

## **4.1 API Service Structure**

```
apps/backend/
├── functions/
│   ├── calculate/
│   │   └── handler.ts          # Main calculation endpoint
│   ├── route/
│   │   └── handler.ts          # Route calculation
│   └── weather/
│       └── handler.ts          # Weather retrieval
├── services/
│   ├── routing/
│   │   └── RoutingService.ts   # Route calculation service
│   ├── weather/
│   │   └── WeatherService.ts   # Weather retrieval service
│   ├── thermal/
│   │   └── ThermalService.ts   # Thermal model service
│   └── cache/
│       └── CacheService.ts     # Caching layer
└── adapters/
    ├── map/
    │   ├── IMapAdapter.ts
    │   └── implementations/
    └── weather/
        ├── IWeatherAdapter.ts
        └── implementations/
```

## **4.2 API Endpoints**

### **POST /api/calculate**
**Purpose:** Complete temperature calculation request

**Request:**
```typescript
{
  origin: { lat: number, lon: number },
  destination: { lat: number, lon: number },
  datetime: string, // ISO8601
  handoffDelaySeconds: number,
  useHotbag: boolean
}
```

**Response:**
```typescript
{
  route: {
    distance: number,
    duration: number, // seconds
    geometry: Polyline,
    origin: Location,
    destination: Location
  },
  weather: {
    ambientTemp: number,
    windSpeed: number,
    windDirection: number,
    humidity?: number,
    precipitation?: string,
    timestamp: string
  },
  temperatures: {
    withBag: number,
    withoutBag: number,
    difference: number,
    initialTemp: number
  },
  metadata: {
    cacheHit: boolean,
    calculationTimeMs: number
  }
}
```

### **GET /api/route**
**Purpose:** Get route information only

**Query Parameters:**
- `originLat`, `originLon`
- `destLat`, `destLon`

**Response:** Route data (same as above)

### **GET /api/weather**
**Purpose:** Get weather data only

**Query Parameters:**
- `lat`, `lon`
- `datetime` (ISO8601)

**Response:** Weather data (same as above)

## **4.3 Service Implementation**

### **Routing Service**
```typescript
class RoutingService {
  constructor(
    private mapAdapter: IMapAdapter,
    private cache: CacheService
  ) {}
  
  async getRoute(origin: LatLon, destination: LatLon): Promise<Route> {
    const cacheKey = this.generateCacheKey(origin, destination);
    
    // Check cache
    const cached = await this.cache.get<Route>(cacheKey);
    if (cached) return cached;
    
    // Fetch from API
    const route = await this.mapAdapter.getRoute(origin, destination);
    
    // Cache result
    await this.cache.set(cacheKey, route, 10 * 60 * 1000); // 10 minutes
    
    return route;
  }
}
```

### **Weather Service**
```typescript
class WeatherService {
  constructor(
    private weatherAdapter: IWeatherAdapter,
    private cache: CacheService
  ) {}
  
  async getWeather(
    coords: LatLon,
    datetime: Date
  ): Promise<WeatherData> {
    const cacheKey = this.generateCacheKey(coords, datetime);
    
    // Check cache
    const cached = await this.cache.get<WeatherData>(cacheKey);
    if (cached) return cached;
    
    // Determine weather type
    const now = new Date();
    let weather: WeatherData;
    
    if (datetime < now) {
      weather = await this.weatherAdapter.getHistorical(coords, datetime);
    } else if (datetime.getTime() - now.getTime() < 3600000) { // 1 hour
      weather = await this.weatherAdapter.getCurrentWeather(coords);
    } else {
      weather = await this.weatherAdapter.getForecast(coords, datetime);
    }
    
    // Cache result
    await this.cache.set(cacheKey, weather, 10 * 60 * 1000);
    
    return weather;
  }
}
```

### **Thermal Service**
```typescript
class ThermalService {
  constructor(
    private thermalModel: ThermalModel
  ) {}
  
  async calculate(
    inputs: ThermalInputs
  ): Promise<TemperatureResult> {
    const startTime = Date.now();
    
    // Calculate both scenarios
    const withBag = this.thermalModel.calculate({
      ...inputs,
      insulation: { box: true, bag: true }
    });
    
    const withoutBag = this.thermalModel.calculate({
      ...inputs,
      insulation: { box: true, bag: false }
    });
    
    const calculationTime = Date.now() - startTime;
    
    return {
      withBag,
      withoutBag,
      difference: withBag - withoutBag,
      calculationTimeMs: calculationTime
    };
  }
}
```

---

# **5. Adapter Layer**

## **5.1 Map Adapter Interface**

```typescript
interface IMapAdapter {
  searchAddress(query: string): Promise<Location[]>;
  getRoute(origin: LatLon, destination: LatLon): Promise<Route>;
  renderMap(container: HTMLElement, options: MapOptions): MapInstance;
}

interface Route {
  distance: number; // meters
  duration: number; // seconds
  geometry: Polyline;
  steps?: RouteStep[];
}
```

### **Implementation: Google Maps Adapter**
```typescript
class GoogleMapsAdapter implements IMapAdapter {
  constructor(private apiKey: string) {}
  
  async getRoute(origin: LatLon, destination: LatLon): Promise<Route> {
    const response = await fetch(
      `https://maps.googleapis.com/maps/api/directions/json?` +
      `origin=${origin.lat},${origin.lon}&` +
      `destination=${destination.lat},${destination.lon}&` +
      `key=${this.apiKey}`
    );
    
    const data = await response.json();
    
    return this.transformToRoute(data);
  }
}
```

## **5.2 Weather Adapter Interface**

```typescript
interface IWeatherAdapter {
  getCurrentWeather(coords: LatLon): Promise<WeatherData>;
  getForecast(coords: LatLon, datetime: Date): Promise<WeatherData>;
  getHistorical(coords: LatLon, datetime: Date): Promise<WeatherData>;
}

interface WeatherData {
  ambientTemp: number; // Celsius
  windSpeed: number; // m/s
  windDirection: number; // degrees
  humidity?: number; // 0-100
  precipitation?: string;
  timestamp: Date;
}
```

---

# **6. Thermal Model Architecture**

## **6.1 Model Structure**

```
packages/thermal-model/
├── src/
│   ├── models/
│   │   ├── NewtonianCooling.ts      # ODE solver
│   │   ├── ThermalResistance.ts     # R-value calculations
│   │   └── ConvectionCoefficient.ts # Wind effects
│   ├── calculators/
│   │   ├── TemperatureCalculator.ts # Main calculator
│   │   └── ScenarioCalculator.ts    # Scenario comparison
│   ├── types/
│   │   └── ThermalTypes.ts
│   └── constants/
│       └── MaterialConstants.ts     # R-values, defaults
└── tests/
    └── unit/
```

## **6.2 Calculation Flow**

```
Input: ThermalInputs
    ↓
1. Compute Total Thermal Resistance (R_total)
   - R_pizza (pizza thermal properties)
   - R_box (cardboard box)
   - R_bag (optional hotbag)
    ↓
2. Compute Convection Coefficient (h)
   - Base coefficient (h_0)
   - Wind speed factor (c_1 × v_wind)
    ↓
3. Compute Cooling Coefficient (k)
   - k = 1 / (R_total × C_pizza)
   - Adjusted for convection
    ↓
4. Numerical Integration (Euler/RK4)
   - Time step: 1 second
   - Iterate for total_time_seconds
   - dT/dt = -k(t) × (T - T_amb)
    ↓
Output: Final Temperature
```

## **6.3 Model Implementation**

```typescript
class TemperatureCalculator {
  calculate(inputs: ThermalInputs): number {
    let temperature = inputs.initialTemp;
    const timeStep = 1; // seconds
    const totalSteps = Math.floor(inputs.totalTimeSecs / timeStep);
    
    for (let step = 0; step < totalSteps; step++) {
      const t = step * timeStep;
      
      // Compute thermal resistance at time t
      const R_total = this.computeTotalResistance(
        inputs.insulation,
        inputs.ambientTemp,
        t
      );
      
      // Compute convection coefficient
      const h = this.computeConvection(
        inputs.windSpeed,
        inputs.ambientTemp
      );
      
      // Compute cooling coefficient
      const k = this.computeCoolingCoefficient(R_total, h, inputs);
      
      // Euler step
      const dT = -k * (temperature - inputs.ambientTemp) * timeStep;
      temperature += dT;
    }
    
    return temperature;
  }
}
```

---

# **7. Caching Architecture**

## **7.1 Cache Strategy**

### **Cache Layers:**
1. **Browser Cache:** Static assets, React Query cache
2. **CDN Cache:** Static Next.js pages
3. **Server Cache:** API responses (in-memory or Redis)

### **Cache Keys:**
```
Route: route:{originLat}:{originLon}:{destLat}:{destLon}
Weather: weather:{lat}:{lon}:{datetime}
```

### **Cache TTL:**
- Route data: 10 minutes
- Weather data: 10 minutes
- Static assets: Long-term (CDN)

## **7.2 Cache Implementation**

```typescript
class CacheService {
  private cache: Map<string, CacheEntry>;
  
  async get<T>(key: string): Promise<T | null> {
    const entry = this.cache.get(key);
    if (!entry) return null;
    
    // Check expiration
    if (Date.now() > entry.expiresAt) {
      this.cache.delete(key);
      return null;
    }
    
    return entry.data as T;
  }
  
  async set<T>(key: string, data: T, ttlMs: number): Promise<void> {
    this.cache.set(key, {
      data,
      expiresAt: Date.now() + ttlMs
    });
  }
}
```

---

# **8. Data Architecture**

## **8.1 Data Flow**

```
User Input
    ↓
Frontend Validation
    ↓
API Request
    ↓
Backend Processing
    ├── Check Cache
    ├── Fetch External APIs (if cache miss)
    ├── Store in Cache
    └── Run Thermal Model
    ↓
Response
    ↓
Frontend Display
```

## **8.2 Data Models**

### **DeliveryRequest**
```typescript
interface DeliveryRequest {
  origin: Location;
  destination: Location;
  datetime: Date;
  handoffDelaySeconds: number;
  useHotbag: boolean;
}
```

### **Route**
```typescript
interface Route {
  distance: number; // meters
  duration: number; // seconds
  geometry: Polyline;
  origin: Location;
  destination: Location;
}
```

### **WeatherData**
```typescript
interface WeatherData {
  ambientTemp: number;
  windSpeed: number;
  windDirection: number;
  humidity?: number;
  precipitation?: string;
  timestamp: Date;
}
```

### **ThermalInputs**
```typescript
interface ThermalInputs {
  initialTemp: number;
  ambientTemp: number;
  windSpeed: number;
  totalTimeSecs: number;
  handoffTimeSecs: number;
  insulation: {
    box: boolean;
    bag: boolean;
  };
}
```

### **TemperatureResult**
```typescript
interface TemperatureResult {
  withBag: number;
  withoutBag: number;
  difference: number;
  calculationTimeMs: number;
}
```

---

# **9. Deployment Architecture**

## **9.1 Infrastructure Components**

```
┌─────────────────────────────────────────────┐
│            CDN / Edge Network                │
│  (Netlify CDN for static assets)            │
└────────────────┬────────────────────────────┘
                 │
┌────────────────▼────────────────────────────┐
│          Frontend Hosting                    │
│  (Netlify Static Site Hosting)              │
│  - Next.js static export                    │
│  - Automatic SSL                            │
└────────────────┬────────────────────────────┘
                 │
                 │ HTTPS
                 │
┌────────────────▼────────────────────────────┐
│        Serverless Functions                  │
│  (Netlify Functions / Supabase Edge)        │
│  - Auto-scaling                             │
│  - Edge locations                           │
└──────┬───────────────┬──────────────────────┘
       │               │
       │               │
┌──────▼──────┐  ┌────▼──────────────────────┐
│  External    │  │   Database                │
│  APIs        │  │   (Supabase PostgreSQL)   │
│  - Maps      │  │   - Minimal data          │
│  - Weather   │  │   - Cache (optional)      │
└─────────────┘  └───────────────────────────┘
```

## **9.2 Deployment Flow**

```
Git Push to Main Branch
    ↓
GitHub Actions Triggered
    ↓
Run Tests (Unit + Integration)
    ↓
Build Frontend (Next.js)
    ↓
Build Backend Functions
    ↓
Deploy to Netlify
    ├── Deploy Static Assets
    └── Deploy Serverless Functions
    ↓
Health Checks
    ↓
Production Live
```

---

# **10. Security Architecture**

## **10.1 Security Layers**

1. **HTTPS/TLS:** All traffic encrypted
2. **API Keys:** Server-side only, environment variables
3. **Input Validation:** Zod schemas on all inputs
4. **Rate Limiting:** Per-user rate limits
5. **CORS:** Configured for specific origins

## **10.2 API Key Management**

```
Environment Variables (Netlify)
    ↓
Serverless Functions Only
    ↓
Never Exposed to Client
    ↓
Automatic Rotation Support
```

---

# **11. Monitoring & Observability**

## **11.1 Logging**

- **Frontend:** Console errors, Sentry integration
- **Backend:** Structured logging to console/cloud
- **API Calls:** Request/response logging
- **Errors:** Error tracking with Sentry

## **11.2 Metrics**

- **Performance:** Response times, thermal model execution time
- **Usage:** Request counts, cache hit rates
- **Errors:** Error rates, error types
- **Costs:** API usage, function invocations

## **11.3 Alerts**

- High error rates
- Performance degradation
- API failures
- Cost threshold exceeded

---

# **12. Scalability Considerations**

## **12.1 Horizontal Scaling**

- **Frontend:** CDN handles unlimited static traffic
- **Backend:** Serverless functions auto-scale
- **Database:** Supabase auto-scales connections

## **12.2 Caching Strategy**

- Aggressive caching reduces external API calls ~90%
- In-memory cache for MVP
- Redis for production scale

## **12.3 Performance Optimization**

- Edge functions reduce latency
- Parallel API calls (map + weather)
- Optimized thermal model (< 50ms)
- Code splitting in frontend

---

# **13. Architecture Diagrams**

## **13.1 Component Diagram**

```
[Frontend] ──HTTPS──> [API Gateway]
                          │
                          ├──> [Routing Service] ──> [Map Adapter] ──> [Google Maps]
                          │
                          ├──> [Weather Service] ──> [Weather Adapter] ──> [OpenWeather]
                          │
                          └──> [Thermal Service] ──> [Thermal Model]
                                  │
                                  └──> [Cache Service]
```

## **13.2 Sequence Diagram**

```
User    Frontend    API        Routing    Weather    Thermal    Cache    External
 │         │         │          Service    Service    Service            APIs
 │         │         │             │          │          │         │         │
 │──Input──>│         │             │          │          │         │         │
 │         │──Request─>│             │          │          │         │         │
 │         │         │──getRoute()──>│          │          │         │         │
 │         │         │             │──checkCache──>│          │         │         │
 │         │         │             │<──miss─────│          │         │         │
 │         │         │             │──fetch──────────────────────────>│         │
 │         │         │             │<──route───────────────────────────│         │
 │         │         │             │──cache───────>│          │         │         │
 │         │         │<──route──────│          │          │         │         │
 │         │         │──getWeather()────────>│          │         │         │
 │         │         │                       │──checkCache─────────────>│         │
 │         │         │                       │<──miss───────────────────│         │
 │         │         │                       │──fetch──────────────────────────────────>│
 │         │         │                       │<──weather──────────────────────────────────│
 │         │         │                       │──cache───────>│          │         │
 │         │         │<──weather─────────────│          │         │         │
 │         │         │──calculate()──────────────────────────>│          │         │
 │         │         │                                        │──compute──>│         │
 │         │         │<──temperatures─────────────────────────│          │         │
 │         │<──Response───────────────────────────────────────────────────────│         │
 │<──Results─────────────────────────────────────────────────────────────────────────────│
```

---

# **14. Technology Decisions**

## **14.1 Key Technologies**

| Component | Technology | Rationale |
|-----------|-----------|-----------|
| Frontend Framework | Next.js 14+ | SSR, static export, excellent DX |
| UI Library | Shadcn/ui | Accessible, customizable |
| State Management | Zustand + React Query | Lightweight, server state handling |
| Backend Runtime | Node.js | TypeScript compatibility, ecosystem |
| Deployment | Netlify | Integrated, auto-scaling |
| Database | Supabase (PostgreSQL) | Managed, generous free tier |
| Maps | Google Maps | Mature, reliable |
| Weather | OpenWeatherMap | Good free tier, global coverage |

## **14.2 Design Patterns**

- **Adapter Pattern:** External service integration
- **Repository Pattern:** Data access abstraction
- **Factory Pattern:** Service instantiation
- **Strategy Pattern:** Multiple model implementations
- **Observer Pattern:** Reactive UI updates

---

# **15. Future Architecture Considerations**

## **15.1 Potential Enhancements**

- **Real-time Updates:** WebSocket support for live temperature tracking
- **GraphQL API:** More flexible data fetching
- **Microservices:** Separate services for scaling
- **Event Sourcing:** Audit trail and replay capabilities
- **Machine Learning:** Improve model accuracy with ML

## **15.2 Migration Paths**

- In-memory cache → Redis
- Single adapter → Multi-adapter with failover
- Basic thermal model → Advanced ML model
- Serverless functions → Container-based if needed

---

**This architecture provides a solid foundation that can scale from MVP to production while maintaining simplicity and cost-effectiveness.**

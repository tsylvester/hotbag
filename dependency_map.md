# **DEPENDENCY MAP**

**Pizza Heat Saver / Pizza Delivery Temperature Calculator**  
**Version:** 0.1  
**Status:** Draft  
**Source:** Dependency analysis based on System Architecture, Technical Approach, and Tech Stack

---

# **1. Overview**

This document maps all dependencies within the Pizza Heat Saver system, including:
- Package dependencies (npm packages)
- Module dependencies (internal code modules)
- Service dependencies (external services)
- Data flow dependencies
- Infrastructure dependencies

---

# **2. Package Dependencies**

## **2.1 Frontend Package Dependencies**

```
apps/frontend/
├── dependencies:
│   ├── next (^14.0.0)
│   ├── react (^18.0.0)
│   ├── react-dom (^18.0.0)
│   ├── typescript (^5.0.0)
│   │
│   ├── UI & Styling:
│   │   ├── @radix-ui/react-* (via shadcn)
│   │   ├── tailwindcss (^3.0.0)
│   │   ├── clsx (^2.0.0)
│   │   └── class-variance-authority (^0.7.0)
│   │
│   ├── State Management:
│   │   ├── zustand (^4.0.0)
│   │   └── @tanstack/react-query (^5.0.0)
│   │
│   ├── Maps:
│   │   └── @react-google-maps/api (^2.0.0)
│   │
│   ├── Forms:
│   │   ├── react-hook-form (^7.0.0)
│   │   └── zod (^3.0.0)
│   │
│   ├── Date/Time:
│   │   └── date-fns (^2.0.0)
│   │
│   ├── Charts (Tier 2):
│   │   └── recharts (^2.0.0)
│   │
│   └── Icons:
│       └── react-icons (^4.0.0)
│
└── devDependencies:
    ├── @types/react (^18.0.0)
    ├── @types/node (^20.0.0)
    ├── eslint (^8.0.0)
    ├── prettier (^3.0.0)
    ├── vitest (^1.0.0)
    └── @testing-library/react (^14.0.0)
```

## **2.2 Backend Package Dependencies**

```
apps/backend/
├── dependencies:
│   ├── @netlify/functions (^2.0.0)
│   ├── typescript (^5.0.0)
│   │
│   ├── HTTP Client:
│   │   └── (native fetch in Node.js 18+)
│   │
│   ├── Caching:
│   │   └── lru-cache (^10.0.0) [MVP]
│   │   └── ioredis (^5.0.0) [Production]
│   │
│   ├── Validation:
│   │   └── zod (^3.0.0)
│   │
│   └── Logging:
│       └── pino (^8.0.0)
│
└── devDependencies:
    ├── @types/node (^20.0.0)
    ├── typescript (^5.0.0)
    └── vitest (^1.0.0)
```

## **2.3 Shared Package Dependencies**

```
packages/
├── types/
│   └── dependencies:
│       └── (none - pure TypeScript types)
│
├── utils/
│   └── dependencies:
│       ├── date-fns (^2.0.0)
│       └── zod (^3.0.0)
│
├── thermal-model/
│   └── dependencies:
│       └── (none - pure TypeScript calculations)
│
├── map-adapter/
│   └── dependencies:
│       ├── @react-google-maps/api (^2.0.0) [Frontend]
│       └── (native fetch for backend)
│
└── weather-adapter/
    └── dependencies:
        └── (native fetch)
```

---

# **3. Module Dependencies**

## **3.1 Frontend Module Dependencies**

```
apps/frontend/
│
├── app/page.tsx
│   ├── depends on: components/, stores/, lib/api/
│   └── uses: React, Next.js
│
├── components/map/MapSelector.tsx
│   ├── depends on: @react-google-maps/api
│   ├── depends on: stores/appStore
│   └── uses: React hooks
│
├── components/forms/DateTimePicker.tsx
│   ├── depends on: components/ui/ (Shadcn)
│   ├── depends on: date-fns
│   └── uses: react-hook-form, zod
│
├── components/results/TemperatureDisplay.tsx
│   ├── depends on: stores/appStore
│   ├── depends on: lib/utils/temperature
│   └── uses: React
│
├── lib/api/client.ts
│   ├── depends on: @tanstack/react-query
│   ├── depends on: packages/types
│   └── uses: fetch API
│
├── lib/hooks/useTemperatureCalculation.ts
│   ├── depends on: lib/api/client
│   ├── depends on: stores/appStore
│   └── uses: React Query
│
└── stores/appStore.ts
    ├── depends on: zustand
    ├── depends on: packages/types
    └── uses: (pure state management)
```

## **3.2 Backend Module Dependencies**

```
apps/backend/
│
├── functions/calculate/handler.ts
│   ├── depends on: services/RoutingService
│   ├── depends on: services/WeatherService
│   ├── depends on: services/ThermalService
│   └── uses: @netlify/functions
│
├── services/RoutingService.ts
│   ├── depends on: adapters/map/IMapAdapter
│   ├── depends on: services/CacheService
│   └── uses: packages/types
│
├── services/WeatherService.ts
│   ├── depends on: adapters/weather/IWeatherAdapter
│   ├── depends on: services/CacheService
│   └── uses: packages/types
│
├── services/ThermalService.ts
│   ├── depends on: packages/thermal-model
│   └── uses: packages/types
│
├── services/CacheService.ts
│   ├── depends on: lru-cache [MVP]
│   └── depends on: ioredis [Production]
│
└── adapters/
    ├── map/GoogleMapsAdapter.ts
    │   ├── depends on: adapters/map/IMapAdapter
    │   └── uses: fetch API
    │
    └── weather/OpenWeatherAdapter.ts
        ├── depends on: adapters/weather/IWeatherAdapter
        └── uses: fetch API
```

## **3.3 Shared Package Module Dependencies**

```
packages/
│
├── types/
│   └── index.ts
│       └── (no dependencies - base types)
│
├── utils/
│   ├── temperature.ts
│   │   └── depends on: packages/types
│   │
│   └── validation.ts
│       ├── depends on: packages/types
│       └── depends on: zod
│
├── thermal-model/
│   ├── models/NewtonianCooling.ts
│   │   └── depends on: packages/types
│   │
│   ├── models/ThermalResistance.ts
│   │   └── depends on: packages/types
│   │
│   ├── calculators/TemperatureCalculator.ts
│   │   ├── depends on: models/NewtonianCooling
│   │   ├── depends on: models/ThermalResistance
│   │   ├── depends on: constants/MaterialConstants
│   │   └── depends on: packages/types
│   │
│   └── constants/MaterialConstants.ts
│       └── (no dependencies)
│
├── map-adapter/
│   ├── interfaces/IMapAdapter.ts
│   │   └── depends on: packages/types
│   │
│   └── implementations/GoogleMapsAdapter.ts
│       ├── depends on: interfaces/IMapAdapter
│       └── uses: fetch API
│
└── weather-adapter/
    ├── interfaces/IWeatherAdapter.ts
    │   └── depends on: packages/types
    │
    └── implementations/OpenWeatherAdapter.ts
        ├── depends on: interfaces/IWeatherAdapter
        └── uses: fetch API
```

---

# **4. Service Dependencies**

## **4.1 External Service Dependencies**

```
System
│
├── Google Maps Platform
│   ├── Used by: apps/backend/adapters/map/GoogleMapsAdapter
│   ├── APIs: Directions API, Geocoding API, Places API
│   ├── Dependency Type: External API
│   └── Risk Level: Medium (adapter pattern mitigates)
│
├── OpenWeatherMap API
│   ├── Used by: apps/backend/adapters/weather/OpenWeatherAdapter
│   ├── APIs: One Call API 3.0
│   ├── Dependency Type: External API
│   └── Risk Level: Medium (adapter pattern mitigates)
│
├── Supabase (PostgreSQL)
│   ├── Used by: apps/backend/services (optional)
│   ├── Features: Database, Cache storage (optional)
│   ├── Dependency Type: Managed Service
│   └── Risk Level: Low (optional, minimal usage)
│
├── Netlify
│   ├── Used by: Entire application
│   ├── Features: Hosting, Functions, CDN
│   ├── Dependency Type: Infrastructure
│   └── Risk Level: Low (standard hosting)
│
└── Sentry (Monitoring)
    ├── Used by: Frontend & Backend
    ├── Features: Error tracking, performance monitoring
    ├── Dependency Type: Monitoring Service
    └── Risk Level: Low (optional, non-critical)
```

## **4.2 Internal Service Dependencies**

```
Frontend Application
    │
    └── depends on ──> Backend API
            │
            ├── depends on ──> Routing Service
            │       │
            │       └── depends on ──> Map Adapter
            │               │
            │               └── depends on ──> Google Maps API
            │
            ├── depends on ──> Weather Service
            │       │
            │       └── depends on ──> Weather Adapter
            │               │
            │               └── depends on ──> OpenWeatherMap API
            │
            ├── depends on ──> Thermal Service
            │       │
            │       └── depends on ──> Thermal Model Package
            │
            └── depends on ──> Cache Service
                    │
                    └── depends on ──> In-Memory Cache / Redis
```

---

# **5. Data Flow Dependencies**

## **5.1 Request Flow Dependencies**

```
User Input
    │
    ├── Frontend Component
    │   │
    │   ├── depends on: Zustand Store
    │   │   └── triggers: React Query Mutation
    │   │
    │   └── depends on: API Client
    │       │
    │       └── HTTP Request ──> Backend API
    │
    └── Backend API Handler
        │
        ├── depends on: Routing Service
        │   │
        │   ├── checks: Cache Service
        │   │
        │   └── depends on: Map Adapter (if cache miss)
        │       │
        │       └── HTTP Request ──> Google Maps API
        │
        ├── depends on: Weather Service
        │   │
        │   ├── checks: Cache Service
        │   │
        │   └── depends on: Weather Adapter (if cache miss)
        │       │
        │       └── HTTP Request ──> OpenWeatherMap API
        │
        └── depends on: Thermal Service
            │
            └── depends on: Thermal Model
                │
                └── calculates: Temperature Result
                    │
                    └── returns: Response to Frontend
```

## **5.2 State Dependencies**

```
Frontend State (Zustand)
    │
    ├── User Inputs
    │   ├── origin ──> used by: Map Component, API Request
    │   ├── destination ──> used by: Map Component, API Request
    │   ├── datetime ──> used by: DatePicker, API Request
    │   ├── handoffDelay ──> used by: Slider, API Request
    │   └── useHotbag ──> used by: Toggle, API Request
    │
    ├── Results
    │   ├── route ──> used by: Map Display, Results Panel
    │   ├── weather ──> used by: Weather Card, Thermal Calculation
    │   └── temperatures ──> used by: Results Panel, Charts
    │
    └── UI State
        ├── loading ──> used by: Loading Indicators
        └── error ──> used by: Error Messages
```

---

# **6. Infrastructure Dependencies**

## **6.1 Deployment Dependencies**

```
Git Repository (GitHub)
    │
    └── triggers: CI/CD Pipeline
        │
        ├── GitHub Actions
        │   │
        │   ├── depends on: Node.js 18+
        │   ├── depends on: pnpm/npm
        │   ├── runs: Tests (Vitest)
        │   ├── runs: Build (Next.js, TypeScript)
        │   └── deploys: Netlify
        │
        └── Netlify Deployment
            │
            ├── builds: Frontend (Next.js)
            │   └── deploys: CDN
            │
            ├── builds: Backend Functions
            │   └── deploys: Serverless Runtime
            │
            └── configures: Environment Variables
                │
                ├── API Keys (Google Maps, OpenWeather)
                ├── Database URL (Supabase)
                └── Monitoring Keys (Sentry)
```

## **6.2 Runtime Dependencies**

```
Production Runtime
    │
    ├── Frontend
    │   ├── depends on: CDN (Netlify)
    │   ├── depends on: Browser (Chrome, Firefox, Safari, Edge)
    │   └── depends on: Internet Connection
    │
    ├── Backend API
    │   ├── depends on: Serverless Runtime (Netlify Functions)
    │   │   └── depends on: Node.js 18+
    │   │
    │   └── depends on: External APIs
    │       ├── Google Maps API
    │       └── OpenWeatherMap API
    │
    └── Database (Optional)
        └── depends on: Supabase Infrastructure
```

---

# **7. Dependency Graph**

## **7.1 Package Dependency Graph**

```
┌─────────────────────────────────────────────────────┐
│                    Frontend App                      │
│  (apps/frontend)                                     │
│                                                      │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐         │
│  │  Next.js │  │  React   │  │ Zustand  │         │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘         │
│       │             │              │                │
│       └─────────────┴──────────────┘                │
│                      │                               │
│       ┌──────────────▼──────────────┐               │
│       │   Shared Packages           │               │
│       │  ┌──────┐  ┌──────┐        │               │
│       │  │Types │  │Utils │        │               │
│       │  └──────┘  └──────┘        │               │
│       └──────────────┬──────────────┘               │
└──────────────────────┼──────────────────────────────┘
                       │
                       │ HTTP API
                       │
┌──────────────────────▼──────────────────────────────┐
│                   Backend API                        │
│  (apps/backend)                                      │
│                                                      │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐         │
│  │Routing   │  │ Weather  │  │ Thermal  │         │
│  │Service   │  │ Service  │  │ Service  │         │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘         │
│       │             │              │                │
│  ┌────▼─────┐  ┌────▼─────┐  ┌────▼─────┐         │
│  │  Map     │  │ Weather  │  │ Thermal  │         │
│  │ Adapter  │  │ Adapter  │  │  Model   │         │
│  └────┬─────┘  └────┬─────┘  └──────────┘         │
│       │             │                               │
└───────┼─────────────┼───────────────────────────────┘
        │             │
        │             │
┌───────▼─────────────▼───────────────┐
│      External Services              │
│  ┌──────────┐  ┌──────────┐        │
│  │  Google  │  │OpenWeather│       │
│  │  Maps    │  │   Map     │       │
│  └──────────┘  └──────────┘        │
└──────────────────────────────────────┘
```

## **7.2 Module Dependency Graph**

```
Frontend Components
    │
    ├── MapSelector ──> Map Adapter Package ──> Google Maps API
    │
    ├── DateTimePicker ──> date-fns
    │
    ├── TemperatureDisplay ──> Utils Package (temperature conversion)
    │
    └── All Components ──> Store (Zustand) ──> Types Package
            │
            └── API Client ──> Backend API
                    │
                    └── Backend Services
                            │
                            ├── Routing Service ──> Map Adapter ──> External API
                            │
                            ├── Weather Service ──> Weather Adapter ──> External API
                            │
                            ├── Thermal Service ──> Thermal Model Package
                            │
                            └── Cache Service ──> LRU Cache / Redis
```

---

# **8. Critical Dependencies**

## **8.1 Single Points of Failure**

| Dependency | Type | Risk | Mitigation |
|------------|------|------|------------|
| Google Maps API | External Service | Medium | Adapter pattern enables switching to Mapbox/OSM |
| OpenWeatherMap API | External Service | Medium | Adapter pattern enables switching to Tomorrow.io/NOAA |
| Netlify Hosting | Infrastructure | Low | Standard hosting, can migrate to Vercel |
| Thermal Model | Internal Package | High | Well-tested, pure functions, minimal dependencies |

## **8.2 Dependency Chain Risks**

### **High-Risk Chains:**
```
User Request
    └──> Backend API
        └──> External API (Google Maps / OpenWeather)
            └──> [FAILURE POINT]
```

**Mitigation:** Caching layer breaks the dependency chain, fallback mechanisms

### **Medium-Risk Chains:**
```
Frontend
    └──> Backend API
        └──> Thermal Model
            └──> [CALCULATION DEPENDENCY]
```

**Mitigation:** Well-tested, pure functions, minimal external dependencies

---

# **9. Dependency Management Strategy**

## **9.1 Version Management**

- **Lock Files:** `package-lock.json` or `pnpm-lock.yaml` committed to Git
- **Version Pinning:** Pin major versions, allow patch updates
- **Regular Updates:** Dependabot for automated dependency updates
- **Security:** `npm audit` for vulnerability scanning

## **9.2 Breaking Changes**

- **Testing:** Comprehensive test suite catches breaking changes
- **Staged Updates:** Test in development, then staging, then production
- **Rollback Plan:** Git-based rollback for dependency issues

## **9.3 External Service Changes**

- **Adapter Pattern:** Enables provider switching
- **Monitoring:** Alert on API changes/deprecations
- **Versioning:** API versioning where supported
- **Documentation:** Track API change logs

---

# **10. Dependency Metrics**

## **10.1 Package Metrics**

| Metric | Value | Target |
|--------|-------|--------|
| Total Dependencies | ~50-70 packages | < 100 |
| Direct Dependencies | ~20-30 packages | < 40 |
| Dev Dependencies | ~15-20 packages | < 30 |
| Bundle Size (Frontend) | TBD | < 500KB (gzipped) |
| Tree-Shakeable | Yes | All packages |

## **10.2 Service Dependency Metrics**

| Metric | Value | Target |
|--------|-------|--------|
| External APIs | 2 (Maps, Weather) | < 3 |
| Infrastructure Services | 2 (Netlify, Supabase) | < 3 |
| Single Points of Failure | 0 (with adapters) | 0 |
| Average Dependency Depth | 2-3 levels | < 4 |

---

# **11. Dependency Documentation**

## **11.1 Dependency Rationale**

Each major dependency should document:
- **Why:** Reason for inclusion
- **Alternatives:** Other options considered
- **Trade-offs:** Benefits and drawbacks
- **Migration Path:** How to switch if needed

## **11.2 License Tracking**

- Track all package licenses
- Ensure license compatibility
- Document any GPL/LGPL dependencies
- Review for commercial use compliance

---

# **12. Dependency Map Summary**

## **12.1 Key Dependencies**

**Frontend:**
- Next.js (framework)
- React (UI library)
- Zustand + React Query (state)
- Shadcn/ui (components)

**Backend:**
- Netlify Functions (runtime)
- Thermal Model Package (core logic)
- Map/Weather Adapters (integrations)

**External:**
- Google Maps (routing)
- OpenWeatherMap (weather)

**Infrastructure:**
- Netlify (hosting)
- Supabase (database, optional)

## **12.2 Dependency Health**

✅ **Well-Managed:**
- Adapter pattern for external services
- Minimal external dependencies
- Type-safe boundaries
- Comprehensive testing

✅ **Low Risk:**
- Standard, well-maintained packages
- Mature external APIs
- Managed infrastructure

⚠️ **Monitor:**
- External API changes
- Package security vulnerabilities
- Version compatibility

---

**This dependency map should be updated regularly as the system evolves and new dependencies are added or removed.**

# Deepenk — Architecture Diagrams
> AI Decision Engine | All Mermaid Diagrams

---

## 1. Sequence Diagram — Intent to Checkout

```mermaid
sequenceDiagram
    participant U as User
    participant FE as Frontend (React.js / RN)
    participant BE as Express Server
    participant Auth as Auth Middleware (JWT)
    participant SE as SearchEngine Service
    participant AI as AIService (GPT-4)
    participant PS as ProductService
    participant Cache as NodeCache (Redis)
    participant MONG as MongoDB
    participant EXT as External APIs (Amazon / Flipkart / Swiggy / Uber)

    U->>FE: Submit query ("Gaming laptop under ₹70k")
    FE->>BE: POST /api/search { query }
    BE->>Auth: Verify JWT token (optional)
    Auth-->>BE: userId (or null for anon)
    BE->>SE: SearchEngine.search(query, userId)
    SE->>SE: NLP Intent Extraction (category, budget, features, domain)
    SE->>Cache: Check NodeCache (TTL 300s)
    alt Cache HIT
        Cache-->>SE: Return cached results
    else Cache MISS
        SE->>PS: searchAmazon(query, filters)
        SE->>PS: searchFlipkart(query, filters)
        SE->>PS: searchMyntra(query, filters)
        PS->>EXT: Parallel RapidAPI calls
        EXT-->>PS: Raw product data
        PS-->>SE: Parsed & normalized products
        SE->>Cache: Store results (TTL 300s)
    end
    SE->>AI: generateProductSummary(query, products)
    AI->>AI: GPT-4 analysis (best deal, value, considerations)
    AI-->>SE: JSON { bestOverall, bestValue, recommendations }
    SE->>AI: generateComparisonInsights(products)
    AI-->>SE: Comparison insights
    SE-->>BE: Final searchResult object
    BE->>MONG: Search.create({ userId, query, domain, intent })
    BE-->>FE: Return optimized result + checkout links
    FE->>U: Display "Best Deal" card with universal checkout
    U->>FE: Click → one-click checkout
    FE->>BE: POST /api/search/track-click { itemUrl, provider, price }
    BE->>MONG: Click.create({ userId, itemUrl, provider })
    FE->>EXT: Direct checkout redirection (offers pre-applied)
```

---

## 2. System Architecture — Component Graph

```mermaid
graph TD
    subgraph Client_Layer["🖥️ Client Layer"]
        A["React.js Web App\n(index.html / SPA)"]
        B["React Native Mobile"]
        C["Voice / IoT Interface\n(Future)"]
    end

    subgraph API_Gateway["⚙️ Backend Services (Express.js)"]
        D["server.js\nHTTP + Socket.IO"]
        E["Auth Routes\n/auth/google\n/auth/login\n/auth/me"]
        F["Search Routes\n/api/search\n/api/search/food\n/api/search/shopping"]
        G["Products Routes\n/api/products"]
        H["Rate Limiter\nMiddleware"]
        I["Auth Middleware\nJWT Verification"]
    end

    subgraph Intelligence_Core["🧠 Intelligence Core"]
        J["SearchEngine\nNLP Intent Extraction\nDomain Classification"]
        K["AIService\nGPT-4 Integration\nProduct Summarization\nComparison Insights"]
        L["ProductService\nMulti-Platform Fetcher\nResponse Parser\nFallback Scraper"]
    end

    subgraph Data_Layer["💾 Data & Persistence"]
        M[("MongoDB\nUsers / Searches / Clicks")]
        N[("NodeCache\nIn-memory TTL 300s")]
        P[("Supabase\nAuth / User Profiles")]
    end

    subgraph External["🌐 External Ecosystem"]
        Q["Amazon (RapidAPI)"]
        R["Flipkart (RapidAPI)"]
        S["Myntra / Meesho"]
        T["Swiggy / Zomato"]
        U["Uber / Ola / Rapido"]
        V["MakeMyTrip / IRCTC"]
        W["OYO / Booking.com"]
        X["ONDC Network"]
    end

    subgraph Auth_Providers["🔐 Auth Providers"]
        Y["Google OAuth 2.0\nPassport.js"]
        Z["JWT Tokens\n7-day expiry"]
    end

    subgraph Realtime["📡 Real-time"]
        WS["WebSocket\nSocket.IO\nPrice drop alerts"]
    end

    Client_Layer -->|HTTPS| API_Gateway
    API_Gateway --> Intelligence_Core
    API_Gateway --> Data_Layer
    Intelligence_Core --> External
    Intelligence_Core --> Data_Layer
    Auth_Providers --> API_Gateway
    D --> WS
```

---

## 3. Data Flow Diagram — Query to Recommendation

```mermaid
graph LR
    UI((User Input\nText or Voice)) --> NLP["NLP Engine\nIntent Extraction"]

    NLP --> Intent{Intent\nClassification}

    Intent -->|E-commerce| EC["ProductService\nAmazon / Flipkart\nMyntra / Meesho"]
    Intent -->|Food| FD["Food APIs\nSwiggy / Zomato\nONDC"]
    Intent -->|Rides| RD["Ride APIs\nUber / Ola\nRapido"]
    Intent -->|Travel| TR["Travel APIs\nIRCTC / MakeMyTrip\nYatra / RedBus"]
    Intent -->|Hospitality| HS["Hotel APIs\nOYO / Booking.com\nAirbnb"]

    EC & FD & RD & TR & HS --> AGG["Data Aggregator\nNormalize & Merge"]

    AGG --> Cache{NodeCache\nHIT?}
    Cache -->|Yes| REC
    Cache -->|No| AI

    AI["AIService GPT-4"] --> SA["Sentiment\nAnalysis"]
    AI --> CPN["Coupon\nValidation"]
    AI --> PP["Price\nPrediction"]
    AI --> PERS["Personalization\nEngine"]

    SA & CPN & PP & PERS --> REC["Recommendation\nEngine"]

    REC --> OUT["Frontend UI\nBest Deal Card"]
    OUT --> CHK["Universal\nCheckout Link"]
    CHK --> TRACK["Click Tracker\nMongoDB"]
```

---

## 4. Authentication Flow — Google OAuth + JWT

```mermaid
sequenceDiagram
    participant U as User
    participant FE as Frontend
    participant BE as Express /auth
    participant GP as Google OAuth 2.0
    participant DB as MongoDB (User model)
    participant JWT as JWT Service

    U->>FE: Click "Sign in with Google"
    FE->>BE: GET /auth/google
    BE->>GP: Redirect (clientID, scope: profile+email)
    GP->>U: Google consent screen
    U->>GP: Grant permission
    GP->>BE: GET /auth/google/callback?code=...
    BE->>GP: Exchange code for accessToken
    GP-->>BE: profile { id, email, displayName, photo }
    BE->>DB: User.findByGoogleId(profile.id)
    alt User EXISTS
        DB-->>BE: Existing user record
        BE->>DB: User.updateLastLogin(user.id)
    else New user
        BE->>DB: User.create({ googleId, email, name, avatar })
        DB-->>BE: New user record
    end
    BE->>JWT: generateToken(user) → sign HS256 7d
    JWT-->>BE: deepenk_token (JWT)
    BE->>FE: Set-Cookie: deepenk_token (httpOnly, secure)
    FE->>U: Redirect to app (auth=success)

    Note over FE,BE: Subsequent requests
    U->>FE: Make search request
    FE->>BE: POST /api/search + Cookie: deepenk_token
    BE->>JWT: jwt.verify(token, JWT_SECRET)
    JWT-->>BE: { id, email, name }
    BE->>BE: Attach userId to request
```

---

## 5. Database Schema — Entity Relationship Diagram

```mermaid
erDiagram
    USER ||--o{ SEARCH_HISTORY : "performs"
    USER ||--o{ CLICK_EVENTS : "triggers"
    SEARCH_HISTORY ||--o{ CLICK_EVENTS : "contains"

    USER {
        string id PK
        string googleId
        string email
        string name
        string avatar
        string provider
        json preferences
        timestamp lastLogin
        timestamp createdAt
    }

    SEARCH_HISTORY {
        string id PK
        string userId FK
        string query
        string domain
        json intent
        int resultsCount
        json topResult
        json alternatives
        timestamp createdAt
    }

    CLICK_EVENTS {
        string id PK
        string userId FK
        string searchId FK
        string itemName
        string itemUrl
        string provider
        float price
        timestamp createdAt
    }

    PRODUCT_CACHE {
        string cacheKey PK
        string platform
        string query
        json products
        int ttl
        timestamp cachedAt
    }

    TRANSACTION {
        string txId PK
        string userId FK
        float amount
        string status
        string platformOrigin
        string domain
        json offerApplied
        timestamp createdAt
    }

    USER ||--o{ TRANSACTION : "executes"
```

---

## 6. State Diagram — Price Drop Prediction Logic

```mermaid
stateDiagram-v2
    [*] --> Idle

    Idle --> Monitoring : User requests price watch
    
    Monitoring --> DataCollection : Trigger price fetch (interval)
    DataCollection --> HistoricalAnalysis : Fetch current price + history
    
    HistoricalAnalysis --> TrendEvaluation : Compare 30/60/90 day data
    
    TrendEvaluation --> PriceSteady : Variance < 5%
    TrendEvaluation --> PriceDropLikely : Downward trend detected\nor festive season upcoming
    TrendEvaluation --> PriceRising : Upward trend / surge

    PriceSteady --> RecommendBuyNow : "Buy now, stable price"
    PriceDropLikely --> PredictDropWindow : Calculate estimated\ndrop date + amount
    PriceRising --> AlertUser : "Price rising, buy now\nor set alert"

    PredictDropWindow --> WaitRecommended : "Expected ₹X drop\nin Y days"
    WaitRecommended --> AutoWatch : User opts to wait
    
    AutoWatch --> PriceCheckLoop : Polling interval (15 min)
    PriceCheckLoop --> DropDetected : Price crosses target
    PriceCheckLoop --> PriceCheckLoop : No drop yet
    
    DropDetected --> PushNotification : Notify user instantly
    PushNotification --> AutoCheckout : User enables auto-book
    PushNotification --> ManualCheckout : User manually reviews

    AutoCheckout --> [*] : Order placed
    ManualCheckout --> [*] : User confirms / declines
    RecommendBuyNow --> [*] : User proceeds to checkout
    AlertUser --> [*] : User decides action
```

---

## 7. API Route Map — Backend Endpoint Structure

```mermaid
graph LR
    ROOT["/"]

    ROOT --> AUTH["/auth"]
    AUTH --> AG["/google\nGET - OAuth redirect"]
    AUTH --> AGC["/google/callback\nGET - OAuth callback"]
    AUTH --> AL["/login\nPOST - Email login"]
    AUTH --> ALO["/logout\nPOST - Clear cookie"]
    AUTH --> AM["/me\nGET - Current user"]

    ROOT --> SEARCH["/api/search"]
    SEARCH --> SP["/\nPOST - Universal search\n+ intent detection"]
    SEARCH --> STC["/track-click\nPOST - Click analytics"]
    SEARCH --> SF["/food\nPOST - Food domain search"]
    SEARCH --> SS["/shopping\nPOST - E-commerce search"]
    SEARCH --> SGG["/suggestions\nGET - Autocomplete"]

    ROOT --> PROD["/api/products"]
    PROD --> PD["/details\nGET - Product details"]
    PROD --> PC["/compare\nPOST - Compare products"]
    PROD --> PA["/alternatives\nGET - Similar items"]

    ROOT --> SCRAPE["/api/scrape"]
    SCRAPE --> SAM["/amazon\nFallback scraper"]

    ROOT --> HLT["/api/health\nGET - Server status"]
```

---

## 8. WebSocket — Real-time Events Flow

```mermaid
sequenceDiagram
    participant U as User Browser
    participant FE as Frontend
    participant WS as Socket.IO Server
    participant BE as Backend Services
    participant MON as Price Monitor

    U->>FE: Open app
    FE->>WS: socket.connect()
    WS-->>FE: connection established

    FE->>WS: emit("watch-price", { productId, targetPrice, userId })
    WS->>BE: Register price watch
    BE->>MON: Start polling interval (15 min)

    loop Price Monitoring
        MON->>BE: Fetch current price
        BE->>BE: Compare vs target
        alt Price dropped
            BE->>WS: Trigger price-drop event
            WS->>FE: emit("price-alert", { product, newPrice, savings })
            FE->>U: Push notification + toast
        end
    end

    FE->>WS: emit("search-update", { searchId })
    WS-->>FE: emit("results-ready", { results })
    FE->>U: Real-time search update
```

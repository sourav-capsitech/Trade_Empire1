# 📄 Trade Empire — README

## 1. Executive Overview
**Trade Empire** is a lightweight, real-time economy simulation built on **Unity** with **Google Firebase** as the end-to-end backend. The solution showcases secure authentication, asynchronous data orchestration, and atomic marketplace transactions, all delivered through a scalable and maintainable Unity architecture.

The core value proposition is demonstrating **data integrity, concurrency safety, and clean separation of concerns** in a production-aligned game backend.

---

## 2. System Architecture
The application follows a **Unity-adapted MVC / Presenter pattern**, ensuring clear ownership of responsibilities while keeping Firebase as the single source of truth for persistence and authentication.

### Core Components

| Component | Role | Description |
|---------|------|-------------|
| **UserData (Model)** | Data Structure | Encapsulates player state including profile data, coins, runtimeInventory, and progression metrics (purchaseCount, sellCount, tradeLevel). |
| **FirebaseAuthManager** | Authentication | Manages login, guest access, registration, and lifecycle of the active FirebaseUser. |
| **MainSceneManager (Controller/Presenter)** | Business Logic & UI | Orchestrates UI navigation, marketplace lifecycle, state transitions, and transaction execution. |
| **MarketListing (Model)** | Data Structure | Represents a single marketplace listing aligned with Firebase schema. |

---

## 3. Asynchronous Data Flow
All Firebase Realtime Database operations are executed asynchronously using **Task-based workflows**.

### Thread Safety Strategy
- UI mutations and gameplay state changes are always executed on the **Unity main thread**.
- Enforced via:
  - `Firebase.Extensions.ContinueWithOnMainThread`
  - `UnityMainThreadDispatcher`

This eliminates race conditions and preserves frame stability.

### Marketplace Refresh Pipeline
- `LoadMarketListings()` clears existing UI state.
- Firebase data is fetched asynchronously.
- Prefab instantiation occurs strictly on the main thread after task completion.
- Guarantees UI parity with backend state.

---

## 4. Atomic Transaction Workflow (Buy Flow)
Marketplace purchases are executed as a **multi-stage atomic workflow** to guarantee data consistency.

### Transaction Stages
1. **Validation**
   - Buyer ≠ Seller
   - Buyer has sufficient coins

2. **Buyer Tentative Update**
   - Coins debited
   - Inventory updated
   - Purchase count incremented (local only)

3. **Seller Synchronization (Critical Path)**
   - Seller profile fetched
   - Coins credited
   - Sell count incremented
   - Seller data persisted to Firebase

4. **Cleanup**
   - Listing removed from `/listings` via `RemoveValueAsync()`
   - Buyer data committed
   - Marketplace UI refreshed

5. **Rollback Strategy**
   - Any failure triggers immediate buyer state reversion
   - Ensures zero partial commits and data drift

This design guarantees **transactional integrity without server-side logic**.

---

## 5. Database Structure
Firebase Realtime Database is organized into two primary nodes.

### `/users/`
| Key | Content | Purpose |
|----|--------|---------|
| `userId (UID)` | displayName, coin, purchaseCount, sellCount, tradeLevel, runtimeInventory | Stores all mutable player data |

### `/listings/`
| Key | Content | Purpose |
|----|--------|---------|
| `listingId (Push ID)` | itemName, sellerId, sellerName, quantity, price | Stores all active marketplace listings |

Listings are deleted immediately upon successful sale or manual unlisting.

---

## 6. Known Issues & Operational Notes

| Category | Description | Resolution |
|-------|------------|-----------|
| Firebase Initialization | App fails due to Bundle ID mismatch | Set Unity Package Name to `com.GameWise.Trade_Empire` |
| Marketplace Layout | Listing prefabs overlap | Add **Vertical Layout Group** + **Content Size Fitter** to ScrollView Content |
| Listing Persistence | Sold items briefly reappear | UI refresh now waits for `RemoveValueAsync()` completion |
| Async UI Errors | UI updated off main thread | Enforce `ContinueWithOnMainThread` or `UnityMainThreadDispatcher` |

---

## 7. Strategic Takeaway
Trade Empire delivers a **production-grade reference implementation** for:
- Firebase-backed Unity games
- Safe async programming
- Client-side atomic transactions
- Scalable UI-state orchestration

Designed to be **lean, deterministic, and enterprise-ready**.

# Test Plan: Free Spins Promotional Integration

## 1. Feature Overview
Integration of a new "Free Spins" promotional wallet API into the existing slot game engine. The feature allows players to use operator-awarded free spins before consuming their real-money balance.

## 2. Risk Assessment
* **Financial Risk:** Deducting funds from the real balance instead of the free spins counter.
* **State Desynchronization:** Player disconnecting during a free spin round, leading to lost spins or unawarded payouts.
* **Exploitation:** Manipulating API requests to trigger unauthorized free spins or bypass validation.

## 3. Scope
* **In-Scope:** API integration (bonus wallet validation), database state transitions, UI synchronization (spin counter), session recovery (handling disconnects).
* **Out-of-Scope:** Base game RNG math model and standard paytable (previously certified and tested).

## 4. Key Test Scenarios & Approach
* **TC-01 (Functional):** Verify the bet is deducted strictly from the Free Spins counter (not real balance) when the promo is active.
* **TC-02 (Integration):** Validate DB state update and API response when transitioning from the final free spin back to real-money play.
* **TC-03 (Edge Case):** Simulate a network drop (socket disconnect) precisely at the moment of a Free Spin win calculation; verify session recovery state upon reload.
* **TC-04 (Security):** Intercept the `/spin` request via Burp Suite and attempt to inject a negative bet value or alter the `currency_id` during the bonus round.

## 5. Test Data & Environment
* **Environment:** Staging (QA-01)
* **Tools Used:** Postman (API simulation), Burp Suite (request manipulation), SQL (database state validation), Chrome DevTools (network throttling).

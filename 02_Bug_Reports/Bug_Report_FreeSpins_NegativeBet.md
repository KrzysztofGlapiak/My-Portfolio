# Bug Report: Negative Bet Value Injection in Free Spins Mode

**Issue ID:** QA-102
**Severity:** Critical
**Priority:** High
**Status:** Open
**Environment:** Staging (QA-01)
**Component:** Backend API / Free Spins Module

## Description
When the Free Spins mode is active, it is possible to manipulate the `/spin` API request using Burp Suite by injecting a negative integer into the `bet_amount` parameter. The backend does not properly validate this field during a promotional spin. This results in the system adding the negative bet value to the player's real-money balance, effectively allowing infinite balance generation.

## Preconditions
1. Player account has an active "10 Free Spins" promotion.
2. Player is logged into the game client.
3. Burp Suite is configured to intercept HTTP traffic between the client and server.

## Steps to Reproduce
1. Trigger a spin within the game client while the Free Spins promotion is active.
2. Intercept the `/spin` POST request in Burp Suite.
3. Locate the JSON payload and modify the `"bet_amount"` field from `0` (or the default free spin value) to `-5000`.
4. Forward the modified request to the server.
5. Observe the API response and the updated player balance in the UI.

## Expected Result
The backend should reject the request with an HTTP 400 Bad Request status code and an appropriate error message indicating invalid input validation (`"Invalid bet_amount"`). The player's real-money balance should remain unchanged.

## Actual Result
The backend accepts the modified request with an HTTP 200 OK status. The system processes the negative bet, which inadvertently adds 5000 credits to the player's real-money balance.

## Evidence / Logs

**Intercepted Request Payload (Snippet):**
```json
POST /api/v1/game/spin HTTP/1.1
Host: staging-api.igaming.local
Authorization: Bearer <token>
Content-Type: application/json

{
  "game_id": "slot_123",
  "currency_id": "EUR",
  "bet_amount": -5000,
  "promo_id": "freespins_10"
}
```

**Server Response (Snippet):**
```json
HTTP/1.1 200 OK
Content-Type: application/json

{
  "status": "success",
  "balance": {
    "real_money": 15000,
    "free_spins_remaining": 9
  }
}
```

## Additional Information
This vulnerability highlights a critical lack of input sanitization and boundary checking on the backend specifically during promotional states. Standard real-money spins seem unaffected, indicating the issue is localized to the Free Spins wallet logic.

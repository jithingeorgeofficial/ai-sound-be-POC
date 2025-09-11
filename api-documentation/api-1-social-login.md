##AUC-59
## Social Authentication Flow (Apple/Google)

### Overview
- **App/web obtains provider id_token** (Apple identityToken or Google ID token).
- App POSTs to backend with provider `id_token` and required device fields.
- Backend verifies provider token, creates/links user + device, records login, and returns your app JWT.

### API Contract
- **Endpoint**: `POST /rest/v1/auth/social-login`
- **Body**:
  - `provider`: `"apple" | "google"`
  - `idToken`: provider-issued JWT
  - `deviceId`, `deviceType` (`ios|android|web`), `deviceFcmToken`, `deviceModel`, `osVersion`, `appVersion`

### Flow
1. App signs in with Apple/Google and obtains `id_token`.
2. App calls `/rest/v1/auth/social-login` with `idToken` + device fields.
3. `AuthController.socialLogin` → `LoginService.socialLogin`.
4. `VerifySocialUser` → `SocialAuthAdapter.verifySocialToken` → provider adapter:
   - `AppleAuthAdapter`: verifies JWT using Apple JWKS, checks `iss`, `aud`, `exp`.
   - `GoogleAuthAdapter`: verifies Google ID token.
5. `RegisterSocialUser`: find/create `User`.
6. `RegisterUserDevice`: upsert `UserDevice`.
7. `RecordUserLoginActivity`: insert login activity.
8. `BuildToken`: issue app JWT; return `AuthTokenResponse`.

### Files (added/changed)
- Web test flow: `src/adapter/in/web/SocialAuthWebController.ts`
- REST: `src/adapter/in/rest/AuthController.ts`
- Service: `src/application/services/LoginService.ts`
- Use cases: `src/application/usecases/VerifySocialUser.ts`, `RegisterSocialUser.ts`, `RegisterUserDevice.ts`, `RecordUserLoginActivity.ts`, `BuildToken.ts`
- Infra adapters: `src/adapter/out/infra/SocialAuthAdapter.ts`, `src/adapter/out/infra/social-auth/AppleAuthAdapter.ts`, `src/adapter/out/infra/social-auth/GoogleAuthAdapter.ts`
- Models: `src/models/http-models/SocialLoginRequest.ts`, `src/models/http-models/DeviceRequest.ts`
- Mounting: `src/Server.ts` (mounts `/rest/:version` and `/` web routes)

### Packages
- `jsonwebtoken`, `jwks-rsa`
- `google-auth-library` (or `openid-client`)
- `@tsed/*`, `typeorm`

### Env Vars
- Apple: `APPLE_CLIENT_ID` (must match id_token `aud`);
- Google: `GOOGLE_CLIENT_ID`; 

### Database Tables Affected
- `users` (create/find social user)
- `user_device` (register/update device)
- `user_login_activity` (insert login row)

### Example Request
```bash
curl -X POST https://<ngrok>/rest/v1/auth/social-login \
  -H "Content-Type: application/json" \
  -d '{
    "provider": "apple",
    "idToken": "<APPLE_OR_GOOGLE_ID_TOKEN_JWT>",
    "deviceId": "device-uuid",
    "deviceType": "ios",
    "deviceFcmToken": "fcm-or-apns",
    "deviceModel": "iPhone 15",
    "osVersion": "17.5",
    "appVersion": "1.0.0"
  }'
```

### Notes
- Send the provider `id_token` (JWT) — not the Apple client secret.
- Backend returns its own JWT after verification.



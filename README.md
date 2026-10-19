# SmartHome Hub

IoT-based mobile app for home automation and device management.
ENGI401 / Capstone — Cyprus International University.

This repo holds the **Software Engineer 1 + 2** work:

| Folder | What | WBS |
|---|---|---|
| `backend/` | Python FastAPI server: auth, homes, devices, rules engine, energy, notifications, MQTT bridge | 3.1 – 3.3 (+ 4.2 starter) |
| `mobile/` | React Native (Expo) app for iOS + Android: login, dashboard, device control, automation, energy charts, alerts | 2.1 – 2.4 |
| `docs/API.md` | The API contract both sides follow | – |
| `tools/simulate_device.py` | Tiny MQTT device simulator for testing | (placeholder for 4.3) |

## 1. Run the backend

```bash
cd backend
python -m venv .venv && source .venv/bin/activate     # Windows: .venv\Scripts\activate
pip install -r requirements.txt
python seed.py                 # demo data: demo@example.com / demo1234
uvicorn app.main:app --reload --host 0.0.0.0
```

- API docs: http://localhost:8000/docs
- Health: http://localhost:8000/api/v1/health
- Tests: `pytest`

Settings are environment variables with prefix `SHH_` (see `app/config.py`), e.g.
`SHH_MQTT_HOST=192.168.1.10`, `SHH_JWT_SECRET=...`, `SHH_DATABASE_URL=postgresql://...`.
Without an MQTT broker the API still works; commands are applied to the database only.

Optional MQTT broker: `docker run -p 1883:1883 eclipse-mosquitto`

## 2. Run the mobile app

```bash
cd mobile
npm install
npx expo start
```

Then press `a` (Android emulator), `i` (iOS simulator), or scan the QR code with **Expo Go** on your phone.

Server address:
- Emulator/simulator: default works (Android uses `10.0.2.2` automatically).
- Real phone: set your PC's LAN IP first:
  `EXPO_PUBLIC_API_URL=http://192.168.1.20:8000/api/v1 npx expo start`

Typecheck: `npm run typecheck`

## 3. How the parts talk

```
 Mobile app  --HTTP/JSON-->  FastAPI  --MQTT cmd-->  Device / simulator
            <--poll 5 s--             <--MQTT state--
```

- Command: `POST /devices/{id}/command` → published to `smarthome/{home}/{device}/cmd`,
  and the expected state is saved right away (optimistic UI).
- Device reports state on `smarthome/{home}/{device}/state` → saved → rules engine runs.
- Rules: `device_state` triggers run on every state change; `schedule` triggers run from a background loop.
- Security events (motion, door unlocked) and rule runs create notifications; push goes out via FCM when `SHH_FIREBASE_CREDENTIALS_FILE` is set.

## 4. What is done / what is next

Done:
- JWT auth, multi-user + multi-home with owner/member roles
- Device CRUD, commands per device type, state reports (MQTT and HTTP)
- IFTTT rules (device-state and schedule), rule builder UI
- Energy readings → kWh totals, per-device split, hourly/daily series, chart
- Notification feed, push-token endpoint, FCM hook
- 7 API tests; app typechecks and runs end to end against the backend

Next (suggested order):
1. Real-time updates: replace 5-second polling with WebSocket or MQTT-over-WebSocket in the app.
2. Push notifications in the app: `expo-notifications` → send token to `/notifications/push-token`.
3. Firebase Firestore adapter if the team wants Firebase as the main database (currently SQLite/PostgreSQL via SQLModel).
4. Edit-rule screen (today: create, enable/disable, delete).
5. Hand the MQTT topic contract (`docs/API.md`) to the Mechatronics engineer for the ESP32 firmware and full simulator.
# SmartHome-Hub

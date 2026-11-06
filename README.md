# SmartTransit Mysuru – V4

A Flask + SQLite demonstration prototype for data-driven public transport reliability and crowding prediction.

## V4 workflow separation

1. **Online passenger booking** (`/passenger`): select a route and one of the next three buses. The generated ticket is bus-specific. There is no seat selection or seat confirmation.
2. **Bus-stop kit** (`/bus-stop-kit`): a separate camera scanner reads a wallet-payment QR payload and records a wallet transaction in its own tables. A successful payment creates a route-specific travel ticket. Duplicate wallet references are rejected.
3. **Conductor kit** (`/conductor-kit`): a separate handheld scanner reads travel-ticket QR codes. Online tickets are accepted only on their assigned bus; wallet tickets are accepted on any bus on the same route. A ticket can be used once. A second scan is rejected.
4. **Controller** (`/control-room`): existing control room and demand forecasting views remain available. Online bookings and bus-stop wallet transactions are retained as separate source records.
5. **Fleet register** (`/fleet`): sample fleet records and bus profiles.

## Run on Windows

1. Extract the ZIP.
2. Open the extracted `smarttransit_mysuru_v2` folder in VS Code.
3. Open Terminal → New Terminal.
4. Run:

```powershell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
python backend\server.py
```

If PowerShell blocks activation, run this in the same terminal and retry activation:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.\.venv\Scripts\Activate.ps1
```

Alternatively, double-click `run_windows.bat` after Python is installed; it creates the virtual environment, installs Flask and starts the server.

Open `http://127.0.0.1:5000`.

## Browser camera access

Camera scanning works on `localhost` (the above address) or an HTTPS deployment. Browsers normally block camera access on an ordinary HTTP address accessed from another device over Wi-Fi. Allow camera permission when prompted. Scanner libraries and map tiles are loaded from public CDNs, so an internet connection is required for those browser features. Manual QR-value entry is available for testing.

## Demo wallet QR format

For local testing, enter a payload such as:

`WALLET:DEMO-REF-0001:10`

The wallet reference must be unique. Reusing it is rejected as a duplicate. This is a simulation only: it does not contact or debit a real wallet. A production integration must verify signed payment status with the actual payment provider before recording a transaction.

## Demo ticket rules

- Online ticket: bus-specific; only the selected bus can validate it.
- Wallet ticket: assigned to the selected bus so its issued-ticket counter is updated.
- Normal conductor-issued ticket: created from the Conductor Kit and immediately included in that bus counter.
- Online, wallet and normal journey tickets: single-use. A successful conductor scan changes the ticket status from `UNUSED` to `USED`; repeat scans are rejected.
- Smart bus pass: created in the Passenger domain with a separate `PASS:` QR payload. The conductor scanner routes pass QR values to a separate validator that checks authenticity, route, active status and validity dates. A repeat scan on the same bus on the same service date is rejected; an active pass may be used on another service day.
- Live ticket counters are shown in the passenger, conductor and control-centre domains. They count issued journey-ticket records from online booking, wallet transactions and normal conductor-issued tickets. Bus passes are tracked separately and are not mixed into the journey-ticket count.
- Both: single-use. A successful conductor scan changes the ticket status from `UNUSED` to `USED`; repeat scans are rejected.
- A wallet payment record and a conductor boarding record are separate events. A payment alone is not counted as confirmed boarding.

## Important limitations

- Bus registrations, crew names, fleet details, route geometry, GPS positions, ETAs and historical demand are illustrative sample data, not verified live KSRTC/Mysuru operator records.
- GPS is simulated unless an authorized device feed is integrated.
- Wallet verification is simulated; no real money is collected.
- The prototype's in-memory bus load simulation resets when the server restarts. Database ticket/transaction records persist in `data/smarttransit.db`.
- The crowding forecast is a transparent demonstration model, not a validated production prediction model.
- Before public deployment, add authentication/roles, signed ticket payloads, payment-provider verification, concurrency-safe ticket redemption, audit logs, HTTPS, backups, and an authorized live GPS/traffic feed.

# 🚑 GOLDENROUTE: A GOLDEN HOUR-AWARE INTELLIGENT EMERGENCY ROUTING AND HEALTHCARE RESOURCE OPTIMIZATION

GoldenRoute is a Django-based emergency healthcare dispatch and intelligent hospital routing system. 
It optimizes the critical "Golden Hour" in emergency medical care by routing ambulances to the best-equipped hospital in real time based on ER room availability, ICU beds, required specialist availability, and live travel time via the OpenRouteService API.

---

## 🌐 Live Production Deployment

- **Main Web Application:** [https://golden--routee.vercel.app/](https://golden--routee.vercel.app/)
- **Django Admin Control Panel:** [https://golden--routee.vercel.app/admin/](https://golden--routee.vercel.app/admin/)

---

## 👥 User Roles, Credentials & Direct URLs

Each role in GoldenRoute has a dedicated dashboard, direct access URL, and specific capabilities.

---

### 🏥 1. Hospital Role

Hospitals manage their real-time emergency capacity and receive pre-arrival patient alerts.

- **Direct Dashboard URL:** [https://golden--routee.vercel.app/hospital/](https://golden--routee.vercel.app/hospital/)
- **Login URL:** [https://golden--routee.vercel.app/login/](https://golden--routee.vercel.app/login/)
- **Demo Credentials:**
  - **Hospital 1 (Apollo):** Username: `apollo_hospital` | Password: `123`
  - **Hospital 2 (City Hospital):** Username: `city_hospital` | Password: `123`
- **Capabilities & Features:**
  - Update live count of free **ER Rooms** and **ICU Beds**.
  - Toggle on-duty specialist flags (**Cardiologist**, **Neurosurgeon**, **Trauma Team**).
  - View real-time incoming **Patient Transfers** with patient medical summary (blood group, age, pre-existing conditions) and arrival ETA.

---

### 🚑 2. Ambulance Driver Role

Ambulance drivers calculate optimal hospital destinations, view route maps, and respond to SOS calls.

- **Direct Dashboard URL:** [https://golden--routee.vercel.app/ambulance/](https://golden--routee.vercel.app/ambulance/)
- **Login URL:** [https://golden--routee.vercel.app/login/](https://golden--routee.vercel.app/login/)
- **Demo Credentials:**
  - **Ambulance 1:** Username: `driver1` | Password: `123`
  - **Ambulance 2:** Username: `driver2` | Password: `123`
  - **Ambulance 3:** Username: `driver3` | Password: `123`
- **Capabilities & Features:**
  - **Update GPS Location:** Broadcast current ambulance coordinates.
  - **Smart Hospital Routing:** Input patient emergency type (`Cardiac`, `Neuro`, `Trauma`) and accident coordinates (e.g. `12.9715`, `80.2500`) to find the best hospital.
  - **🗺️ Interactive Map & Navigation:** View embedded Google Map of the destination hospital and launch **Google Maps turn-by-turn navigation**.
  - **Patient Confirmation:** Confirm transport with patient's Aadhaar to notify the receiving hospital.
  - **🚨 Distress Signals:** View nearby citizen SOS signals with one-click **"🚑 Navigate to Citizen"** GPS routing.

---

### 👤 3. Citizen Role

Citizens register their emergency medical profiles and broadcast instant distress SOS signals.

- **Direct Dashboard URL:** [https://golden--routee.vercel.app/citizen/](https://golden--routee.vercel.app/citizen/)
- **Signup URL:** [https://golden--routee.vercel.app/citizen-signup/](https://golden--routee.vercel.app/citizen-signup/)
- **Edit Profile URL:** [https://golden--routee.vercel.app/edit-profile/](https://golden--routee.vercel.app/edit-profile/)
- **Login URL:** [https://golden--routee.vercel.app/login/](https://golden--routee.vercel.app/login/)
- **Demo Credentials:**
  - **Pre-existing Citizen:** Username: `batman` | Password: `123`
  - **Self-Registration:** Sign up at `/citizen-signup/` with any unique username and password.
- **Capabilities & Features:**
  - Store emergency health data (Aadhaar number, Blood group, Diabetes, Heart disease, Emergency contact).
  - Trigger a 1-click **Distress SOS Signal** with GPS location that auto-dispatches to the nearest active ambulance.

---

### 🛠️ 4. Admin / Superuser Role

System administrators manage all database tables, users, and hospital records.

- **Direct Admin URL:** [https://golden--routee.vercel.app/admin/](https://golden--routee.vercel.app/admin/)
- **Credentials:**
  - Username: `admin` | Password: `123`
- **Capabilities & Features:**
  - Full CRUD access to `User`, `UserProfile`, `Hospital`, `EmergencyCase`, `HealthProfile`, `PatientTransfer`, and `DistressSignal` models.

---

## 🗺️ How to Test Route Navigation Step-by-Step

### 1. Test Hospital Route & Turn-by-Turn Navigation
1. Open [`https://golden--routee.vercel.app/login/`](https://golden--routee.vercel.app/login/)
2. Log in with **`driver1`** / **`123`**.
3. Under **Find Hospital for Patient**:
   - **Patient Type:** Select `Cardiac`
   - **Accident Latitude:** `12.9715`
   - **Accident Longitude:** `80.2500`
4. Click **`🔎 Find Hospitals`**.
5. View the **⭐ Recommended Hospital** section:
   - 🗺️ **Embedded Google Map** displays the hospital location.
   - 🧭 Click **"Open Route in Google Maps"** to open live turn-by-turn driving directions from accident to hospital.

### 2. Test Citizen SOS to Ambulance Dispatch Route
1. Log into [`https://golden--routee.vercel.app/login/`](https://golden--routee.vercel.app/login/) as Citizen (**`batman`** / **`123`**).
2. Click **Send Distress Signal** with coordinates (e.g. `12.9650`, `80.2480`).
3. Log in as Ambulance (**`driver1`** / **`123`**).
4. Under **🚨 Distress Signals**, click **`🚑 Navigate to Citizen`** to launch Google Maps routing directly to the citizen's coordinates.

---

## 📐 Hospital Scoring Algorithm

Implemented in `core/utils.py → calculate_hospital_score()`:

$$\text{Score} = (\text{ER rooms} \times 3) + (\text{ICU beds} \times 2) + (\text{Specialist Match} \ ? \ +20 : -20) - (\text{Travel Time in mins} \times 1.5)$$

---

## 💻 Tech Stack

| Layer | Technology |
|---|---|
| Backend Framework | Django 5.x / 6.x |
| API Layer | Django REST Framework |
| Static Assets | WhiteNoise |
| Database | SQLite3 (configured for `/tmp` writable storage on Vercel) |
| Routing API | OpenRouteService API v2 |
| Navigation | Google Maps (deep-link integration) |
| Deployment | Vercel Serverless Functions (`@vercel/python`) |
| Language | Python 3.11 / 3.12 |

---

## 📂 Architecture & Project Structure

```
goldenroute/                        ← Root Django project folder
│
├── manage.py                       ← Django management CLI entry point
├── db.sqlite3                      ← Pre-seeded SQLite database
├── requirements.txt                ← Python package dependencies
├── vercel.json                     ← Vercel serverless deployment config
├── .gitignore                      ← Git ignore rules
│
├── goldenroute/                    ← Django project config package
│   ├── settings.py                 ← App settings, WhiteNoise, /tmp DB logic
│   ├── urls.py                     ← Root URL dispatcher
│   ├── wsgi.py                     ← WSGI server entry point (Vercel handler)
│   └── asgi.py                     ← ASGI server entry point
│
└── core/                           ← Main application
    ├── models.py                   ← All database models
    ├── views.py                    ← Business logic, auth & role dispatching
    ├── urls.py                     ← App-level URL patterns
    ├── admin.py                    ← Django admin registrations
    ├── utils.py                    ← Scoring algorithm & routing API client
    ├── serializers.py              ← DRF serializers
    ├── tests.py                    ← Unit tests
    │
    ├── migrations/                 ← Database schema migrations (0001–0016)
    │
    └── templates/
        ├── base.html               ← Shared base HTML layout
        └── core/
            ├── login.html          ← Role-based login page
            ├── citizen_signup.html ← Citizen self-registration
            ├── citizen_dashboard.html
            ├── edit_health_profile.html
            ├── ambulance_dashboard.html
            └── hospital_dashboard.html
```

---

## 🚀 How to Run Locally

```bash
# 1. Clone repo
git clone https://github.com/LithkeshBalajiB/goldenroute.git
cd goldenroute

# 2. Virtual environment
python -m venv venv
venv\Scripts\activate  # Windows
# source venv/bin/activate  # macOS/Linux

# 3. Install dependencies
pip install -r requirements.txt

# 4. Run server
python manage.py runserver
```
Visit `http://127.0.0.1:8000/` in your browser.

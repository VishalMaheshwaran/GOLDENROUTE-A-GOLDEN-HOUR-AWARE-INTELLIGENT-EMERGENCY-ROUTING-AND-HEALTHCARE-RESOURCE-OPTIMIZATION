# 🚑 GOLDENROUTE: A GOLDEN HOUR-AWARE INTELLIGENT EMERGENCY ROUTING AND HEALTHCARE RESOURCE OPTIMIZATION

GoldenRoute is a Django-based emergency healthcare dispatch and routing web application. 
It intelligently routes ambulances to the best available hospital in real time, taking into account ER availability, ICU beds, required specialist presence,
and live travel time via the OpenRouteService API. The platform also allows citizens to send distress signals that are automatically dispatched 
to the nearest ambulance.

---

## 🌐 Live Production Deployment

- **Live Web App:** [https://golden--routee.vercel.app/](https://golden--routee.vercel.app/)
- **Admin Panel:** [https://golden--routee.vercel.app/admin/](https://golden--routee.vercel.app/admin/)

---

## 🔑 Pre-Configured Demo Credentials

All pre-seeded test accounts use the password: **`123`**

| Role | Username | Password | Login URL | Access Scope / Target |
|---|---|---|---|---|
| 🏥 **Hospital** | `apollo_hospital` | `123` | [`/login/`](https://golden--routee.vercel.app/login/) | Hospital Dashboard (ER rooms, ICU beds, incoming transfers) |
| 🏥 **Hospital** | `city_hospital` | `123` | [`/login/`](https://golden--routee.vercel.app/login/) | Hospital Dashboard (ER rooms, ICU beds, incoming transfers) |
| 🚑 **Ambulance** | `driver1` | `123` | [`/login/`](https://golden--routee.vercel.app/login/) | Ambulance Dashboard (GPS location, find hospital, distress SOS) |
| 🚑 **Ambulance** | `driver2` | `123` | [`/login/`](https://golden--routee.vercel.app/login/) | Ambulance Dashboard (GPS location, find hospital, distress SOS) |
| 👤 **Citizen** | `batman` | `123` | [`/login/`](https://golden--routee.vercel.app/login/) | Citizen Dashboard (Health profile, emergency contact, SOS trigger) |
| 👤 **New Citizen** | Self-signup | Your password | [`/citizen-signup/`](https://golden--routee.vercel.app/citizen-signup/) | Citizen Self-Registration |
| 🛠️ **Admin** | `admin` | `123` | [`/admin/`](https://golden--routee.vercel.app/admin/) | Full Django Admin Control Panel |

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Tech Stack](#tech-stack)
3. [Architecture & Project Structure](#architecture--project-structure)
4. [Data Models](#data-models)
5. [User Roles & Dashboards](#user-roles--dashboards)
6. [Core Features](#core-features)
7. [Hospital Scoring Algorithm](#hospital-scoring-algorithm)
8. [API Integrations](#api-integrations)
9. [URL Routes](#url-routes)
10. [How to Run Locally](#how-to-run-locally)
11. [Vercel Deployment Guide](#vercel-deployment-guide)

---

## Project Overview

GoldenRoute solves a critical problem in emergency healthcare: **which hospital should an ambulance go to?**

When an ambulance responds to an emergency, the driver needs to quickly decide which hospital is best — not just the closest, but the one that has:
- Available ER rooms and ICU beds
- The right specialist on duty (cardiologist, neurosurgeon, trauma team)
- The shortest real-world travel time

GoldenRoute automates this decision with a weighted scoring algorithm and live routing data. It also enables citizens 
to trigger a distress signal from their dashboard, which gets routed to the nearest available ambulance.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Backend Framework | Django 5.x / 6.x |
| API Layer | Django REST Framework |
| Static Assets | WhiteNoise |
| Database | SQLite3 (configured for `/tmp` writable storage on Vercel) |
| Routing API | OpenRouteService API v2 |
| Navigation | Google Maps (deep-link integration) |
| Templating | Django Templates (server-side HTML) |
| Deployment | Vercel Serverless Functions (`@vercel/python`) |
| Language | Python 3.11 / 3.12 |

---

## Architecture & Project Structure

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

## Data Models

### `UserProfile`
Extends Django's built-in `User` model with a role and real-time GPS location.

| Field | Type | Description |
|---|---|---|
| `user` | OneToOneField (User) | Link to Django auth user |
| `role` | CharField | One of: `ambulance`, `hospital`, `citizen` |
| `latitude` / `longitude` | FloatField | Current GPS coordinates |
| `location_updated_at` | DateTimeField | Timestamp of last location update |

---

### `Hospital`
Represents a medical facility and its live resource availability.

| Field | Type | Description |
|---|---|---|
| `user` | OneToOneField (User) | Linked hospital user account |
| `name` | CharField | Hospital name |
| `latitude` / `longitude` | FloatField | Facility GPS location |
| `er_rooms_available` | IntegerField | Free emergency room count |
| `icu_beds_available` | IntegerField | Free ICU bed count |
| `has_cardiologist` | BooleanField | Specialist on duty flag |
| `has_neurosurgeon` | BooleanField | Specialist on duty flag |
| `has_trauma_team` | BooleanField | Specialist on duty flag |
| `last_updated` | DateTimeField | When resources were last modified |

---

### `HealthProfile`
Citizen medical and emergency contact records.

| Field | Type | Description |
|---|---|---|
| `user` | OneToOneField (User) | Linked citizen account |
| `name` | CharField | Full name |
| `aadhar_number` | CharField | Unique ID for lookup |
| `age` | IntegerField | Patient age |
| `blood_group` | CharField | Blood type |
| `diabetes` / `heart_disease` | BooleanField | Medical condition flags |
| `emergency_contact` | CharField | Primary emergency phone number |
| `comments` | TextField | Additional medical notes |

---

### `DistressSignal`
Citizen-triggered SOS signals assigned to the nearest ambulance.

| Field | Type | Description |
|---|---|---|
| `citizen` | ForeignKey (HealthProfile) | Citizen who sent the SOS |
| `latitude` / `longitude` | FloatField | GPS location of the distress call |
| `assigned_ambulance` | ForeignKey (User) | Auto-assigned nearest ambulance |
| `emergency_phone` | CharField | Emergency contact number |
| `created_at` | DateTimeField | Timestamp |

---

### `PatientTransfer`
Confirmed transfer record sent from an ambulance to a hospital.

| Field | Type | Description |
|---|---|---|
| `hospital` | ForeignKey (Hospital) | Destination hospital |
| `citizen` | ForeignKey (HealthProfile) | Patient profile |
| `ambulance_user` | ForeignKey (User) | Driver making the transfer |
| `patient_type` | CharField | Emergency category (cardiac, neuro, trauma) |
| `eta_minutes` | FloatField | Calculated travel time |
| `sent_time` | DateTimeField | Confirmation timestamp |

---

## User Roles & Dashboards

GoldenRoute provides 3 distinct user roles with automated role-based routing upon login:

### 🚑 Ambulance Driver
- Updates real-time GPS coordinates.
- Selects emergency patient type (Cardiac, Neuro, Trauma).
- Uses the **Smart Routing Algorithm** to find the highest-scoring hospital with live travel time and ETA.
- Confirms patient transport, instantly alerting the hospital.
- Receives auto-assigned distress SOS signals from citizens with direct Google Maps navigation links.

### 🏥 Hospital
- Manages and updates live ER room counts and ICU bed availability.
- Toggles specialist availability on duty.
- Views live incoming patient notifications with full medical profiles and arrival ETAs.

### 👤 Citizen
- Registers an account and maintains emergency health details (Aadhaar, blood group, medical history).
- Triggers a GPS-based Distress Signal that auto-dispatches to the closest active ambulance.

---

## Hospital Scoring Algorithm

Defined in `core/utils.py → calculate_hospital_score()`:

$$\text{Score} = (\text{ER rooms} \times 3) + (\text{ICU beds} \times 2) + (\text{Specialist Match} \ ? \ +20 : -20) - (\text{Travel Time in mins} \times 1.5)$$

The highest-scoring hospital is recommended to the ambulance driver, balancing medical capability with travel proximity.

---

## URL Routes

| URL | View | Name | Access Scope |
|---|---|---|---|
| `/` | `login_view` | `home` | Public |
| `/login/` | `login_view` | `login` | Public |
| `/citizen-signup/` | `citizen_signup` | `citizen_signup` | Public |
| `/citizen/` | `citizen_dashboard` | `citizen_dashboard` | Citizen Role |
| `/edit-profile/` | `edit_health_profile` | `edit_health_profile` | Citizen Role |
| `/distress/` | `distress_signal` | `distress_signal` | Citizen Role |
| `/ambulance/` | `ambulance_dashboard` | `ambulance_dashboard` | Ambulance Role |
| `/hospital/` | `hospital_dashboard` | `hospital_dashboard` | Hospital Role |
| `/admin/` | Django Admin | — | Superuser |

---

## How to Run Locally

### 1. Clone the repository
```bash
git clone https://github.com/LithkeshBalajiB/goldenroute.git
cd goldenroute
```

### 2. Create and activate virtual environment
```bash
# Windows
python -m venv venv
venv\Scripts\activate

# macOS / Linux
python3 -m venv venv
source venv/bin/activate
```

### 3. Install dependencies
```bash
pip install -r requirements.txt
```

### 4. Run the development server
```bash
python manage.py runserver
```
Visit `http://127.0.0.1:8000/` in your browser.

---

## Vercel Deployment Guide

This repository is pre-configured for 1-click deployment on Vercel:

1. Push your repository to GitHub.
2. Import the repository in [Vercel](https://vercel.com).
3. The root `vercel.json` and `requirements.txt` will automatically build the `@vercel/python` serverless WSGI runtime.
4. Set your production domain in **Project Settings > Domains**.

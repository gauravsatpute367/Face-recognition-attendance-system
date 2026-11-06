# Orbit — Face Recognition Attendance Studio

A local-first attendance dashboard built with Python, Flask, SQLite, and OpenCV's LBPH face recognizer. It includes a live camera scanner, face enrollment, daily attendance tracking, a live weekly chart, roster management, and CSV export.

> **Privacy and intended use:** Face descriptors are biometric data. This demo stores them on the machine running the app and does not upload them to a third-party service. Use it only with informed consent, restrict access to the computer, and delete records when they are no longer needed. Do not expose this development server directly to the public internet. Provide a non-biometric attendance option and follow the laws and policies that apply to your organization.

## Features

- Enroll people using a browser camera; only one visible face is accepted.
- Student and employee accounts with password login; first-time admin setup and admin-created accounts.
- Self-registration for students and employees, held pending until an administrator links an enrolled face and approves the account.
- Admin-configurable geofence with live location on the map, an inside/outside indicator, radius, and a server-side GPS distance check before a face check-in.
- Recognize the signed-in user's enrolled face and record one check-in per person per local calendar day.
- Refreshable dashboard with present count, attendance rate, weekly activity, recent check-ins, and a 7-day activity streak.
- Manage enrolled people and remove their attendance history.
- Export attendance records to CSV.
- Responsive dark dashboard with animated 3D-style face scanner, orbital rings, and reduced-motion support.
- Local SQLite storage; no cloud account or hosted face API required.

## Requirements

- Python 3.10–3.12 recommended.
- A webcam and a modern browser.
- OpenCV contrib provides the face recognition API; this avoids building `dlib` on Windows.

## Run locally

```powershell
cd "FACE RECOGINATION  ATTENDANCE SYSTEM"
py -3.11 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
python app.py
```

Open [http://127.0.0.1:5000](http://127.0.0.1:5000) and allow camera access. Camera access is normally restricted to `localhost` or HTTPS.

On the first visit, create the administrator account. The admin can then set the attendance point by searching for a place/address, clicking the map, or using the device's live location; set a radius; enroll faces; approve new registrations; and link each account to one enrolled face. Selecting a search result only moves the proposed center—press **Save attendance area** to apply it. Place searches are sent to the OpenStreetMap Nominatim geocoding service. Students and employees can show their current GPS location on the map and see whether they are inside or outside the saved attendance boundary. Their live location is also checked during face attendance. Staff accounts see their own attendance and the configured area map.

The database is created as `attendance.db` beside `app.py` on first run. It is intentionally ignored by Git. To use a different local database path, set `ATTENDANCE_DB` before starting the app.

## API

The API is available at the same local server as the dashboard:

| Method | Endpoint | Purpose |
| --- | --- | --- |
| `GET` | `/api/health` | Check API and OpenCV face-recognition readiness |
| `GET` | `/api/auth/state` | Check first-time setup and current login |
| `POST` | `/api/setup-admin` | Create the first admin account |
| `POST` | `/api/register` | Request a student/employee account (pending approval) |
| `POST` | `/api/login` / `/api/logout` | Sign in/out |
| `GET` | `/api/dashboard` | Read the role-filtered dashboard and attendance |
| `GET` / `PUT` | `/api/area` | Read or admin-configure the allowed location area |
| `POST` | `/api/enroll` | Admin enrolls a face image |
| `POST` | `/api/recognize` | Match the signed-in user's face and validate their location |
| `GET` | `/api/admin/accounts` | Admin list of users and available face profiles |
| `POST` | `/api/admin/accounts` | Admin creates a linked student/employee account |
| `POST` | `/api/admin/accounts/<id>/approve` | Admin links a profile and approves registration |
| `DELETE` | `/api/admin/accounts/<id>` | Admin removes a login account |
| `DELETE` | `/api/people/<id>` | Admin removes a face profile and attendance history |
| `GET` | `/api/attendance.csv` | Admin downloads attendance CSV |

Enrollment supports JSON with an image data URL:

```powershell
curl.exe -X POST http://127.0.0.1:5000/api/enroll `
  -H "Content-Type: application/json" `
  -d '{\"name\":\"Alex Morgan\",\"imageData\":\"data:image/jpeg;base64,<BASE64_IMAGE>\"}'
```

Or multipart form data, convenient for API clients:

```powershell
curl.exe -X POST http://127.0.0.1:5000/api/enroll `
  -F "name=Alex Morgan" `
  -F "image=@C:\path\to\face.jpg"
```

The image must contain exactly one visible face. Older dlib face descriptors are ignored by the OpenCV recognizer; re-enrolling someone with the same name replaces their old descriptor while preserving their attendance history.

## Configuration

| Environment variable | Default | Description |
| --- | --- | --- |
| `ATTENDANCE_DB` | `attendance.db` in the project folder | SQLite database file path |
| `FACE_MATCH_THRESHOLD` | `75` | OpenCV LBPH distance cutoff; lower values are stricter |
| `FLASK_DEBUG` | unset | Set to `1` for Flask debug mode during local development only |

Recognition is a convenience feature, not proof of identity. OpenCV LBPH is a lightweight local recognizer and may be less reliable than modern embedding-based systems; lighting, camera quality, pose, and demographic differences can affect accuracy. Browser GPS may be spoofed, so geofencing is a convenience control and not a strong security boundary. Use HTTPS outside localhost, review check-ins, and offer a manual alternative.

Set `FLASK_SECRET_KEY` to a long, random value before using the app beyond local development; otherwise sessions are invalidated whenever the app restarts.

## GitHub

Create an empty repository on GitHub (do not add a README or license there), then run these commands from this project folder:

```powershell
git init
git add .
git commit -m "Create Orbit attendance studio"
git branch -M main
git remote add origin https://github.com/<USERNAME>/<REPOSITORY>.git
git push -u origin main
```

Replace `<USERNAME>` and `<REPOSITORY>` with your GitHub account and repository. The generated database is ignored by Git; never commit real face images or biometric descriptors.

# Mergington High School Activities API

A super simple FastAPI application that allows students to view and sign up for extracurricular activities.

## Features

- View all available extracurricular activities
- Teacher-only activity registration and unregistration
- Public activity and participant lists

## Getting Started

1. Install the dependencies:

   ```
   pip install fastapi uvicorn
   ```

2. Run the application:

   ```
   python app.py
   ```

3. Open your browser and go to:
   - API documentation: http://localhost:8000/docs
   - Alternative documentation: http://localhost:8000/redoc

## API Endpoints

| Method | Endpoint                                                          | Description                                                         |
| ------ | ----------------------------------------------------------------- | ------------------------------------------------------------------- |
| GET    | `/activities`                                                     | Get all activities and their current participants                   |
| POST   | `/auth/login`                                                      | Verify a teacher username and password                              |
| POST   | `/activities/{activity_name}/signup?email=student@mergington.edu` | Register a student (teacher authentication required)                |
| DELETE | `/activities/{activity_name}/unregister?email=student@mergington.edu` | Unregister a student (teacher authentication required)            |

## Assigning teacher credentials

Teacher accounts are provisioned by editing `src/teachers.json`; there is no
account-management page or default teacher account. Passwords are stored as
PBKDF2-SHA256 hashes with a per-account random salt. Generate a salt and hash
for each assigned password with:

```sh
python -c 'import hashlib, getpass, secrets; p=getpass.getpass("Password: "); s=secrets.token_bytes(16); print("salt:", s.hex()); print("password_hash:", hashlib.pbkdf2_hmac("sha256", p.encode(), s, 600000).hex())'
```

Add the resulting values to the `teachers` object, keyed by username:

```json
{
  "teachers": {
    "teacher1": {
      "salt": "the-generated-salt",
      "password_hash": "the-generated-hash"
    }
  }
}
```

The browser keeps the teacher login only in memory, so teachers need to log in
again after reloading the page. Serve the application over HTTPS outside local
development because teacher credentials authorize registration changes.

## Data Model

The application uses a simple data model with meaningful identifiers:

1. **Activities** - Uses activity name as identifier:

   - Description
   - Schedule
   - Maximum number of participants allowed
   - List of student emails who are signed up

2. **Students** - Uses email as identifier:
   - Name
   - Grade level

Activity data is stored in memory, which means registrations will be reset when
the server restarts. Teacher credentials are read from `src/teachers.json`.

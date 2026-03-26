---

# Smart Skill Exchange Platform 

## Project Description

This is a simple **Smart Skill Exchange Platform** written in Python. It helps users connect with others to teach and learn skills efficiently.

Users can **register, login, add skills they can teach, add skills they want to learn, view users, find matching partners, and rate users**.

The system stores user data and ratings using **file handling**, making it persistent for future use.

---

## Files in the Project

* **skill_exchange.py** – Main Python program
* **users.txt** – Stores user credentials and skills
* **ratings.txt** – Stores ratings given to users

---

## Features

* User registration and login
* Add or update skills: teach and learn
* View all users with their skills
* Find skill exchange matches
* Rate skill partners after exchange
* Show average rating for each user
* File handling for persistent storage

---

## Requirements

* Python 3 installed on your system

---

## How to Run

```bash
python skill_exchange.py
```

---

## Login Details (for testing)

| Username | Password |
| -------- | -------- |
| admin    | 1234     |



---

## Code

```
import tkinter as tk
from tkinter import messagebox, simpledialog
import os

# Files
USER_FILE = "users.txt"
RATING_FILE = "ratings.txt"

# Ensure files exist
for file in [USER_FILE, RATING_FILE]:
    if not os.path.exists(file):
        open(file, "w").close()

# --------------------- Helper Functions ---------------------
def register_user(username, password):
    with open(USER_FILE, "r") as f:
        for line in f:
            if line.strip().split(",")[0] == username:
                return False  # Username exists
    with open(USER_FILE, "a") as f:
        f.write(f"{username},{password},,,\n")
    return True

def login_user(username, password):
    with open(USER_FILE, "r") as f:
        for line in f:
            parts = line.strip().split(",")
            if parts[0] == username and parts[1] == password:
                return True
    return False

def update_skills(username, teach, learn):
    lines = []
    with open(USER_FILE, "r") as f:
        for line in f:
            parts = line.strip().split(",")
            if parts[0] == username:
                lines.append(f"{username},{parts[1]},{teach},{learn}\n")
            else:
                lines.append(line)
    with open(USER_FILE, "w") as f:
        f.writelines(lines)

def view_users():
    data = ""
    with open(USER_FILE, "r") as f:
        for line in f:
            parts = line.strip().split(",")
            if len(parts) >= 4 and parts[2] and parts[3]:
                data += f"{parts[0]} | Teaches: {parts[2]} | Wants: {parts[3]}\n"
    if data == "":
        data = "No users with skills added."
    messagebox.showinfo("Users List", data)

def find_matches(username):
    matches = ""
    with open(USER_FILE, "r") as f:
        users = [line.strip().split(",") for line in f if len(line.strip().split(","))>=4]
    user_skill = None
    for u in users:
        if u[0] == username:
            user_skill = u
            break
    if not user_skill or not user_skill[2] or not user_skill[3]:
        messagebox.showinfo("Matches", "Add your skills first!")
        return
    for u in users:
        if u[0] != username:
            if user_skill[2].lower() == u[3].lower() and user_skill[3].lower() == u[2].lower():
                matches += f"{username} ↔ {u[0]}\n"
    if matches == "":
        matches = "No matches found"
    messagebox.showinfo("Matches", matches)

def rate_user(username):
    ratee = simpledialog.askstring("Rate User", "Enter username to rate:")
    rating = simpledialog.askinteger("Rate User", "Rate 1-5:")
    if not ratee or not rating or rating<1 or rating>5:
        messagebox.showerror("Error", "Invalid input")
        return
    with open(RATING_FILE, "a") as f:
        f.write(f"{ratee},{rating}\n")
    messagebox.showinfo("Success", f"{ratee} rated {rating} stars")

def show_average_rating():
    ratings = {}
    with open(RATING_FILE, "r") as f:
        for line in f:
            user, rate = line.strip().split(",")
            rate = int(rate)
            if user not in ratings:
                ratings[user] = []
            ratings[user].append(rate)
    data = ""
    for user, r in ratings.items():
        avg = sum(r)/len(r)
        data += f"{user} → Average Rating: {avg:.1f}\n"
    if data == "":
        data = "No ratings yet"
    messagebox.showinfo("Ratings", data)

# --------------------- GUI ---------------------
root = tk.Tk()
root.title("Skill Exchange Platform 🌍")
root.geometry("450x500")
root.configure(bg="#e6f2ff")

tk.Label(root, text="Skill Exchange Platform", font=("Arial",14,"bold"), bg="#e6f2ff").pack(pady=10)

# --- Login/Register Frame ---
frame_login = tk.Frame(root, bg="#e6f2ff")
frame_login.pack(pady=5)

tk.Label(frame_login, text="Username", bg="#e6f2ff").grid(row=0,column=0)
entry_username = tk.Entry(frame_login)
entry_username.grid(row=0,column=1)

tk.Label(frame_login, text="Password", bg="#e6f2ff").grid(row=1,column=0)
entry_password = tk.Entry(frame_login, show="*")
entry_password.grid(row=1,column=1)

def handle_register():
    user = entry_username.get()
    pw = entry_password.get()
    if register_user(user, pw):
        messagebox.showinfo("Success", "Registered successfully")
    else:
        messagebox.showerror("Error", "Username exists")

def handle_login():
    user = entry_username.get()
    pw = entry_password.get()
    if login_user(user, pw):
        messagebox.showinfo("Success", "Login successful")
        frame_login.pack_forget()
        show_main_interface(user)
    else:
        messagebox.showerror("Error", "Invalid credentials")

tk.Button(frame_login, text="Register", bg="green", fg="white", command=handle_register).grid(row=2,column=0,pady=5)
tk.Button(frame_login, text="Login", bg="blue", fg="white", command=handle_login).grid(row=2,column=1,pady=5)

# --- Main Interface ---
def show_main_interface(user):
    frame_main = tk.Frame(root, bg="#e6f2ff")
    frame_main.pack(pady=10)

    tk.Label(frame_main, text=f"Welcome, {user}", font=("Arial",12,"bold"), bg="#e6f2ff").pack(pady=5)

    # Add Skills
    tk.Label(frame_main, text="Skill You Teach", bg="#e6f2ff").pack()
    entry_teach = tk.Entry(frame_main)
    entry_teach.pack()

    tk.Label(frame_main, text="Skill You Want to Learn", bg="#e6f2ff").pack()
    entry_learn = tk.Entry(frame_main)
    entry_learn.pack()

    tk.Button(frame_main, text="Add / Update Skills", bg="orange", fg="white",
              command=lambda: update_skills(user, entry_teach.get(), entry_learn.get())).pack(pady=5)

    tk.Button(frame_main, text="View Users", bg="purple", fg="white", command=view_users).pack(pady=5)
    tk.Button(frame_main, text="Find Matches", bg="green", fg="white", command=lambda: find_matches(user)).pack(pady=5)
    tk.Button(frame_main, text="Rate User", bg="blue", fg="white", command=lambda: rate_user(user)).pack(pady=5)
    tk.Button(frame_main, text="Show Average Ratings", bg="pink", fg="white", command=show_average_rating).pack(pady=5)

root.mainloop()
```

---

## Output
<img width="1907" height="1004" alt="Screenshot 2026-03-26 183352" src="https://github.com/user-attachments/assets/56837d41-6605-4e34-96c9-1d4e4281579f" />

<img width="312" height="223" alt="Screenshot 2026-03-26 183930" src="https://github.com/user-attachments/assets/50716335-2f77-4f28-8afa-d22f7209e333" />

<img width="257" height="221" alt="Screenshot 2026-03-26 184436" src="https://github.com/user-attachments/assets/4aeaefdb-18fe-4527-bab0-9ed706f7f763" />

<img width="1908" height="799" alt="Screenshot 2026-03-26 183710" src="https://github.com/user-attachments/assets/d96d553a-d5d2-42cd-b7a3-d959156d45cf" />

<img width="456" height="230" alt="Screenshot 2026-03-26 183541" src="https://github.com/user-attachments/assets/4cf9d57e-59a3-4e93-8b29-61dcc33e7c3b" />

<img width="360" height="225" alt="Screenshot 2026-03-26 183734" src="https://github.com/user-attachments/assets/0bb4ed3c-06cd-4ef0-bac5-6e6573b27228" />

<img width="572" height="327" alt="Screenshot 2026-03-26 183954" src="https://github.com/user-attachments/assets/fdf13fc0-dd6f-4fbe-ba25-c7c6c04385bd" />

<img width="312" height="223" alt="Screenshot 2026-03-26 183930" src="https://github.com/user-attachments/assets/50716335-2f77-4f28-8afa-d22f7209e333" />

<img width="257" height="221" alt="Screenshot 2026-03-26 184436" src="https://github.com/user-attachments/assets/4aeaefdb-18fe-4527-bab0-9ed706f7f763" />

<img width="1919" height="999" alt="image" src="https://github.com/user-attachments/assets/4445b713-ea96-461e-8c94-7d004f788ae2" />



<img width="456" height="225" alt="Screenshot 2026-03-26 184038" src="https://github.com/user-attachments/assets/88f900d8-ac7d-4e9e-b5ca-03a042495e0d" />

<img width="273" height="222" alt="Screenshot 2026-03-26 184113" src="https://github.com/user-attachments/assets/55bd5c41-6ef9-487c-88b2-287e4f874d37" />



## Result

The **Smart Skill Exchange Platform** application was successfully implemented using Python.

The program allows users to:

* Connect with skill partners efficiently
* Store and update user data using file handling
* Find suitable skill exchange matches automatically
* Rate users and track performance

The system is **useful for real-world learning, peer education, and skill development**, supporting SDG-4 (Quality Education) and SDG-8 (Decent Work and Economic Growth).

---

## Author

Priyanka S 212224040255

---


Do you want me to do that next?

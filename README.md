# Social-media-analytics-dashboard-
import instaloader
import tkinter as tk
from tkinter import messagebox
from tkinter import simpledialog
import threading
import time

# Initialize Instaloader
L = instaloader.Instaloader()
L.context.user_agent = "Mozilla/5.0 (Linux; Android 10; SM-G973F) AppleWebKit/537.36 Chrome/88.0.4324.93 Mobile Safari/537.36"

# Login function
def login_instagram(username, password):
    try:
        L.login(username, password)
        return True
    except instaloader.exceptions.BadCredentialsException:
        messagebox.showerror("Login Failed", "Wrong username or password.")
    except instaloader.exceptions.ConnectionException as ce:
        messagebox.showerror("Connection Error", str(ce))
    return False

# Get profile information
def get_profile_information(username):
    try:
        profile = instaloader.Profile.from_username(L.context, username)
        return profile
    except Exception as e:
        messagebox.showerror("Error", f"Unable to load profile: {e}")
        return None

# Fetch post data
def fetch_post_data(profile, choice, max_posts, output_text):
    output_text.insert(tk.END, f"\nFetching last {max_posts} posts...\n")
    for i, post in enumerate(profile.get_posts()):
        if i >= max_posts:
            break

        data = {
            "post_id": post.mediaid,
            "likes": post.likes,
            "comments": post.comments,
            "caption": post.caption
        }

        output_text.insert(tk.END, f"\nPost {i + 1}:\n")
        output_text.insert(tk.END, f"Post ID: {data['post_id']}\n")

        if choice == "1":
            output_text.insert(tk.END, f"Likes: {data['likes']}\n")
        elif choice == "2":
            output_text.insert(tk.END, f"Comments: {data['comments']}\n")
        elif choice == "3":
            output_text.insert(tk.END, f"Caption: {data['caption']}\n")
        elif choice == "4":
            output_text.insert(tk.END, f"Likes: {data['likes']}\n")
            output_text.insert(tk.END, f"Comments: {data['comments']}\n")
            output_text.insert(tk.END, f"Caption: {data['caption']}\n")
        else:
            output_text.insert(tk.END, "Invalid choice.\n")
            break

        time.sleep(5)  # Respect Instagram rate limits

# Thread-safe execution
def start_fetching():
    username = username_entry.get()
    password = password_entry.get()
    choice = metric_var.get()
    try:
        max_posts = int(post_count_entry.get())
    except ValueError:
        messagebox.showerror("Input Error", "Enter a valid number for post count.")
        return

    if not login_instagram(username, password):
        return

    profile = get_profile_information(username)
    if profile is None:
        return

    output_text.delete(1.0, tk.END)
    fetch_post_data(profile, choice, max_posts, output_text)

def run_in_thread():
    threading.Thread(target=start_fetching).start()

# --- GUI ---
root = tk.Tk()
root.title("Instagram Scraper")

tk.Label(root, text="Instagram Username:").grid(row=0, column=0, sticky="e")
username_entry = tk.Entry(root, width=30)
username_entry.grid(row=0, column=1)

tk.Label(root, text="Instagram Password:").grid(row=1, column=0, sticky="e")
password_entry = tk.Entry(root, show="*", width=30)
password_entry.grid(row=1, column=1)

tk.Label(root, text="Number of Posts:").grid(row=2, column=0, sticky="e")
post_count_entry = tk.Entry(root, width=30)
post_count_entry.grid(row=2, column=1)

tk.Label(root, text="Select Metrics:").grid(row=3, column=0, sticky="ne")
metric_var = tk.StringVar(value="4")
tk.Radiobutton(root, text="Likes", variable=metric_var, value="1").grid(row=3, column=1, sticky="w")
tk.Radiobutton(root, text="Comments", variable=metric_var, value="2").grid(row=4, column=1, sticky="w")
tk.Radiobutton(root, text="Caption", variable=metric_var, value="3").grid(row=5, column=1, sticky="w")
tk.Radiobutton(root, text="All", variable=metric_var, value="4").grid(row=6, column=1, sticky="w")

fetch_button = tk.Button(root, text="Fetch Posts", command=run_in_thread)
fetch_button.grid(row=7, column=0, columnspan=2, pady=10)

output_text = tk.Text(root, height=20, width=70)
output_text.grid(row=8, column=0, columnspan=2, padx=10, pady=10)

root.mainloop()

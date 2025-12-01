

# app.py
import streamlit as st
from datetime import date, datetime
import json
import os
import uuid

DATA_FILE = "data.json"

# --------------------
# Utility: Data store
# --------------------
def load_data():
    """Load data from disk into a Python dict. Return default structure if missing."""
    default = {
        "todos": [],        # list of dicts: {id, text, done, created_at}
        "habits": []        # list of dicts: {id, name, created_at, completed_dates: [ISO dates strings]}
    }
    if os.path.exists(DATA_FILE):
        try:
            with open(DATA_FILE, "r", encoding="utf-8") as f:
                raw = json.load(f)
            # merge with defaults to avoid schema break
            for k, v in default.items():
                if k not in raw:
                    raw[k] = v
            return raw
        except Exception:
            # If file corrupt, back it up and start fresh
            backup_name = f"{DATA_FILE}.bak.{int(datetime.now().timestamp())}"
            os.rename(DATA_FILE, backup_name)
            return default
    else:
        return default


def save_data(data):
    """Save data (dict) to disk."""
    with open(DATA_FILE, "w", encoding="utf-8") as f:
        json.dump(data, f, indent=2, ensure_ascii=False)


# --------------
# Helpers
# --------------
def new_id():
    return str(uuid.uuid4())


def ensure_session_state():
    """Ensure session state contains the app data (loaded once per session)."""
    if "app_data" not in st.session_state:
        st.session_state.app_data = load_data()


def add_todo(text):
    if not text.strip():
        return
    todos = st.session_state.app_data["todos"]
    todos.append({
        "id": new_id(),
        "text": text.strip(),
        "done": False,
        "created_at": datetime.now().isoformat()
    })
    save_data(st.session_state.app_data)


def delete_todo(todo_id):
    st.session_state.app_data["todos"] = [t for t in st.session_state.app_data["todos"] if t["id"] != todo_id]
    save_data(st.session_state.app_data)


def toggle_todo(todo_id):
    for t in st.session_state.app_data["todos"]:
        if t["id"] == todo_id:
            t["done"] = not t.get("done", False)
            break
    save_data(st.session_state.app_data)


def add_habit(name):
    if not name.strip():
        return
    habits = st.session_state.app_data["habits"]
    habits.append({
        "id": new_id(),
        "name": name.strip(),
        "created_at": datetime.now().isoformat(),
        "completed_dates": []  # store ISO date strings
    })
    save_data(st.session_state.app_data)


def delete_habit(habit_id):
    st.session_state.app_data["habits"] = [h for h in st.session_state.app_data["habits"] if h["id"] != habit_id]
    save_data(st.session_state.app_data)


def toggle_habit_today(habit_id, checked):
    today = date.today().isoformat()
    for h in st.session_state.app_data["habits"]:
        if h["id"] == habit_id:
            if checked and today not in h["completed_dates"]:
                h["completed_dates"].append(today)
            elif (not checked) and today in h["completed_dates"]:
                h["completed_dates"].remove(today)
            break
    save_data(st.session_state.app_data)


def clear_all_data(confirm=False):
    if confirm:
        st.session_state.app_data = {"todos": [], "habits": []}
        save_data(st.session_state.app_data)


# -----------------
# UI components
# -----------------
def header():
    st.markdown(
        """
        <div style="display:flex;align-items:center;gap:12px">
          <div style="width:48px;height:48px;border-radius:10px;background:linear-gradient(135deg,#6ee7b7,#3b82f6);
                      display:flex;align-items:center;justify-content:center;color:white;font-weight:700;">
            ✓
          </div>
          <div>
            <h2 style="margin:0;padding:0">To-Do & Habit Tracker</h2>
            <div style="color:gray;margin-top:2px">Simple Streamlit app — add tasks, track habits, and persist during runtime.</div>
          </div>
        </div>
        """,
        unsafe_allow_html=True,
    )


def todo_page():
    st.subheader("To-Do List")
    st.write("Add tasks, mark complete, and delete items. Tasks persist to `data.json` during runtime.")
    with st.form("add_todo_form", clear_on_submit=True):
        new_task = st.text_input("New task", placeholder="e.g., Write report")
        submitted = st.form_submit_button("Add task")
        if submitted:
            add_todo(new_task)
            st.rerun()

    todos = st.session_state.app_data["todos"]
    # Sort with not done first, then created time
    todos_sorted = sorted(todos, key=lambda x: (x.get("done", False), x.get("created_at", "")))
    if not todos_sorted:
        st.info("No tasks yet. Add one above!")
    else:
        for t in todos_sorted:
            cols = st.columns([0.05, 0.8, 0.15])
            checked = cols[0].checkbox("", value=t.get("done", False), key=f"todo_chk_{t['id']}")
            # If checkbox changed, toggle
            if checked != t.get("done", False):
                toggle_todo(t["id"])
                st.rerun()
            text_style = "text-decoration: line-through; color: gray;" if t.get("done", False) else ""
            cols[1].markdown(f"<div style='{text_style}'>{st.session_state.app_data['todos'] and t['text']}</div>", unsafe_allow_html=True)
            if cols[2].button("Delete", key=f"todo_del_{t['id']}"):
                delete_todo(t["id"])
                st.rerun()


def habit_page():
    st.subheader("Habit Tracker")
    st.write("Create daily habits and tick them off for today. Progress shows percentage of habits completed today.")
    with st.form("add_habit_form", clear_on_submit=True):
        habit_name = st.text_input("New habit", placeholder="e.g., Drink water")
        submitted = st.form_submit_button("Add habit")
        if submitted:
            add_habit(habit_name)
            st.rerun()

    habits = st.session_state.app_data["habits"]
    today = date.today().isoformat()

    if not habits:
        st.info("No habits yet. Add a new habit above!")
    else:
        # Display checkboxes and allow toggling
        done_count = 0
        for h in habits:
            checked = (today in h.get("completed_dates", []))
            cols = st.columns([0.05, 0.7, 0.25])
            new_checked = cols[0].checkbox("", value=checked, key=f"habit_chk_{h['id']}")
            if new_checked != checked:
                toggle_habit_today(h["id"], new_checked)
                st.rerun()
            cols[1].markdown(f"**{h['name']}**")
            created = datetime.fromisoformat(h["created_at"]).strftime("%b %d, %Y")
            cols[2].markdown(f"<small>Created: {created}</small>", unsafe_allow_html=True)
            if today in h.get("completed_dates", []):
                done_count += 1

        # Show progress
        progress = (done_count / len(habits)) if habits else 0.0
        st.metric("Today's habits completed", f"{done_count} / {len(habits)}", delta=None)
        st.progress(progress)

        # Quick view to mark any habit for other days (optional advanced feature)
        with st.expander("View habit completion history"):
            for h in habits:
                cd = sorted(h.get("completed_dates", []), reverse=True)
                sample = ", ".join(cd[:10]) if cd else "No completions yet"
                st.write(f"- **{h['name']}**: {sample}")


# -----------------
# Sidebar & Layout
# -----------------
def sidebar():
    st.sidebar.title("Navigation")
    page = st.sidebar.radio("Choose page", ["To-Do List", "Habit Tracker", "Settings"])
    st.sidebar.markdown("---")
    st.sidebar.write("Data storage:")
    st.sidebar.info("Data is stored in `data.json` in the app folder during runtime. On Hugging Face Spaces a container restart will reset uncommitted changes.")
    return page


def settings_page():
    st.subheader("Settings & Data")
    st.write("You can export or reset the app data here.")
    data = st.session_state.app_data

    # Export JSON
    st.download_button("Download data.json", json.dumps(data, indent=2, ensure_ascii=False), file_name="data.json", mime="application/json")

    if st.button("Backup & Reset (clear all tasks & habits)"):
        st.warning("This will delete all to-dos and habits from the current runtime.", icon="⚠️")
        if st.button("Confirm: Clear all data"):
            clear_all_data(confirm=True)
            st.success("All data cleared.")
            st.rerun()

    st.markdown("### Current data preview")
    st.write(data)


# -----------------
# App entry point
# -----------------
def main():
    st.set_page_config(page_title="To-Do & Habit Tracker", page_icon="✅", layout="centered")
    ensure_session_state()
    header()
    page = sidebar()
    if page == "To-Do List":
        todo_page()
    elif page == "Habit Tracker":
        habit_page()
    elif page == "Settings":
        settings_page()
    else:
        st.write("Page not found")


if __name__ == "__main__":
    main()


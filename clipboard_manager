import pyperclip
import keyboard
import threading
import json
import os
import time
import tkinter as tk
from pystray import Icon, MenuItem, Menu
from PIL import Image

MAX_ITEMS = 500
SAVE_FILE = "clipboard_data.json"

history = []
favorites = set()
last_clip = ""

# --------------------
# Load / Save
# --------------------

def load_data():
    global history, favorites

    if os.path.exists(SAVE_FILE):
        with open(SAVE_FILE,"r",encoding="utf8") as f:
            data = json.load(f)
            history = data.get("history",[])
            favorites = set(data.get("favorites",[]))

def save_data():
    with open(SAVE_FILE,"w",encoding="utf8") as f:
        json.dump({
            "history":history,
            "favorites":list(favorites)
        },f)

# --------------------
# Clipboard Monitor
# --------------------

def monitor_clipboard():
    global last_clip

    while True:
        try:
            text = pyperclip.paste()

            if text != last_clip and text.strip():
                last_clip = text

                if text not in history:
                    history.insert(0,text)

                    if len(history) > MAX_ITEMS:
                        history.pop()

                    save_data()
        except:
            pass

        time.sleep(0.5)

# --------------------
# UI
# --------------------

def open_window():

    root = tk.Tk()
    root.title("Clipboard Manager")
    root.geometry("600x450")

    search_var = tk.StringVar()

    top = tk.Frame(root)
    top.pack(fill="x")

    search = tk.Entry(top,textvariable=search_var)
    search.pack(fill="x",padx=5,pady=5)

    listbox = tk.Listbox(root,font=("Consolas",10))
    listbox.pack(fill="both",expand=True,padx=5,pady=5)

    def refresh():

        listbox.delete(0,tk.END)

        term = search_var.get().lower()

        for item in history:

            if term in item.lower():

                star = "⭐ " if item in favorites else ""
                preview = item.replace("\n"," ")[:100]

                listbox.insert(tk.END,star + preview)

    def copy_item(event=None):

        sel = listbox.curselection()

        if sel:
            text = history[sel[0]]
            pyperclip.copy(text)
            root.destroy()

    def toggle_fav(event=None):

        sel = listbox.curselection()

        if sel:
            item = history[sel[0]]

            if item in favorites:
                favorites.remove(item)
            else:
                favorites.add(item)

            save_data()
            refresh()

    search_var.trace("w",lambda *args: refresh())

    listbox.bind("<Double-Button-1>",copy_item)

    root.bind("<Return>",copy_item)
    root.bind("f",toggle_fav)

    refresh()

    root.mainloop()

# --------------------
# Tray
# --------------------

def quit_app(icon,item):
    icon.stop()
    os._exit(0)

def tray():

    image = Image.new("RGB",(64,64),(60,60,60))

    menu = Menu(
        MenuItem("Open Clipboard",lambda: threading.Thread(target=open_window).start()),
        MenuItem("Quit",quit_app)
    )

    icon = Icon("ClipboardManager",image,menu=menu)

    icon.run()

# --------------------
# Start
# --------------------

load_data()

threading.Thread(target=monitor_clipboard,daemon=True).start()

keyboard.add_hotkey("ctrl+shift+v",lambda: threading.Thread(target=open_window).start())

threading.Thread(target=tray,daemon=True).start()

keyboard.wait()

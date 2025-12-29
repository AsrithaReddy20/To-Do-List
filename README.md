# To-Do-List
A modern Tkinter checklist with a mobile-inspired "My Day" UI. It features interactive circular checkboxes, a scrollable task area, and dynamic strikethrough effects. A robust backend automatically manages a /tasks/ directory to ensure persistent data storage, while sleek white task cards provide a clean, responsive user experience.
import tkinter as tk
from tkinter import messagebox
import os

# --- Modern Color Palette ---
BG_COLOR = "#F7F9FC"      
CARD_COLOR = "#FFFFFF"    
TEXT_COLOR = "#2D3436"    
CIRCLE_COLOR = "#B2BEC3"  
DONE_COLOR = "#2ECC71"    
STRIKE_COLOR = "#A2A2A2"  

class TodoApp:
    def __init__(self, root):
        self.root = root
        self.root.title("My Day - Checklist")
        self.root.geometry("400x600")
        self.root.configure(bg=BG_COLOR)
        
        # --- BACKEND SETUP ---
        # Create a folder called 'tasks' if it doesn't exist
        self.folder_name = "tasks"
        if not os.path.exists(self.folder_name):
            os.makedirs(self.folder_name)
        
        self.file_path = os.path.join(self.folder_name, "task_data.txt")
        
        # Header
        tk.Label(root, text="My Day", font=("Helvetica", 24, "bold"), 
                 bg=BG_COLOR, fg="#2980B9").pack(pady=(20, 5), padx=20, anchor="w")
        
        # Entry Area
        self.entry_frame = tk.Frame(root, bg=BG_COLOR)
        self.entry_frame.pack(fill="x", padx=20, pady=10)
        
        self.task_entry = tk.Entry(self.entry_frame, font=("Helvetica", 14), bd=0, highlightthickness=1)
        self.task_entry.pack(side="left", fill="x", expand=True, ipady=8)
        self.task_entry.bind("<Return>", lambda e: self.add_task())
        
        tk.Button(self.entry_frame, text="+", command=self.add_task, font=("Helvetica", 18),
                  bg="#3498DB", fg="white", bd=0, width=3, cursor="hand2").pack(side="right", padx=(5, 0))

        # Scrollable Task Area
        self.canvas = tk.Canvas(root, bg=BG_COLOR, highlightthickness=0)
        self.scroll_frame = tk.Frame(self.canvas, bg=BG_COLOR)
        self.scrollbar = tk.Scrollbar(root, orient="vertical", command=self.canvas.yview)
        
        self.canvas.configure(yscrollcommand=self.scrollbar.set)
        self.scrollbar.pack(side="right", fill="y")
        self.canvas.pack(side="left", fill="both", expand=True, padx=10)
        self.canvas.create_window((0, 0), window=self.scroll_frame, anchor="nw")
        
        self.scroll_frame.bind("<Configure>", lambda e: self.canvas.configure(scrollregion=self.canvas.bbox("all")))
        
        self.load_tasks()

    def add_task(self, task_text=None, is_done=False):
        text = task_text if task_text else self.task_entry.get().strip()
        if not text: return

        row = tk.Frame(self.scroll_frame, bg=CARD_COLOR, pady=10, padx=10)
        row.pack(fill="x", pady=5, padx=5)
        
        circle_btn = tk.Canvas(row, width=30, height=30, bg=CARD_COLOR, highlightthickness=0, cursor="hand2")
        circle_btn.pack(side="left")
        
        task_label = tk.Label(row, text=text, font=("Helvetica", 12), bg=CARD_COLOR, fg=TEXT_COLOR)
        task_label.pack(side="left", padx=10)

        del_btn = tk.Button(row, text="✕", command=lambda: self.remove_row(row), 
                           bg=CARD_COLOR, fg="#E74C3C", bd=0, font=("Helvetica", 10), cursor="hand2")
        del_btn.pack(side="right")

        def draw_circle(done):
            circle_btn.delete("all")
            if done:
                circle_btn.create_oval(5, 5, 25, 25, outline=DONE_COLOR, width=2, fill=DONE_COLOR)
                circle_btn.create_text(15, 15, text="✓", fill="white", font=("Helvetica", 10, "bold"))
                task_label.config(fg=STRIKE_COLOR, font=("Helvetica", 12, "overstrike"))
            else:
                circle_btn.create_oval(5, 5, 25, 25, outline=CIRCLE_COLOR, width=2)
                task_label.config(fg=TEXT_COLOR, font=("Helvetica", 12))

        state = {"done": is_done}
        draw_circle(state["done"])

        def toggle(e):
            state["done"] = not state["done"]
            draw_circle(state["done"])
            self.save_tasks()

        circle_btn.bind("<Button-1>", toggle)
        
        if not task_text:
            self.task_entry.delete(0, tk.END)
            self.save_tasks()

    def remove_row(self, row):
        row.destroy()
        # Delay save slightly to allow widget to be destroyed
        self.root.after(10, self.save_tasks)

    def save_tasks(self):
        """Saves tasks to the /tasks/ folder."""
        with open(self.file_path, "w", encoding="utf-8") as f:
            for row in self.scroll_frame.winfo_children():
                if row.winfo_exists():
                    widgets = row.winfo_children()
                    label = widgets[1]
                    is_done = "overstrike" in str(label.cget("font"))
                    f.write(f"{'DONE' if is_done else 'TODO'}|{label.cget('text')}\n")

    def load_tasks(self):
        """Loads tasks from the /tasks/ folder."""
        if os.path.exists(self.file_path):
            with open(self.file_path, "r", encoding="utf-8") as f:
                for line in f:
                    if "|" in line:
                        parts = line.strip().split("|", 1)
                        if len(parts) == 2:
                            status, text = parts
                            self.add_task(text, is_done=(status == "DONE"))

if __name__ == "__main__":
    root = tk.Tk()
    app = TodoApp(root)
    root.mainloop()

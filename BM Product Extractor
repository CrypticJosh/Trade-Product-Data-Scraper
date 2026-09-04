import json
import ctypes
import csv
import sys
import platform
from pathlib import Path
from bs4 import BeautifulSoup
import tkinter as tk
from tkinter import filedialog, messagebox, ttk
import requests

# ---------------- DPI AWARENESS (Windows only) ---------------- #
if platform.system() == "Windows":
    try:
        ctypes.windll.shcore.SetProcessDpiAwareness(2)
    except Exception:
        pass

# ---------------- OPTIONAL DRAG & DROP SUPPORT ---------------- #
try:
    from tkinterdnd2 import TkinterDnD, DND_FILES
    DND_AVAILABLE = True
except ImportError:
    DND_AVAILABLE = False
    TkinterDnD = None
    DND_FILES = None


# ---------------- LICENSE CHECK ---------------- #
def check_app_status(root):
    """
    Verifies license/subscription status before allowing the app to run.
    """
    try:
        url = "https://app-control-6e8a6-default-rtdb.europe-west1.firebasedatabase.app/apps/bm_extractor.json"
        response = requests.get(url, timeout=5)
        status = response.json()

        if status != "OK":
            messagebox.showerror(
                "License Required",
                "A valid license is required to continue using this application.\n\n"
                "Plans: £19.99 / month or £200 / year.\n\n"
                "Please contact your account administrator or the developer to renew."
            )
            root.destroy()
            sys.exit()

    except Exception:
        messagebox.showerror(
            "Unable to Verify License",
            "This application couldn't verify your license for this session.\n\n"
            "Please check your internet connection and try again. "
            "If the problem continues, contact support."
        )
        root.destroy()
        sys.exit()


# ---------------- EXTRACTION LOGIC (UNCHANGED) ---------------- #

def extract_products(html_text):
    """
    Extract product title, SKU and availability
    from a Bachmann category page.
    """

    soup = BeautifulSoup(html_text, "html.parser")
    products = []

    for h3 in soup.find_all("h3"):

        link = h3.find("a")
        if not link:
            continue

        title = link.get_text(strip=True)

        container = h3.parent

        sku = ""
        availability = ""

        sku_element = container.find(
            "p",
            class_="form-control-static"
        )

        if sku_element:
            sku = sku_element.get_text(strip=True)

        availability_element = container.find(
            "span",
            class_=lambda x: x and "FlexAvailability" in str(x)
        )

        if availability_element:
            availability = availability_element.get_text(strip=True)

        if sku:
            products.append({
                "title": title,
                "sku": sku,
                "availability": availability
            })

    return products


def save_json(products, output_file):
    with open(output_file, "w", encoding="utf-8") as f:
        json.dump(
            products,
            f,
            indent=2,
            ensure_ascii=False
        )


def save_csv(products, output_file):
    with open(
        output_file,
        "w",
        newline="",
        encoding="utf-8"
    ) as f:

        writer = csv.DictWriter(
            f,
            fieldnames=[
                "title",
                "sku",
                "availability"
            ]
        )

        writer.writeheader()
        writer.writerows(products)


# ---------------- DESIGN TOKENS ---------------- #
# Light, neutral, corporate palette. No neon, no glow, no emoji.

BG = "#f8fafc"           # slate-50 — app background
SURFACE = "#ffffff"      # card / panel background
SURFACE_ALT = "#f1f5f9"  # slate-100 — secondary surface (log rows, inputs)
BORDER = "#e2e8f0"       # slate-200
BORDER_STRONG = "#cbd5e1"  # slate-300
TEXT_MAIN = "#0f172a"    # slate-900
TEXT_MUTED = "#64748b"   # slate-500
ACCENT = "#2563eb"       # blue-600 — primary action
ACCENT_HOVER = "#1d4ed8" # blue-700
ACCENT_SOFT = "#eff6ff"  # blue-50 — logo chip / subtle highlight
SUCCESS = "#15803d"      # green-700
SUCCESS_BG = "#f0fdf4"
ERROR = "#b91c1c"        # red-700
ERROR_BG = "#fef2f2"
FONT_FAMILY = "Segoe UI"


# ---------------- APP ---------------- #

class ExtractorApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Bachmann Product Extractor")

        base_w, base_h = 600, 660
        base_min_w, base_min_h = 520, 600

        zoom = 1.25

        w = int(base_w * zoom)
        h = int(base_h * zoom)
        min_w = int(base_min_w * zoom)
        min_h = int(base_min_h * zoom)

        self.root.geometry(f"{w}x{h}")
        self.root.minsize(min_w, min_h)
        self.root.configure(bg=BG)

        self._build_style()
        self._build_ui()

        if DND_AVAILABLE:
            self.drop_zone.drop_target_register(DND_FILES)
            self.drop_zone.dnd_bind("<<Drop>>", self._on_drop)

    # ---------- styling ---------- #
    def _build_style(self):
        style = ttk.Style(self.root)
        try:
            style.theme_use("clam")
        except tk.TclError:
            pass

        style.configure(
            "Primary.TButton",
            background=ACCENT,
            foreground="white",
            font=(FONT_FAMILY, 11, "bold"),
            padding=(18, 10),
            borderwidth=0,
            relief="flat",
        )
        style.map(
            "Primary.TButton",
            background=[("active", ACCENT_HOVER), ("pressed", ACCENT_HOVER)],
        )

        style.configure(
            "Secondary.TButton",
            background=SURFACE,
            foreground=TEXT_MAIN,
            font=(FONT_FAMILY, 10),
            padding=(14, 8),
            borderwidth=1,
            relief="flat",
        )
        style.map(
            "Secondary.TButton",
            background=[("active", SURFACE_ALT), ("pressed", SURFACE_ALT)],
        )

        style.configure(
            "Clean.Horizontal.TProgressbar",
            troughcolor=SURFACE_ALT,
            background=ACCENT,
            borderwidth=0,
            thickness=6,
        )

    # ---------- layout ---------- #
    def _build_ui(self):
        # ----- Top bar with simple brand mark ----- #
        topbar = tk.Frame(self.root, bg=SURFACE, highlightbackground=BORDER, highlightthickness=1)
        topbar.pack(fill="x", side="top")

        topbar_inner = tk.Frame(topbar, bg=SURFACE)
        topbar_inner.pack(fill="x", padx=24, pady=16)

        # Simple square "logo" mark — initials, no emoji/icon font dependency
        logo = tk.Label(
            topbar_inner,
            text="BM",
            font=(FONT_FAMILY, 12, "bold"),
            fg=ACCENT,
            bg=ACCENT_SOFT,
            width=3,
            height=1,
        )
        logo.pack(side="left", ipady=4)

        title_block = tk.Frame(topbar_inner, bg=SURFACE)
        title_block.pack(side="left", padx=(12, 0))

        tk.Label(
            title_block,
            text="Bachmann Product Extractor",
            font=(FONT_FAMILY, 13, "bold"),
            fg=TEXT_MAIN,
            bg=SURFACE,
        ).pack(anchor="w")

        tk.Label(
            title_block,
            text="Product data extraction tool",
            font=(FONT_FAMILY, 10),
            fg=TEXT_MUTED,
            bg=SURFACE,
        ).pack(anchor="w")

        # ----- Body ----- #
        body = tk.Frame(self.root, bg=BG)
        body.pack(fill="both", expand=True, padx=24, pady=20)

        tk.Label(
            body,
            text="Import saved pages",
            font=(FONT_FAMILY, 11, "bold"),
            fg=TEXT_MAIN,
            bg=BG,
        ).pack(anchor="w", pady=(0, 4))

        tk.Label(
            body,
            text="Drop .html or .txt files below, or browse to select them.",
            font=(FONT_FAMILY, 10),
            fg=TEXT_MUTED,
            bg=BG,
        ).pack(anchor="w", pady=(0, 12))

        # Drop zone — flat card, dashed-style border via highlight
        self.drop_zone = tk.Frame(
            body,
            bg=SURFACE,
            highlightbackground=BORDER_STRONG,
            highlightcolor=ACCENT,
            highlightthickness=1,
            bd=0,
        )
        self.drop_zone.pack(fill="x")
        self.drop_zone.pack_propagate(False)
        self.drop_zone.configure(height=170)

        drop_content = tk.Frame(self.drop_zone, bg=SURFACE)
        drop_content.place(relx=0.5, rely=0.5, anchor="center")

        drop_msg = (
            "Drag and drop files here"
            if DND_AVAILABLE
            else "Drag and drop unavailable in this build"
        )
        self.drop_label = tk.Label(
            drop_content,
            text=drop_msg,
            font=(FONT_FAMILY, 11),
            fg=TEXT_MAIN,
            bg=SURFACE,
        )
        self.drop_label.pack(pady=(0, 10))

        browse_btn = ttk.Button(
            drop_content,
            text="Browse Files",
            style="Primary.TButton",
            command=self.browse_files,
        )
        browse_btn.pack()

        for widget in (self.drop_zone, drop_content, self.drop_label):
            widget.bind("<Button-1>", lambda e: self.browse_files())

        # Progress bar (hidden until processing)
        self.progress = ttk.Progressbar(
            body,
            style="Clean.Horizontal.TProgressbar",
            mode="determinate",
        )

        # ----- Results card ----- #
        results_frame = tk.Frame(
            body, bg=SURFACE, highlightbackground=BORDER, highlightthickness=1
        )
        results_frame.pack(fill="both", expand=True, pady=(16, 0))

        results_header = tk.Frame(results_frame, bg=SURFACE)
        results_header.pack(fill="x", padx=16, pady=(14, 8))

        tk.Label(
            results_header,
            text="Activity",
            font=(FONT_FAMILY, 10, "bold"),
            fg=TEXT_MAIN,
            bg=SURFACE,
        ).pack(side="left")

        self.summary_label = tk.Label(
            results_header,
            text="No files processed yet",
            font=(FONT_FAMILY, 10),
            fg=TEXT_MUTED,
            bg=SURFACE,
        )
        self.summary_label.pack(side="right")

        divider = tk.Frame(results_frame, bg=BORDER, height=1)
        divider.pack(fill="x")

        list_container = tk.Frame(results_frame, bg=SURFACE)
        list_container.pack(fill="both", expand=True, padx=16, pady=(8, 16))

        scrollbar = tk.Scrollbar(list_container)
        scrollbar.pack(side="right", fill="y")

        self.log_list = tk.Listbox(
            list_container,
            bg=SURFACE_ALT,
            fg=TEXT_MAIN,
            selectbackground=ACCENT_SOFT,
            selectforeground=TEXT_MAIN,
            font=("Consolas", 10),
            bd=0,
            highlightthickness=0,
            yscrollcommand=scrollbar.set,
            activestyle="none",
        )
        self.log_list.pack(side="left", fill="both", expand=True)
        scrollbar.config(command=self.log_list.yview)

        # ----- Footer ----- #
        footer = tk.Frame(self.root, bg=BG)
        footer.pack(fill="x", padx=24, pady=(0, 18))

        tk.Label(
            footer,
            text="© 2026 Joshua Cheetham",
            font=(FONT_FAMILY, 9),
            fg=TEXT_MUTED,
            bg=BG,
        ).pack(side="left")

        help_btn = ttk.Button(
            footer,
            text="Instructions",
            style="Secondary.TButton",
            command=self.show_instructions,
        )
        help_btn.pack(side="right")

    # ---------- helpers ---------- #
    def show_instructions(self):
        messagebox.showinfo(
            "Instructions",
            "1. Go to the relevant Bachmann page.\n"
            "2. Press Ctrl+U, then on the opened page press Ctrl+A then Ctrl+C.\n"
            "3. Paste into a text file named after the page, e.g. \"Coaches Page 1\".\n"
            "4. Once done for all pages needed, drag them into this window "
            "(or use Browse Files).\n"
            "5. CSV and JSON files are exported to the same folder, using the "
            "same file name as the source."
        )

    def browse_files(self):
        filenames = filedialog.askopenfilenames(
            title="Select Bachmann Pages",
            filetypes=[
                ("HTML/TXT Files", "*.html *.htm *.txt"),
                ("All Files", "*.*")
            ]
        )
        if filenames:
            self.run_batch(filenames)

    def _on_drop(self, event):
        raw = event.data
        paths = self.root.tk.splitlist(raw)
        valid_ext = {".html", ".htm", ".txt"}
        filenames = [p for p in paths if Path(p).suffix.lower() in valid_ext]

        if not filenames:
            self._log("No supported files (.html/.htm/.txt) were dropped.", ERROR)
            return

        self.run_batch(filenames)

    def _log(self, message, color=TEXT_MAIN):
        self.log_list.insert("end", message)
        self.log_list.itemconfig("end", fg=color)
        self.log_list.see("end")

    def _set_processing_visual(self, active):
        if active:
            self.drop_zone.configure(highlightbackground=ACCENT)
            self.progress.pack(fill="x", pady=(12, 0))
            self.progress["value"] = 0
        else:
            self.drop_zone.configure(highlightbackground=BORDER_STRONG)
            self.progress.pack_forget()

    # ---------- core batch processing (logic unchanged) ---------- #
    def run_batch(self, filenames):
        self._set_processing_visual(True)
        self.progress["maximum"] = len(filenames)

        success_count = 0
        total_products = 0
        failed_files = []

        for i, filename in enumerate(filenames, start=1):
            name = Path(filename).name
            try:
                html = Path(filename).read_text(
                    encoding="utf-8",
                    errors="ignore"
                )

                products = extract_products(html)

                source = Path(filename)

                json_file = source.with_suffix(".json")
                csv_file = source.with_suffix(".csv")

                save_json(products, json_file)
                save_csv(products, csv_file)

                success_count += 1
                total_products += len(products)

                self._log(f"Done   {name} — {len(products)} products", SUCCESS)

            except Exception as e:
                failed_files.append(f"{name}: {str(e)}")
                self._log(f"Failed  {name} — {str(e)}", ERROR)

            self.progress["value"] = i
            self.root.update_idletasks()

        self._set_processing_visual(False)

        self.summary_label.configure(
            text=f"{success_count} file(s) processed · {total_products} product(s) found"
        )

        message = (
            f"Files Processed: {success_count}\n"
            f"Products Found: {total_products}"
        )

        if failed_files:
            message += "\n\nFailed Files:\n"
            message += "\n".join(failed_files)

        messagebox.showinfo(
            "Batch Processing Complete",
            message
        )


# ---------------- ENTRY POINT ---------------- #

def main():
    if DND_AVAILABLE:
        root = TkinterDnD.Tk()
    else:
        root = tk.Tk()

    # Hide the window until the license check resolves
    root.withdraw()

    check_app_status(root)

    if not DND_AVAILABLE:
        messagebox.showinfo(
            "Drag and Drop Unavailable",
            "The 'tkinterdnd2' package is not installed, so drag-and-drop is disabled.\n\n"
            "You can still use the Browse Files button."
        )

    app = ExtractorApp(root)
    root.deiconify()
    root.mainloop()


if __name__ == "__main__":
    main()

import cv2
import glob
import os
import threading
import tkinter as tk
from tkinter import ttk, filedialog, messagebox

# ── Colors & Constants ────────────────────────────────────────────────────────
BG        = "#1a1a2e"
PANEL     = "#16213e"
CARD      = "#0f3460"
ACCENT    = "#e94560"
ACCENT2   = "#f5a623"
FG        = "#eaeaea"
FG_DIM    = "#8892a4"
SUCCESS   = "#4ecca3"
FONT_MAIN = ("Segoe UI", 10)
FONT_H1   = ("Segoe UI", 16, "bold")
FONT_H2   = ("Segoe UI", 11, "bold")
FONT_MONO = ("Consolas", 9)

# ── Interpolation methods ─────────────────────────────────────────────────────
INTERPOLATION_MAP = {
    "NEAREST        - Fastest, pixelated edges":   cv2.INTER_NEAREST,
    "NEAREST_EXACT  - Precise nearest-neighbor":   cv2.INTER_NEAREST_EXACT,
    "LINEAR         - Fast, balanced quality":     cv2.INTER_LINEAR,
    "LINEAR_EXACT   - Exact bilinear (slower)":    cv2.INTER_LINEAR_EXACT,
    "CUBIC          - High quality, slower":       cv2.INTER_CUBIC,
    "AREA           - Best for downscaling":       cv2.INTER_AREA,
    "LANCZOS4       - Highest quality, slowest":   cv2.INTER_LANCZOS4,
}
METHOD_KEYS  = list(INTERPOLATION_MAP.keys())
METHOD_SHORT = [k.split()[0] for k in METHOD_KEYS]

# ── Supported file extensions ─────────────────────────────────────────────────
SUPPORTED_EXTS = [
    ".png", ".jpg", ".jpeg", ".bmp",
    ".tiff", ".tif", ".webp",
    ".ppm", ".pgm", ".pbm",
    ".exr", ".hdr",
    ".sr", ".ras",
    ".jp2",
]


# ── Helper: non-conflicting output folder ─────────────────────────────────────
def unique_dir(base):
    """Return base if it doesn't exist, otherwise base_2, base_3, …"""
    if not os.path.exists(base):
        return base
    counter = 2
    while True:
        candidate = f"{base}_{counter}"
        if not os.path.exists(candidate):
            return candidate
        counter += 1


# ── Helper: collect images from folder ───────────────────────────────────────
def collect_images(folder):
    found = []
    for ext in SUPPORTED_EXTS:
        found.extend(glob.glob(os.path.join(folder, f"*{ext}")))
        found.extend(glob.glob(os.path.join(folder, f"*{ext.upper()}")))
    seen = set()
    unique = []
    for path in found:
        key = path.lower()
        if key not in seen:
            seen.add(key)
            unique.append(path)
    return unique


# ── Widget helpers ────────────────────────────────────────────────────────────
def styled_button(parent, text, command, bg=ACCENT, fg="white", **kw):
    btn = tk.Button(
        parent, text=text, command=command,
        bg=bg, fg=fg, activebackground=ACCENT2, activeforeground="white",
        relief="flat", cursor="hand2", font=FONT_H2,
        padx=14, pady=6, **kw
    )
    btn.bind("<Enter>", lambda e: btn.config(bg=ACCENT2))
    btn.bind("<Leave>", lambda e: btn.config(bg=bg))
    return btn


def section_label(parent, text, row):
    tk.Label(parent, text=text, bg=PANEL, fg=FG,
             font=FONT_H2, anchor="w", padx=14
             ).grid(row=row, column=0, columnspan=2,
                    sticky="ew", pady=(10, 2))


def percent_entry(parent, label, col, default):
    """Labeled percentage entry. Returns StringVar."""
    tk.Label(parent, text=label, bg=PANEL, fg=FG_DIM,
             font=FONT_MAIN).grid(row=0, column=col, sticky="w")
    var = tk.StringVar(value=default)
    tk.Entry(parent, textvariable=var, bg=CARD, fg=FG,
             insertbackground=ACCENT, font=FONT_MAIN,
             relief="flat", width=6, highlightthickness=1,
             highlightbackground=CARD, highlightcolor=ACCENT
             ).grid(row=0, column=col + 1, padx=(4, 0), sticky="w", ipady=4)
    return var


def stat_card(parent, label, row, col):
    card = tk.Frame(parent, bg=CARD)
    card.grid(row=row, column=col, padx=4, pady=4, sticky="nsew")
    parent.rowconfigure(row, weight=1)
    tk.Label(card, text=label, bg=CARD, fg=FG_DIM,
             font=FONT_MAIN, pady=6).pack()
    value_label = tk.Label(card, text="0", bg=CARD, fg=ACCENT2,
                           font=("Segoe UI", 22, "bold"), pady=2)
    value_label.pack()
    return value_label


def apply_combo_style():
    style = ttk.Style()
    style.theme_use("default")
    style.configure("TCombobox",
                    fieldbackground=CARD, background=CARD,
                    foreground=FG, selectbackground=ACCENT,
                    selectforeground="white", bordercolor=CARD,
                    arrowcolor=ACCENT, relief="flat")
    style.map("TCombobox",
              fieldbackground=[("readonly", CARD)],
              foreground=[("readonly", FG)],
              background=[("readonly", CARD)])


# ── Main Application ──────────────────────────────────────────────────────────
class App(tk.Tk):
    def __init__(self):
        super().__init__()
        self.title("Batch Image Resizer")
        self.configure(bg=BG)
        self.resizable(True, True)
        self.minsize(700, 560)
        self._running = False
        self._syncing = False   # prevents recursive ratio-lock callbacks
        self._build_ui()
        self._center()

    def _center(self):
        self.update_idletasks()
        w, h = 780, 640
        self.geometry(f"{w}x{h}+"
                      f"{(self.winfo_screenwidth()  - w) // 2}+"
                      f"{(self.winfo_screenheight() - h) // 2}")

    # ── UI Construction ───────────────────────────────────────────────────────
    def _build_ui(self):
        # Header bar
        header = tk.Frame(self, bg=CARD, height=56)
        header.pack(fill="x")
        tk.Label(header, text="Batch Image Resizer",
                 bg=CARD, fg=FG, font=FONT_H1,
                 pady=12, padx=20).pack(side="left")
        tk.Label(header, text="Powered by OpenCV",
                 bg=CARD, fg=FG_DIM, font=FONT_MAIN,
                 pady=12, padx=20).pack(side="right")

        # Body grid
        body = tk.Frame(self, bg=BG)
        body.pack(fill="both", expand=True, padx=20, pady=16)
        body.columnconfigure(0, weight=1)
        body.columnconfigure(1, weight=1)
        body.rowconfigure(2, weight=1)

        # ── Left settings panel ───────────────────────────────────────────────
        left_panel = tk.Frame(body, bg=PANEL)
        left_panel.grid(row=0, column=0, rowspan=2, sticky="nsew",
                        padx=(0, 10))
        left_panel.columnconfigure(1, weight=1)

        # Source folder row
        section_label(left_panel, "Source Folder", 0)
        self.src_var = tk.StringVar()
        folder_row = tk.Frame(left_panel, bg=PANEL)
        folder_row.grid(row=1, column=0, columnspan=2,
                        sticky="ew", padx=14, pady=(0, 10))
        folder_row.columnconfigure(0, weight=1)
        tk.Entry(folder_row, textvariable=self.src_var,
                 bg=CARD, fg=FG, font=FONT_MAIN, relief="flat",
                 state="readonly", readonlybackground=CARD,
                 highlightthickness=1, highlightbackground=CARD,
                 highlightcolor=ACCENT
                 ).grid(row=0, column=0, sticky="ew", ipady=5)
        styled_button(folder_row, "Browse", self._pick_folder,
                      bg="#253c6e").grid(row=0, column=1,
                      padx=(6, 0), sticky="ns")

        # Scale percentage entries
        section_label(left_panel, "Scale (%)", 2)
        scale_row = tk.Frame(left_panel, bg=PANEL)
        scale_row.grid(row=3, column=0, columnspan=2,
                       sticky="ew", padx=14, pady=(0, 12))
        scale_row.columnconfigure(1, weight=1)
        scale_row.columnconfigure(3, weight=1)

        self.width_pct_var  = percent_entry(scale_row, "Width:",  0, "100")
        self.height_pct_var = percent_entry(scale_row, "Height:", 2, "100")

        self.lock_ratio_var = tk.BooleanVar(value=True)
        tk.Checkbutton(
            scale_row, text="Lock ratio", variable=self.lock_ratio_var,
            bg=PANEL, fg=FG_DIM, selectcolor=CARD,
            activebackground=PANEL, activeforeground=FG,
            font=FONT_MAIN, bd=0, cursor="hand2"
        ).grid(row=0, column=4, padx=(12, 0), rowspan=2, sticky="ns")

        # Bidirectional ratio-lock traces
        self.width_pct_var.trace_add("write",  self._on_width_change)
        self.height_pct_var.trace_add("write", self._on_height_change)

        # Interpolation method selector
        section_label(left_panel, "Interpolation Method", 4)
        self.method_var = tk.StringVar(value=METHOD_KEYS[0])
        ttk.Combobox(
            left_panel, textvariable=self.method_var,
            values=METHOD_KEYS, state="readonly",
            font=FONT_MAIN, width=40
        ).grid(row=5, column=0, columnspan=2,
               sticky="ew", padx=14, pady=(0, 14))
        apply_combo_style()

        # Output path preview
        section_label(left_panel, "Output Folder (auto)", 6)
        self.output_preview_lbl = tk.Label(
            left_panel, text="-", bg=PANEL, fg=ACCENT2,
            font=FONT_MAIN, anchor="w", wraplength=300)
        self.output_preview_lbl.grid(row=7, column=0, columnspan=2,
                                     sticky="ew", padx=14, pady=(0, 14))

        # Action buttons
        btn_row = tk.Frame(left_panel, bg=PANEL)
        btn_row.grid(row=8, column=0, columnspan=2,
                     sticky="ew", padx=14, pady=(4, 16))

        self.start_btn = styled_button(btn_row, "Start", self._start)
        self.start_btn.pack(side="left", padx=(0, 8))

        self.stop_btn = styled_button(btn_row, "Stop", self._stop, bg="#555")
        self.stop_btn.pack(side="left")
        self.stop_btn.config(state="disabled")

        styled_button(btn_row, "Clear Log", self._clear_log,
                      bg="#333").pack(side="right")

        # ── Right panel: stat cards ───────────────────────────────────────────
        right_panel = tk.Frame(body, bg=BG)
        right_panel.grid(row=0, column=1, sticky="nsew")
        right_panel.columnconfigure(0, weight=1)
        right_panel.columnconfigure(1, weight=1)

        self.stat_total   = stat_card(right_panel, "Total",   0, 0)
        self.stat_done    = stat_card(right_panel, "Done",    0, 1)
        self.stat_skipped = stat_card(right_panel, "Skipped", 1, 0)
        self.stat_errors  = stat_card(right_panel, "Errors",  1, 1)

        # Progress bar
        progress_frame = tk.Frame(body, bg=BG)
        progress_frame.grid(row=1, column=1, sticky="new", pady=(8, 0))
        progress_frame.columnconfigure(0, weight=1)

        self.progress_lbl = tk.Label(progress_frame, text="Ready.",
                                     bg=BG, fg=FG_DIM, font=FONT_MAIN)
        self.progress_lbl.pack(anchor="w")

        pb_style = ttk.Style()
        pb_style.theme_use("default")
        pb_style.configure("PB.Horizontal.TProgressbar",
                           troughcolor=CARD, background=ACCENT,
                           thickness=14, borderwidth=0)
        self.progress_bar = ttk.Progressbar(
            progress_frame, style="PB.Horizontal.TProgressbar",
            orient="horizontal", mode="determinate")
        self.progress_bar.pack(fill="x", pady=(4, 0))

        # ── Log area ──────────────────────────────────────────────────────────
        log_frame = tk.Frame(body, bg=PANEL)
        log_frame.grid(row=2, column=0, columnspan=2,
                       sticky="nsew", pady=(14, 0))
        log_frame.rowconfigure(1, weight=1)
        log_frame.columnconfigure(0, weight=1)

        tk.Label(log_frame, text="Operation Log",
                 bg=PANEL, fg=FG_DIM, font=FONT_MAIN,
                 anchor="w", padx=10, pady=6
                 ).grid(row=0, column=0, sticky="ew")

        self.log_box = tk.Text(
            log_frame, bg="#0a0a18", fg=FG, font=FONT_MONO,
            relief="flat", wrap="word", state="disabled",
            height=10, selectbackground=CARD)
        self.log_box.grid(row=1, column=0, sticky="nsew")
        self.log_box.tag_config("ok",     foreground=SUCCESS)
        self.log_box.tag_config("err",    foreground=ACCENT)
        self.log_box.tag_config("warn",   foreground=ACCENT2)
        self.log_box.tag_config("info",   foreground=FG_DIM)
        self.log_box.tag_config("header", foreground=FG, font=FONT_H2)

        scrollbar = tk.Scrollbar(log_frame, command=self.log_box.yview,
                                 bg=PANEL, troughcolor=CARD,
                                 activebackground=ACCENT)
        scrollbar.grid(row=1, column=1, sticky="ns")
        self.log_box.config(yscrollcommand=scrollbar.set)

        # Output label update watchers
        for var in (self.src_var, self.width_pct_var,
                    self.height_pct_var, self.method_var):
            var.trace_add("write", self._update_output_preview)

    # ── Ratio lock (bidirectional) ────────────────────────────────────────────
    def _on_width_change(self, *_):
        if self._syncing or not self.lock_ratio_var.get():
            return
        self._syncing = True
        self.height_pct_var.set(self.width_pct_var.get())
        self._syncing = False

    def _on_height_change(self, *_):
        if self._syncing or not self.lock_ratio_var.get():
            return
        self._syncing = True
        self.width_pct_var.set(self.height_pct_var.get())
        self._syncing = False

    # ── Folder picker ─────────────────────────────────────────────────────────
    def _pick_folder(self):
        path = filedialog.askdirectory(title="Select Source Folder")
        if path:
            self.src_var.set(path)

    # ── Output folder preview ─────────────────────────────────────────────────
    def _update_output_preview(self, *_):
        src = self.src_var.get()
        if not src:
            self.output_preview_lbl.config(text="-")
            return
        method_idx   = METHOD_KEYS.index(self.method_var.get())
        method_short = METHOD_SHORT[method_idx]
        width_val    = self.width_pct_var.get()  or "?"
        height_val   = self.height_pct_var.get() or "?"
        base_path    = os.path.join(src, f"{method_short}_%{width_val}_%{height_val}")
        final_path   = unique_dir(base_path)
        note = "  [renamed, folder exists]" if final_path != base_path else ""
        self.output_preview_lbl.config(text=final_path + note)

    # ── Log helpers ───────────────────────────────────────────────────────────
    def _log(self, msg, tag="info"):
        self.log_box.config(state="normal")
        self.log_box.insert("end", msg + "\n", tag)
        self.log_box.see("end")
        self.log_box.config(state="disabled")

    def _clear_log(self):
        self.log_box.config(state="normal")
        self.log_box.delete("1.0", "end")
        self.log_box.config(state="disabled")

    def _set_stat(self, widget, value):
        widget.config(text=str(value))

    # ── Start ─────────────────────────────────────────────────────────────────
    def _start(self):
        src_path = self.src_var.get().strip()
        if not src_path:
            messagebox.showerror("Error", "Please select a source folder.")
            return
        if not os.path.isdir(src_path):
            messagebox.showerror("Error", "Selected folder does not exist.")
            return
        try:
            width_pct  = int(self.width_pct_var.get())
            height_pct = int(self.height_pct_var.get())
            if not (1 <= width_pct <= 800 and 1 <= height_pct <= 800):
                raise ValueError
        except ValueError:
            messagebox.showerror("Error",
                                 "Scale values must be integers between 1 and 800.")
            return

        method_idx   = METHOD_KEYS.index(self.method_var.get())
        method_short = METHOD_SHORT[method_idx]
        method_cv2   = INTERPOLATION_MAP[self.method_var.get()]
        base_dir     = os.path.join(src_path,
                                    f"{method_short}_%{width_pct}_%{height_pct}")
        output_dir   = unique_dir(base_dir)

        self._running = True
        self.start_btn.config(state="disabled")
        self.stop_btn.config(state="normal")
        for stat_widget in (self.stat_total, self.stat_done,
                            self.stat_skipped, self.stat_errors):
            self._set_stat(stat_widget, 0)

        threading.Thread(
            target=self._worker,
            args=(src_path, output_dir, width_pct, height_pct,
                  method_cv2, method_short),
            daemon=True
        ).start()

    # ── Stop ──────────────────────────────────────────────────────────────────
    def _stop(self):
        self._running = False
        self._log("Stopped by user.", "warn")

    # ── Conversion worker thread ──────────────────────────────────────────────
    def _worker(self, src_path, output_dir,
                width_pct, height_pct, method_cv2, method_name):
        divider = "-" * 58
        try:
            self._log(f"\n{divider}", "info")
            self._log(f"  Source  : {src_path}", "header")
            self._log(f"  Output  : {output_dir}", "header")
            self._log(f"  Method  : {method_name}  |  "
                      f"Scale: {width_pct}% x {height_pct}%", "header")
            self._log(f"{divider}\n", "info")

            image_files = collect_images(src_path)
            total       = len(image_files)
            self._set_stat(self.stat_total, total)

            if total == 0:
                self._log("No supported images found in the selected folder.",
                          "warn")
                self._finish()
                return

            os.makedirs(output_dir, exist_ok=True)
            self.progress_bar["maximum"] = total
            done = skipped = errors = 0

            for index, file_path in enumerate(image_files):
                if not self._running:
                    break

                file_name = os.path.basename(file_path)
                self.progress_lbl.config(text=f"Processing: {file_name}")

                try:
                    img = cv2.imread(file_path, cv2.IMREAD_UNCHANGED)
                    if img is None:
                        self._log(f"  Could not read: {file_name}", "warn")
                        skipped += 1
                        self._set_stat(self.stat_skipped, skipped)
                    else:
                        orig_h, orig_w = img.shape[:2]
                        new_w = max(1, int(orig_w * width_pct  / 100))
                        new_h = max(1, int(orig_h * height_pct / 100))
                        resized      = cv2.resize(img, (new_w, new_h),
                                                  interpolation=method_cv2)
                        output_path  = os.path.join(output_dir, file_name)

                        if os.path.exists(output_path):
                            self._log(
                                f"  SKIP   {file_name}  (already exists)",
                                "warn")
                            skipped += 1
                            self._set_stat(self.stat_skipped, skipped)
                        else:
                            cv2.imwrite(output_path, resized)
                            done += 1
                            self._set_stat(self.stat_done, done)
                            self._log(
                                f"  OK     {file_name}"
                                f"  [{orig_w}x{orig_h}] -> [{new_w}x{new_h}]",
                                "ok")

                except Exception as exc:
                    errors += 1
                    self._set_stat(self.stat_errors, errors)
                    self._log(f"  ERROR  {file_name}: {exc}", "err")

                self.progress_bar["value"] = index + 1

            self._log(f"\n{divider}", "info")
            self._log(
                f"  Finished: {done} converted, {skipped} skipped, "
                f"{errors} errors  /  {total} total", "header")
            self._log(f"{divider}\n", "info")
            self.progress_lbl.config(
                text=f"Finished - {done}/{total} images converted.")

        except Exception as exc:
            self._log(f"Critical error: {exc}", "err")
        finally:
            self._finish()

    def _finish(self):
        self._running = False
        self.start_btn.config(state="normal")
        self.stop_btn.config(state="disabled")
        self._update_output_preview()


# ── Entry point ───────────────────────────────────────────────────────────────
if __name__ == "__main__":
    app = App()
    app.mainloop()

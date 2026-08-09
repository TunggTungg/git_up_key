import os
import sys
import json
import subprocess
import threading
import urllib.request
import urllib.error
from pathlib import Path
import tkinter as tk
from tkinter import ttk, messagebox, scrolledtext

# =========================================================================
# 1. TỰ ĐỘNG NẠP CẤU HÌNH TỪ CONFIG_PRIVATE.PY (NẾU CÓ)
# =========================================================================
try:
    from config_private import USER_CONFIG
except ImportError:
    USER_CONFIG = {
        "key_title": "My-Workstation-PC",
        "token": "",
        "username": "",
        "email": "",
        "passphrase": ""
    }


# ==========================================
# 2. GIAO DIỆN CHÍNH (TKINTER DARK MODE)
# ==========================================
class GitHubSSHApp(tk.Tk):
    def __init__(self):
        super().__init__()

        self.title("GitHub SSH & Git Identity Manager")
        self.geometry("750x660")
        self.resizable(False, False)

        # Bảng màu Dark Mode
        self.COLOR_BG = "#1e1e1e"
        self.COLOR_PANEL = "#252526"
        self.COLOR_TEXT = "#d4d4d4"
        self.COLOR_ACCENT = "#007acc"
        self.COLOR_SUCCESS = "#4ec9b0"
        self.COLOR_ERROR = "#f14c4c"
        self.COLOR_ENTRY_BG = "#3c3c3c"

        self.configure(bg=self.COLOR_BG)
        self.setup_styles()
        self.create_widgets()

    def setup_styles(self):
        style = ttk.Style(self)
        style.theme_use("clam")

        style.configure(".", background=self.COLOR_BG, foreground=self.COLOR_TEXT, font=("Segoe UI", 10))
        style.configure("TFrame", background=self.COLOR_BG)
        style.configure("Panel.TFrame", background=self.COLOR_PANEL, relief="flat")
        style.configure("TLabel", background=self.COLOR_PANEL, foreground=self.COLOR_TEXT, font=("Segoe UI", 10))
        style.configure("Title.TLabel", font=("Segoe UI", 16, "bold"), foreground="#ffffff")

        style.configure(
            "Primary.TButton",
            font=("Segoe UI", 10, "bold"),
            background=self.COLOR_ACCENT,
            foreground="#ffffff",
            borderwidth=0,
            focusthickness=0,
            padding=8
        )
        style.map("Primary.TButton", background=[("active", "#005999"), ("disabled", "#555555")])

    def create_widgets(self):
        # Header
        header_frame = ttk.Frame(self)
        header_frame.pack(fill="x", padx=20, pady=(15, 5))
        
        title_label = ttk.Label(header_frame, text="GitHub SSH & Git Setup Tool", style="Title.TLabel")
        title_label.pack(anchor="w")
        
        sub_label = ttk.Label(
            header_frame, 
            text="Công cụ 1-Click Setup: Tự động khởi tạo SSH Key, cấu hình Git Identity và đẩy Public Key lên GitHub.",
            font=("Segoe UI", 9),
            foreground="#888888"
        )
        sub_label.pack(anchor="w", pady=(2, 0))

        # Main Input Panel
        panel = ttk.Frame(self, style="Panel.TFrame")
        panel.pack(fill="x", padx=20, pady=10, ipady=10)

        fields = [
            ("Tên nhận diện Key (Key Title):", "entry_key_name", False, USER_CONFIG.get("key_title", "My-Workstation-PC")),
            ("Personal Access Token (PAT):", "entry_token", True, USER_CONFIG.get("token", "")),
            ("GitHub Username:", "entry_username", False, USER_CONFIG.get("username", "")),
            ("Git Email:", "entry_email", False, USER_CONFIG.get("email", "")),
            ("SSH Passphrase (Mật khẩu Key):", "entry_passphrase", True, USER_CONFIG.get("passphrase", ""))
        ]

        for i, (label_text, attr_name, is_show_star, default_val) in enumerate(fields):
            lbl = ttk.Label(panel, text=label_text)
            lbl.grid(row=i, column=0, sticky="w", padx=15, pady=8)

            show_char = "*" if is_show_star else ""
            entry = tk.Entry(
                panel,
                font=("Segoe UI", 10),
                bg=self.COLOR_ENTRY_BG,
                fg=self.COLOR_TEXT,
                insertbackground=self.COLOR_TEXT,
                relief="flat",
                show=show_char
            )
            entry.grid(row=i, column=1, sticky="ew", padx=(0, 15), pady=8, ipady=4)
            if default_val:
                entry.insert(0, default_val)
            setattr(self, attr_name, entry)

        panel.columnconfigure(1, weight=1)

        # Action Button
        self.btn_run = ttk.Button(
            self,
            text="🚀 BẮT ĐẦU TỰ ĐỘNG CẤU HÌNH",
            style="Primary.TButton",
            command=self.start_process_thread
        )
        self.btn_run.pack(fill="x", padx=20, pady=5)

        # Console Log Box
        log_frame = ttk.Frame(self)
        log_frame.pack(fill="both", expand=True, padx=20, pady=(10, 15))

        log_label = ttk.Label(log_frame, text="Nhật ký hệ thống (Console Output):", font=("Segoe UI", 10, "bold"))
        log_label.pack(anchor="w", pady=(0, 5))

        self.txt_log = scrolledtext.ScrolledText(
            log_frame,
            font=("Consolas", 9.5),
            bg="#121212",
            fg="#cccccc",
            insertbackground="#ffffff",
            relief="flat",
            state="disabled"
        )
        self.txt_log.pack(fill="both", expand=True)

        self.txt_log.tag_config("INFO", foreground="#d4d4d4")
        self.txt_log.tag_config("SUCCESS", foreground=self.COLOR_SUCCESS)
        self.txt_log.tag_config("ERROR", foreground=self.COLOR_ERROR)
        self.txt_log.tag_config("WARNING", foreground="#ce9178")

    def log(self, message: str, level: str = "INFO"):
        """Ghi log vào ô Text Box (Thread-safe)."""
        self.txt_log.config(state="normal")
        self.txt_log.insert(tk.END, f"[{level}] {message}\n", level)
        self.txt_log.see(tk.END)
        self.txt_log.config(state="disabled")

    def start_process_thread(self):
        self.btn_run.config(state="disabled")
        threading.Thread(target=self.run_automation_process, daemon=True).start()

    # ==========================================
    # 3. LUỒNG XỬ LÝ TỰ ĐỘNG HÓA CHÍNH
    # ==========================================
    def run_automation_process(self):
        try:
            key_name = self.entry_key_name.get().strip()
            token = self.entry_token.get().strip()
            username = self.entry_username.get().strip()
            email = self.entry_email.get().strip()
            passphrase = self.entry_passphrase.get().strip()

            if not all([key_name, token, username, email]):
                self.log("Vui lòng điền đầy đủ các thông tin bắt buộc!", "ERROR")
                messagebox.showerror("Lỗi", "Vui lòng không để rỗng các ô thông tin!")
                return

            ssh_dir = Path.home() / ".ssh"
            ssh_dir.mkdir(parents=True, exist_ok=True)
            key_path = ssh_dir / "id_ed25519"
            pub_key_path = ssh_dir / "id_ed25519.pub"

            # 1. Tạo SSH Key
            self.log("Đang tiến hành khởi tạo Ed25519 SSH Key...", "INFO")
            if key_path.exists():
                self.log(f"Phát hiện Key đã tồn tại tại {key_path}. Bỏ qua bước tạo mới.", "WARNING")
            else:
                cmd_keygen = [
                    "ssh-keygen",
                    "-t", "ed25519",
                    "-C", email,
                    "-f", str(key_path),
                    "-N", passphrase
                ]
                res = subprocess.run(cmd_keygen, capture_output=True, text=True)
                if res.returncode == 0:
                    self.log("Tạo SSH Key mới thành công!", "SUCCESS")
                else:
                    self.log(f"Lỗi khi tạo SSH Key: {res.stderr}", "ERROR")
                    return

            # 2. Tạo ~/.ssh/config ngắt tự động cache Passphrase
            self.log("Đang cấu hình file ~/.ssh/config...", "INFO")
            config_path = ssh_dir / "config"
            config_content = (
                "Host github.com\n"
                "  HostName github.com\n"
                "  User git\n"
                "  IdentityFile ~/.ssh/id_ed25519\n"
                "  IdentitiesOnly yes\n"
                "  AddKeysToAgent no\n"
            )
            with open(config_path, "w", encoding="utf-8") as f:
                f.write(config_content)
            self.log("Cấu hình ~/.ssh/config hoàn tất (Yêu cầu nhập Passphrase thủ công khi dùng Git).", "SUCCESS")

            # 3. Đọc Public Key
            if not pub_key_path.exists():
                self.log("Không tìm thấy file id_ed25519.pub!", "ERROR")
                return
            with open(pub_key_path, "r", encoding="utf-8") as f:
                public_key_str = f.read().strip()

            # 4. Đẩy Public Key lên GitHub qua REST API
            self.log("Đang đồng bộ Public Key lên tài khoản GitHub...", "INFO")
            api_url = "https://api.github.com/user/keys"
            payload = json.dumps({"title": key_name, "key": public_key_str}).encode("utf-8")
            
            req = urllib.request.Request(
                api_url,
                data=payload,
                headers={
                    "Authorization": f"Bearer {token}",
                    "Accept": "application/vnd.github+json",
                    "Content-Type": "application/json",
                    "User-Agent": "Python-Git-SSH-App"
                },
                method="POST"
            )

            try:
                with urllib.request.urlopen(req) as resp:
                    if resp.status == 201:
                        self.log("Đã thêm SSH Key lên tài khoản GitHub thành công!", "SUCCESS")
            except urllib.error.HTTPError as e:
                err_body = e.read().decode()
                if e.code == 422 and "key_already_exists" in err_body:
                    self.log("Key này đã tồn tại trên tài khoản GitHub của bạn.", "WARNING")
                else:
                    self.log(f"Lỗi REST API GitHub [{e.code}]: {err_body}", "ERROR")
                    return

            # 5. Cấu hình Git Global Identity
            self.log("Đang thiết lập Git Global User...", "INFO")
            subprocess.run(["git", "config", "--global", "user.name", username], check=True)
            subprocess.run(["git", "config", "--global", "user.email", email], check=True)
            self.log(f"Đã set Git Config: {username} <{email}>", "SUCCESS")

            self.log("HOÀN TẤT BƯỚC THIẾT LẬP VÀ ĐỒNG BỘ KEY LÊN GITHUB!", "SUCCESS")
            messagebox.showinfo("Thành công", "Đã tạo Key và đẩy lên GitHub thành công!")

        except Exception as ex:
            self.log(f"Lỗi không xác định: {str(ex)}", "ERROR")
            messagebox.showerror("Lỗi", str(ex))
        finally:
            self.btn_run.config(state="normal")


if __name__ == "__main__":
    app = GitHubSSHApp()
    app.mainloop()
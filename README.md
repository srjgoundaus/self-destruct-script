# 🧨 Self-Destructing Python Script — Advanced README

> **Important:** This repository is strictly for **educational** and **benign** purposes (e.g., cleanup utilities, temporary scripts, demos). **Do not** use it for malicious purposes. By using this code you agree to follow all applicable laws and institutional policies.

---

## 📂 Project Structure

```
self-destruct-script/
│── src/
│   └── self_destruct.py   # main CLI script
│
│── gui/
│   └── app.py             # PySide6 GUI scaffold
│
│── assets/
│   └── screenshots/       # placeholder images for GUI
│
│── requirements.txt       # dependencies
│── .gitignore             # files to ignore
│── LICENSE                # open-source license (MIT)
│── README.md              # documentation (this file)
```

---

## ✅ Platform Behavior

* **Linux/macOS** → The file deletes itself directly.
* **Windows** → If direct deletion fails, the script creates a `.bat` file to delete the script after Python exits.

---

## 🚀 Quickstart (CLI)

```bash
git clone https://github.com/<your-username>/self-destruct-script.git
cd self-destruct-script/src
python self_destruct.py
```

---

## 🎨 GUI (Optional — PySide6)

Install dependencies:

```bash
python -m venv .venv
source .venv/bin/activate    # Linux/macOS
.\.venv\Scripts\activate   # Windows
pip install -r requirements.txt
```

Run GUI:

```bash
python -m gui.app
```

---

## 📜 Requirements

`requirements.txt`

```
PySide6
```

---

## 📝 Example Files

### src/self\_destruct.py

```python
import os
import sys
import time
import tempfile

def safe_remove(path):
    try:
        if os.path.isfile(path) and os.access(path, os.W_OK):
            size = os.path.getsize(path)
            with open(path, 'r+b') as f:
                f.write(b'\x00' * size)
                f.flush()
                os.fsync(f.fileno())
        os.remove(path)
        return True
    except Exception:
        return False

def schedule_windows_delete(path):
    try:
        bat = tempfile.mktemp(suffix='.bat')
        with open(bat, 'w') as f:
            f.write(f"""@echo off
ping 127.0.0.1 -n 3 > nul
del /F /Q "{path}"
del /F /Q "%~f0"
""")
        os.startfile(bat)
        return True
    except Exception:
        return False

def main():
    print("This script will attempt to self-destruct after execution.")
    time.sleep(1)
    path = os.path.abspath(__file__) if '__file__' in globals() else sys.argv[0]
    if not safe_remove(path) and os.name == 'nt':
        if schedule_windows_delete(path):
            print("Deletion scheduled via batch file (Windows).")
        else:
            print("Failed to schedule deletion.")
    else:
        print("Self-destruction complete.")

if __name__ == '__main__':
    main()
```

### gui/app.py

```python
from PySide6.QtWidgets import QApplication, QMainWindow, QPushButton, QTextEdit, QWidget, QVBoxLayout, QMessageBox
import sys

class MainWindow(QMainWindow):
    def __init__(self):
        super().__init__()
        self.setWindowTitle('Self-Destruct Demo — Safe Mode')
        container = QWidget()
        layout = QVBoxLayout()

        self.log = QTextEdit()
        self.log.setReadOnly(True)
        run_btn = QPushButton('Run (Dry-run)')
        del_btn = QPushButton('Self-Destruct (CONFIRM)')

        run_btn.clicked.connect(self.dry_run)
        del_btn.clicked.connect(self.confirm_delete)

        layout.addWidget(self.log)
        layout.addWidget(run_btn)
        layout.addWidget(del_btn)
        container.setLayout(layout)
        self.setCentralWidget(container)

    def dry_run(self):
        self.log.append('[DRY-RUN] Would execute safe checks...')

    def confirm_delete(self):
        reply = QMessageBox.question(self, 'Confirm', 'Type DELETE in your console to confirm',
                                     QMessageBox.Yes | QMessageBox.No)
        if reply == QMessageBox.Yes:
            self.log.append('[ACTION] User confirmed deletion — run safe removal here.')

if __name__ == '__main__':
    app = QApplication(sys.argv)
    w = MainWindow()
    w.show()
    sys.exit(app.exec())
```

### .gitignore

```
__pycache__/
*.pyc
*.pyo
*.bat
.venv/
```

### LICENSE

MIT License (c) 2025 <Your Name>

Permission is hereby granted, free of charge, to any person obtaining a copy...

---

## ⚠️ Disclaimer

This script is for **educational purposes only**. Do not use in production or for malicious activities.

---

## ❤️ Contributing

1. Fork the repo
2. Create a feature branch (`feat/my-feature`)
3. Commit your changes
4. Push and create a Pull Request

---

## 📜 License

This project is licensed under the MIT License — see `LICENSE`.

---

## Project Files (ready to copy)

Below are the project files prepared for you. Copy each into your repository following the same structure.

### `src/self_destruct.py`

```python
#!/usr/bin/env python3
"""Self-destructing Python script (cross-platform)

Usage:
    python src/self_destruct.py [--dry-run]

Notes:
- Dry-run logs the actions without removing files.
- On Windows, if direct removal fails, a temporary .bat file is created to delete the script after the Python process exits.
"""

import os
import sys
import time
import tempfile
import traceback
import argparse
from datetime import datetime

LOGFILE = os.path.join(os.path.abspath(os.path.dirname(__file__)), '..', 'self_destruct.log')


def log(msg):
    ts = datetime.utcnow().isoformat() + 'Z'
    line = f"[{ts}] {msg}
"
    try:
        with open(LOGFILE, 'a', encoding='utf-8') as f:
            f.write(line)
    except Exception:
        # best-effort logging to stderr
        sys.stderr.write(line)


def safe_remove(path):
    """Attempt to overwrite with zeros (best-effort) and then delete.
    This is NOT a forensic wipe.
    Returns (ok: bool, error: Exception|None)
    """
    try:
        if os.path.isfile(path) and os.access(path, os.W_OK):
            size = os.path.getsize(path)
            # avoid huge allocations
            chunk = 8192
            written = 0
            with open(path, 'r+b') as f:
                f.seek(0)
                while written < size:
                    towrite = min(chunk, size - written)
                    f.write(b'�' * towrite)
                    written += towrite
                f.flush()
                try:
                    os.fsync(f.fileno())
                except Exception:
                    pass
        os.remove(path)
        return True, None
    except Exception as e:
        return False, e


def schedule_windows_delete(path):
    """Create and launch a temporary .bat that waits and then deletes the file and itself."""
    try:
        # mktemp is used intentionally for a simple single-use script
        bat = tempfile.mktemp(suffix='.bat')
        with open(bat, 'w', encoding='utf-8') as f:
            f.write(f"@echo off
")
            # small wait using ping (~2 seconds)
            f.write(f"ping 127.0.0.1 -n 3 > nul
")
            f.write(f"del /F /Q \"{path}\"
")
            f.write(f"del /F /Q \"%~f0\"
")
        try:
            os.startfile(bat)
        except Exception:
            # fallback: spawn via cmd
            try:
                import subprocess
                subprocess.Popen(['cmd', '/c', bat], creationflags=0)
            except Exception as e:
                return False, e
        return True, None
    except Exception as e:
        return False, e


def determine_path():
    path = None
    if '__file__' in globals():
        path = os.path.abspath(__file__)
    else:
        path = os.path.abspath(sys.argv[0]) if sys.argv and sys.argv[0] else None
    return path


def main():
    parser = argparse.ArgumentParser(description='Self-destructing script (educational)')
    parser.add_argument('--dry-run', action='store_true', help='Do not delete; only show actions')
    args = parser.parse_args()

    log('Started session')
    print('This script will attempt to self-destruct after execution.')
    log('User ran script; dry-run=' + str(args.dry_run))

    # Put your primary logic here — example sleep to simulate work
    try:
        print('Performing main work...')
        time.sleep(0.5)
    except Exception:
        traceback.print_exc()

    path = determine_path()
    if not path:
        print('Could not determine script path. Nothing will be deleted.')
        log('Path resolution failed; aborting deletion')
        return

    print('Script path:', path)
    log('Resolved path: ' + path)

    if args.dry_run:
        print('[DRY-RUN] Would attempt safe overwrite and deletion (if permitted).')
        log('Dry-run: simulated deletion')
        return

    ok, err = safe_remove(path)
    if ok:
        print('Self-destruction complete.')
        log('Self-destruction succeeded')
        return

    log('Direct deletion failed: ' + repr(err))

    if os.name == 'nt':
        ok2, err2 = schedule_windows_delete(path)
        if ok2:
            print('Deletion scheduled via batch file (Windows).')
            log('Scheduled deletion via .bat')
            return
        else:
            print('Batch file scheduling failed:', err2)
            log('Batch scheduling failed: ' + repr(err2))

    print('Error during self-destruction:', err)
    log('Final error: ' + repr(err))


if __name__ == '__main__':
    main()
```

### `gui/app.py` (PySide6 scaffold)

```python
# gui/app.py  — educational GUI shell (does not auto-delete without confirmation)
from PySide6.QtWidgets import (
    QApplication, QMainWindow, QPushButton, QTextEdit, QWidget,
    QVBoxLayout, QMessageBox, QLabel, QLineEdit, QCheckBox
)
import sys
import os

class MainWindow(QMainWindow):
    def __init__(self):
        super().__init__()
        self.setWindowTitle('Self-Destruct Demo — Safe Mode')
        container = QWidget()
        layout = QVBoxLayout()

        self.info = QLabel(f'Python: {sys.version.split()[0]} | OS: {os.name}')
        self.log = QTextEdit()
        self.log.setReadOnly(True)
        self.confirm_input = QLineEdit()
        self.confirm_input.setPlaceholderText('Type DELETE to enable destructive action')
        self.safe_mode = QCheckBox('Safe Mode (recommended)')
        self.safe_mode.setChecked(True)

        run_btn = QPushButton('Run (Dry-run)')
        del_btn = QPushButton('Self-Destruct (CONFIRM)')

        run_btn.clicked.connect(self.dry_run)
        del_btn.clicked.connect(self.confirm_delete)

        layout.addWidget(self.info)
        layout.addWidget(self.log)
        layout.addWidget(self.confirm_input)
        layout.addWidget(self.safe_mode)
        layout.addWidget(run_btn)
        layout.addWidget(del_btn)
        container.setLayout(layout)
        self.setCentralWidget(container)

    def dry_run(self):
        self.log.append('[DRY-RUN] Would perform safe checks...')

    def confirm_delete(self):
        text = self.confirm_input.text().strip()
        if text != 'DELETE':
            QMessageBox.warning(self, 'Confirmation required', 'Type DELETE in the input box to confirm')
            return
        if self.safe_mode.isChecked():
            ok = QMessageBox.question(self, 'Safe Mode active', 'Safe mode is ON; continue?')
            if ok != QMessageBox.StandardButton.Yes:
                return
        self.log.append('[ACTION] User confirmed deletion — running safe removal...')
        QMessageBox.information(self, 'Action queued', 'Deletion routines would run now (see logs).')
        # Integrate the deletion function here with caution.

if __name__ == '__main__':
    app = QApplication(sys.argv)
    w = MainWindow()
    w.resize(800, 600)
    w.show()
    sys.exit(app.exec())
```

### `requirements.txt`

```
PySide6>=6.5
```

### `LICENSE` (MIT)

```text
MIT License

Copyright (c) 2025 <Your Name>

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

### `.gitignore`

```
__pycache__/
*.pyc
*.pyo
*.log
.env
.venv/
/dist/
/build/
*.exe
*.spec
/assets/screenshots/*
```

---

## Next steps — How to push to GitHub (commands)

```bash
# from project root (where README.md sits)
git init
git add .
git commit -m "Initial commit: self-destruct script + GUI scaffold"
git branch -M main
git remote add origin https://github.com/<your-username>/self-destruct-script.git
git push -u origin main
```

Replace `<your-username>` with your GitHub username. If you want me to generate a zip of these files or prepare a commit patch, tell me and I will provide the patch text here.

---

If you'd like, I can also:

* Create a minimal `tests/` folder with non-destructive unit tests,
* Provide a ready-made GitHub Actions workflow that runs linting and the test suite (no destructive actions),
* Produce a patch file you can apply with `git apply`.

Tell me which of those you want next.

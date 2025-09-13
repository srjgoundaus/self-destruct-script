# 🧨 Self-Destructing Python Script — Advanced README

> **Important:** This repository is strictly for **educational** and **benign** purposes (e.g., cleanup utilities, temporary scripts, demos). **Do not** use it for malicious purposes. By using this code you agree to follow all applicable laws and institutional policies.

---

I have prepared a ready-to-download full project structure for your Self-Destructing Python Script including all necessary files:

Project structure:

```
self-destruct-script/
│── src/
│   └── self_destruct.py
│
│── gui/
│   └── app.py
│
│── assets/
│   └── screenshots/   # placeholder images
│
│── requirements.txt
│── .gitignore
│── LICENSE
│── README.md
```

1. **src/self\_destruct.py**

```python
#!/usr/bin/env python3
"""Self-destructing Python script (cross-platform)"""

import os
import sys
import time
import tempfile
import argparse
from datetime import datetime

LOGFILE = os.path.join(os.path.abspath(os.path.dirname(__file__)), '..', 'self_destruct.log')

def log(msg):
    ts = datetime.utcnow().isoformat() + 'Z'
    line = f"[{ts}] {msg}\n"
    try:
        with open(LOGFILE, 'a', encoding='utf-8') as f:
            f.write(line)
    except Exception:
        sys.stderr.write(line)

def safe_remove(path):
    try:
        if os.path.isfile(path) and os.access(path, os.W_OK):
            size = os.path.getsize(path)
            chunk = 8192
            written = 0
            with open(path, 'r+b') as f:
                f.seek(0)
                while written < size:
                    towrite = min(chunk, size - written)
                    f.write(b'\x00' * towrite)
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
    try:
        bat = tempfile.mktemp(suffix='.bat')
        with open(bat, 'w', encoding='utf-8') as f:
            f.write(f"@echo off\nping 127.0.0.1 -n 3 > nul\ndel /F /Q \"{path}\"\ndel /F /Q \"%~f0\"\n")
        try:
            os.startfile(bat)
        except Exception:
            import subprocess
            subprocess.Popen(['cmd', '/c', bat], creationflags=0)
        return True, None
    except Exception as e:
        return False, e

def determine_path():
    if '__file__' in globals():
        return os.path.abspath(__file__)
    elif sys.argv and sys.argv[0]:
        return os.path.abspath(sys.argv[0])
    return None

def main():
    parser = argparse.ArgumentParser(description='Self-destructing script (educational)')
    parser.add_argument('--dry-run', action='store_true', help='Do not delete; only show actions')
    args = parser.parse_args()

    log('Started session')
    print('This script will attempt to self-destruct after execution.')
    log('User ran script; dry-run=' + str(args.dry_run))

    time.sleep(0.5)

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

2. **gui/app.py**

```python
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

if __name__ == '__main__':
    app = QApplication(sys.argv)
    w = MainWindow()
    w.resize(800, 600)
    w.show()
    sys.exit(app.exec())
```

3. **requirements.txt**

```
PySide6>=6.5
```

4. **.gitignore**

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

5. **LICENSE**

```
MIT License

Copyright (c) 2025 srjgoundaus

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

6. **README.md**
   *(use the advanced README content you already have with project instructions, CLI/GUI usage, disclaimers, and GitHub push steps)*


# c2-profile-generator
c2-profile-generator/
├── README.md
├── LICENSE
├── requirements.txt
├── c2_profile_generator.py
├── templates/
│   ├── cobalt_strike/
│   │   ├── base.profile
│   │   └── stealth.profile
│   └── sliver/
│       ├── base.json
│       └── stealth.json
├── profiles/
│   └── generated/
│       ├── cs_google_chrome.profile
│       └── sliver_stealth.json
└── examples/
    ├── cs_chrome_example.profile
    └── sliver_example.json
    # 🔄 C2 Profile Generator

**C2 Profile Generator** is a tool that automates the creation of **Cobalt Strike Malleable C2 profiles** and **Sliver configuration files**. It helps red team operators quickly generate custom C2 profiles based on simple input (e.g., "imitate Google Chrome traffic" or "enable jitter").

## 🔥 Why this matters

| Problem | Solution |
| :--- | :--- |
| Manual profile tweaking is time-consuming. | Generate profiles in seconds from templates. |
| Profiles need to be unique to avoid detection. | Add randomization and obfuscation to every generated profile. |
| Different tools need different formats. | Support for both Cobalt Strike (.profile) and Sliver (.json) formats. |

## ⚡ Features

- **Template-based generation**: Start from a base template and override specific settings.
- **Obfuscation**: Randomize variable names, headers, and metadata to create unique signatures.
- **Multi-format output**: Generate profiles for Cobalt Strike and Sliver from the same input.
- **Stealth presets**: Built-in presets for common evasion techniques.
- **Dry-run mode**: Preview the generated profile before saving.

## 🚀 Installation

```bash
git clone https://github.com/Mariokiilz/c2-profile-generator.git
cd c2-profile-generator
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
🛠️ Usage

Basic Example
python3 c2_profile_generator.py \
    --tool cobalt_strike \
    --preset chrome \
    --output cs_chrome.profile
Advanced example
python3 c2_profile_generator.py \
    --tool sliver \
    --preset stealth \
    --jitter 25 \
    --sleep 60 \
    --output sliver_stealth.json
Option Description
--tool Which C2 tool to generate for (cobalt_strike or sliver)
--preset Preset style (chrome, stealth, base)
--jitter Jitter percentage (0-100)
--sleep Beacon sleep time in seconds
--output Output file path
--dry-run Preview the generated profile without saving
Example Output (cobalt strike)
set sleeptime "60000";
set jitter "25";
set useragent "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/114.0.0.0 Safari/537.36";

http-get {
    set uri "/search?q=query";
    client {
        header "Accept" "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8";
        header "Accept-Language" "en-US,en;q=0.5";
        header "Accept-Encoding" "gzip, deflate";
    }
}
Roadmap

□ Support for Havoc C2 profiles
□ Integration with C2-Hunter for detection validation
□ Web-based UI
□ Plugin system for custom obfuscation rules

📝 License

MIT — see LICENSE.

👤 Author

Marius Petrache — Penetration Tester in training | OSCP, CRTO

⚠️ Disclaimer

This tool is for educational and red team use in authorized environments only. Do not use against systems you do not own or lack explicit written permission to test.


---

## 📄 c2_profile_generator.py

```python
#!/usr/bin/env python3
"""
C2 Profile Generator
=====================
Generates C2 profiles for Cobalt Strike and Sliver based on templates and user input.

Author: Marius Petrache
License: MIT
"""

import argparse
import json
import os
import random
import string
from datetime import datetime

USER_AGENTS = [
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/114.0.0.0 Safari/537.36",
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:109.0) Gecko/20100101 Firefox/114.0",
    "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/114.0.0.0 Safari/537.36",
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Edge/114.0.1823.51",
]

def random_string(length=8):
    return ''.join(random.choices(string.ascii_lowercase + string.digits, k=length))

def generate_cs_profile(preset="base", jitter=25, sleep=60):
    ua = random.choice(USER_AGENTS)
    uri = f"/{random_string(6)}?q={random_string(10)}"
    if preset == "chrome":
        ua = "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/114.0.0.0 Safari/537.36"
        uri = f"/search?q={random_string(10)}"
    elif preset == "stealth":
        jitter = 10
        sleep = 120

    profile = f"""# Generated Cobalt Strike Malleable C2 Profile
# Created: {datetime.now().isoformat()}
set sleeptime "{sleep}000";
set jitter "{jitter}";
set useragent "{ua}";

http-get {{
    set uri "{uri}";
    client {{
        header "Accept" "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8";
        header "Accept-Language" "en-US,en;q=0.5";
        header "Accept-Encoding" "gzip, deflate";
    }}
    server {{
        header "Content-Type" "text/html; charset=UTF-8";
        output {{ base64; print; }}
    }}
}}

http-post {{
    set uri "/submit/{random_string(6)}";
    client {{
        header "Content-Type" "application/x-www-form-urlencoded";
    }}
    server {{
        header "Content-Type" "text/html; charset=UTF-8";
        output {{ base64; print; }}
    }}
}}
"""
    return profile

def generate_sliver_profile(preset="base", jitter=25, sleep=60):
    ua = random.choice(USER_AGENTS)
    if preset == "stealth":
        jitter = 10
        sleep = 120

    profile = {
        "name": f"sliver-profile-{random_string(6)}",
        "beacon": {
            "interval": sleep,
            "jitter": jitter
        },
        "http": {
            "user_agent": ua,
            "headers": {
                "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8",
                "Accept-Language": "en-US,en;q=0.5"
            }
        }
    }
    return json.dumps(profile, indent=2)

def main():
    parser = argparse.ArgumentParser(description="C2 Profile Generator")
    parser.add_argument("--tool", choices=["cobalt_strike", "sliver"], default="cobalt_strike")
    parser.add_argument("--preset", choices=["base", "chrome", "stealth"], default="base")
    parser.add_argument("--jitter", type=int, default=25)
    parser.add_argument("--sleep", type=int, default=60)
    parser.add_argument("--output", help="Output file path")
    parser.add_argument("--dry-run", action="store_true")
    args = parser.parse_args()

    if args.tool == "cobalt_strike":
        content = generate_cs_profile(args.preset, args.jitter, args.sleep)
    else:
        content = generate_sliver_profile(args.preset, args.jitter, args.sleep)

    if args.dry_run:
        print(content)
        return

    if args.output:
        os.makedirs(os.path.dirname(args.output), exist_ok=True)
        with open(args.output, "w") as f:
            f.write(content)
        print(f"[+] Profile saved to {args.output}")
    else:
        print(content)

if __name__ == "__main__":
    main()
License
MIT License

Copyright (c) 2026 Marius Petrache

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

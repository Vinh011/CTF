# OLP - Communication | CTF Writeup

<img width="619" height="556" alt="image" src="https://github.com/user-attachments/assets/bea7dd7b-24a1-451d-b554-aad43ba348a9" />

---

## Files đề bài cung cấp

| File | Mô tả |
|------|-------|
| `er.pcap` | Network capture (~40MB, 13.643 packets) |
| `flag_txt_txt.a296eXliZWFy` | File flag bị mã hóa (59 bytes) |

**Mô tả đề bài:** *"Suspicious data stream."*

---


## Phân tích với Wireshark

### Mở file pcap

Mở Wireshark → **File → Open** → chọn `er.pcap`

### Xem Protocol Hierarchy

**Statistics → Protocol Hierarchy**


```
eth                        frames: 13.643
  ip
    tcp
      http                 frames: 1.132   ← 
        data               frames: 557
      tls                  frames: 2.858
    udp
      dns                  frames: 250
      quic                 frames: 261
```


### Xem Conversations

**Statistics → Conversations → TCP tab**

<img width="1918" height="873" alt="image" src="https://github.com/user-attachments/assets/0afbcc66-030a-428c-81ba-f90e83a03476" />


Chú ý cặp IP nội bộ giao tiếp nhiều nhất:
- `192.168.214.131` ↔ `192.168.214.133`

### Filter HTTP requests đáng ngờ

Trong thanh filter của Wireshark, gõ:

```
http && ip.addr == 192.168.214.133
```

<img width="1919" height="873" alt="image" src="https://github.com/user-attachments/assets/911655c6-bccb-4e4d-8502-ffa2bf667595" />


Nhấn **Enter**. Trong cột **Info**, scan qua các request:

```
GET /encrypt.exe HTTP/1.1     ←
POST /                        
POST /
POST /
...
```
---

## Havoc C2 Framework

### Xem raw data của POST request đầu tiên

Click vào packet frame #5 (POST request đầu tiên) → **Follow → HTTP Stream**

Trong hex dump, chú ý các byte đầu của body:

<img width="755" height="348" alt="image" src="https://github.com/user-attachments/assets/424b7c2b-6e93-4727-b571-da12e1c7510f" />

```
00 00 00 fe  de ad be ef  34 62 db 4a  00 00 00 63
└─ length ─┘ └── magic ──┘ └─ agent─┘ └─ type ─┘
```

**`deadbeef`** — magic bytes của **Havoc C2 Framework**!

> ### Havoc C2 Framework
> 
> **Havoc** là một C2 (Command & Control) framework mã nguồn mở, thường được dùng trong red team operations và đáng tiếc là cả trong actual malware.
> 
> **Cách hoạt động:**
> - **Demon** (agent/implant): chạy trên máy victim, beacon về server
> - **Team Server**: nhận beacon, gửi lệnh về
> - **Protocol:** HTTP POST với custom binary protocol
> 
> **Packet structure của Havoc:**
> ```
> [4B size][4B magic=0xDEADBEEF][4B agent_id][8B ignored][32B AES_KEY][16B AES_IV][payload...]
> ```
> 
> **Mã hóa:** AES-256 ở chế độ **CTR** (Counter Mode) cho C2 traffic.
> 
> **Reference:** [Immersive Labs - Havoc C2 Forensics](https://github.com/Immersive-Labs-Sec/HavocC2-Forensics)

### Trong Wireshark

Filter:
```
http.request.method == "POST" && ip.dst == 192.168.214.133
```

Click packet đầu tiên → **Packet Details pane** → expand **Hypertext Transfer Protocol** → **File Data**

Nhìn hex:
```
000000fe deadbeef 3462db4a 00000063 00000000
f20c2894 0e98eeca e0cc0a28 ee609a02   ← AES KEY (32 bytes)
a426b4a0 0ce05af2 9418e08a cab0a83a
78a2521c 1670aa50 ccf03818 f2702a78   ← AES IV (16 bytes)
...
```

---

## Extract AES Key và IV

Dùng script python của github [này](https://github.com/Immersive-Labs-Sec/HavocC2-Forensics/tree/main) để lấy **KEY - IV**:

```python
# Copyright (C) 2024 Kev Breen, Immersive Labs
# https://github.com/Immersive-Labs-Sec/HavocC2-Forensics
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:

# The above copyright notice and this permission notice shall be included in all
# copies or substantial portions of the Software.

# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
# SOFTWARE.
import os
import argparse
import struct
import binascii
from binascii import unhexlify

from uuid import uuid4


try:
    import pyshark
except ImportError:
    print("[-] Pyshark not installed, please install with 'pip install pyshark'")
    exit(0)

try:
    from Crypto.Cipher import AES
    from Crypto.Util import Counter
except ImportError:
    print("[-] PyCryptodome not installed, please install with 'pip install pycryptodome'")
    exit(0)



demon_constants = {
    1: "GET_JOB",
    10: 'COMMAND_NOJOB',
    11: 'SLEEP',
    12: 'COMMAND_PROC_LIST',
    15: 'COMMAND_FS',
    20: 'COMMAND_INLINEEXECUTE',
    21: 'COMMAND_JOB',
    22: 'COMMAND_INJECT_DLL',
    24: 'COMMAND_INJECT_SHELLCODE',
    26: 'COMMAND_SPAWNDLL',
    27: 'COMMAND_PROC_PPIDSPOOF',
    40: 'COMMAND_TOKEN',
    99: 'DEMON_INIT',
    100: 'COMMAND_CHECKIN',
    2100: 'COMMAND_NET',
    2500: 'COMMAND_CONFIG',
    2510: 'COMMAND_SCREENSHOT',
    2520: 'COMMAND_PIVOT',
    2530: 'COMMAND_TRANSFER',
    2540: 'COMMAND_SOCKET',
    2550: 'COMMAND_KERBEROS',
    2560: 'COMMAND_MEM_FILE', # Beacon Object File
    4112: 'COMMAND_PROC', # Shell Command
    4113: 'COMMMAND_PS_IMPORT',
    8193: 'COMMAND_ASSEMBLY_INLINE_EXECUTE',
    8195: 'COMMAND_ASSEMBLY_LIST_VERSIONS',
}


# Used to store the AES Keys for each session
sessions = {}


def tsharkbody_to_bytes(hex_string):
    """
    Converts a TShark hex formated string to a byte string.

    :param hex_string: The hex string from TShark.
    :return: The byte string.
    """
    # its concatonated strings
    hex_string = hex_string.replace(':', '')
    #unhex it
    hex_bytes = unhexlify(hex_string)
    return hex_bytes



def aes_decrypt_ctr(aes_key, aes_iv, encrypted_payload):
    """
    Decrypts an AES-encrypted payload in CTR mode.

    :param aes_key: The AES key as a byte string.
    :param aes_iv: The AES IV (Initialization Vector) for the counter, as a byte string.
    :param encrypted_payload: The encrypted payload as a byte string.
    :return: The decrypted plaintext as a byte string.
    """
    # Initialize the counter for CTR mode
    ctr = Counter.new(128, initial_value=int.from_bytes(aes_iv, byteorder='big'))

    # Create the cipher in CTR mode
    cipher = AES.new(aes_key, AES.MODE_CTR, counter=ctr)

    # Decrypt the payload
    decrypted_payload = cipher.decrypt(encrypted_payload)

    return decrypted_payload



def parse_header(header_bytes):
    """
    Parses a 20-byte header into an object.

    :param header_bytes: A 20-byte header.
    :return: A dictionary representing the parsed header.
    """
    if len(header_bytes) != 20:
        raise ValueError("Header must be exactly 20 bytes long")

    # Unpack the header
    payload_size, magic_bytes, agent_id, command_id, mem_id = struct.unpack('>I4s4sI4s', header_bytes)

    # Convert bytes to appropriate representations
    magic_bytes_str = binascii.hexlify(magic_bytes).decode('ascii')
    agent_id_str = binascii.hexlify(agent_id).decode('ascii')
    mem_id_str = binascii.hexlify(mem_id).decode('ascii')
    command_name = demon_constants.get(command_id, f'Unknown Command ID: {command_id}')

    return {
        'payload_size': payload_size,
        'magic_bytes': magic_bytes_str,
        'agent_id': agent_id_str,
        'command_id': command_name,
        'mem_id': mem_id_str
    }


def parse_request(http_pair, magic_bytes, save_path):
    request = http_pair['request']
    response = http_pair['response']#

    unique_id = uuid4()

    print("[+] Parsing Request")


    try:
        request_body = tsharkbody_to_bytes(request.get('file_data', ''))
        header_bytes = request_body[:20]
        request_payload = request_body[20:]
        request_header = parse_header(header_bytes)
    except Exception as e:
        print(f"[!] Error parsing request body: {e}")
        return


    # If there is no magic this is not Havoc
    if request_header.get("magic_bytes", '') != magic_bytes:
        return


    if request_header['command_id'] == 'DEMON_INIT':
        print("[+] Found Havoc C2")
        print(f"  [-] Agent ID: {request_header['agent_id']}")
        print(f"  [-] Magic Bytes: {request_header['magic_bytes']}")
        print(f"  [-] C2 Address: {request.get('uri')}")

        aes_key = request_body[20:52]
        aes_iv = request_body[52:68]

        print(f"  [+] Found AES Key")
        print(f"    [-] Key: {binascii.hexlify(aes_key).decode('ascii')}")
        print(f"    [-] IV: {binascii.hexlify(aes_iv).decode('ascii')}")

        if request_header['agent_id'] not in sessions:
            sessions[request_header['agent_id']] = {
                "aes_key": aes_key,
                "aes_iv": aes_iv
            }

        # We dont want to process the rest of the request
        response_payload = None

    elif request_header['command_id'] == 'GET_JOB':
        print("  [+] Job Request from Server to Agent")
         # if the pcap did not contain an init or we have manually passed keys add the found keys message

        # Grab the response header to get the incoming request.

        try:
            response_body = tsharkbody_to_bytes(response.get('file_data', ''))

        except Exception as e:
            print(f"[!] Error parsing request body: {e}")
            return


        header_bytes = response_body[:12]
        response_payload = response_body[12:]
        command_id = struct.unpack('<H', header_bytes[:2])[0]

        command = demon_constants.get(command_id, f'Unknown Command ID: {command_id}')

        print(f"    [-] C2 Address: {request.get('uri')}")
        print(f"    [-] Comamnd: {command}")

    else:
        print(f"  [+] Unknown Command: {request_header['command_id']}")
        response_payload = None

    # If we have keys lets decode the payload
    aes_keys = sessions.get(request_header['agent_id'], None)

    if not aes_keys:
        print(f"[!] No AES Keys for Agent with ID {request_header['agent_id']}")
        return

    # If save_path is set, make sure the directory exists
    if save_path and not os.path.exists(save_path):
        print(f"[!] Save path {save_path} does not exist, creating")
        os.makedirs(save_path)

    # Decrypt the Request Body
    if request_payload:
        print("  [+] Decrypting Request Body")
        decrypted_request = aes_decrypt_ctr(aes_keys['aes_key'], aes_keys['aes_iv'], request_payload)

        # Always print
        decoded = decrypted_request.decode('utf-8', errors='ignore').strip()
        print(f"\n\033[92m[Decrypted Request]\033[0m\n{decoded}\n\n")

        # Save only if save_path is set
        if save_path:
            save_file = f'{save_path}/{unique_id}-request-{request_header["agent_id"]}.bin'
            with open(save_file, 'wb') as output_file:
                output_file.write(decrypted_request)

    # Decrypt the Response Body
    if response_payload:
        print("  [+] Decrypting Response Body")

        decrypted_response = aes_decrypt_ctr(aes_keys['aes_key'], aes_keys['aes_iv'], response_payload)

        # Always print
        decoded = decrypted_response.decode('utf-8', errors='ignore').strip()
        print(f"\n\033[94m[Decrypted Response - {command}]\033[0m\n{decoded}\n\n")

        # Save only if save_path is set
        if save_path:
            save_file = f'{save_path}/{unique_id}-response-{request_header["agent_id"]}.bin'
            with open(save_file, 'wb') as output_file:
                output_file.write(decrypted_response)


def read_pcap_and_get_http_pairs(pcap_file, magic_bytes, save_path):
    capture = pyshark.FileCapture(pcap_file, display_filter='http')
    http_pairs = {}
    current_stream = None
    request_data = None

    print("[+] Parsing Packets")

    for packet in capture:
        try:
            # Check if we are still in the same TCP stream
            if current_stream != packet.tcp.stream:
                # Reset for a new stream
                current_stream = packet.tcp.stream
                request_data = None

            if 'HTTP' in packet:
                if hasattr(packet.http, 'request_method'):
                    # This is a request
                    request_data = {
                        'method': packet.http.request_method,
                        'uri': packet.http.request_full_uri,
                        'headers': packet.http.get_field_value('request_line'),
                        'file_data': packet.http.file_data if hasattr(packet.http, 'file_data') else None
                    }
                elif hasattr(packet.http, 'response_code') and request_data:
                    # This is a response paired with the previous request
                    response_data = {
                        'code': packet.http.response_code,
                        'phrase': packet.http.response_phrase,
                        'headers': packet.http.get_field_value('response_line'),
                        'file_data': packet.http.file_data if hasattr(packet.http, 'file_data') else None
                    }
                    # Pair them together in a dictionary
                    http_pairs[f"{current_stream}_{packet.http.request_in}"] = {
                        'request': request_data,
                        'response': response_data
                    }

                    parse_request(http_pairs[f"{current_stream}_{packet.http.request_in}"], magic_bytes, save_path)

                    #print(http_pairs[f"{current_stream}_{packet.http.request_in}"])

                    request_data = None  # Reset request data after pairing
        except AttributeError as e:
            # Ignore packets that don't have the necessary HTTP fields
            pass



if __name__ == "__main__":
    parser = argparse.ArgumentParser(description='Extract Havoc Traffic from a PCAP')

    parser.add_argument(
        '--pcap',
        help='Path to pcap file',
        required=True)


    parser.add_argument(
        "--aes-key",
        help="AES key",
        required=False)

    parser.add_argument(
        "--aes-iv",
        help="AES initialization vector",
        required=False)

    parser.add_argument(
        "--agent-id",
        help="Agent ID",
        required=False)

    parser.add_argument(
        '--save',
        help='Save decrypted payloads to file',
        default=False,
        required=False)

    parser.add_argument(
        '--magic',
        help='Set the magic bytes marker for the Havoc C2 traffic',
        default='deadbeef',
        required=False)


    # Parse the arguments
    args = parser.parse_args()

    # Custom check for the optional values
    if any([args.aes_key, args.aes_iv, args.agent_id]) and not all([args.aes_key, args.aes_iv, args.agent_id]):
        parser.error("[!] If you provide one of 'aes-key', 'aes-iv', or 'agent-id', you must provide all three.")

    if args.agent_id and args.aes_key and args.aes_iv:
        sessions[args.agent_id] = {
            "aes_key": unhexlify(args.aes_key),
            "aes_iv": unhexlify(args.aes_iv)
        }
        print(f"[+] Added session keys for Agent ID {args.agent_id}")

    #find_havoc_packets(packets, args.save)


    # Usage example
    http_pairs = read_pcap_and_get_http_pairs(args.pcap, args.magic, args.save)

```

Output:

```
[+] Parsing Packets
[+] Parsing Request
[+] Found Havoc C2
  [-] Agent ID: 3462db4a
  [-] Magic Bytes: deadbeef
  [-] C2 Address: http://192.168.214.133/
  [+] Found AES Key
    [-] Key: f20c28940e98eecae0cc0a28ee609a02a426b4a00ce05af29418e08acab0a83a
    [-] IV: 78a2521c1670aa50ccf03818f2702a78
  [+] Decrypting Request Body
```

---

## Decrypt C2 Traffic để tìm lệnh

Bây giờ ta có key. Nhưng cần tìm **password** để decrypt file flag. Password đó được attacker gõ vào lệnh qua C2. Vậy → decrypt server responses để xem lệnh đã gửi.

> Dùng tool ở trên có thể tìm được pass

### Kết quả

```
[Decrypted Response - COMMAND_PROC]
8c:\windows\system32\cmd.exe~/c msedge.exe flag.txt.txt -p IfimissyoucanwebacktothepartKozy
```

> Attacker đổi tên `encrypt.exe` thành `msedge.exe` (ngụy trang thành Microsoft Edge) rồi dùng để mã hóa `flag.txt.txt` với password `IfimissyoucanwebacktothepartKozy`.

---

## Phân tích encrypt.exe


### Phân tích với strings

```bash
strings encrypt.exe | grep -iE "key|aes|gcm|usage|encrypt|password|secret|sha"
```

Output quan trọng:
```
Usage: %s <filename> -p <secret_string>
Success: File encrypted to %s
EVP_aes_256_gcm                 ← AES-256-GCM!
SHA512                          ← Dùng SHA512 để derive key!
encrypt_file
```

### Phân tích với ghidra

Mở hàm **encrypt_file** tôi thấy:

<img width="1919" height="858" alt="image" src="https://github.com/user-attachments/assets/c70b5a36-43d5-4992-8b2c-dec35d48fa77" />


> ### AES-256-GCM
> 
> **AES-GCM** (Galois/Counter Mode) là mode mã hóa xác thực (AEAD):
> - **Mã hóa:** AES-CTR mode
> - **Xác thực:** GHASH authentication tag (16 bytes)
> - **Output format thường gặp:** `[IV (12 bytes)][Ciphertext][Tag (16 bytes)]`
> 
> Khác với AES-CBC hay CTR thuần, GCM còn sinh ra **authentication tag** — nếu tag match thì đảm bảo dữ liệu không bị tamper.
> 
> **OpenSSL EVP API** (`EVP_EncryptInit_ex`, `EVP_aes_256_gcm`) là thư viện C tiêu chuẩn để implement.


Binary dùng:
- `SHA512` — hash password
- `EVP_aes_256_gcm` — mã hóa

**SHA512(password) = 64 bytes → lấy 32 bytes đầu làm key, 12 bytes tiếp làm IV (nonce cho GCM).**

```
SHA512("IfimissyoucanwebacktothepartKozy")
= 175bebc9bbb47bca5518d96272f6b81e726f34d7f56a65841226fcaf8849d7d2
  fe26117cc235fef149c26586f78431644c9493f905c1fb3440dd445a189c0990

KEY = sha512[:32] = 175bebc9bbb47bca5518d96272f6b81e726f34d7f56a65841226fcaf8849d7d2
IV  = sha512[32:44] = fe26117cc235fef149c26586
```

---

## Decrypt file flag

### Script giải mã

```python
from Crypto.Cipher import AES
import hashlib

# Password tìm được từ C2 traffic
password = b"IfimissyoucanwebacktothepartKozy"

# Đọc file flag
with open("flag_txt_txt.a296eXliZWFy", "rb") as f:
    flag_raw = f.read()

print(f"File size: {len(flag_raw)} bytes")  # 59 bytes

# Key derivation: SHA512
sha512 = hashlib.sha512(password).digest()
key = sha512[:32]   # 32 bytes = AES-256 key
iv  = sha512[32:44] # 12 bytes = GCM nonce

print(f"Key: {key.hex()}")
print(f"IV:  {iv.hex()}")

# AES-256-GCM decrypt
# Format: [ciphertext (43 bytes)][tag (16 bytes)]
ciphertext = flag_raw[:-16]
tag        = flag_raw[-16:]

cipher = AES.new(key, AES.MODE_GCM, nonce=iv)
plaintext = cipher.decrypt(ciphertext)

# Verify authentication tag
try:
    cipher.verify(tag)
    print("Authentication tag verified!")
except Exception:
    print("Tag mismatch (data may be corrupted)")

print(f"\nFLAG: {plaintext.decode('utf-8')}")
```

### Kết quả

```
File size: 59 bytes
Key: 175bebc9bbb47bca5518d96272f6b81e726f34d7f56a65841226fcaf8849d7d2
IV:  fe26117cc235fef149c26586
Authentication tag verified!

FLAG: TLUCTF2025{YouR3allyD3cryptH4vocC2tr4ff1c?}
```

*Write-up by: Tham khảo từ [Immersive Labs Havoc C2 Guide](https://www.immersivelabs.com/resources/c7-blog/havoc-c2-framework-a-defensive-operators-guide) và [HavocC2-Forensics](https://github.com/Immersive-Labs-Sec/HavocC2-Forensics)*

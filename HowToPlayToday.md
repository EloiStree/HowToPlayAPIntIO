![image](https://github.com/user-attachments/assets/25ea11e2-cb33-48f9-b818-4274ba4c77fd)

---------------- 

# Today we play at: Stealth Bastard

[![image](https://github.com/user-attachments/assets/71058209-1afb-477b-ab8e-8ca5139f0dc2)](https://store.steampowered.com/app/209190/Stealth_Bastard_Deluxe/)  
[https://store.steampowered.com/app/209190/Stealth_Bastard_Deluxe/](https://store.steampowered.com/app/209190/Stealth_Bastard_Deluxe/)  

Arrow: Left, Right, Up, Down
Jump: A
Carry/Throw: Z
Gadget: E
Restart Level: R

**Key to Int**

| Action         | Key        | Dec | Hex   | Int1 | Int2 |
|----------------|------------|-----|-------|------|------|
| Move Left      | LeftArrow  | 37  | 0x25  | 1037 | 2037 |
| Move Right     | RightArrow | 39  | 0x27  | 1039 | 2039 |
| Move Up        | UpArrow    | 38  | 0x26  | 1038 | 2038 |
| Move Down      | DownArrow  | 40  | 0x28  | 1040 | 2040 |
| Jump           | A          | 65  | 0x41  | 1065 | 2065 |
| Carry/Throw    | Z          | 90  | 0x5A  | 1090 | 2090 |
| Gadget         | E          | 69  | 0x45  | 1069 | 2069 |
| Restart Level  | R          | 82  | 0x52  | 1082 | 2082 |

[https://github.com/EloiStree/2024_08_29_ScratchToWarcraft](https://github.com/EloiStree/2024_08_29_ScratchToWarcraft)

UDP = 3615
WEBSOCKET= 3617
WEBSOCKET_PORT_WSS = 3717

By UDP:
``` py 

import socket
import string
import random
import time
import struct

def random_bytes():
    return struct.pack("<ii", random.randint(0, 100), random.randint(0, 100))

def random_text(length):
    letters = string.ascii_letters
    return ''.join(random.choice(letters) for _ in range(length))

def push_data(ip, port, data: bytes):
    with socket.socket(socket.AF_INET, socket.SOCK_DGRAM) as s:
        print (f"Pushing data to {ip}:{port}| {data}")
        s.sendto(data, (ip, port))

while True:
    string_ddns_server= "apint.ddns.net"
    try:
        string_ipv4_server = socket.gethostbyname(string_ddns_server)
        print(f"Resolved {string_ddns_server} to {string_ipv4_server}")
    except socket.gaierror as e:
        print(f"Failed to resolve {string_ddns_server}: {e}")
        string_ipv4_server = "127.0.0.1"  # Fallback to localhost

    # Push eight random bytes to 127.0.0.1:3615
    data_bytes = random_bytes()
    push_data(string_ipv4_server, 3615, data_bytes)
    
    # Uncomment the following lines to push random text of 6 characters to 127.0.0.1:3614
    # data_text = random_text(6).encode('utf-8')
    # push_data('127.0.0.1', 3614, data_text)
    
    time.sleep(1)  # Sleep for a second to avoid overwhelming the server
```


By Secure Websocket:
```

import asyncio
import websockets
import os
import time
import ssl
import struct
import random

ssl_context = ssl.SSLContext(ssl.PROTOCOL_TLS_CLIENT)
ssl_context.check_hostname = False
ssl_context.verify_mode = ssl.CERT_NONE

player_index = random.randint(-100, -1)
if player_index > 0:
    player_index = -player_index

queue_bytes :bytes=[]

async def send_random_bytes():
    while True:
        await asyncio.sleep(1)
        int_value= random.randint(0, 100)
        bytes = struct.pack("<ii", player_index, int_value)
        queue_bytes.append(bytes)
        
class UDPListenerProtocol(asyncio.DatagramProtocol):
    def datagram_received(self, data, addr):
        print(f"Received {data} from {addr}")
        bytes_array = bytes(data)
        queue_bytes.append(bytes_array)

async def listen_to_udp():
    udp_ip = "127.0.0.1"
    udp_port = 3615
    loop = asyncio.get_event_loop()
    transport, protocol = await loop.create_datagram_endpoint(
        lambda: UDPListenerProtocol(),
        local_addr=(udp_ip, udp_port)
    )

    print(f"Listening for UDP packets on {udp_ip}:{udp_port}")

    try:
        while True:
            await asyncio.sleep(3600)  # Keep the listener running
    except asyncio.CancelledError:
        transport.close()
    
            
            
async def push_queue_to_wss():
    uri = "wss://193.150.14.47:3717"
    while True:
        try:
            async with websockets.connect(uri, ssl=ssl_context, ping_interval=20, ping_timeout=300) as websocket:
                while True:
                    while len(queue_bytes) > 0:
                        b : bytes = queue_bytes.pop(0)
                        await websocket.send(b)
                        print(f"Sent: {b}")
                    await asyncio.sleep(0.001)
        except (websockets.ConnectionClosed, websockets.InvalidURI, websockets.InvalidHandshake, ssl.SSLError) as e:
            print(f"Connection lost due to {e}. Reconnecting in 5 seconds...")
            await asyncio.sleep(5)

if __name__ == "__main__":
    asyncio.get_event_loop().run_until_complete(asyncio.gather(
        send_random_bytes(),
        listen_to_udp(),
        push_queue_to_wss()
    ))
    asyncio.get_event_loop().run_forever()
    

```


Server sur le PI:
https://github.com/EloiStree/2024_12_17_SingleTunnelAsymmetricalWSUDP/blob/main/README.md

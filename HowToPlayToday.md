![image](https://github.com/user-attachments/assets/25ea11e2-cb33-48f9-b818-4274ba4c77fd)

---------------- 




# Today we play at: Silksong

[<img width="966" height="480" alt="image" src="https://github.com/user-attachments/assets/08fa3aa5-ebf8-4764-9315-821e09dc2b1c" />](https://store.steampowered.com/app/1030300/Hollow_Knight_Silksong/)
[https://store.steampowered.com/app/1030300/Hollow_Knight_Silksong/](https://store.steampowered.com/app/1030300/Hollow_Knight_Silksong/)

Int input to send: https://github.com/EloiStree/2024_08_29_ScratchToWarcraft?tab=readme-ov-file#xbox-xinput-version

I am currently writing a tutorial for Godot here:  
[https://github.com/EloiStree/HelloGodotRemoteControlHub/blob/main/EN/HollowKnightSilksong/ReadMe.md](https://github.com/EloiStree/HelloGodotRemoteControlHub/blob/main/EN/HollowKnightSilksong/ReadMe.md)  

Want to play on your computer (without Arduino gamepad) ?    
Tutorial about use of XOMI and VIGem: [https://youtu.be/_vMG_CROAi4](https://youtu.be/_vMG_CROAi4)    
_(⚠️You should play it on a old computer with an other Steam account. Third party application are always against TOS and bannable.)_  


A bit of code if you want to try in Python on the Twitch live.
Replace the '193.150.14.47' by '127.0.0.1' if you are on your own computer.

``` py
import socket
import time
import struct

def send_numbers(numbers, delay):
    with socket.socket(socket.AF_INET, socket.SOCK_DGRAM) as s:
        s.setsockopt(socket.SOL_SOCKET, socket.SO_BROADCAST, 1)
        for num in numbers:
            data = struct.pack('<i', num)
            s.sendto(data, ('193.150.14.47', 3615))
            time.sleep(delay)

print("Starting in 5 seconds...")
send_numbers([1330,2330,1340,2340], 0.1)
time.sleep(5)
print ("Starting now!")

while True:
    time.sleep(1)
    send_numbers([1300,2300,1301,2301], 0.1)
    time.sleep(1)
    send_numbers([1337,2337,1333,2333], 0.1)
```



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

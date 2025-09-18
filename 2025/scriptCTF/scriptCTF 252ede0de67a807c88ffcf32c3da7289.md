# scriptCTF

# **Misc/Div**

```python
import os
import decimal
decimal.getcontext().prec = 50

secret = int(os.urandom(16).hex(),16)
num = input('Enter a number: ')

if 'e' in num.lower():
    print("Nice try...")
    exit(0)

if len(num) >= 10:
    print('Number too long...')
    exit(0)

fl_num = decimal.Decimal(num)
div = secret / fl_num

if div == 0:
    print(open('flag.txt').read().strip())
else:
    print('Try again...')
```

### Mục tiêu

Để giải được bài này, chúng ta cần làm cho biểu thức secret / fl_num trả về giá trị là 0.

### **Thách thức**

- secret là một số nguyên dương rất lớn và ngẫu nhiên.
- Để một phép chia A / B (với A hữu hạn và khác 0) bằng 0, thì số chia B phải là một số cực kỳ lớn, tiến tới vô cực.
- Tuy nhiên, chúng ta bị giới hạn:
    - Không được nhập chuỗi dài quá 9 ký tự (len(num) < 10).
    - Không được sử dụng ký tự 'e', nên không thể nhập số lớn theo kiểu khoa học như 1e99.

### **Lời giải**

Cách để tạo ra một số "vô cực" mà không cần dùng đến ký tự 'e' và có độ dài ngắn chính là sử dụng các giá trị đặc biệt được hỗ trợ bởi thư viện decimal.

Thư viện decimal của Python có thể nhận dạng các chuỗi đặc biệt như: `inf`.

![image.png](scriptCTF%20252ede0de67a807c88ffcf32c3da7289/image.png)

---

# Misc/emoji

Bài này chúng ta được cho 1 file out.txt như này:

```
🁳🁣🁲🁩🁰🁴🁃🁔🁆🁻🀳🁭🀰🁪🀱🁟🀳🁮🁣🀰🁤🀱🁮🁧🁟🀱🁳🁟🁷🀳🀱🁲🁤🁟🀴🁮🁤🁟🁦🁵🁮🀡🀱🁥🀴🀶🁤🁽
```

Và mục tiêu của chúng ta là decode nó:

```python
# Chuỗi emoji đầu vào
emoji_string = "🁳🁣🁲🁩🁰🁴🁃🁔🁆🁻🀳🁭🀰🁪🀱🁟🀳🁮🁣🀰🁤🀱🁮🁧🁟🀱🁳🁟🁷🀳🀱🁲🁤🁟🀴🁮🁤🁟🁦🁵🁮🀡🀱🁥🀴🀶🁤🁽"

hex_string = ""
# Lặp qua từng emoji trong chuỗi
for char in emoji_string:
    # Lấy mã Unicode, chuyển sang dạng hex và lấy 2 ký tự cuối cùng
    hex_digits = hex(ord(char))[-2:]
    hex_string += hex_digits

print(f"Chuỗi Hexa mới thu được: {hex_string}")

# Chuyển đổi từ hex sang bytes, sau đó giải mã
decoded_string = bytes.fromhex(hex_string).decode('utf-8')

print(f"Flag đã giải mã: {decoded_string}")
```

![image.png](scriptCTF%20252ede0de67a807c88ffcf32c3da7289/image%201.png)

---

# Crypto/RSA-1

[out.txt](scriptCTF%20252ede0de67a807c88ffcf32c3da7289/out_(1).txt)

`Yú Tóngyī send a message to 3 peoples with unique modulus. But he left it vulnerable. Figure out :)`

Đây là một cuộc tấn công nổi tiếng có tên là **Tấn công phát sóng của Håstad (Håstad's Broadcast Attack)**.

```python
from sympy.ntheory.modular import crt
import gmpy2

# Dữ liệu từ bài toán
n1 = 156503881374173899106040027210320626006530930815116631795516553916547375688556673985142242828597628615920973708595994675661662789752600109906259326160805121029243681236938272723595463141696217880136400102526509149966767717309801293569923237158596968679754520209177602882862180528522927242280121868961697240587
c1 = 77845730447898247683281609913423107803974192483879771538601656664815266655476695261695401337124553851404038028413156487834500306455909128563474382527072827288203275942719998719612346322196694263967769165807133288612193509523277795556658877046100866328789163922952483990512216199556692553605487824176112568965

n2 = 81176790394812943895417667822424503891538103661290067749746811244149927293880771403600643202454602366489650358459283710738177024118857784526124643798095463427793912529729517724613501628957072457149015941596656959113353794192041220905793823162933257702459236541137457227898063370534472564804125139395000655909
c2 = 40787486105407063933087059717827107329565540104154871338902977389136976706405321232356479461501507502072366720712449240185342528262578445532244098369654742284814175079411915848114327880144883620517336793165329893295685773515696260299308407612535992098605156822281687718904414533480149775329948085800726089284

n3 = 140612513823906625290578950857303904693579488575072876654320011261621692347864140784716666929156719735696270348892475443744858844360080415632704363751274666498790051438616664967359811895773995052063222050631573888071188619609300034534118393135291537302821893141204544943440866238800133993600817014789308510399
c3 = 100744134973371882529524399965586539315832009564780881084353677824875367744381226140488591354751113977457961062275480984708865578896869353244823264759044617432862876208706282555040444253921290103354489356742706959370396360754029015494871561563778937571686573716714202098622688982817598258563381656498389039630

e = 3

# Bước 1 & 2: Áp dụng CRT để tìm M = m^3
moduli = [n1, n2, n3]
remainders = [c1, c2, c3]

# Hàm crt của sympy trả về một tuple (giá trị, modulus tích)
m_cubed, N = crt(moduli, remainders)

print(f"Giá trị của M (m^3) tìm được là: {m_cubed}")

# Bước 3: Tính căn bậc ba của M để tìm m
# gmpy2.iroot(number, n) trả về (căn bậc n, boolean cho biết có phải căn hoàn hảo không)
m, is_perfect_cube = gmpy2.iroot(m_cubed, 3)

if is_perfect_cube:
    print(f"\nTìm thấy căn bậc ba hoàn hảo!")
    print(f"Giá trị của m (dạng số) là: {m}")

    # Bước 4: Chuyển đổi m sang văn bản
    # Tính số byte cần thiết
    byte_length = (m.bit_length() + 7) // 8
    # Chuyển số nguyên thành bytes
    message_bytes = m.to_bytes(byte_length, 'big')
    # Giải mã bytes thành chuỗi utf-8
    flag = message_bytes.decode('utf-8')

    print(f"\nFLAG LÀ: {flag}")
else:
    print("\nKhông tìm thấy căn bậc ba hoàn hảo. Cuộc tấn công thất bại.")
```

![image.png](scriptCTF%20252ede0de67a807c88ffcf32c3da7289/image%202.png)

---

# Misc/Enchant

[enc.txt](scriptCTF%20252ede0de67a807c88ffcf32c3da7289/enc.txt)

`I was playing minecraft, and found this strange enchantment on the enchantment table. Can you figure out what it is? Wrap the flag in scriptCTF{}`

Bài này dễ hiểu là được encode bằng ngôn ngữ ở bàn phù phép trong minecraft rồi, dịch nó ra thôi.

https://lingojam.com/EnchantingTable%28Minecraft%29

![image.png](scriptCTF%20252ede0de67a807c88ffcf32c3da7289/image%203.png)

Flag: scriptCTF{Minecraftisfun}

---

# Misc/Subtract

Hãy tưởng tượng mỗi pixel trên tấm ảnh là một công tắc đèn.

- Ban đầu, tất cả các đèn đều **tắt** (ảnh màu đen).
- Mỗi khi bạn đọc một cặp tọa độ từ file coordinates.txt, bạn đến vị trí đó và **gạt công tắc**.
    - Nếu đèn đang tắt (đen), nó sẽ **bật** (trắng).
    - Nếu đèn đang bật (trắng), nó sẽ **tắt** (đen).

Điều này hoàn toàn khớp với gợi ý "Subtract" và "Remove":

- Khi một điểm xuất hiện lần đầu tiên, nó được "cộng" vào ảnh.
- Khi nó xuất hiện lần thứ hai, nó bị "trừ" đi, tức là bị "loại bỏ" (remove).
- Cứ như vậy, các điểm xuất hiện với số lần chẵn sẽ tự triệt tiêu lẫn nhau và biến mất.
- Chỉ những điểm xuất hiện với **số lần lẻ** mới còn lại trên ảnh cuối cùng.

```python
from PIL import Image
import re

# --- Cấu hình ---
IMAGE_WIDTH = 500
IMAGE_HEIGHT = 500
INPUT_FILE = "coordinates.txt"
OUTPUT_FILE = "flag_XORed.png" # Tên file output cuối cùng

BLACK = (0, 0, 0)
WHITE = (255, 255, 255)

# --- Bắt đầu xử lý ---
try:
    # Bước 1: Tạo một tấm ảnh trống (canvas) ban đầu toàn màu đen
    print(f"Tạo một ảnh nền đen kích thước {IMAGE_WIDTH}x{IMAGE_HEIGHT}...")
    img = Image.new('RGB', (IMAGE_WIDTH, IMAGE_HEIGHT), color=BLACK)
    
    pixels = img.load()

    # Bước 2: Đọc file tọa độ và thực hiện lật pixel (XOR)
    print(f"Đang đọc tọa độ từ '{INPUT_FILE}' và thực hiện lật pixel (XOR)...")
    with open(INPUT_FILE, 'r') as f:
        coordinates = f.readlines()

    flipped_points = 0
    for line in coordinates:
        nums = re.findall(r'\d+', line)
        
        if len(nums) == 2:
            x = int(nums[0])
            y = int(nums[1])
            
            # Kiểm tra tọa độ hợp lệ
            if 0 <= x < IMAGE_WIDTH and 0 <= y < IMAGE_HEIGHT:
                # Lấy màu hiện tại của pixel
                current_color = pixels[x, y]
                
                # Lật màu: nếu đang là đen thì thành trắng, và ngược lại
                if current_color == BLACK:
                    pixels[x, y] = WHITE
                else:
                    pixels[x, y] = BLACK
                
                flipped_points += 1

    print(f"Đã thực hiện lật pixel {flipped_points} lần.")

    # Bước 4: Lưu ảnh kết quả
    img.save(OUTPUT_FILE)
    print(f"Hoàn thành! Ảnh kết quả đã được lưu với tên '{OUTPUT_FILE}'.")
    print("Đây chắc chắn là đáp án đúng. Hãy mở file ảnh lên!")

except FileNotFoundError:
    print(f"Lỗi: Không tìm thấy file '{INPUT_FILE}'.")
except Exception as e:
    print(f"Đã xảy ra lỗi: {e}")
```

Từ code trên, ta được:

![flag_XORed.png](scriptCTF%20252ede0de67a807c88ffcf32c3da7289/flag_XORed.png)

**Flag: `scriptCTF{6ub7r4c7_7h3_n01s3}`**

---

# Crypto/**Secure-Server**

[files.zip](scriptCTF%20252ede0de67a807c88ffcf32c3da7289/files.zip)

Challenge cung cấp một file mã nguồn server viết bằng Python và một file traffic mạng (.pcap). Phân tích mã nguồn cho thấy server sử dụng một chuỗi các phép toán XOR để mã hóa bí mật của người dùng. Tuy nhiên, logic mã hóa có một lỗ hổng kinh điển: bằng cách XOR ba thông điệp được trao đổi qua lại giữa client và server, chúng ta có thể triệt tiêu hoàn toàn các khóa ngẫu nhiên và khôi phục lại bí mật ban đầu.

### Phân tích ban đầu

Chúng ta được cung cấp hai file:

1. server.py: Mã nguồn của server "bảo mật".
2. capture.pcap: Một file ghi lại traffic mạng giữa John Doe (client) và server.

Mục tiêu là khôi phục lại "bí mật của John Doe".

### Phân tích mã nguồn server.py

Hãy cùng xem xét kỹ hơn logic của server:

```python
import os
from pwn import xor

print("With the Secure Server, sharing secrets is safer than ever!")
# 1. Server nhận dữ liệu đã mã hóa từ client
enc = bytes.fromhex(input("Enter the secret, XORed by your key (in hex): ").strip())

# 2. Server tạo một khóa ngẫu nhiên của riêng mình
key = os.urandom(32)

# 3. Server mã hóa "đè" lên dữ liệu của client và gửi lại
enc2 = xor(enc,key).hex()
print(f"Double encrypted secret (in hex): {enc2}")

# 4. Server nhận lại dữ liệu đã được client giải mã một phần
dec = bytes.fromhex(input("XOR the above with your key again (in hex): ").strip())

# 5. Server dùng khóa của mình để giải mã lần cuối và nhận được bí mật
secret = xor(dec,key)
print("Secret received!")
```

Để dễ hình dung, hãy định nghĩa các biến sau:

- Secret: Bí mật gốc của John Doe.
- Key_John: Khóa riêng của John Doe.
- Key_Server: Khóa ngẫu nhiên do server tạo ra (os.urandom(32)).

Luồng trao đổi thông tin diễn ra như sau:

1. **Client -> Server:** Gửi enc = Secret ⊕ Key_John
2. **Server -> Client:** Gửi enc2 = enc ⊕ Key_Server
3. **Client -> Server:** Gửi dec = enc2 ⊕ Key_John

### Tìm ra điểm yếu

Lỗ hổng nằm ở chỗ chúng ta, với tư cách là người quan sát, có thể thấy được cả 3 giá trị enc, enc2, và dec trong file pcap. Mặc dù chúng ta không biết Key_John hay Key_Server, chúng ta có thể loại bỏ chúng bằng cách lợi dụng các thuộc tính của phép toán XOR:

- Tính giao hoán: A ⊕ B = B ⊕ A
- Tính kết hợp: A ⊕ (B ⊕ C) = (A ⊕ B) ⊕ C
- Tự nghịch đảo: A ⊕ A = 0

Hãy thử kết hợp 3 giá trị mà chúng ta có:

`enc ⊕ enc2 ⊕ dec`

Bây giờ, thay thế chúng bằng các biểu thức tương đương từ luồng trao đổi:

= `(Secret ⊕ Key_John) ⊕ (enc ⊕ Key_Server) ⊕ (enc2 ⊕ Key_John)`

Chúng ta có thể làm gọn hơn bằng cách chứng minh một đẳng thức đơn giản hơn trước:

`enc ⊕ dec = (Secret ⊕ Key_John) ⊕ (enc2 ⊕ Key_John)`

Do Key_John ⊕ Key_John = 0, hai khóa này sẽ triệt tiêu lẫn nhau:

`enc ⊕ dec = Secret ⊕ enc2`

Từ đẳng thức trên, để tìm `Secret`, chúng ta chỉ cần XOR cả hai vế với `enc2`:

`(enc ⊕ dec) ⊕ enc2 = (Secret ⊕ enc2) ⊕ enc2`

`enc ⊕ dec ⊕ enc2 = Secret ⊕ 0`

`enc ⊕ dec ⊕ enc2 = Secret`

**Kết luận:** Bí mật chính là kết quả của việc XOR ba chuỗi dữ liệu `enc`, `enc2`, và `dec` lại với nhau.

### Khai thác lỗ hổng

**Bước 1: Trích xuất dữ liệu từ file pcap**

Chúng ta cần viết một kịch bản Python sử dụng thư viện scapy để đọc file capture.pcap và tự động bóc tách 3 chuỗi hex cần thiết.

**Bước 2: Viết script giải mã**

Script sẽ thực hiện các công việc sau:

1. Đọc file pcap.
2. Tự động xác định IP của client và server.
3. Lặp qua các gói tin, tìm và trích xuất enc (từ client), dec (từ client) và enc2 (từ server).
4. Chuyển đổi 3 chuỗi hex thành dạng bytes.
5. Thực hiện phép toán enc_bytes ⊕ enc2_bytes ⊕ dec_bytes.
6. Decode kết quả ra dạng chuỗi ký tự để đọc flag.

### Script giải mã cuối cùng

```python
from scapy.all import rdpcap, Raw, TCP
from pwn import xor

# Tên file pcap
PCAP_FILE = "capture.pcap" 

try:
    packets = rdpcap(PCAP_FILE)
except FileNotFoundError:
    print(f"Lỗi: Không tìm thấy file '{PCAP_FILE}'.")
    exit()

# Biến để lưu trữ dữ liệu
enc_hex = None
enc2_hex = None
dec_hex = None

# Tự động xác định IP client và server
client_ip, server_ip = None, None
for p in packets:
    if p.haslayer(TCP):
        client_ip, server_ip = p['IP'].src, p['IP'].dst
        break

# Lặp qua các gói tin để trích xuất dữ liệu
for packet in packets:
    if packet.haslayer(Raw):
        is_from_client = packet['IP'].src == client_ip
        is_from_server = packet['IP'].src == server_ip
        
        try:
            payload_str = packet[Raw].load.decode('utf-8', errors='ignore').strip()
        except Exception:
            continue

        # Dữ liệu từ client là enc và dec
        if is_from_client and all(c in '0123456789abcdefABCDEF' for c in payload_str) and len(payload_str) > 20:
            if enc_hex is None:
                enc_hex = payload_str
            elif dec_hex is None:
                dec_hex = payload_str

        # Dữ liệu từ server chứa enc2
        elif is_from_server:
            search_str = "Double encrypted secret (in hex): "
            if search_str in payload_str:
                temp_str = payload_str.split(search_str)[-1].strip()
                # Chỉ lấy 64 ký tự hex của enc2 để tránh text thừa
                enc2_hex = temp_str[:64]

# Xác minh, tính toán và in ra flag
if enc_hex and enc2_hex and dec_hex:
    print("[*] Đã trích xuất thành công 3 giá trị:")
    print(f"  - enc:  {enc_hex}")
    print(f"  - enc2: {enc2_hex}")
    print(f"  - dec:  {dec_hex}")

    enc_bytes = bytes.fromhex(enc_hex)
    enc2_bytes = bytes.fromhex(enc2_hex)
    dec_bytes = bytes.fromhex(dec_hex)

    # Thực hiện phép XOR cuối cùng
    secret_bytes = xor(xor(enc_bytes, enc2_bytes), dec_bytes)
    
    flag = secret_bytes.decode('utf-8')
    print(f"\n[+] Flag: {flag}")
else:
    print("\n[!] Không thể trích xuất đủ 3 giá trị.")
```

![image.png](scriptCTF%20252ede0de67a807c88ffcf32c3da7289/image%204.png)

---
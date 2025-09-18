# Full Weak Engineer CTF 2025

# OSINT/**GeoGuessr1**

- Bài này ta phải tìm định vị của 1 bức ảnh

![GeoGuessr1.jpg](Full%20Weak%20Engineer%20CTF%202025%2025eede0de67a8094b15fd409ccbda30c/GeoGuessr1.jpg)

- Có thể thấy ngay đầu tiên đập vào mắt là KFC với mã số 1065
- Đơn giản bây giờ là lên google search với cụm từ **“KFC 1065”**, ta sẽ được:

![image.png](Full%20Weak%20Engineer%20CTF%202025%2025eede0de67a8094b15fd409ccbda30c/image.png)

- Có rất nhiều trang đều cung cấp 1 địa chỉ như này: ***1065 E El Camino Real # 234 Sunnyvale, CA 94087***
- Tìm nó với ggmaps với chế độ xem phố, ta thấy ngay được như vậy:

![image.png](Full%20Weak%20Engineer%20CTF%202025%2025eede0de67a8094b15fd409ccbda30c/image%201.png)

- And định vị của nó là: ***37.3523268,-122.0035919***

---

# Crypto/**baby-crypto**

- Đề bài cho 1 flag đã bị mã hóa là: **sjrpgs{ebg13rq_zrffntr!}**
- Có thể check đơn giản thì đây là ROT13 và decode nó, ta được: **fwectf{rot13ed_message!}**

---

# Web/**regex-auth**

- Có
    
    [regex-auth.zip](Full%20Weak%20Engineer%20CTF%202025%2025eede0de67a8094b15fd409ccbda30c/regex-auth.zip)
    
- Trong [`app.py`](http://app.py) có thể thấy flag được đặt trong biến môi trường và nó được gọi khi người dùng đăng nhập trong phần uid ở trong cookie là 1 cái rỗng

![image.png](Full%20Weak%20Engineer%20CTF%202025%2025eede0de67a8094b15fd409ccbda30c/image%202.png)

- Đăng nhập với 1 tên bất kỳ thì nó sẽ đặt là `guest_12345`, còn đăng nhập bằng user có sẵn thì nó sẽ nhảy vào trong `USER`

![image.png](Full%20Weak%20Engineer%20CTF%202025%2025eede0de67a8094b15fd409ccbda30c/image%203.png)

- Mặt khác ta biết được rằng cái này nó xác định bằng cookie và cookie được encode bằng base64
- Với việc đăng nhập vào 1 username bất kỳ, ta sẽ được:

![image.png](Full%20Weak%20Engineer%20CTF%202025%2025eede0de67a8094b15fd409ccbda30c/image%204.png)

- Vậy bên server sẽ decode cái phần value của uid và xác định nó thuộc loại gì. Nếu bây giờ muốn phần kia là 1 cái rỗng thì mình có thể xóa value của uid đi, nhưng khi thử nó, ngay lập tức đoạn code phía dưới sẽ thực hiện xác định và kick người dùng quay về trang đăng nhập username. Vậy làm sao để bypass được nó? Ta có thể dùng `IA==` để thay thế cho dấu cách  `` .

![image.png](Full%20Weak%20Engineer%20CTF%202025%2025eede0de67a8094b15fd409ccbda30c/image%205.png)

---

# Web/AED

- Không hiểu cơ chế bài này lắm, nhưng mà nó là SSRF
- hmmm, idk, but AISTUDIO wil help me write this chall write-up hẹ hẹ hẹ

### Tóm tắt bài toán

Khi truy cập vào trang web, chúng ta thấy một giao diện đơn giản với hình trái tim đang đập và một chuỗi các dấu chấm hỏi (`?`). Mục tiêu là tìm ra cách để thay thế các dấu hỏi này bằng flag thực sự.

### Phân tích mã nguồn

Mã nguồn được cung cấp cho chúng ta thấy ứng dụng được xây dựng bằng Hono (một framework JS/TS) và chạy trên hai cổng nội bộ khác nhau:

1. **Máy chủ chính (Port 3000):**
    - Đây là máy chủ công khai, được ánh xạ ra cổng `8004` để chúng ta truy cập.
    - Nó có một biến trạng thái quan trọng là `pwned`, mặc định là `false`.
    - **Endpoint `/heartbeat`**: Trang web liên tục gọi API này.
        - Khi `pwned = false`, API trả về các ký tự ngẫu nhiên.
        - Khi `pwned = true`, API sẽ trả về **từng ký tự của flag** theo thứ tự.
    - **Endpoint `/fetch`**: Đây là nơi chứa lỗ hổng. Nó cho phép máy chủ `fetch` một URL do người dùng cung cấp. Đây chính là một lỗ hổng **SSRF**.
2. **Máy chủ nội bộ (Port 4000):**
    - Đây là một máy chủ quản trị, không thể truy cập trực tiếp từ bên ngoài.
    - **Endpoint `/toggle`**: Khi endpoint này được gọi, nó sẽ chuyển biến `pwned` thành `true`.

**Mục tiêu rất rõ ràng:** Chúng ta phải tìm cách khiến máy chủ chính (cổng 3000) tự gửi một request đến máy chủ nội bộ (cổng 4000) tại đường dẫn `/toggle`.

### Lỗ hổng và Cơ chế bảo vệ

Lỗ hổng nằm ở endpoint `/fetch`, nhưng nó có một cơ chế bảo vệ:

```jsx
const isAllowedURL = (u: URL) => u.protocol === "http:" && !["localhost", "0.0.0.0", "127.0.0.1"].includes(u.hostname)

```

Hàm này tạo ra một danh sách đen (blacklist), ngăn chặn chúng ta sử dụng các hostname phổ biến nhất trỏ về máy chủ cục bộ.

### Quá trình Khai thác: Từ Thất bại đến Thành công

Lộ trình khai thác là phải vượt qua bộ lọc hostname trên.

### Thử nghiệm 1 & 2: Các địa chỉ Loopback cơ bản (Thất bại)

Các payload bypass SSRF phổ biến đầu tiên được thử nghiệm:

- `http://127.0.0.2:4000/toggle`: Sử dụng một IP khác trong dải loopback.
- `http://[::1]:4000/toggle`: Sử dụng địa chỉ loopback của IPv6.

Cả hai đều không thành công, cho thấy bộ lọc có thể thông minh hơn và chặn toàn bộ dải IP `127.0.0.0/8`.

### Thử nghiệm 3: Hostname `0` (Bị chặn với lỗi `forbidden`)

Tiếp theo, chúng ta thử một kỹ thuật khác: `http://0:4000/toggle`.

- **Logic:** `0` là một cách viết tắt trong nhiều hệ thống để trỏ về `0.0.0.0` (localhost). Chuỗi hostname `"0"` không nằm trong danh sách đen.
- **Kết quả:** Máy chủ trả về lỗi `forbidden`. Đây là một manh mối quan trọng! Nó cho thấy máy chủ đã **phân giải (resolve)** hostname `0` thành địa chỉ IP nội bộ, sau đó mới áp dụng bộ lọc trên địa chỉ IP đã phân giải đó. Điều này chứng tỏ bộ lọc có hai lớp.

### Giải pháp cuối cùng: Bypass bằng DNS với `lvh.me`

Khi bộ lọc đủ thông minh để phân giải hostname trước khi kiểm tra, chúng ta cần một kỹ thuật tinh vi hơn: sử dụng một tên miền DNS công khai được cấu hình sẵn để trỏ về `127.0.0.1`. Tên miền `lvh.me` (Local Virtual Host) là một công cụ hoàn hảo cho việc này.

**Quá trình hoạt động của payload cuối cùng:**

1. **Payload được gửi:**
    
    ```
    <http://3a9e00d5975545ebb13793bf1836ac570.chal3.fwectf.com:8004/fetch?url=http://lvh.me:4000/toggle>
    
    ```
    
2. **Vòng kiểm tra đầu tiên (Kiểm tra chuỗi Hostname):**
    - Máy chủ nhận được URL payload là `http://lvh.me:4000/toggle`.
    - Hàm `isAllowedURL` kiểm tra hostname là `"lvh.me"`.
    - Vì `"lvh.me"` không nằm trong danh sách đen `["localhost", "0.0.0.0", "127.0.0.1"]`, nên bộ lọc **được vượt qua**.
3. **Thực thi `fetch` và Phân giải DNS:**
    - Hàm `fetch` bắt đầu được thực thi.
    - Máy chủ cần tìm địa chỉ IP của `lvh.me`, vì vậy nó gửi một truy vấn DNS.
    - Máy chủ DNS công cộng trả lời: `lvh.me` trỏ về `127.0.0.1`.
4. **Gửi Request nội bộ:**
    - Máy chủ bây giờ gửi một request HTTP đến `127.0.0.1:4000/toggle`.
    - Endpoint `/toggle` trên máy chủ nội bộ được kích hoạt, và biến `pwned` được chuyển thành `true`.

### Các bước tổng kết để lấy Flag

1. Sử dụng lỗ hổng SSRF tại `/fetch` với payload đã được chế tạo đặc biệt để bypass bộ lọc DNS:
    
    ```
    <http://3a9e00d5975545ebb13793bf1836ac570.chal3.fwectf.com:8004/fetch?url=http://lvh.me:4000/toggle>
    
    ```
    
2. Sau khi truy cập URL trên và nhận được phản hồi `OK`, quay trở lại trang chủ:
    
    ```
    <http://3a9e00d5975545ebb13793bf1836ac570.chal3.fwectf.com:8004/>
    
    ```
    
3. Kiên nhẫn chờ đợi trang web tự động gọi API `/heartbeat` và điền đầy đủ flag lên màn hình.

**Kết luận:** Đây là một bài CTF rất hay, dạy chúng ta rằng không nên tin tưởng vào các bộ lọc SSRF dựa trên danh sách đen đơn giản. Việc kiểm tra hostname ở nhiều lớp (cả chuỗi ký tự và địa chỉ IP đã phân giải) là cần thiết, và việc sử dụng các bản ghi DNS là một kỹ thuật mạnh mẽ để vượt qua các cơ chế bảo vệ như vậy.

---
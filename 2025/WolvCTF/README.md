# WolvCTF 2025

# JWT Learning

- Với bài này, khi vào trang web nó có giới thiệu về "JWT"
- Sang trang 2, nó cho mình tạo 1 username và tạo ra 1 `JWT Token`

![image](https://hackmd.io/_uploads/rJyDt5Chyx.png)

- Tạo 1 username bất kỳ và nó cho ta `access-token`
- Copy nó và lên [https://jwt.io/](https://jwt.io/)

![image](https://hackmd.io/_uploads/HJ0J5qRnJx.png)

- Có thể thấy trong json phần "isAdmin" là `false`, ta cần sửa nó thành `true` để có thể gửi đi, nhưng để sửa được nó thì ta cần 1 `Secret Key`
- Trong trang tiếp theo có nhắc, giới thiệu đến `/robots.txt`, vào thử trang đó

![image](https://hackmd.io/_uploads/HkMbo5C2ke.png)

- Ta tìm được đường dẫn đến `/TOKEN_SECRET.txt`. Và ở đây ta đã tìm được `secret key` kia. Quay lại trang [jwt.io](http://jwt.io/)

![image](https://hackmd.io/_uploads/HJSii9RnJl.png)

- Sửa giá trị "isAdmin" thành `true`, copy đoạn jwt đó rồi vào BurpSuite gửi đi

![image](https://hackmd.io/_uploads/B1B73c0nyx.png)

# Javascript Puzzle

- Trong bài này, chúng ta được cung cấp `app.js`. Nhìn vào đoạn code sau:

```jsx
app.get('/', (req, res) => {
    try {
        const username = req.query.username || 'Guest'
        const output = 'Hello ' + username
        res.send(output)
    }
    catch (error) {
        res.sendFile(__dirname + '/flag.txt')
    }
})

```

- Để lấy được flag, ta cần gây lỗi cho server để server kích hoạt `catch(error)` và in ra flag từ `flag.txt`
- Server ghép chuỗi `username` vào `output` mà không kiểm tra kiểu dữ liệu. Nếu có lỗi xảy ra trong *try*, server sẽ gửi file `flag.txt`.
- Thử `username` là 1 chuỗi bình thường:

![image](https://hackmd.io/_uploads/rJBDVq0n1e.png)

- Thử `username` là 1 object `{}`:

![image](https://hackmd.io/_uploads/rypyScC21l.png)

- Thử `username` là 1 object:

![image](https://hackmd.io/_uploads/SksvSqC2yx.png)

Có thể thấy, khi thực hiện việc nối chuỗi với 1 object thì javascript sẽ tự động gọi `toString()` của object đó và trả về [object Object]

- Vậy nên, nếu chúng ta gán `toString` thành một chuỗi thay vì một hàm, JavaScript sẽ không thể gọi được nó, gây lỗi.

![image](https://hackmd.io/_uploads/BkUAv5Chye.png)

# Limited 1

![image](https://hackmd.io/_uploads/rJJtwHe6ye.png)

- Nhìn vào câu truy vấn trong code ta có thể khai thác SQLi với 3 chỗ là {price_op}, {price} và {limit}
- Nhưng {price_op} bị giới hạn là 4 ký tự, còn {price} nếu khi ta đưa vào 1 giá trị mà hàm float() không thể xác định, thì nó sẽ đặt giá trị của `price = 0.0`
    
    ![image](https://hackmd.io/_uploads/rJMxYHxTJe.png)
    
- Thêm nữa, ta bị hạn chế trong việc sử dụng `;`
- Từ câu truy vấn gốc đó, ta có thể nghĩ đến việc bỏ hẳn cả `price` và `limit` bằng `/* */`
- Câu truy vấn sẽ như thế này:
`SELECT /*{FLAG1}*/category, name, price, description FROM Menu WHERE price {price_op} /*{price} ORDER BY 1 LIMIT {limit}*/`
- Mà {price} chắc chắn chỉ có số, thì ta có thể đặt `/*` vào cuối {price_op}
- Tức là khi đó:

```sql
SELECT /*{FLAG1}*/category, name, price, description
FROM Menu
WHERE price {price_op}/* {price}
ORDER BY 1 LIMIT {limit}*/

```

- Khi này điều kiện bị thiếu giá trị so sánh, nên ta cần thêm vào trước hoặc sau `/* */` (Tức là 1 giá trị thay thế cho {price})

```sql
SELECT /*{FLAG1}*/category, name, price, description
FROM Menu
WHERE price {price_op}/* {price}
ORDER BY 1 LIMIT {limit}*/4.99

```

![image](https://hackmd.io/_uploads/SkmjjSx6yl.png)

- Vì bị hạn chế sử dụng `;` nên ta có thể dùng `UNION`
- Ở đây, ta cũng biết flag cần tìm ở ngay trong truy vấn sql đang chạy. Vậy nên có thể sử dụng `INFORMATION_SCHEMA.PROCESSLIST` cùng với `info` để xem truy vấn đang được chạy:
`/query?price=4.99&price_op=%3D/*&limit=5*/4.99+UNION+SELECT+info,+NULL,+NULL,+NULL+FROM+INFORMATION_SCHEMA.PROCESSLIST`
    
    ![image](https://hackmd.io/_uploads/SJpt6rxTJl.png)
    

`FLAG: wctf{bu7_my5ql_h45_n0_curr3n7_qu3ry_func710n_l1k3_p0576r35_d035_25785458}`

# Limited 2

![image](https://hackmd.io/_uploads/HyQX0BgTkg.png)

- Với bài số 2 ta được biết, flag đang nằm ở 1 bảng khác với tên bảng bắt đầu bằng `Flag_`
- Sử dụng `table_name` với `information_schema.tables` để hiện lên tất cả tên bảng
    
    ![image](https://hackmd.io/_uploads/rk6RRHxpye.png)
    
- Rồi tương tự với việc lấy tên cột bằng `column_name` và `information_schema.columns`, ta lấy được tên cột là `value`
    
    ![image](https://hackmd.io/_uploads/HJcp1Uxayg.png)
    
- có tên cột và bảng rồi thì select thui
    
    ![image](https://hackmd.io/_uploads/r114xLx61e.png)
    

`Flag: wctf{r34d1n6_07h3r_74bl35_15_fun_96427235634}`

# Limited 3

![image](https://hackmd.io/_uploads/rJrQmUxakx.png)

- Trong db có nói đến việc thêm 1 người dùng `flag` vào bảng `mysql.user` với pass là flag cần tìm
- Mật khẩu có 13 ký tự.

![image](https://hackmd.io/_uploads/SySYN8gTJe.png)

![image](https://hackmd.io/_uploads/rJF9NUlTJx.png)

- Tìm trong bảng `mysql.user` có thể dùng `user`, `password` và `authentication_string` để tìm phần mật khẩu đã lưu cho user flag
    
    ![image](https://hackmd.io/_uploads/S1NALUxayg.png)
    
- Đoạn này em không có biết nên em xem ké
    
    ![image](https://hackmd.io/_uploads/HyKHD8lpJg.png)
    

`UNION+SELECT+User,2,3,CONCAT('$mysql',LEFT(authentication_string,6),'*',INSERT(HEX(SUBSTR(authentication_string,8)),41,0,'*'))+AS+hash+FROM+mysql.user+WHERE+plugin+%3d+'caching_sha2_password'`

![image](https://hackmd.io/_uploads/Bk3IYLlpkg.png)

- Tạo file 13 ký tự từ file `rockyou.txt`:
`awk 'length($0) == 13' rockyou.txt > rockyou13.txt`
- Xong rùi chạy: `hashcat hash.txt rockyou13.txt`
    
    ![image](https://hackmd.io/_uploads/SkOiRUgpJg.png)
    

`Flag: wctf{maricrissarah}`
# CookieArena - Part 1

# [Baby HTTP Method](https://battle.cookiearena.org/challenges/web/baby-http-method)

![image](https://hackmd.io/_uploads/SJiiGq4-le.png)

- Mở đầu vào thì có 1 nút bấm, bấm vào chẳng được cái gì cả
- check source:
    
    ![image](https://hackmd.io/_uploads/ry2Cf9Ebxl.png)
    
- Nó có gợi ý với `/src`, check tiếp:
    
    ![image](https://hackmd.io/_uploads/HJfZmqVbge.png)
    
- Có thể lấy flag tại `/super-secret-route-nobody-will-guess` với method PUT
    
    ![image](https://hackmd.io/_uploads/r1OHmcEZlg.png)
    

# [Youtube Download](https://battle.cookiearena.org/challenges/web/youtube-downloader)

![image](https://hackmd.io/_uploads/B1hVz4I-ge.png)

- Thử thêm `;ls` thì trong source có đoạn hiện lên là `index.php`
    
    ![image](https://hackmd.io/_uploads/Sy59GNLblg.png)
    
- Tiếp `;cat${IFS}/flag.txt`:
    
    ![image](https://hackmd.io/_uploads/SkG6GVU-eg.png)
    

# [eViewer](https://battle.cookiearena.org/challenges/web/eviewer)

![image](https://hackmd.io/_uploads/rJHGE48-xx.png)

- Trang web có chức năng đọc file, sơ qua thì ko mở được `index.php`, 2 cái kia thì được
- Mục tiêu là mở file /flag.txt
- Thử `../../../../../../../flag.txt`
    
    ![image](https://hackmd.io/_uploads/SyBYV48Zee.png)
    

# [Cookie Comic](https://battle.cookiearena.org/challenges/web/cookie-comic)

![image](https://hackmd.io/_uploads/SkHE_4Lbxx.png)

- Thử qua sương sương mấy trang, xong ctrl+U lên thì thấy phần ảnh được gọi từ file local
    
    ![image](https://hackmd.io/_uploads/HkbTd48-ge.png)
    
- Vậy nếu mình thử gọi file `/flag.txt` ở đây thì sao?
- `http://103.97.125.56:31204/image.php?file=/flag.txt`: và mình nhận được 1 file .php trống trơn
- thử với path traversal thì seo? `http://103.97.125.56:31204/image.php?file=../../../../../flag.txt`
- Bùm
    
    ![image](https://hackmd.io/_uploads/HkcVYEL-lx.png)
    

# [My Deploy](https://battle.cookiearena.org/challenges/web/my-deploy)

- Bài này nó cho mình 1 cái web có khả năng deploy 1 cái source web của mình
    
    ![image](https://hackmd.io/_uploads/ryxgaE8-gg.png)
    
    ![image](https://hackmd.io/_uploads/BkgbpEL-le.png)
    
- Nó cho mình up lên chỉ duy nhất file zip
- Tạo 1 file php có code như này:

để khi mở file này thì code trong file sẽ được chạy và khi đó ta có thể thêm para `?cmd=<lệnh gì đó trong shell>` và RCE
    
    ![image](https://hackmd.io/_uploads/BkprpNU-gx.png)
    
- Đóng zip vào rồi ném lên
    
    ![image](https://hackmd.io/_uploads/BkAT6NLZll.png)
    
- Về lại index có thể thấy file zip đã được upload, giờ thì mở extracted/ lên để coi phần code đc extracted
    
    ![image](https://hackmd.io/_uploads/BJoW0NIWlg.png)
    
- Đó, file php kia đc upload rồi, bây giờ thì chạy rồi theo kịch bản kia thui
    
    ![image](https://hackmd.io/_uploads/rJxv0EUWxe.png)
    
    ![image](https://hackmd.io/_uploads/BJahAELblx.png)
    
- hehe, thấy rồi, giờ cat nó thôi

![image](https://hackmd.io/_uploads/BkvgkrL-gx.png)
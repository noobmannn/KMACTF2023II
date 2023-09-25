# Backer

![](https://github.com/konate47/KMACTF2023II/blob/867c6df6788823eee53656142d626225cd0c73c3/Backer/Img/Backer.png)

Bài cho chúng ta một file PE32, khi chạy thử chương trình, nó yêu cầu mình nhập Key, sai thì trả về ```Try again!```

![](https://github.com/konate47/KMACTF2023II/blob/23d20c87a969178d5a2f6156da13fb632e5e709d/Backer/Img/1.png)

Tiến hành load file vào IDA32 và xem hàm ```main``` của nó

![](https://github.com/konate47/KMACTF2023II/blob/f1600dae4362918703b606d0a94089c981609048/Backer/Img/2.png)

Mình sẽ xem thử bên trong hàm ```sub_4011E0```

![](https://github.com/konate47/KMACTF2023II/blob/eda7f5a94f7e9bc71cd8ba58862c30322e2c60dc/Backer/Img/3.png)

Giữa vào các giá trị ```0x6A09E667```, ```0xBB67AE85```, ```0x3C6EF372```, ```0xA54FF53A```, ```0x510E527F```, ```0x9B05688C```, ```0x1F83D9AB```, ```0x5BE0CD19```, mình nhận ra rằng hàm này sẽ biến đổi Key đầu vào mình nhập thành một đoạn mã SHA-256 (có thể xem cụ thể cách nó hoạt động ở đây: https://viblo.asia/p/cach-hoat-dong-cua-sha-256-1VgZvJPmZAw)

Bởi việc viết Script để Decrypt một đoạn mã SHA-256 là một việc cực kì khó khăn, nên mình sẽ giải bài này theo một cách khác.

Đặt breakpoint tại ```0x005E1B88``` sau đó tiến hành Debug 

![](https://github.com/konate47/KMACTF2023II/blob/609a1b788086f8cbf67e1c7122a8df9adda95daa/Backer/Img/4.png)

Theo dõi sự thay đổi giá trị của thanh ghi ```esi```

![](https://github.com/konate47/KMACTF2023II/blob/097f47935ab31e847a8a3892db5046544154be38/Backer/Img/5.png)



Lúc này giá trị của thanh ghi ```esi``` là ```71616d1f41fa7f1d72d56793252afff87b8aebf20afbf799b8fdc459c52a3d2c```, đây chính là giá trị được đem ra so sánh với mã SHA-256 được biến đổi từ Key nhập vào

Mình thử ném đoạn mã trên vào trang web https://md5hashing.net/hash/sha256 và thử Decrypt xem được không, và kết quả trả về chuỗi gốc ban đầu là ```s9cr9t_k6y```

![](https://github.com/konate47/KMACTF2023II/blob/416e6071a4011943b164da5347b6c1a7314b34ac/Backer/Img/6.png)

Chạy lại chương trình với Input là đoạn Key mình vừa tìm được, và ta có được Flag :)))))

![](https://github.com/konate47/KMACTF2023II/blob/2fb969b1cfd2fb7b1e45a1beeb3ef7d365093bc3/Backer/Img/7.png)

# Flag

```KMACTF{do_you_like_it}```

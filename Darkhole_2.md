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
Tại bước này cứ nghĩ là có IDOR vì change id=2,3,4 ... đều được hết. Nhưng không có cái trang nào là admin :(

Thử 1 paylaod đơn giản xem có bị dính SQLi không

> ' order by 7 -- -

Khi order by 7 thì trang bị lỗi :) -> dính SQLi rồi

![image](https://user-images.githubusercontent.com/97771705/225812296-1ef5d23e-aba7-42f5-af85-db087c621dea.png)

Sử dụng sqlmap để dò thôi :)



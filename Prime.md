## Step 1
Máy prime cho ta biết một vài thông số: user là victor và có 1 tệp password.txt

![image](https://user-images.githubusercontent.com/97771705/223301082-ac55005f-a2a5-4beb-aa2e-0729ac8c54f3.png)

Dùng lệnh **netdiscover -r 10.0.2.0/24** ta tìm thấy IP của máy Prime

![image](https://user-images.githubusercontent.com/97771705/223301622-661a51b0-459b-4cea-b43a-430b263775ff.png)

Dùng lệnh **nmap -A 10.0.2.15** ta tìm thấy 2 port, trong đó có 1 port ssh và 1 port http

![image](https://user-images.githubusercontent.com/97771705/223301935-8330a925-14d7-4542-9df3-160903b3ac54.png)

## Step 2
Sử dụng browser truy cập vào trang 10.0.2.15 

![image](https://user-images.githubusercontent.com/97771705/223302137-31566638-75b5-463d-962a-0c65eac9d32c.png)

Khi dùng **dirsearch -u 10.0.2.15**, ta phát hiện vài trang wordpress

![image](https://user-images.githubusercontent.com/97771705/223303789-a8201541-0554-47e4-a838-0ce8e396db79.png)
![image](https://user-images.githubusercontent.com/97771705/223303964-c09f6ca3-4222-4adf-a259-6abb113d2e8d.png)

Trang js yêu cầu có quền cao hơn để vào

![image](https://user-images.githubusercontent.com/97771705/223304140-816b055c-b2ec-409d-b5fa-55c5cde35615.png)

Trang wordpress không có gì kì lạ

![image](https://user-images.githubusercontent.com/97771705/223431905-5bf7c1c5-c0e3-46c0-af6c-e6e97ce52f1c.png)

Sử dụng lệnh gobuster để tìm các trang ẩn 

> gobuster dir -u 10.0.2.15 -w /usr/share/wordlists/dirb/common.txt -x html,php,txt

Tự dưng lòi ra thêm secret.txt

![image](https://user-images.githubusercontent.com/97771705/223306643-3effc41b-3c1a-4613-8b18-5517ce71cb5e.png)

Secret.txt dẫn ta tới 1 trang github

![image](https://user-images.githubusercontent.com/97771705/223306787-f30e75c4-3a4e-4f42-877d-0f61ef08a73c.png)

Trên trang này có nhiều payload nhưng paylaod thứ nhất cho phép ta tìm được parameter "file"

>wfuzz -c -w /usr/share/wfuzz/wordlist/general/common.txt --hc 404 http://10.0.2.15/index.php?FUZZ

Ta thấy word = 12W là common lengh của tất cả các word. Thêm --hw 12 vào command trên

>wfuzz -c -w /usr/share/wfuzz/wordlist/general/common.txt --hc 404 --hw 12 http://10.0.2.15/index.php?FUZZ

![image](https://user-images.githubusercontent.com/97771705/223308523-804f4165-f876-49b4-a74b-821938939d68.png)

Truy cập vào /index.php?file=location.txt -> có gợi ý về 1 parameter khác

![image](https://user-images.githubusercontent.com/97771705/223308777-b2e7fb9f-634c-4d05-b5f0-e6252428003a.png)

## Step 3
Khi ta dùng gobuster, chỉ có 2 trang php trả về response 200. Ngoài trang index đã sử dụng thì chỉ còn image.php. Ta cần tìm value ứng với parameter secrettier360  của trang này
Làm tương tự như trên nhưng với lệnh

>wfuzz -c -w /usr/share/wfuzz/wordlist/general/common.txt --hc 404 http://10.0.2.15/image.php?secrettier360=FUZZ

![image](https://user-images.githubusercontent.com/97771705/223311316-dc739b8c-fd23-4df0-a1db-f511f4396713.png)

Common word là 17

>wfuzz -c -w /usr/share/wfuzz/wordlist/general/common.txt --hc 404 --hw 17 http://10.0.2.15/image.php?secrettier360=FUZZ

Ta tìm được value là dev

![image](https://user-images.githubusercontent.com/97771705/223311427-90a6129f-aaa4-4c23-8584-6efe31057f0c.png)

![image](https://user-images.githubusercontent.com/97771705/223311994-f42cd4d4-1ea1-422f-9ca4-53e106d22a53.png)

Nhưng tại sao tên value là dev, nó giống y hệt 1 folder của linux. Và nhắc đến folder của linux thì phải nhắc đến 1 path thường gặp nhất với những bài path travelsal /etc/passwd
Thử nhập path đó vào url 

![image](https://user-images.githubusercontent.com/97771705/223312437-46a45885-603c-4d67-848e-343b43de138e.png)

Boom!! Phát hiện password.txt
Đi theo path /home/saket/password.txt

![image](https://user-images.githubusercontent.com/97771705/223312884-6e9b3ee3-5c5d-419f-ac28-298ed4833f3c.png)

Password là follow_the_ippsec. Nhập thử vào máy Prime -> không phải pass. Vậy đây là pass của cái gì?
 Trước chúng ta dò được 1 trang login của wp, đến lúc xài rồi.
 
 Với username là victor và pass là follow_the_ippsec
 
 ![image](https://user-images.githubusercontent.com/97771705/223432311-3b8ed0af-34b8-4d6f-8a27-67aa81ab738f.png)

 Tại theme editor, tìm thấy secret.php
 
 ![image](https://user-images.githubusercontent.com/97771705/223432858-3c652fd4-57c3-4a88-9dbc-3b345756d3e4.png)

Secret.php cho phép nhập văn bản -> có thể lợi dụng để up reverse shell

## Step 4
(Từ đoạn này đổi máy: máy kali là 10.0.2.15, máy prime là 10.0.2.5)

Trên kali có sẵn một file reverse shell php tại /usr/share/webshells/php/php-reverse-shell.php

Copy text của file đó vào secret.php của wordpress theme editor và sứ ip và port

![image](https://user-images.githubusercontent.com/97771705/224310901-ccc47d4b-5c64-4a82-bedb-202404041675.png)

Tại root kali

![image](https://user-images.githubusercontent.com/97771705/224311108-f94f18d9-31d5-44b0-b491-f2ac5323a6d9.png)

Trên browser, truy cập trang http://10.0.2.5/wordpress/wp-content/themes/twentynineteen/secret.php

Ngay lúc đó, tại terminal

![image](https://user-images.githubusercontent.com/97771705/224312233-1bd453d9-9f3e-4a3c-b3a1-747974286c23.png)

Tìm được flag đầu tiên

![image](https://user-images.githubusercontent.com/97771705/224312612-6f90b555-e86d-4de5-a967-a9af17d30bfc.png)

## Step 5
Truy cập thử folder victor nhưng không đủ quyền 

![image](https://user-images.githubusercontent.com/97771705/224312781-334b0205-cc15-40f7-9ba6-9f488fc48080.png)

Tìm kiếm phiên bản kernel 

![image](https://user-images.githubusercontent.com/97771705/224314063-13fd798c-4204-4784-b232-b40c7c9d2567.png)







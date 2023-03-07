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


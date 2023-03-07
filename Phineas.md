## Step 1
Dùng **netdiscover -r 10.0.2.0/24** để dò ip của victim
![image](https://user-images.githubusercontent.com/97771705/222623213-96b7c935-f64e-456b-886f-9848fee3f4eb.png)

IP là *10.0.2.5*

Quét port ra 4 port, trong đó có 1 port mysql 

![image](https://user-images.githubusercontent.com/97771705/223318597-503e6337-d3a5-4494-90b9-c779ee69317a.png)

![image](https://user-images.githubusercontent.com/97771705/223318808-c6786e72-4619-48f4-a1ba-4e6f5cc147db.png)

Tìm path ẩn bằng gobuster
>gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.0.2.5 -x html,php,txt

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

> gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.0.2.5/structure -x html,php,txt

Trang index.php chuyển hướng sang start

![image](https://user-images.githubusercontent.com/97771705/223322704-44a5ab48-433a-499b-92f2-1d471019b544.png)

Thử thay url bằng /index.php/fuel 

![image](https://user-images.githubusercontent.com/97771705/223323114-ac21dfa1-9395-4f2c-99a2-5dbd381c6337.png)







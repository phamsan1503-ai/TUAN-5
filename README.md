# Bài tập HDH Nhúng - Biên dịch chéo thư viện và ứng dụng
## Bài tập 01: Biên dịch ứng dụng với thư viện đã có
### Bật cJSON trong Buildroot
Trong folder buildroot:
```
make menuconfig
```
vào:
```
Target packages
   Libraries
      JSON
         [*] cJSON
```
sau đó build lại:
```
make
```

<img width="605" height="341" alt="Picture1" src="https://github.com/user-attachments/assets/62d8f86d-66f7-4c53-8456-a874f05e8959" />

Buildroot sẽ cài cJSON vào sysroot.
________________________________________
### Viết chương trình HelloJSON
Tạo file:
```
nano HelloJSON.c
```
code:
```
#include <stdio.h>
#include <cjson/cJSON.h>

int main()
{
    char json_string[] = "{\"name\":\"BBB\",\"year\":2025}";

    cJSON *json = cJSON_Parse(json_string);

    cJSON *name = cJSON_GetObjectItem(json, "name");
    cJSON *year = cJSON_GetObjectItem(json, "year");

    printf("Name: %s\n", name->valuestring);
    printf("Year: %d\n", year->valueint);

    cJSON_Delete(json);

    return 0;
}
```

<img width="605" height="243" alt="Picture2" src="https://github.com/user-attachments/assets/4b1f886d-a093-4ad2-ac46-20b4ab33d63e" />

________________________________________
### Cross compile
Dùng toolchain của buildroot
ví dụ:
```
output/host/bin/arm-linux-gcc
```
compile:
```
output/host/bin/arm-linux-gcc HelloJSON.c -o HelloJSON -lcjson
```
<img width="605" height="31" alt="Picture3" src="https://github.com/user-attachments/assets/e7c60b38-57c2-44cd-b141-9e09be9a331a" />


_______________________________________

### Mount thẻ SD trên Ubuntu
Cắm SD card vào máy tính.
Kiểm tra thiết bị:
```
lsblk
```
Tạo thư mục mount:
```
sudo mkdir /mnt/sd
```
Mount phân vùng rootfs:
```
sudo mount /dev/sdb2 /mnt/sd
```
________________________________________
### Copy chương trình vào thẻ SD
Copy file vừa compile vào rootfs của thẻ SD:
```
cp HelloJSON /mnt/sd/
```
Kiểm tra file:
```
ls /mnt/sd
```
Nếu thấy:
```
HelloJSON
```
thì đã copy thành công.
________________________________________
### Unmount thẻ SD
Sau khi copy xong cần tháo mount:
```
sudo umount /mnt/sd
```
Sau đó rút thẻ SD khỏi máy tính.

<img width="400" height="202" alt="Picture4" src="https://github.com/user-attachments/assets/0129266c-4665-483a-9568-3ef0bfae15ee" />

________________________________________
### Boot BeagleBone Black từ thẻ SD
Thực hiện các bước sau:
1.	Cắm SD card vào BBB
2.	Nhấn giữ nút BOOT (S2)
3.	Cắm nguồn cho BBB
4.	Giữ nút khoảng 3–5 giây rồi thả ra
BBB sẽ boot hệ điều hành Buildroot từ thẻ SD.
________________________________________
### Đăng nhập vào BBB qua UART
Mở terminal:
```
screen /dev/ttyUSB0 115200
```
Sau khi hệ thống khởi động xong sẽ hiện:
```
buildroot login:
```
Đăng nhập:
```
root
```
________________________________________
### Chạy chương trình
Kiểm tra file:
```
ls
```
Nếu thấy:
```
HelloJSON
```
thì chạy chương trình:
```
chmod +x HelloJSON
./HelloJSON
```
________________________________________
### Kết quả
Chương trình sẽ hiển thị:
```
Name: BBB
Year: 2025
```
<img width="605" height="171" alt="Picture5" src="https://github.com/user-attachments/assets/c43f04fa-0dee-46ae-af33-ee72dbdd3b7a" />


Điều này chứng tỏ chương trình đã được cross compile thành công và chạy trên BeagleBone Black.

## Bài tập 02: Tự tạo thư viện cá nhân
### Tạo file thư viện
Trong Ubuntu vào Buildroot:
```
cd ~/buildroot
```
Tạo file source:
```
nano mylib.c
```
Code:
```
#include <stdio.h>

void hello()
{
    printf("Hello from my library!\n");
}
```

<img width="605" height="167" alt="Picture1" src="https://github.com/user-attachments/assets/02389f82-ee4d-4bd0-8882-e424378b0453" />

Tạo file header:
```
nano mylib.h
```
Code:
```
void hello();
```

<img width="605" height="127" alt="Picture3" src="https://github.com/user-attachments/assets/3c5a675a-3858-4ebb-a524-065f64fe73bf" />

### ross compile object file
Dùng gcc của Buildroot:
```
output/host/bin/arm-linux-gcc -c mylib.c -o mylib.o
```
Kiểm tra:
```
ls
```
phải thấy:
```
mylib.c
mylib.h
mylib.o
```

<img width="605" height="265" alt="Picture4" src="https://github.com/user-attachments/assets/f6c39205-8b6b-4ca7-b8f5-64ef4aa38eb2" />

### Tạo static library (.a)
Dùng ar:
```
output/host/bin/arm-linux-ar rcs libmylib.a mylib.o
```
Kiểm tra:
```
ls
```
phải thấy:
```
libmylib.a
```


<img width="605" height="116" alt="Picture5" src="https://github.com/user-attachments/assets/b39aacb1-74b5-42b1-abaa-698ace7a14c0" />


### Tạo shared library (.so)

Compile lại với -fPIC:
```
output/host/bin/arm-linux-gcc -fPIC -c mylib.c
```
Sau đó:
```
output/host/bin/arm-linux-gcc -shared -o libmylib.so mylib.o
```
Kiểm tra:
```
libmylib.so
```

<img width="605" height="136" alt="Picture6" src="https://github.com/user-attachments/assets/325b6465-6a8e-4490-b72e-4776bd6aadfb" />

ĐƯA THƯ VIỆN ĐÃ BIÊN DỊCH THÀNH CÔNG VÀO SYSROOT

### Viết chương trình sử dụng thư viện
Tạo file:
```
nano main.c
```
Code:
```
#include "mylib.h"

int main()
{
    hello();
    return 0;
}

```

<img width="605" height="143" alt="Picture7" src="https://github.com/user-attachments/assets/9c21d5e1-c8d0-4811-8244-9e921b57a568" />

### Compile chương trình với static library
```
output/host/bin/arm-linux-gcc main.c -L. -lmylib -o app_static
```
Kiểm tra:
```
file app_static
```
kết quả:```  ELF 32-bit ARM ```

### Compile chương trình với shared library

```
output/host/bin/arm-linux-gcc main.c -L. -lmylib -o app_shared
```

<img width="605" height="115" alt="Picture8" src="https://github.com/user-attachments/assets/00394ae9-7c17-413f-869a-d47ca947a943" />

###  Mount SD card (giống phần 1)
Cắm SD vào Ubuntu rồi chạy:
```
lsblk
```
ví dụ thấy:
```
sdb
sdb1
sdb2
```
Mount rootfs:
```
sudo mount /dev/sdb2 /mnt/sd
```

### Copy chương trình và thư viện
Copy:
```
sudo cp app_static /mnt/sd/
sudo cp app_shared /mnt/sd/
sudo cp libmylib.so /mnt/sd/usr/lib/
```
- Lý do:
shared library phải nằm trong /usr/lib thì BBB mới tìm thấy.

### Unmount SD

```
sudo umount /mnt/sd
```

<img width="605" height="75" alt="Picture9" src="https://github.com/user-attachments/assets/a01cd185-f583-4cf4-9435-4d6bbb327f93" />

### Boot BBB Chạy chương trình

```
sudo screen /dev/ttyUSB0 115200
```
login:
```
root
```
Static:
```
/app_static
```
Shared:
```
/app_shared
```
<img width="605" height="184" alt="Picture10" src="https://github.com/user-attachments/assets/4bbd7aa4-c0b2-4b49-9e53-cbf838bc1a4d" />

## Bài tập 03: Tích hợp ứng dụng và thư viện và Buildroot
### Tạo source code cho chương trình
#### Tạo thư mục source:
```
mkdir myapp
cd myapp
```

#### Tạo file chương trình
```
nano main.c
```
dán:
```
#include <stdio.h>
#include <cjson/cJSON.h>
#include "mylib.h"

int main()
{
    hello();

    cJSON *root = cJSON_CreateObject();
    cJSON_AddStringToObject(root, "board", "BBB");

    printf("%s\n", cJSON_Print(root));

    return 0;
}
```

<img width="605" height="185" alt="Picture11" src="https://github.com/user-attachments/assets/f0a9362e-4b5a-4ebd-b2d1-b3fe7abcc55f" />

lưu:
```
Ctrl+O
Enter
Ctrl+X
```

#### Quay lại buildroot
```
cd ~/buildroot
```
- Tạo package cho myapp
```
mkdir -p package/myapp
```
- Tạo Config.in
```
nano package/myapp/Config.in
```
dán:
```
config BR2_PACKAGE_MYAPP
    bool "myapp"
    select BR2_PACKAGE_CJSON
    select BR2_PACKAGE_MYLIB
```
lưu. 


<img width="605" height="111" alt="Picture12" src="https://github.com/user-attachments/assets/9fd5637c-99fa-44c9-a129-f58f6e939b98" />


#### Tạo file myapp.mk
```
nano package/myapp/myapp.mk
```
dán:
```
MYAPP_VERSION = 1.0
MYAPP_SITE = $(TOPDIR)/myapp
MYAPP_SITE_METHOD = local

MYAPP_DEPENDENCIES = cjson mylib

define MYAPP_BUILD_CMDS
	$(TARGET_CC) $(@D)/main.c -o $(@D)/myapp -lcjson -lmylib
endef

define MYAPP_INSTALL_TARGET_CMDS
	$(INSTALL) -D -m 0755 $(@D)/myapp $(TARGET_DIR)/usr/bin/myapp
endef

$(eval $(generic-package))
```
lưu. 

<img width="605" height="164" alt="Picture13" src="https://github.com/user-attachments/assets/f32f0076-9b96-4b27-89dd-f8be2e764c78" />


TẠO SOURCE CHO MYLIB
1️⃣ quay ra home
```
cd ~
```
2️⃣ tạo thư viện
```
mkdir mylib
cd mylib
```
3️⃣ tạo file header
```
nano mylib.h
#ifndef MYLIB_H
#define MYLIB_H

void hello();

#endif
```
4️⃣ tạo file source
```
nano mylib.c
#include <stdio.h>
#include "mylib.h"

void hello()
{
    printf("Hello from mylib\n");
}
```
#### Thêm package vào Buildroot
Mở:
```
nano package/Config.in
```
thêm 2 dòng này ở cuối file
```
source "package/mylib/Config.in"
source "package/myapp/Config.in"
```
lưu.

#### Bật package trong menuconfig
```
make menuconfig
```
vào:
```
Target packages
```
tìm:
```
myapp
```
tick:
```
[*] myapp
```
Save → Exit.

#### Build lại
```
make
```
đợi build xong.

#### Sau khi build xong
flash lại image ra SD:
```
sudo dd if=output/images/sdcard.img of=/dev/sdb bs=4M status=progress
sync
```

#### Boot BBB
sau khi boot

<img width="605" height="123" alt="Picture14" src="https://github.com/user-attachments/assets/ca715997-1b05-4ca2-8016-0f272d51fb82" />



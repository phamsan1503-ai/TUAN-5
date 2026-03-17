<img width="945" height="159" alt="image" src="https://github.com/user-attachments/assets/7b33cd07-76ac-4f7b-8fc9-a26d7f55b0d8" /># Bài tập HDH Nhúng - Biên dịch chéo thư viện và ứng dụng
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
nano package/Hello_Json/src/HelloJSON.c
```
code:
```
#include <stdio.h>
#include <stdlib.h>
#include <cjson/cJSON.h> 

int main() {
    const char *json_string = "{\"name\":\"BeagleBone\", \"version\":\"Black\", \"id\":123}";
    cJSON *json = cJSON_Parse(json_string);
    
    if (json == NULL) {
        printf("Loi parse JSON!\n");
        return 1;
    }

    cJSON *name = cJSON_GetObjectItemCaseSensitive(json, "name");
    cJSON *version = cJSON_GetObjectItemCaseSensitive(json, "version");
    cJSON *id = cJSON_GetObjectItemCaseSensitive(json, "id");

    printf("--- Thong tin JSON tu Buildroot Package ---\n");
    if (cJSON_IsString(name)) printf("Device: %s\n", name->valuestring);
    if (cJSON_IsString(version)) printf("Model: %s\n", version->valuestring);
    if (cJSON_IsNumber(id)) printf("ID: %d\n", id->valueint);

    cJSON_Delete(json);
    return 0;
}
```

<img width="945" height="434" alt="image" src="https://github.com/user-attachments/assets/e765d74f-aed4-47a6-84e8-aea567a2e5a3" />


________________________________________
### Viết HelloJSON.mk
Dùng toolchain của buildroot
Mở:
```
nano package/Hello_Json/HelloJSON.mk
```
Viết chương trình:
```
HELLO_JSON_VERSION = 1.0
HELLO_JSON_SITE = $(HELLO_JSON_PKGDIR)/src
HELLO_JSON_SITE_METHOD = local

HELLO_JSON_DEPENDENCIES = cjson

define HELLO_JSON_BUILD_CMDS
	$(TARGET_CC) $(TARGET_CFLAGS) $(@D)/HelloJSON.c -o $(@D)/Hello_Json $(TARGET_LDFLAGS) -lcjson
endef

define HELLO_JSON_INSTALL_TARGET_CMDS
	$(INSTALL) -D -m 0755 $(@D)/Hello_Json $(TARGET_DIR)/usr/bin/Hello_Json
endef

$(eval $(generic-package))

```

File này nói cho Buildroot biết:
-	compile HelloJSON.c
-	link thư viện cjson
-	copy binary vào /usr/bin.



_______________________________________

### Viết config.in
Cắm SD card vào máy tính.
Mở:
```
nano package/Hello_Json/Config.in
```
Viết chương trình:
```
config BR2_PACKAGE_HELLO_JSON
	bool "Hello_Json"
	select BR2_PACKAGE_CJSON
	help
	  Chuong trinh parse JSON don gian cho bai tap BBB.

```
Ý nghĩa:
-	tạo option Hello_Json trong menuconfig
-	tự bật thư viện cJSON.

_______________________________________
### Đăng ký package vào Buildroot
Mở file chính:
```
nano package/Config.in
```
Kéo xuống cuối file, thêm:
```
source "package/Hello_Json/Config.in"
```
Lưu lại.
________________________________________
### Build lại image và chạy trên BBB

<img width="945" height="64" alt="image" src="https://github.com/user-attachments/assets/09ed2935-0faa-464a-888e-c9fe6fc261eb" />

<img width="945" height="220" alt="image" src="https://github.com/user-attachments/assets/51b01941-9ddb-43e1-bdd0-f2f8037045b6" />


## Bài tập 02: Tự tạo thư viện cá nhân
Vào thư mục package của repo
```
cd ~/buildroot
```

### Tạo cấu trúc thư viện ` mylib `
Gõ lệnh:
```
nano mylib.c
```
Lúc này ta sẽ có cấu trúc:
```
#include <stdio.h>

buildroot
└── package
    └── mylib
        └── src

```
- Tạo file header ` mylib.h`

```
nano mylib/src/mylib.h
#ifndef MYLIB_H
#define MYLIB_H

int add(int a, int b);

#endif

```

- Tạo file ` mylib.c`
```
nano mylib/src/mylib.c
#include "mylib.h"

int add(int a, int b)
{
    return a + b;
}

```
- Tạo ` Config.in`
```
nano mylib/Config.in
config BR2_PACKAGE_MYLIB
    bool "mylib"
    help
      Thu vien tu tao thuc hien phep cong.

```

- Tạo ` mylib.mk`
```
nano mylib/mylib.mk
MYLIB_VERSION = 1.0
MYLIB_SITE = $(MYLIB_PKGDIR)/src
MYLIB_SITE_METHOD = local
MYLIB_INSTALL_STAGING = YES

define MYLIB_BUILD_CMDS

	# Build static library
	$(TARGET_CC) $(TARGET_CFLAGS) -c $(@D)/mylib.c -o $(@D)/mylib.o
	$(TARGET_AR) rcs $(@D)/libmylib.a $(@D)/mylib.o

	# Build shared library
	$(TARGET_CC) $(TARGET_CFLAGS) -fPIC -shared $(@D)/mylib.c -o $(@D)/libmylib.so

endef

define MYLIB_INSTALL_STAGING_CMDS

	$(INSTALL) -D -m 0644 $(@D)/mylib.h $(STAGING_DIR)/usr/include/mylib.h
	$(INSTALL) -D -m 0755 $(@D)/libmylib.a $(STAGING_DIR)/usr/lib/libmylib.a
	$(INSTALL) -D -m 0755 $(@D)/libmylib.so $(STAGING_DIR)/usr/lib/libmylib.so

endef

define MYLIB_INSTALL_TARGET_CMDS

	$(INSTALL) -D -m 0755 $(@D)/libmylib.so $(TARGET_DIR)/usr/lib/libmylib.so

endef

$(eval $(generic-package))

```

### Đăng ký package vào Buildroot
Mở file:
```
nano ~/buildroot/package/Config.in
```
Thêm dòng:
```
source "package/mylib/Config.in"
```

### Viết chương trình test `mylib_test`
Tạo thư mục:
```
cd ~/buildroot/package
mkdir -p mylib_test/src

```
- Tạo file ` main.c`
```
nano mylib_test/src/main.c
#include <stdio.h>
#include <mylib.h>

int main()
{
    printf("--- Bai tap 02 ---\n");
    printf("15 + 25 = %d\n", add(15,25));
    return 0;
}

```

- Tạo ` mylib_test.mk`
```
nano mylib_test/mylib_test.mk
MYLIB_TEST_VERSION = 1.0
MYLIB_TEST_SITE = $(MYLIB_TEST_PKGDIR)/src
MYLIB_TEST_SITE_METHOD = local
MYLIB_TEST_DEPENDENCIES = mylib

define MYLIB_TEST_BUILD_CMDS

	# Dynamic link
	$(TARGET_CC) $(TARGET_CFLAGS) $(@D)/main.c -o $(@D)/app_dynamic -lmylib

	# Static link
	$(TARGET_CC) $(TARGET_CFLAGS) $(@D)/main.c -o $(@D)/app_static -L$(STAGING_DIR)/usr/lib -lmylib -static

endef

define MYLIB_TEST_INSTALL_TARGET_CMDS

	$(INSTALL) -D -m 0755 $(@D)/app_dynamic $(TARGET_DIR)/usr/bin/app_dynamic
	$(INSTALL) -D -m 0755 $(@D)/app_static $(TARGET_DIR)/usr/bin/app_static

endef

$(eval $(generic-package))
```
- Tạo ` Config.in`
```
nano mylib_test/Config.in
config BR2_PACKAGE_MYLIB_TEST
    bool "mylib_test"
    select BR2_PACKAGE_MYLIB
    help
      Chuong trinh test thu vien.
```

- Đăng ký package và Bật package
Mở lại:
```
nano ~/buildroot/package/Config.in
```

Thêm:
```
source "package/mylib_test/Config.in"
```
Bật package chọn
```
[*] mylib
[*] mylib_test
```

<img width="945" height="647" alt="image" src="https://github.com/user-attachments/assets/cce7c041-1342-43d0-b21b-174b538e7964" />


### Build
```
make
```
Sau khi build xong Binary nằm ở ` output/target/usr/bin`
Sẽ có:

```
app_dynamic
app_static

```

<img width="878" height="325" alt="image" src="https://github.com/user-attachments/assets/1c6cb94e-589d-496f-8cc7-99e5f9de81d0" />


### So sánh kích thước chương trình
Dùng lệnh:
```
ls -lh app_*
```

<img width="930" height="120" alt="image" src="https://github.com/user-attachments/assets/f7f7db98-dd31-47c4-83fa-0eb15a4cfc9f" />

-	Dynamic link → dùng thư viện ngoài (libmylib.so)
-	Static link → nhúng toàn bộ thư viện vào file

###  Kiểm tra dependencies bằng ` readelf`
Với chương trình dynamic
```
readelf -d app_dynamic
```
Sẽ thấy:


<img width="945" height="159" alt="image" src="https://github.com/user-attachments/assets/a8d96050-bd9f-41d9-840c-0b017e3261aa" />

Với chương trình static

```
readelf -d app_static
```
Kết quả thường:


<img width="945" height="79" alt="image" src="https://github.com/user-attachments/assets/006a3273-889d-43bb-872f-73eda982df69" />


## Bài tập 03: Tích hợp ứng dụng và thư viện và Buildroot

### Cấu trúc package
```
package/APP_ADD_JSON
│
├── Config.in
├── APP_ADD_JSON.mk
└── src
    └── main.c
```

### Code chương trình
```
#include <stdio.h>
#include <cjson/cJSON.h>
#include <mylib.h>

int main()
{
    int a = 20;
    int b = 30;

    int sum = add(a, b);

    cJSON *root = cJSON_CreateObject();

    cJSON_AddStringToObject(root, "Title", "Bai Tap 03 - Tong Hop");
    cJSON_AddNumberToObject(root, "Input_A", a);
    cJSON_AddNumberToObject(root, "Input_B", b);
    cJSON_AddNumberToObject(root, "Result_Sum", sum);

    char *json_print = cJSON_Print(root);

    printf("%s\n", json_print);

    cJSON_Delete(root);

    return 0;
}

```
#### File ` APP_ADD_JSON.mk`
```
APP_ADD_JSON_VERSION = 1.0
APP_ADD_JSON_SITE = $(APP_ADD_JSON_PKGDIR)/src
APP_ADD_JSON_SITE_METHOD = local

# Ràng buộc phải có 2 thư viện này thì mới build app
APP_ADD_JSON_DEPENDENCIES = cjson mylib

define APP_ADD_JSON_BUILD_CMDS
	# Sử dụng biến $(@D) để trỏ vào thư mục build tạm thời
	$(TARGET_CC) $(TARGET_CFLAGS) $(@D)/main.c -o $(@D)/APP_ADD_JSON \
		$(TARGET_LDFLAGS) -lcjson -lmylib
endef

define APP_ADD_JSON_INSTALL_TARGET_CMDS
	# Ép file thực thi vào thư mục /usr/bin trên SD Card
	$(INSTALL) -D -m 0755 $(@D)/APP_ADD_JSON $(TARGET_DIR)/usr/bin/APP_ADD_JSON
endef

$(eval $(generic-package))
```

### File ` Config.in`
```
config BR2_PACKAGE_APP_ADD_JSON
    bool "APP_ADD_JSON"
    select BR2_PACKAGE_CJSON
    select BR2_PACKAGE_MYLIB
    help
      Chuong trinh tong hop su dung cJSON (Bai 1) va mylib (Bai 2).

```
### Build ứng dụng

+	Chạy make menuconfig và bật APP_ADD_JSON.
+	Build bằng make.

### Kết quả
Binary ` output/target/usr/bin/APP_ADD_JSON` được tạo.
Tạo SD card và boot
```
cd ~/Documents/buildroot/
sudo dd if=output/images/sdcard.img of=/dev/sda bs=4M status=progress conv=fsync
sync
```
### Chạy chương trình trên thiết bị

```
$ APP_ADD_JSON
{
 "Title": "Bai Tap 03 - Tong Hop",
 "Input_A": 20,
 "Input_B": 30,
 "Result_Sum": 50
}

```

 <img width="945" height="548" alt="image" src="https://github.com/user-attachments/assets/40cca8e3-479b-433f-95de-5c0ae94da6c2" />




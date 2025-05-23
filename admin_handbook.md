# 101 bài admin server thiếu nhi
## I. Xử lý khi server sập từ A-Z (software only)
### 1. Cài lại centos
#### ************ Chỉ cài lại OS trong trường hợp bất khả kháng ***************
a. **Thuật ngữ**: không hiểu thì chatgpt google dịch gì đó giùm cái :)))  
b. **Chuẩn bị**: 1 ubs boot centos (image ở đây: https://drive.google.com/drive/folders/1B3CQqbP8FTMMioJ1iPqqWj6XhhCtxTWi?usp=drive_link) (có thể burn bằng balena etcher, rufus, create boot device trong GNOME, ...), 1 hoặc nhiều ổ đĩa sao lưu và 1 tâm hồn đẹp :)))  
***Lưu ý***: Nên phân vùng cho usb sao lưu bằng mfs.ext4 của chính server để tránh rắc rối trong việc nhận usb sau này
c. ***Nên sao lưu***: `/etc`, `/root`, cẩn trọng khi xử lý `/home`, `/` như bên dưới  
  
**ND chính**: Tham khảo vid dưới đây nhưng thao tác khác 1 chút như sau:  
Do os cũ vẫn tồn tại trong ổ đĩa nên ta vẫn thấy được các phân vùng của OS cũ: `/boot`, `/boot/efi`, `/`, `/home`. Xử lý chúng như sau:
+ Xóa các phân vùng boot của os cũ `/boot`, `/boot/efi`
+ Với `/home`: ***chỉ xóa khi sao lưu toàn bộ*** hoặc chọn mount point là `/home` (như vậy `/home` sẽ giữ nguyên trong OS mới)
+ Với `/`:  xóa khi đã sao lưu lại những file quan trọng hoặc chọn mount point trong hệ thống mới (VD: `/mnt/old)`. Không thể mount `/` hay bất kỳ mount point cũ nào vào `/` trong os mới 
+ Kích thước các phân vùng /boot, /boot/efi để theo giá trị mặc định (1024/2048mb), phân vùng tối đa cho swap (hình như là 800gb, cứ phân hẳn 1Tb nó sẽ tự bóp xuống)

Link vid: https://www.youtube.com/watch?v=hYBXyooK6yk  

### 2. Fix yum repo EOL và thiết lập GUI
###### Giải thích: Do centos 7 đã bị ngừng hỗ trợ (end of life/EOL) nên yum repo chính thức, mặc định đã không còn tồn tại. Rất may chúng ta vẫn còn nhiều mirror hỗ trợ
Fix yum repo EOL: google nguyên cụm từ này **hoặc** chạy lệnh như sau nếu đã sao lưu
```
cp /mnt/usb/etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo
```
Thay `/mnt/usb` bằng mount point usb khác tùy chúng m  
Sau đó thì là những câu lệnh hết sức kinh điển (không hiểu thì hỏi gpt):
```
yum clean all
yum update
yum -y groups install "GNOME Desktop"
startx
```
### 3. Set up tailnet
###### 1 trong những mắt xích quan trọng nhất của hệ thống server này là tailscale. Sự ưu việt của nó nằm ở chỗ nó là 1 hệ thống VPN zerotrust (không qua server trung gian của bên thứ 3), miễn phí và hoàn toàn không cần config phức tạp và có thể kết nối vượt gateway, tức là không cần mở public ip và vì vậy rất an toàn. Nếu ai có nhu cầu có thể đọc thêm ở đây: https://tailscale.com/blog/how-tailscale-works)
Tải tailscale:
- Mở link này trong firefox của root: https://tailscale.com/download. Copy lệnh mẫu của họ paste vào terminal chạy
- Chạy xong sẽ ra 1 link trong terminal, mở link này trong firefox rồi đăng nhập vào mail server để chạy. Cụ thể mail và pass server là gì thì không ghi ở đây được vì repo này đang public :)))
- Để chắc chắn, vào tab admin console để check rằng đã thêm machine. Nếu thêm rồi thì di chuột vào gần dấu ... như ảnh để share
  ![image](https://github.com/user-attachments/assets/70d79a1b-267f-4851-b0e6-2eae0669e12f)
Tạo reusable link rồi gửi lên cho mn truy cập
![image](https://github.com/user-attachments/assets/55d60fb2-3e87-410c-abc3-08b52b20a350)


Hướng dẫn set up tailscale ở phía client: https://github.com/cern143/edabk_SoC_doc/blob/main/edabk_server_manual.md
### 4. Khôi phục user và các service
#### Khôi phục group và user
Nếu còn đã sao lưu root (có thể phải sửa đường dẫn):
```
cd /root
cp /mnt/usb/etc/group /etc/group
cp /mnt/usb/root/* .
chmod +x restore_and_clean_shadow.sh
./restore_and_clean_shadow.sh
```
Giải thích: group là 1 trong 3 loại đối tượng sở hữu trong linux: owner, group, others, tức là 1 file sẽ có 3 phân quyền khác nhau cho owner, group và other (nếu muốn biết thêm google 3 lệnh: chmod, chown và chgrp). `etc/group` là file chứa thông tin về các group, copy nó sang là khôi phục dc thông tin này
Nội dung script `restore_and_clean_shadow.sh` như sau:
```
#!/bin/bash

# Source and destination paths
SRC_DIR="/mnt/usb/etc"
DEST_DIR="/etc"

# Files
PASSWD_FILE="passwd"
SHADOW_FILE="shadow"

# Backup originals
timestamp=$(date +%s)
cp "${DEST_DIR}/${PASSWD_FILE}" "${DEST_DIR}/${PASSWD_FILE}.bak.$timestamp"
cp "${DEST_DIR}/${SHADOW_FILE}" "${DEST_DIR}/${SHADOW_FILE}.bak.$timestamp"

echo "Backed up current passwd and shadow files with timestamp: $timestamp"

# Copy new files from USB
cp "${SRC_DIR}/${PASSWD_FILE}" "${DEST_DIR}/" || { echo "Failed to copy passwd"; exit 1; }
cp "${SRC_DIR}/${SHADOW_FILE}" "${DEST_DIR}/" || { echo "Failed to copy shadow"; exit 1; }

echo "Copied passwd and shadow from USB."

# Clean up shadow file
TMP_FILE="/tmp/shadow.clean.$$"
> "$TMP_FILE"

users=$(cut -d: -f1 "${DEST_DIR}/${SHADOW_FILE}" | sort | uniq)

for user in $users; do
    lines=$(grep "^$user:" "${DEST_DIR}/${SHADOW_FILE}")
    good_line=$(echo "$lines" | grep -Ev "^$user:!!:")
    
    if [ -n "$good_line" ]; then
        echo "$good_line" >> "$TMP_FILE"
    else
        echo "$lines" | head -n 1 >> "$TMP_FILE"
    fi
done

mv "$TMP_FILE" "${DEST_DIR}/${SHADOW_FILE}"


echo "Cleaned duplicate locked entries from shadow file."
echo "Done."
```
Giải thích: thông tin của 1 user được chứa trong `/etc/passwd`, còn mật khẩu được mã hóa hash được chứa trong `/etc/shadow`. Script này sẽ sao lưu 2 file này trước (vì nó khá quan trọng) rồi ghi đè từ hệ thống cũ sang. Nhưng khi ghi đè như vậy linux sẽ tự khóa entry của từng user vì cách thêm này bị coi là không hợp lệ (không qua lệnh useradd). Rất may mọi thứ trong linux chỉ là file, cách nó khóa entry là thêm line khóa vào /etc/shadow nên (script) xóa mấy dòng đó đi là được.





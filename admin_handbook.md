# 101 bài admin server thiếu nhi
## I. Xử lý khi server sập từ A-Z (software only)
### 1. Cài lại centos
#### ************ Chỉ cài lại OS trong trường hợp bất khả kháng ***************
**Thuật ngữ**: không hiểu thì chatgpt google dịch gì đó giùm cái :)))  
**Chuẩn bị**: 1 ubs boot centos (image ở đây: https://drive.google.com/drive/folders/1B3CQqbP8FTMMioJ1iPqqWj6XhhCtxTWi?usp=drive_link) (có thể burn bằng balena etcher, rufus, create boot device trong GNOME, ...), 1 hoặc nhiều ổ đĩa sao lưu và 1 tâm hồn đẹp :)))  
***Nên sao lưu***: `/etc`, `/root`, cẩn trọng khi xử lý `/home`, `/` như bên dưới  
Tham khảo vid dưới đây nhưng thao tác khác 1 chút như sau:  
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




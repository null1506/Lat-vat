#### Use ja-netfilter agent (jetbrain)

- Truy cập vào đường link sau để tải Jetbra: `https://3.jetbra.in/`
- Download jetbra.zip
- Giải nén và tìm đến file `\jetbra\vmoptions\idea.vmoptions`, copy từ dòng thứ 2 đến hết và thay thế từ dòng thứ 2 đến hết của file `\IntelliJ IDEA 2024.1.4\bin\idea64.exe.vmoptions` (tìm đến file này thì chuột phải 
vào shortcut của Intellij chọn open file location rồi tìm trong thư mục đó)
- tìm đến file `\jetbra\vmoptions\jetbrains_client.vmoptions`, copy từ dòng thứ 2 đến hết và thay thế từ dòng thứ 2 đến hết của file `\IntelliJ IDEA 2024.1.4\bin\jetbrains_client64.exe.vmoptions` (tìm đến file này thì chuột phải 
vào shortcut của Intellij chọn open file location rồi tìm trong thư mục đó)
- Thêm dòng này của cuối của mỗi file vừa rồi `-javaagent:/path/to/ja-netfilter.jar=jetbrains` (`/path/to` là đường dẫn đến file ja-netfilter, file này nằm trong thư mục jetbra đã giải nén, copy đường dẫn và dán thay thế
cho /path/to (VD: `javaagent:D:\jetbra-5a50fc03d68a014f893b7fc3aa465380d59f9095\jetbra\ja-netfilter.jar=jetbrains`)
- Bật Intellij lên và vào tab Active Code, Copy code Intellij trong https://3.jetbra.in/, sau đó dán vào là được

Một vấn đề nữa đọc ở đây: https://gist.github.com/nort3x/dde6757cb630afe052331e04ca1ab79e
- Cá nhân tôi đã gặp tình trạng chậm và các vấn đề khi sử dụng agent ja-netfilter.
Tôi quyết định kiểm tra cách JetBrains xác minh tính hợp lệ của giấy phép (vì mặc dù tôi đã chỉ rõ với JetBrains là làm việc offline, nhưng nó vẫn kiểm tra license).
Đây là các kết luận của tôi: có hai domain chịu trách nhiệm thu hồi (revoke) các license không hợp lệ:
`www.jetbrains.com`
`account.jetbrains.com`
- Bạn có thể chặn chúng ở router hoặc chặn cục bộ bằng các lệnh iptables sau:
```
sudo iptables -I OUTPUT -p udp --dport 53 -m string --hex-string "|03|www|09|jetbrains|03|com|" --algo bm -j DROP
sudo iptables -I OUTPUT -p udp --dport 53 -m string --hex-string "|07|account|09|jetbrains|03|com|" --algo bm -j DROP
sudo ip6tables -I OUTPUT -p udp --dport 53 -m string --hex-string "|03|www|09|jetbrains|03|com|" --algo bm -j DROP
sudo ip6tables -I OUTPUT -p udp --dport 53 -m string --hex-string "|07|account|09|jetbrains|03|com|" --algo bm -j DROP
```
### Cảnh báo (Warning)
- Các luật iptables sẽ bị reset khi reboot nếu bạn không cấu hình cho chúng tồn tại lâu dài (persistent).
- Để làm cho iptables persistent trên Ubuntu (sau khi chạy các lệnh iptables ở trên):
sudo apt install iptables-persistentsudo iptables-save | sudo tee /etc/iptables/rules.v4 > /dev/null
sudo ip6tables-save | sudo tee /etc/iptables/rules.v6 > /dev/null
- Tại sao lại “gắt” như vậy?
Khi tôi chỉnh sửa /etc/hosts, JetBrains bắt đầu từ chối 127.0.0.1 làm server kiểm tra tính hợp lệ và bắt đầu gửi các gói UDP tới các DNS provider phổ biến:
8.8.8.8
1.1.1.1
1.0.0.1
8.8.4.4
9.9.9.9
- Tôi không ngờ tới điều này, nên tôi quyết định chặn hoàn toàn việc IntelliJ được resolve DNS trên hệ thống, từ bất kỳ nguồn nào.

### Trên Windows, cơ chế bảo mật mạng hoạt động khác. Windows Firewall mặc định không hỗ trợ chặn gói tin dựa trên "chuỗi Hex" (Deep Packet Inspection) giống như Linux.
- Tuy nhiên, bạn vẫn có thể đạt được mục đích "chặn JetBrains gọi về nhà" trên Windows bằng cách làm theo 2 bước sau. Cách này tương đương với giải pháp "gắt" mà tác giả đoạn văn nhắc tới:
- Bước 1: Sửa file Hosts (Cơ bản)
Đây là bước đầu tiên để điều hướng các địa chỉ của JetBrains về "ngõ cụt" (0.0.0.0).
Mở Start Menu, gõ Notepad.
Chuột phải vào Notepad chọn Run as administrator (Chạy với quyền Admin).
Trong Notepad, chọn File > Open, dán đường dẫn này vào ô Address bar rồi Enter: C:\Windows\System32\drivers\etc\
Chọn hiển thị All Files (.) ở góc dưới bên phải để thấy file tên là hosts.
Mở file hosts, thêm các dòng sau vào cuối file:
Lưu lại (Ctrl + S).
Lưu ý: Như đoạn hướng dẫn gốc của bạn đã cảnh báo, JetBrains rất "khôn". Nếu nó thấy file hosts chặn, nó sẽ tự động bỏ qua và cố kết nối trực tiếp đến Google DNS (8.8.8.8) để tìm IP thật. Vì thế, trên Windows, bạn bắt buộc phải làm thêm Bước 2 để chặn triệt để.
- Bước 2: Dùng Windows Firewall chặn ứng dụng (Nâng cao)
Thay vì chặn DNS phức tạp như Linux, trên Windows cách hiệu quả nhất là cấm hoàn toàn ứng dụng IDE kết nối Internet.
Mở Start Menu, gõ "Windows Defender Firewall with Advanced Security" và mở nó lên.
Ở cột bên trái, chọn Outbound Rules (Luật đi ra).
Ở cột bên phải, chọn New Rule...
Chọn Program > Next.
Chọn This program path, bấm Browse và tìm đến file chạy của IDE bạn đang dùng.
Ví dụ IntelliJ: C:\Program Files\JetBrains\IntelliJ IDEA 2023.x\bin\idea64.exe
Bấm Next, chọn Block the connection (quan trọng nhất).
Tích chọn cả 3 ô: Domain, Private, Public > Next.
Đặt tên (ví dụ: Block IntelliJ) và bấm Finish.
Tại sao làm thế này lại hiệu quả trên Windows?
Cách làm này còn "gắt" hơn cả đoạn code Linux kia.
Linux code: Chỉ chặn gói tin hỏi đường (DNS). Nếu IDE biết sẵn IP, nó vẫn kết nối được.
Windows Firewall (Bước 2): Cắt hoàn toàn đường ra Internet của riêng ứng dụng đó. Dù IDE có đổi DNS, có dùng 4G hay Wifi, nó cũng không thể gửi bất kỳ byte dữ liệu nào ra ngoài.

Nhược điểm: Bạn sẽ không thể update IDE tự động hoặc tải plugin từ Marketplace trong IDE (phải tải file plugin về cài tay). Nếu bạn chấp nhận làm việc offline hoàn toàn thì đây là cách an toàn nhất trên Windows.



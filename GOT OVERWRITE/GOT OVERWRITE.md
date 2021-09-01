> P/s: Một số tài liệu tham khảo mình dùng để viết research này:
> - [GOT OVERFLOW](https://infosecwriteups.com/got-overwrite-bb9ff5414628) của [AidenPearce369](https://aidenpearce369.medium.com/)  
> - [Global Offset Table (GOT) and Procedure Linkage Table (PLT)](https://www.youtube.com/watch?v=kUk5pw4w0h4) của [LiveOverflow](https://www.youtube.com/channel/UClcE-kVhqyiHCcjYwcpfj9w)


# Cơ bản cần phải biết
Trong quá trình compiling chương trình, compiler sẽ **triệu hồi** linker =))) để 
thực hiện việc liên kết object code với (các object code khác, static lib, dynamic lib) để tạo ra file .EXE  

Khi một executable được tạo, vị trí của bất cứ hàm thư viện được tham chiếu bởi excutable đó phải được tìm ra. Linker có hai phương thức cho việc giải quyết các lời gọi tới các hàm thư viện:
- Liên kết tĩnh (Static link) 
- Liên kết động (Dynamic link)

Các đối số của command line cung cấp đến linker để xác định xem phương thức nào được sử dụng.

Một excutable có thể được liên kết tĩnh, liên kết động, hoặc cả hai.

## Dynamic Link Library (Thư viện liên kết động)  

Là file, thư viện chứa các mã nguồn, hàm dữ liệu sẽ được tải vào chương trình (executable) trong quá trình chạy của chương trình (runtime)  

Nhiều chương trình có thể cùng lúc sử dụng 1 thư viện vì thế nó còn được gọi là shared library  

Thư viện liên kết động Window sẽ có đuôi `.dll`, Linux là `.so`, MacOS là `.dylib`  

#### Ưu điểm
- Giảm không gian sử dụng bộ nhớ
- Có thể đóng gói và đưa vào chương trình khác
- Dễ nâng cấp chương trình, ứng dụng. Khi nâng cấp/ thay đổi, chỉ cần thay file .dll cũ bằng file .dll mới
- Có khả năng tương tác giữa các ngôn ngữ lập trình

#### Nhược điểm
- Chương trình, ứng dụng sẽ không thể chạy khi thiếu 1 file .dll nhất định
- Chương trình sử dụng thư viện động thường chạy chậm hơn những chương trình sử dụng thư viện tĩnh  

![image](https://user-images.githubusercontent.com/74854445/129679105-b4a36f35-1f25-4548-9855-d9d36c8c93d1.png)


## Static Link Library (Thư viện liên kết tĩnh)

Khác với thư viện liên kêt động, khi các chương trình chạy hay được dịch sang mã nhị phân, chúng sẽ sao chép các đoạn mã có sẵn trong thư viện tĩnh vào trong mã nguồn của mình và chạy như thư viện là 1 phần của chương trình.  

Trong window, thư viện tĩnh có đuôi là `.lib`, còn trong Linux là `.a`  

#### Ưu điểm
- Các chương trình sử dụng thư viện tĩnh thông thường sẽ chạy rất nhanh bởi chúng không mất thời gian để mở thư viện ra mà dịch
- Các chương trình có thể chạy độc lập mà không cần file đính kèm
- Dễ thực hiện

#### Nhược điểm
- Do khi chạy, các chương trình sẽ copy toàn bộ thư viện, kích thước của các chương trình/ ứng dụng sẽ phình to, dẫn tới tốn bộ nhớ
- Khi thay đổi/ nâng cấp thư viện thì cần chỉnh lại toàn bộ các file chương trình   

![image](https://user-images.githubusercontent.com/74854445/129679153-97aca3d1-128f-4174-97cb-80c73bfcc0d8.png)


## PROCEDURE LINKAGE TABLE (PLT)  

PLT là một section "chỉ đọc"  

Nó chịu trách nhiệm cho việc gọi trình liên kết động (dynamic linker) trong và sau thời gian chạy của chương trình để xử lý địa chỉ của những hàm được yêu cầu  

Trong quá trình biên dịch chúng ta không thể đề cập đến những địa chỉ vì những địa chỉ này là không xác định, mỗi hệ thống (system) sẽ có một địa chỉ riêng và đối tượng được chia sẻ thư viện cũng không có sẵn  

=> PLT đóng vai trò quan trọng trong việc giải quyết, xử lý địa chỉ của những hàm này trong quá trình chạy của chương trình *(during runtime)*  

PLT lớn hơn GOT  

Mỗi chương trình sẽ có 1 PLT của riêng nó và PLT đó cũng chỉ có tác dụng với chương trình đó  

Khi một hàm được gọi/yêu cầu, yêu cầu đó được gửi tới PLT bởi calling function (lời gọi hàm) lúc này địa chỉ của GOT sẽ được đẩy vào (push) thanh ghi xử lý  

## GLOBAL OFFSET TABLE (GOT)  

GOT được lấy ra (pop) bởi dynamic linker trong quá trình chạy của chương trình  

Dynamic linker nhận được 1 địa chỉ xác định của hàm được yêu cầu và cập nhật GOT theo yêu cầu  

Nhiều hàm sẽ không được xử lý, giải quyết, thực hiện trong thời gian chạy. Chúng chỉ được xử lý trong lần gọi hạm đầu tiên  

> Tạm hiểu là trong quá trình chạy chương trình hàm nào được gọi thì sẽ xử lý hàm đó. Hàm nào không được gọi thì không cần xử lý  

=> Quá trình trên còn được gọi là "lazy linking" (liên kết lười =))) ), quá trình này giúp tiết kiệm được tài nguyên  

Một khi mà địa chỉ PLT của hàm được liên kết vơi địa chỉ GOT của hàm, chương trình có thể gọi được hàm đó từ thư viện ngay lập tức nhờ sự giúp đỡ của PLT và GOT  

![image](https://user-images.githubusercontent.com/74854445/129679252-81c6a5e5-09b1-4393-966a-85a41fea9ad6.png)

# GOT OVERWRITE  

> Giờ mới vô vấn đề chính nè :)))  

GOT OVERWRITE là khi địa chỉ hàm trong GOT bị thay thế bằng địa chỉ mà chúng ta mong muốn  

Giả sử khi chúng ta sẽ gọi hàm `printf()` trong chương trình  

Chương trình sẽ tiến hành kiểm tra địa chỉ của hàm trong PLT trước, đồng thời cũng tìm địa chỉ của hàm trong GOT để liên kết 2 địa chỉ lại với nhau  

Từ đó `printf()` có thể được gọi ngay lập tức từ thư viện  

Nếu chúng ta ghi đè địa chỉ của hàm trong GOT với địa chỉ của một hàm khác mà chúng ta muốn  

Khi hàm `printf()` được gọi, chương trình sẽ tiến hành tìm địa chỉ trong PLT rồi tìm địa chỉ trong GOT, tìm thấy địa chỉ đã bị chúng ta thay đổi và thực thi hàm mà ta muốn  

Tham số (agrument) được chuẩn bị cho hàm `printf()` cũng được chuyển sang thành tham só cho hàm mà ta muốn  

# KHAI THÁC
> Phần này mình sẽ làm thử một ví dụ về GOT OVERFLOW nhưng do đang hơi lười, nên mọi người xem tạm VD [GOT OVERFLOW](https://infosecwriteups.com/got-overwrite-bb9ff5414628) của [AidenPearce369](https://aidenpearce369.medium.com/)  
> 
> Khi nào rảnh thì mình sẽ làm một cái riêng sau :3

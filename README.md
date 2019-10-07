## Thông tin cơ bản:
Một chương trình C trên hệ thống Linux để phục vụ như một giao diện shell chấp nhận các lệnh của người dùng và sau đó thực thi mỗi lệnh trong một quy trình riêng biệt. Việc triển khai của bạn sẽ hỗ trợ chuyển hướng đầu vào và đầu ra, cũng như các đường ống dưới dạng IPC giữa một cặp lệnh.

## Hướng dẫn sử dụng
* Quá trình thực hiện cơ bản
* Thực hiện lệnh trong một tiến trình con (Executing Command in a Child Process)
* Tính năng lịch sử (History)
* Chuyển hướng đầu vào và đầu ra (Redirecting Input and Output)
* Giao tiếp lệnh qua đường ống (Communication via a Pipe)

## Quá trình thực hiện cơ bản 
Hàm `main()` trình bày dấu nhắc `osh>` ở mỗi dòng. Người dùng sẽ nhập lệnh cần thực hiện vào sau dấu nhắc `osh>` và `Enter` để thực thi

Tiến trình cha đọc những gì người dùng nhập vào dòng lệnh và sau đó tạo một quy trình con riêng biệt thực hiện lệnh. Trừ khi có quy định khác, quy trình cha chờ con thoát ra trước khi tiếp tục. 

Hàm main () liên tục lặp vòng miễn là mode `run` bằng 1

Khi người dùng nhập `exit` tại dấu nhắc, mode `run` được đặt về `0` và chương trình chấm dứt.

## Thực hiện lệnh trong một tiến trình con (Executing Command in a Child Process)
Quá trình con riêng biệt được tạo bằng cách sử dụng lệnh gọi hệ thống `fork()` và lệnh người dùng được thực thi bằng một trong các lệnh gọi hệ thống trong họ `exec()`. Một tiến trình con được rẽ nhánh và thực thi lệnh được chỉ định bởi người dùng

Ở đây, phân tích chuỗi lệnh người dùng nhập vào thành từng mã thông báo riêng, mỗi mã lưu trong một chuỗi kĩ tự. Gọi hàm `execvp()(args[0], args)` để thực thi. Nếu kiểm tra thấy người dùng có bao gồm kí tự `&` ở cuối lệnh thì xác định quy trình cha không chờ đợi con thoát ra.

## Tính năng lịch sử (History)
Cho phép người dùng thực thi lệnh gần đây nhất bằng cách nhập `!!`. 

Ví dụ: nếu người dùng nhập lệnh `ls -l`, thì anh ta có thể thực hiện lại lệnh đó bằng cách nhập `!!` tại dấu nhắc. 

Bất kỳ lệnh nào được thực hiện theo kiểu này sẽ được lặp lại trên màn hình người dùng và lệnh này cũng được đặt trong bộ đệm lịch sử làm lệnh tiếp theo. 

Nếu không có lệnh gần đây trong lịch sử, hãy nhập `!!` sẽ dẫn đến một thông báo `“No commands in history.”`.

Đồng thời, nếu kiểm tra lệnh cũ là `!!` thì sẽ vẫn giữ lại bộ đệm lịch sử, hoặc không thì sẽ giải phóng bộ đệm và bắt đầu ghi

## Chuyển hướng đầu vào và đầu ra (Redirecting Input and Output)
`>` Hồi chuyển hướng đầu ra của một lệnh thành một tệp.
`<` Hồi chuyển hướng đầu vào thành một lệnh từ một tệp. 

Ví dụ: Nếu người dùng nhập `ls> out.txt`, đầu ra từ lệnh `ls` sẽ được chuyển hướng đến tệp `out.txt`. 
Tương tự, đầu vào cũng có thể được chuyển hướng. 
Ví dụ: nếu người dùng nhập `sort <in.txt`, tệp `in.txt` sẽ đóng vai trò là đầu vào cho lệnh `sort`.

Việc quản lý chuyển hướng của cả đầu vào và đầu ra sẽ liên quan đến việc sử dụng hàm `dup2()`, sao chép một bộ mô tả tệp hiện có sang một bộ mô tả tệp khác. 
Ví dụ: nếu `f` là một mô tả tệp cho tệp `out.txt`, sau dòng lệnh `dup2(f, STDOUT_FILENO);` f được nhân bản thành đầu ra tiêu chuẩn `stdout`. Điều này có nghĩa là bất kỳ ghi vào `stdout` sẽ được gửi đến tệp `out.txt`.

## Giao tiếp lệnh qua đường ống (Communication via a Pipe)
Cho phép đầu ra của một lệnh được dùng làm đầu vào cho một lệnh khác bằng cách sử dụng một đường ống. 

Chúng ta tách lệnh ban đầu thành 2 lệnh con có dạng `command1 | command2`, sau đó nối 1 đầu pipe vào `stdout` và thực thi lệnh `command1` để lấy kết quả
Bước tiếp theo, ta nối đầu còn lại pipe vào `stdin` và chạy `command2`

Ví dụ: chuỗi lệnh sau `ls -l | less` có đầu ra của lệnh `ls -l` đóng vai trò là đầu vào cho lệnh `less`. 
Cả hai lệnh `ls` và `less` sẽ chạy như các tiến trình riêng biệt và sẽ giao tiếp bằng cách sử dụng hàm `pipe()` của UNIX. 

Ở đây chúng ta tách lệnh ban đầu thành 2 lệnh con có dạng `command1: ls -l` và `command2: less`, sau đó nối 1 đầu pipe vào `stdout` và thực thi lệnh `command1` để lấy kết quả, ta được 
Bước tiếp theo, ta nối đầu còn lại pipe vào `stdin` và chạy `command2`
Vậy là đầu ra của lệnh `ls -la` được dùng làm đầu vào của `less`. Lệnh người dùng được hoàn tất thực thi

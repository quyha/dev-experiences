![dns](https://github.com/quyha/dev-experiences/blob/main/assets/dns.jpg)

### Khu vực 1

**Cấu hình trỏ tên miền thông thường sử dụng hiện nay**

* WWW với loại là Cname dùng để chuyển hướng từ www sang không sử dụng www
* @ với loại là A dùng để trỏ về địa chỉ hosting.

Như vậy sau khi bạn thiết lập xong khu vực một thì nó luôn ở dạng không có www và được trỏ về hosting vps

### Khu vực 2

Khu vực này khá quan trọng bởi lẽ với Host là giá trị * và loại là A và đi kèm là đại chỉ IP sẽ cho ra kết quả tất cả các tên miền phụ như sub.domain.com cho tới giá trị n subomain domain.com luôn trỏ về địa chỉ ip mà bạn thiết lập.

Ưu điểm của nó là bạn sẽ không phải mất công mỗi lần tạo địa subdomain mới là phải cấu hình trỏ về ip đó dù ip đó dùng chung. Nhưng nhược điểm là nếu mỗi subdomain trên hosting hay vps khác nhau thì điều này không đảm nhận được và bạn phải sang khu vực 3.

### Khu vực 3

Với khu vực này thì mỗi subdomain bạn có thể cấu hình theo nơi bạn muốn trỏ. Ví dụ bạn cấu hình subdomain A về địa chỉ ip nào đó và subdomain B về địa chỉ ip riêng.

Như hình trên có 2 subdomain phukien, lienviet và trỏ về 2 địa chỉ ip. Nếu mà muốn trỏ thêm thì tiến hành làm tương tự với

* Host: tên subdomain
* Loại: A
* Địa chỉ: là địa chỉ ip muốn tạo

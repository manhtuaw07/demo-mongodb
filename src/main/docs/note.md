I. Overall
1. Tại sao chúng ta cần schema?  
Dù MongoDB là schemaless, tức là không cần phải định nghĩa trước cấu trúc, nhưng việc sử dụng schema vẫn mang lại 1 số lợi ích như: đảm bảo tính nhất quán, dễ dàng validate, tối ưu hóa hiệu năng và index, dễ bảo trì.
2. Data types?
3. Mối quan hệ, khi nào dùng embedded, khi nào dùng references?
- Embedded khi:  
    1. 2 thực thể luôn xuất hiện và được truy vấn cùng nhau
    2. Cardinality thấp: mối quan hệ 1-1 hoặc 1-N, N rất thấp. Ví dụ: 1 user có vài địa chỉ.

- Reference khi:
    1. Dữ liệu độc lập: 2 thực thể có vòng đời tách biệt, cập nhập, mở rộng riêng rẽ.
    2. Cadinality cao: 1-N, N rất lớn, hoặc N-N.
    3. Chỉ truy vấn khi cần, không phải lúc nào cùng cần dữ liệu liên quan.  

- Mối quan hệ N-N thông thường trong sql sẽ lưu các id thông qua 1 bảng phụ:  
VD: customer, product, order sẽ là bảng trung gian lưu thông tin giữa customer và các product họ đặt.  
Còn trong MongoDB, chúng ta có cách tiếp cận khác, không dùng bảng trung gian, chúng ta lưu order như là 1 field trong customer, giá trị của nó là 1 document, có các trường meta data khác kèm theo đó là trường productID reference tới productID trong product tương ứng. Điều này giúp giảm thiểu các dữ liệu trùng lặp.
Đây là sự kết hợp giữ embedded và referrence.  

? Thiết kế DB cho Post Application:  
Collections: User, Post consists of: tags, comments.  
User:  
{
    _id: 1,
    name: 'Alex'
    email:'alex@test.com'
}  

Post:  
{
    _id: 1,
    text: 'Hello world',
    tags: ['news', 'tech'],
    comments: {
        userId: 1,
        text: 'like shit'
        createdAt: new Date()
    }
}

4. Joining with $lookup  
5. Schema Validation  
6. Config file
7. Sự phân tán của MongoDB
Được thiết kế để phân tán dữ liệu theo chiều ngang ngay từ đầu. Cơ chế chính là Sharding, được tích hợp sẵn, là 1 phần cốt lõi của kiến trúc.  
Shards trong MongoDB có sự tương đồng vs Partition trong Kafka:
Mỗi shards đc phân phối trên nhiều máy chủ khác nhau, mỗi shards là 1 replica set, replica chính là primary chịu trách nhiệm ghi, sau đó sao chép vào các máy chủ phụ. Điều này dẫn tới option Write Concern (tương tự như ACKS)

II. CRUD
1. Create
Ordered insert: mặc định, MongoDB sẽ thực thi câu lệnh insertMany theo thứ tự (Ordered inserts), thành công tới đâu commit tới đó, nếu có lỗi ở docs nào thì sẽ dừng lại ở docs đó.  
nếu để option là ordered: 'false', khi có lỗi ở 1 docs, nó vẫn tiếp tục thực hiện các docs khác.  
Đảm bảo Atomicity khi insertOne và không khi insertMany.  

1.1. Write Concern
Cho phép bạn quyết định xem khi nào 1 thao tác đc coi là thành công.
- Unacknowledged (w: 0): ko chờ xác nhận
- Acknowledged (w: 1) (default): xác nhận từ Priority.
- Journaled (w: 1, j: true): xác nhận ừ Priority và Journal(nhật ký) đc ghi vào đĩa.
- Replica Acknowledged (w: "majority"): chờ xác nhận từ phần lớn các node trong replica set.

Write Concern có thể được thiết lập trên DB, Collection hoặc từ mỗi câu lệnh.

2. Read
2.1. Read Concern
2.2. Các operators thông dụng
- @and: mậc định khi dùng cho 1 filter truyền vào là 1 array. Hữu dụng khi phép so sánh lặp lại key, khi ko dùng @and thì key cuối cùng sẽ ghi đè. Dùng @and để kết hợp condition giữa các same keys.
- $type: kiểm tra kiểu dữ liệu của field
- $exists: kiểm tra field đó có tồn tại hay không, field = null thì $exist vẫn là true, trong trường hợp này cần check thêm null.
- $expr: kiểm tra giữa các fields trong cùng 1 document, các field đc đánh dấu bằng $ + tên field.  
VD: {$expr: {$gt: ["$volumn", "$target"]}}  
- $cond: trong $cond là các câu điều kiện rẽ nhánh: if then else

2.3 Array
VD: user: {
    hobbies:
    [
        {
            title: "sport"
            frequency: 1
        },
        {
            title: "sing"
            frequency: 2
        }
    }
}

- là 1 mảng các doc, chúng ta có thể truy cập tới các field trong doc thông qua Tên array trỏ tới các field đó. VD: hobbies.title
- tìm kiếm với 1 mảng đầu vào, nếu dùng ":", nó sẽ là order master. Nếu yc cầu không quan tâm thứ tự mà chỉ quan tâm tới giá trị đang tìm kiếm thì dùng $all
- Lưu ý khi tìm kiếm với $and, với nhiều điều kiện đầu vào, mỗi phần tử match với mỗi điều kiện khác nhau đều thỏa mãn.  VD: Điều kiện: $and: [{title: "sport"}, {frequency: 2}] => matching.  
Nếu muốn áp dụng điều kiện trên cùng 1 element thì dùng $elemMatch

2.4. Cursor, skip và limit
Kết quả của find và aggregate là 2 cursor

3. Update
4. Delete

Best practice:  
- dùng insertOne, insertMany, tránh dùng insert => dễ dàng sửa lỗi và bảo trì.

III. Indexes

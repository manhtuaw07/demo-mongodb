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


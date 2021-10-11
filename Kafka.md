### KAFKA

– Đó là hệ thống **message pub/sub** phân tán (distributed messaging system). Bên **pulbic** dữ liệu được gọi là **producer**, bên **subscribe** nhận dữ liệu theo topic được gọi là **consumer**. **Kafka** có khả năng truyền một lượng lớn **message** theo thời gian thực, trong trường hợp bên nhận chưa nhận message vẫn được lưu trữ sao lưu trên một hàng đợi và cả trên ổ đĩa bảo đảm an toàn. Đồng thời nó cũng được **replicate** trong **cluster** giúp phòng tránh mất dữ liệu.

![kafka](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/5ubjohwx8q_cautrucchitiet.png)

- Mô hình cấu trúc kafka gồm các thành phần sau
  - **Message**: thông tin được gửi đi, có thể là text, binary, json hoặc một định dạng format nào đó
  - **Broker**: Một host có thể chạy nhiều server kafka, mỗi server như vậy gọi là một broker. Các broker này cùng trỏ tới chung 1 zookeeper, gọi là cụm broker hay là **clusters**. Broker là nơi chứa các partition. Một broker có thể chứa nhiều partition.
  - **Topic**: Là nơi chứa các message được publish từ **Producer** tới kafka, nhìn về mặt database thì topic như một table trong cơ sở dữ liệu quan hệ, và mỗi massage như một bản ghi của table đó. Dữ liệu truyền trong Kafka theo topic, khi cần truyền dữ liệu cho các ứng dụng khác nhau thì sẽ tạo ra cá topic khác nhau.
  - **Partition**: Đây là nơi dữ liệu cho một topic được lưu trữ. Một topic có thể có một hay nhiều partition. Trên mỗi partition thì dữ liệu lưu trữ cố định và được gán cho một ID gọi là offset. Trong một Kafka cluster thì một partition có thể replicate (sao chép) ra nhiều bản. Trong đó có một bản leader chịu trách nhiệm đọc ghi dữ liệu và các bản còn lại gọi là follower. Khi bản leader bị lỗi thì sẽ có một bản follower lên làm leader thay thế. Nếu muốn dùng nhiều consumer đọc song song dữ liệu của một topic thì topic đó cần phải có nhiều partition.
  - **Producer**: Chương trình/ service tạo ra message, đẩy massage publish vào **Topic**
    - Ackowledgement: Producer sẽ nhận được phản hồi khi tất cả replication leader và IRS[đồng bộ bản nháp in sync replication] write data thành công.
    - Round-robin: Message 1 đi vào broker 1, message 2 đi vào broker 2, message 3 đi vào broker 3,các message được write balance giữa các broker .
    - Message key - Hash key partitioning: Mỗi message được gắn 1 key nếu muốn điều hướng message đến chính xác **partition** mong muốn đảm bảo message ordering và các message liên quan với nha sẽ được gửi vào cùng partition và order theo thứ tự.
    - Trước khi xử lý, Kafka sẽ thực hiện phân loại và lưu trữ các message dựa theo topic của chúng. Producer có nhiệm vụ publish message vào các topic thích hợp. Sau đó, khi dữ liệu được gửi đến partition của topic được lưu trữ tại Broker.
  - **Consumer**: Chương trình/ service có chức năng subscribe vào một Topic đã tiêu thụ, xử lý các message đó.[1 consusmer có thể đọc toàn bộ message của tất cả partition thuộc cùng topic]
    - Đây là nơi nhận các message và xử lý. Consumer đọc message từ topic xác định bằng topic name.
    - Consumer biết nên đọc message từ broker nào. Nếu chưa read xong mà broker gặp sự cố, consumer cũng có cơ chế tự phục hồi.
    - Việc read message trong một partition diễn ra tuần tự để đảm bảo message ordering. Consumer không thể đọc offset=3 khi chưa đọc message offset=2
    - Một consumer cũng có thể đọc message từ một hoặc nhiều hoặc tất cả partition trong một topic.
      **Chú ý :** + Message ording chỉ đảm bảo trong một partition. Việc đọc ghi message giữa nhiều partition không đảm bảo thứ tự + Message offset = 5 ở partition 0 có thể được đọc trước message offset = 2 ở partition 1.
  - **Consumer Group**: Giải quyết việc nếu số lượng producer tăng lên và đồng thời gửi message đến tất cả parititon trong khi chỉ có duy nhất một consumer thì khả năng xử lý sẽ rất chậm, có thể dẫn tới bottle-neck[bị bóp]. Nên giải pháp là phải tăng số lượng consumer, các consumer có thể xử lý đồng thời message từ nhiều partition. Và tất cả các consumer sẽ thuộc cùng một nhóm được gọi là consumer group. + Mỗi consumer thuộc consumer group sẽ đọc toàn bộ dât của một hoặc nhiều partition để đảm bảo mesage ordering. Không tồn tại nhiều consumer cùng đọc message từ một partition - **Chú ý** : Một consumer có thể nhận message từ nhiều partition. Nhưng một parititon không thể gửi mesage cho nhiều consumer trong cùng consumer group.
    ![consumegroup](https://i.imgur.com/m8kQHt7.png) + Trong một vài trường hợp consumer trong group lớn hơn số lượng partition thì nếu một active consumer gặp vấn đề thì một trong nhwuxng inactive consumer còn lại được đẩy lên thay thế và tiếp tục công việc ngay lập tức. Nếu không có inactive consumer nào thì message sẽ được route tới một active consumer bất kì khác. + Quá trình re-assign này được gọi là partition rebalance
  - Một chương trình/service có thể vừa là Producer vừa là Consumer
  - **Kafka cluster** là một nhóm các server và mỗi nhóm server này sẽ được gọi là broker.
    ![broker](https://wiki.tino.org/wp-content/uploads/2021/07/word-image-1417.png)
  - **ZOOKEEPER**: được dùng để quản lý và bố trí các broker.

### Message broker

- Có 2 hình thức giao tiếp basic với 1 message broker
  - Publish và Subscribe (Topics) -> Group
  - Point to Point (Queue) -> Message 1-1

### Cách hoạt động

- **Queuing** cho phép dữ liệu có thể được xử lý phân tán trên nhiều consumer và tạo ra khả năng mở rộng cao.
  ![Queuing](https://wiki.tino.org/wp-content/uploads/2021/07/word-image-1414.png)
- **Publish-subscribe** sẽ tiếp cận cùng lúc nhiều subscribe và các message sẽ được gửi đến nhiều subscribe, không thể sử dụng để phân tán công việc cho nhiều worker.
  ![Publish-subcribe](https://wiki.tino.org/wp-content/uploads/2021/07/word-image-1415.png)


### SOCKETCLUSTER 

- Là một mã nguồn mở, một framework real-time cho node.js
- Support cả giao tiếp giữa client với server, một nhóm thông qua sơ chế pub/sub channel.[tạo ra 1 kênh riêng để các phần tử trong 1 nhóm gia tiếp với nhau]

- SocketCluster được thiết kế để dễ dàng mở rộng số processes/host khi xây dựng hệ thống.
- Ví dụ hiện tại bạn chỉ có 1 con server Node.js để nhận và xử lý request, nhưng sau đấy số lượng request nhiều nên, 1 server không thể đáp ứng, ta sẽ dùng SocketCluster để gán thêm 1 con server y hệt như thế vào hệ thống nhằm giảm tải áp lực cho hệ thống

### Một số khái niệm trọng SocketCluster

- **Worker**: là đối tượng thực thi các chức năng của server, một server sockecluster có thể có nhiều thể hiện worker giúp giải quyết vấn đề single I/O của NODE.JS. 
- Ví dụ máy bạn có 2 CPU thì có thể tạo 2 thể hiện workder, khi có 2 request đồng thời thì nó sẽ chia ra và được xử lý bởi 2 worker trong cùng 1 lúc.
- **Broker** : đối tượng chịu trách nhiệm giao tiếp giữa các worker 
- **SocketCluster/SocketClient**: là đối tượng tạo kết nối socket, cho phép giao tiếp với server 1 cách real-time
- **Channel** : Kênh giao tiếp.  
- Ví dụ trong ứng dụng chat thì mỗi room chat sẽ là 1 channel, khi một client gửi tin nhắn vào room (publish) thì chỉ có những client đang trong room đó (subscribe) mới nhận được tin nhắn, còn các client ở trong channel/room khác thì ko nhận được.
# II. Sơ lược về DST và Mod

## 1. Cấu trúc server DST

Giống như mọi game online multiplayer khác, cấu trúc DST bao gồm server và client.

- **Server** là nơi diễn ra mọi sự kiện (*event*) quan trọng, ảnh hưởng tới thế giới (*world*) và người chơi. Ví dụ: thời tiết, mùa, quái, boss, vị trí các vật thể, các chỉ số, v.v. Nói chung là hầu hết mọi thứ.
- **Client** là tiến trình (*process*) nơi bạn chơi game. Đa số các sự kiện diễn ra ở Server sẽ được đẩy xuống client qua mạng (*network*), và client sẽ hiển thị tương ứng. Đồng thời các thao tác trên client do người chơi lại gửi lên server, tiếp tục tạo ra sự kiện và đẩy đến các client khác. Ngoài ra, ở client cũng diễn ra những sự kiện chỉ dành cho người chơi đó, không quá quan trọng và ảnh hưởng đến những người chơi khác hay thế giới, ví dụ cảnh báo tentacle của Wurt, những dòng chữ nói chuyện của Heo hay Thỏ, v.v. Việc những sự kiện này chỉ diễn ra trên client vừa để cá nhân hóa góc nhìn của người chơi, cũng vừa để giảm tải cho server, ví dụ rõ ràng tất cả người chơi không cần có góc nhìn đồng nhất về hội thoại của Heo/Thỏ/Merm, mỗi người có thể nhìn thấy 1 hội thoại khác nhau.

Khi bạn tạo 1 world DST không có Caves, có thể hiểu là bạn đang chơi ngay trên chính server, client của host chính là server.

Khi bạn tạo 1 world DST có caves, có thể hiểu trên máy bạn có Server và Client chạy riêng. Client của host có toàn quyền điểu khiển Server, ví dụ gửi command.

Với dedicated server, server chạy riêng, người chơi tham gia và trở thành client.

## 2. Sơ lược về game engine

Sau đây là những ý hiểu của mình về game engine của DST:

- Mỗi world chạy chủ yếu trên 1 luồng (*thread*), tức là chỉ có thể sử dụng chủ yếu 1 nhân (*core*) của CPU. Mặt đất (*Forest* trong setting tạo world) và Hang (*Caves*) được tính là 2 world riêng, do đó chạy song song trên 2 thread. Số world tối đa nên host trong 1 máy nên giới hạn bởi số core + bộ nhớ (*memory*).
- Mỗi world trên mỗi core vận hành theo kiểu event-based, nếu bạn đã từng code Javascript/NodeJS thì y hệt. Tóm gọn lại tức là, mọi thứ trong 1 world thực ra đều chạy **LẦN LƯỢT**. Nếu bạn thấy 2 con nhện cắn bạn cùng một lúc, thực chất nó sẽ là như sau: con nhện 1 cắn bạn, rồi con nhện 2 cắn bạn, hoặc ngược lại, nhưng vì CPU chạy rất nhanh nên cảm tưởng nó là cùng một lúc. Trong đa số mọi trường hợp, bạn sẽ không phải quá quan tâm về vấn đề này, mình sẽ đi sâu hơn về nó khi làm mod.
- Server và client đều chạy như vậy, và giao tiếp với nhau qua mạng. Với mod đơn giản, bạn sẽ không phải quá quan tâm về việc giao tiếp giữa server và client. Nếu bạn làm Server-and-all-client-required mod, đa số mọi thứ đều được tự động đẩy từ Server xuống Client. Nếu bạn làm Client-only mod, bạn càng không phải quan tâm đến Server. Chỉ khi có những tính năng game không hỗ trợ giao tiếp server-client sẵn, bạn mới phải nhúng tay vào.
- Code game mà mình có thể đọc được viết bằng Lua. Lua được biết là một ngôn ngữ nhúng, tức là nó có thể chạy trên nền một ngôn ngữ khác. Theo mình đoán, phần code game bằng Lua này chỉ để lập trình các logic cho game, còn phần engine thật nằm bên dưới, có thể được viết bằng 1 ngôn ngữ bậc thấp cho hiệu năng cao. Cái này không quá quan trọng :).
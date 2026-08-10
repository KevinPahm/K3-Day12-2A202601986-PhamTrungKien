# Phiếu Phản Ánh — K3 Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Phạm Trung Kiên  Mã học viên: 2A202601986

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> *Ví dụ khi deploy lên server, nếu quên cấu hình AGENT_API_KEY, app sẽ báo lỗi ngay khi khởi động thay vì chạy với key "changeme". Nhờ vậy tôi phát hiện cấu hình sai sớm và tránh việc app chạy nhưng các request tới API bị lỗi hoặc dùng một API key không an toàn.*

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> *Ví dụ một dòng log JSON tôi thu được {"answer":"Theo mình hiểu, What is Docker liên quan tới cách hệ thống được đóng gói và vận hành. Điểm mấu chốt là tách cấu hình ra khỏi code và giữ service ở trạng thái stateless.","user_id":"sv-test","history_length":0,"cost_usd":2.565e-05,"tokens":{"in":3,"out":42}} Từ log này, tôi có thể lọc request theo user_id và theo dõi chi phí, số token sử dụng qua cost_usd và tokens. Với print("đã trả lời xong"), tôi không có các thông tin có cấu trúc này để máy tự động phân tích.*

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t agent:single .
docker build -t agent:multi .
docker images | grep agent
```

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | 1730 MB |
| Multi-stage | 270 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> *Multi-stage nhỏ hơn vì image cuối chỉ giữ lại các thành phần cần thiết để chạy ứng dụng, còn các dependency/build tools và file trung gian dùng trong quá trình build không được đưa vào image cuối.*

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> *Khi chỉ sửa một ký tự trong app/main.py, các layer trước COPY như cài dependency vẫn được lấy từ cache, còn layer COPY và các layer phía sau phải chạy lại. Nếu đặt COPY . . trước RUN pip install thì mỗi lần thay đổi source code, Docker có thể phải chạy lại pip install, làm thời gian build lâu hơn.*

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> *Nếu code Python có lỗ hổng và kẻ tấn công khai thác được để thực thi lệnh trong container, quyền của process trong container sẽ là quyền root. Nếu container có cấu hình hoặc quyền truy cập nguy hiểm, điều này có thể làm tăng khả năng ảnh hưởng tới host. USER chuyển process sang user thường, nên khi bị khai thác, kẻ tấn công không có quyền root trong container và chuỗi leo thang quyền bị hạn chế.*

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> *Nếu giới hạn là 10 request/phút nhưng reset theo phút đồng hồ, người dùng có thể gửi tối đa 20 request trong khoảng 2 giây: 10 request ngay trước thời điểm chuyển sang phút mới và 10 request ngay sau khi reset ở giây 00. Sliding window tránh được cách lách giới hạn này vì luôn xét 60 giây gần nhất.*

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> *Rate limit giới hạn số lượng request, còn cost guard giới hạn chi phí/tài nguyên tiêu thụ.Rate limit cho qua nhưng cost guard chặn: gửi ít request nhưng mỗi request yêu cầu xử lý rất lớn hoặc sử dụng nhiều token. Cost guard cho qua nhưng rate limit chặn: gửi rất nhiều request nhỏ, mỗi request rẻ nhưng vượt quá số request cho phép trong một khoảng thời gian.*

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> *Nếu gộp hai endpoint và /health cũng kiểm tra Redis, khi Redis mất kết nối thì cả 3 container đều có thể trả trạng thái lỗi. Orchestrator sẽ coi các container là không khỏe và có thể restart chúng. Vì Redis vẫn đang mất kết nối, các container mới tiếp tục fail health check và có thể bị restart lặp lại trong 30 giây. Vì vậy /health nên chỉ kiểm tra process còn sống, còn /ready mới kiểm tra dependency như Redis.*

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> *Nếu lịch sử được lưu trong Redis, cả 3 container đều có thể truy cập cùng một lịch sử nên history_length tăng tương đối ổn định sau mỗi request. Nếu lưu trong một dict Python, mỗi container có một dict riêng. Request được load-balancing sang container khác có thể thấy history_length thấp hơn hoặc quay lại giá trị trước đó. Vì vậy kết quả sẽ không ổn định giữa các request*

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> *Tôi không gặp lỗi khi deploy lên cloud nên không có lỗi thực tế để ghi lại. Quá trình deploy chạy thành công và các container có thể khởi động bình thường.*

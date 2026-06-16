# README (cho người) — Giao việc cho AI để làm công cụ soạn hợp đồng trên 1 file HTML

> Tài liệu này giải thích **concept** của công cụ và **cách giao việc cho AI** để tạo ra nó. Viết cho người đọc — kể cả người không chuyên lập trình. Bản thân công cụ là **bản TEST, không có giá trị pháp lý, không chịu trách nhiệm**.

---

## 1. Công cụ này là gì (1 phút)

Một **file HTML duy nhất** (~410KB), mở offline, chứa 10 loại hợp đồng thông dụng (lao động, thuê nhà, vay tiền, dịch vụ, mua bán hàng hóa, đặt cọc, cộng tác viên, NDA, nguyên tắc, ủy quyền).

Cách dùng: chọn loại hợp đồng → trả lời bảng câu hỏi → công cụ tự ráp và định dạng thành hợp đồng hoàn chỉnh → sửa, in, lưu PDF hoặc tải Word.

Ba điểm khiến nó hơn một "mẫu Word điền chỗ trống":
- **Phân biệt 3 loại điều khoản:** *bắt buộc* (thiếu là hỏng), *có mặc định của luật* (để trống thì luật tự áp dụng), *tùy chọn*.
- **Hiển thị mặc định minh bạch:** để trống ô "có mặc định" thì công cụ in thẳng nội dung luật vào hợp đồng, kèm điều luật.
- **Cảnh báo tuân thủ:** báo ngay khi vượt giới hạn luật (lãi suất, phạt vi phạm, lương tối thiểu, thử việc…).

---

## 2. Ý tưởng cốt lõi — và vì sao nó hợp để giao cho AI

Nguyên tắc quan trọng nhất: **tách DỮ LIỆU ra khỏi BỘ MÁY.**

- **Bộ máy (engine)** không biết gì về luật. Nó chỉ biết: sinh bảng câu hỏi, điền câu trả lời vào chỗ trống, kiểm tra ràng buộc, in/lưu.
- **Tri thức pháp lý** nằm trong **dữ liệu**: mỗi loại hợp đồng là một "bản mô tả" (gồm: hỏi gì, thân hợp đồng có điều khoản nào, ràng buộc luật ra sao).

Vì sao điều này quan trọng khi giao cho AI? Vì nó **chẻ bài toán lớn thành nhiều việc nhỏ độc lập**:
- Engine làm một lần, dùng chung.
- 10 hợp đồng = 10 việc nghiên cứu/soạn riêng → AI có thể làm **song song** (nhiều agent), miễn là tất cả tuân theo cùng một "khuôn dữ liệu".

Nói cách khác: bạn không bảo AI "viết một phần mềm khổng lồ", mà bảo nó "làm cái khung chung trước, rồi điền từng mảnh dữ liệu vào khung đó".

---

## 3. Hai cách để có "mẫu" hợp đồng (đây là lựa chọn quan trọng nhất)

Đây là điểm bạn phải quyết khi giao việc: **AI tự tạo mẫu, hay dùng mẫu sẵn có của bạn?**

| | (A) AI tự tạo mẫu | (B) Dùng mẫu sẵn có của bạn |
|---|---|---|
| **Cách làm** | Bạn mô tả loại hợp đồng → AI tự nghiên cứu luật, soạn nội dung điều khoản, phân loại trường | Bạn đưa mẫu hợp đồng chuẩn (của firm) → AI chỉ "số hóa" thành cấu trúc dữ liệu (đánh dấu chỗ điền, phân tầng điều khoản, đặt ràng buộc) |
| **Ưu điểm** | Nhanh, phủ rộng nhiều loại, không cần có sẵn mẫu | Nội dung **đáng tin** vì do người chuyên môn duyệt; rủi ro pháp lý thấp hơn nhiều |
| **Nhược điểm** | AI **có thể sai hoặc bịa điều luật** → BẮT BUỘC phải kiểm định độc lập + người chuyên môn rà | Cần có sẵn mẫu tốt; tốn công chuẩn bị mẫu |
| **Hợp khi** | Làm bản TEST/demo, khám phá ý tưởng, phủ nhanh | Dùng thật trong văn phòng, cần độ chính xác cao |

**(C) Cách kết hợp (khuyến nghị cho dùng thật):** để AI tạo *bản nháp* → người có chuyên môn sửa thành *mẫu chuẩn của mình* → từ đó công cụ dùng mẫu đã được duyệt. Vừa nhanh vừa an toàn.

> Công cụ TEST này được làm theo cách (A): AI tự nghiên cứu luật và soạn 10 mẫu, **mỗi mẫu có một AI khác kiểm định độc lập** (tự tra luật lại). Đủ tốt cho bản demo, nhưng để dùng thật vẫn nên chuyển sang (C).

---

## 4. Quy trình giao việc cho AI (từng bước)

Đây là cách thực tế đã dùng — bạn có thể lặp lại với bất kỳ ý tưởng "công cụ trên 1 file HTML" nào.

**Bước 1 — Bàn trước, chưa code.** Mở đầu bằng *thảo luận ý tưởng*: tính khả thi, rủi ro, nên thêm/bớt gì. Đừng để AI nhảy vào code ngay. (Chính dự án này bắt đầu bằng một buổi "discuss, chưa build".)

**Bước 2 — Chốt định vị & phạm vi.** Bạn (không phải AI) quyết: dùng để TEST hay dùng thật? Cho ai? Bao nhiêu loại hợp đồng? Có disclaimer/miễn trách nhiệm không? Luật nước nào?

**Bước 3 — Yêu cầu AI tách "khung" và "dữ liệu".** Nói rõ: làm một *engine dùng chung* + mỗi hợp đồng là một *bản mô tả dữ liệu* theo một khuôn thống nhất. Đây là chìa khóa để mở rộng và để chạy song song.

**Bước 4 — Chốt "khuôn dữ liệu" trước khi sản xuất hàng loạt.** Khuôn này (mỗi hợp đồng cần khai báo những gì) chính là "giao kèo" để mọi mảnh ghép khớp nhau. Sai khuôn ở đây thì hỏng hàng loạt về sau.

**Bước 5 — Chọn cách lấy mẫu** (mục 3): AI tự tạo / mẫu sẵn / kết hợp.

**Bước 6 — Giao song song + LUÔN có kiểm định độc lập.** Nếu để AI tự tạo mẫu: với mỗi mẫu, cho **một AI khác đóng vai người rà**, tự tra luật để kiểm tra — không tin lời AI soạn. (Đây là điểm sống còn về độ tin cậy.)

**Bước 7 — Nghiệm thu bằng cách CHẠY THẬT.** Đừng tin câu "đã xong". Mở file lên, bấm thử từng mẫu, xem có lỗi, có thiếu thông tin, có sai số không. (Dự án này được kiểm bằng cách chạy thử cả 10 mẫu trong trình duyệt.)

**Bước 8 — Người chuyên môn rà cuối** trước khi dùng thật.

---

## 5. Những việc người PHẢI tự quyết (đừng để AI tự quyết)

- **Định vị & trách nhiệm pháp lý:** TEST hay sản phẩm? Có chịu trách nhiệm không? → ảnh hưởng disclaimer, mức độ cẩn trọng.
- **Mẫu chuẩn:** nếu dùng thật, nội dung điều khoản nên do người có chuyên môn duyệt.
- **Phạm vi:** loại hợp đồng nào, luật nước nào, ngôn ngữ gì.
- **Đánh đổi:** nhanh-rộng (AI tự tạo) hay chắc-chậm (mẫu duyệt sẵn).

AI giỏi ở phần *lặp đi lặp lại* và *dựng khung*; phán đoán pháp lý và chịu trách nhiệm vẫn là của con người.

---

## 6. Mẹo để AI làm ra thứ "đúng", không chỉ "trông như đúng"

Đây là vài nguyên tắc kỷ luật (mình dùng quy trình tên **Axiom**) giúp giảm rủi ro AI "tự tin sai":

1. **Bắt AI nghiên cứu trước, đừng tin trí nhớ của nó.** Với nội dung luật, yêu cầu nó *tra cứu nguồn* rồi mới viết.
2. **Phân rã rồi mới làm.** Yêu cầu kế hoạch + khuôn dữ liệu *trước*, đừng để nó viết một khối khổng lồ.
3. **Kiểm định độc lập.** Một AI khác (ngữ cảnh khác) tự tra cứu lại và phản biện — quan trọng hơn việc AI tự nói "tôi đã kiểm".
4. **Đòi bằng chứng, không đòi lời hứa.** "Đã verify" mà không có bằng chứng (đã chạy gì, tra nguồn nào) thì coi như chưa làm.
5. **Gắn độ tin cậy cho từng khẳng định.** Cái gì chắc, cái gì suy đoán, cái gì cần người kiểm — nói rõ.
6. **Nghiệm thu bằng hành vi thật**, không bằng mô tả.

> Ví dụ thật: trong lúc kiểm định, AI rà đã bắt được một trích dẫn sai (gán nhầm một cơ chế cho Nghị định thay vì Thông tư) và vài lỗi hiển thị — những thứ mà nếu chỉ tin "đã xong" thì sẽ lọt.

---

## 7. Giới hạn & trách nhiệm

- Đây là **bản TEST, không có giá trị pháp lý, không chịu trách nhiệm.**
- AI có thể sai hoặc lỗi thời về luật → **phải có người chuyên môn rà** trước khi dùng thật.
- Công cụ giúp *dựng nhanh và đỡ sai sót cơ học*, không thay thế tư vấn pháp lý.

---

*Kèm theo: `README-for-AI.md` — bản handover để một AI khác có thể bàn với bạn và dựng lại công cụ theo ý tưởng này.*

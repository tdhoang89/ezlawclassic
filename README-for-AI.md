# README (cho AI) — Handover để dựng lại "công cụ soạn hợp đồng trên 1 file HTML"

> Bạn là một AI nhận tài liệu này để **bàn với một người dùng và dựng lại công cụ theo concept dưới đây.** Tài liệu đúc kết từ một lần build có thật (10 mẫu hợp đồng Việt Nam, 1 file HTML, đã kiểm thử). Hãy coi nó là *điểm khởi đầu có kỷ luật*, không phải khuôn cứng — thích nghi theo nhu cầu người dùng.

---

## 0. Cách dùng tài liệu này

1. **ĐỪNG code ngay.** Trước hết hãy *thảo luận* với người dùng (mục 1).
2. Giữ nguyên **nguyên tắc kiến trúc** (mục 2–5) — đó là phần làm concept này hoạt động.
3. Khi build: theo **recipe điều phối** (mục 7) và **luật liêm chính/kiểm định** (mục 8).
4. Trước khi giao: đạt **Definition of Done** (mục 10).

---

## 1. Việc ĐẦU TIÊN: thảo luận với người dùng (hỏi trước khi build)

Hỏi và chốt các điểm sau — chúng thay đổi cả thiết kế:

- **Tài phán / luật áp dụng:** nước nào, lĩnh vực nào? (nội dung điều khoản phụ thuộc hoàn toàn vào đây)
- **Định vị & trách nhiệm:** TEST/demo hay dùng thật? Có disclaimer/miễn trách nhiệm không? (quyết định mức cẩn trọng + lời cảnh báo)
- **Nguồn mẫu — điểm quyết định (xem mục 6):** **AI tự tạo mẫu, hay người dùng đưa mẫu sẵn có để số hóa, hay kết hợp?**
- **Phạm vi:** những loại hợp đồng nào, ưu tiên loại nào trước?
- **Đối tượng dùng:** nội bộ (chuyên môn rà) hay người cuối tự dùng? (ảnh hưởng disclaimer & độ "cầm tay chỉ việc")
- **Mức độ "thông minh" của v1:** chỉ điền-và-ráp, hay có thêm *hiển thị mặc định của luật* + *cảnh báo tuân thủ*? (khuyến nghị có — đó là phần giá trị nhất)
- **Ngôn ngữ** đầu ra.

ES (escalation): nếu các điểm trên còn mơ hồ → hỏi, đừng tự giả định. Nếu người dùng yêu cầu "tự động hoàn toàn" → chọn mặc định hợp lý và **ghi rõ các mặc định đã chọn**.

---

## 2. Nguyên tắc kiến trúc cốt lõi (không thương lượng)

**Tách DỮ LIỆU (templates) khỏi BỘ MÁY (engine generic).**
- Engine không chứa tri thức pháp lý. Nó chỉ render form, điền chỗ trống, kiểm tra, in/lưu.
- Mỗi loại hợp đồng = một **object dữ liệu** theo schema ở mục 3.
- Thêm 1 loại hợp đồng = thêm 1 object, **không sửa engine**.

Hệ quả: bài toán chẻ thành (1) engine làm một lần + (2) N mảnh dữ liệu độc lập → build song song được.

---

## 3. Mô hình dữ liệu (schema) — "giao kèo" giữa dữ liệu và engine

Chốt schema này TRƯỚC khi sản xuất hàng loạt. Mọi mẫu phải tuân thủ để khớp engine.

```jsonc
// Một object = một loại hợp đồng
{
  "id": "vay_tien",                       // slug không dấu, duy nhất
  "name": "Hợp đồng vay tiền",
  "group": "Cá nhân ↔ Cá nhân",           // nhóm quan hệ, để gom nhóm ở trang chủ
  "icon": "💵",                            // emoji (không dùng file ảnh)
  "summary": "Mô tả ngắn khi nào dùng",
  "title": "HỢP ĐỒNG VAY TÀI SẢN",        // tiêu đề in hoa trong văn bản
  "preamble": "Căn cứ ...\nHôm nay, ngày {{ngay}}, gồm:\n- Bên A: {{ben_a}}...",
  "legalBasis": ["Bộ luật Dân sự 2015, Điều 463-471"],
  "parties": { "a": "Bên cho vay", "b": "Bên vay" },

  "fields": [                              // bộ câu hỏi
    {
      "key": "so_tien",                    // khớp với {{so_tien}} trong body
      "label": "Số tiền vay?",
      "section": "Nội dung vay",           // gom nhóm câu hỏi
      "tier": "required",                  // required | default | optional  (xem mục 4)
      "type": "money",                     // text|textarea|number|money|date|select|tel|email
      "help": "Giải thích ngắn dễ hiểu",
      "placeholder": "VD: 100000000",
      "legalRef": "Điều 463 BLDS",         // tùy chọn
      "defaultText": "...",                // BẮT BUỘC nếu tier=default (xem mục 4)
      "options": ["Có", "Không"],          // chỉ cho type=select
      "condition": { "key": "co_lai", "equals": "Có" }  // tùy chọn: chỉ hiện khi field khác = giá trị
    }
  ],

  "lint": [                               // ràng buộc luật (xem mục 5)
    { "field": "lai_suat", "op": "max", "value": 20,
      "message": "Vượt trần lãi suất 20%/năm", "legalRef": "Điều 468 BLDS", "severity": "error" }
    // op: "min" | "max" | "note"  (note = luôn hiển thị nhắc nhở, cho ràng buộc phức tạp/chéo trường)
  ],

  "body": [                               // thân hợp đồng, theo thứ tự các "Điều"
    { "heading": "ĐIỀU 1: SỐ TIỀN VAY", "text": "Bên A cho Bên B vay {{so_tien}}." },
    { "heading": "ĐIỀU 2: LÃI SUẤT", "text": "Lãi suất {{lai_suat}}%/năm.",
      "condition": { "key": "co_lai", "equals": "Có" } }   // điều khoản có điều kiện
  ],

  "closing": "Hợp đồng gồm 02 bản có giá trị như nhau...",

  "verifyNotes": ["(tùy chọn) nhật ký kiểm định: nguồn đã tra, lỗi đã sửa, cảnh báo còn lại"]
  // engine KHÔNG dùng verifyNotes; nó là dấu vết kiểm định ở lớp dữ liệu, thỏa "để lại bằng chứng" ở mục 8
}
```

**Quy tắc nhất quán bắt buộc:** mọi `{{key}}` trong `preamble`/`body`/`closing` phải có `field.key` tương ứng. Engine có lưới an toàn cho trường hợp lệch, nhưng đừng dựa vào đó.

---

## 4. "Công thức pháp lý" cốt lõi: phân 3 tầng điều khoản

Đây là phần khiến công cụ "hiểu luật" mà không cần AI lúc chạy. Gắn đúng `tier` cho mỗi field:

- **`required`** = điều khoản **chủ yếu**: thiếu thì hợp đồng khuyết/không hình thành → buộc điền. Để trống → engine in cảnh báo đỏ trong văn bản.
- **`default`** = điều khoản **thường lệ** (luật đã quy định sẵn): nếu các bên không thỏa thuận thì luật tự áp dụng. **BẮT BUỘC** kèm `defaultText` nêu đúng nội dung mặc định + `legalRef`. Để trống → engine **in thẳng `defaultText`** vào hợp đồng (minh bạch, không phải khoảng trắng).
- **`optional`** = điều khoản **tùy nghi**: thỏa thuận thêm nếu muốn. Để trống → in "………".

**Quy tắc vàng (và cái bẫy):** "không ghi → áp dụng mặc định của luật" CHỈ đúng với `default`, **KHÔNG** đúng với `required` (bỏ trống là hỏng). Nhiều mẫu Word bỏ qua đúng chỗ này.

---

## 5. Trách nhiệm của ENGINE (phần generic, làm một lần)

Engine vanilla JS, không thư viện, nhúng trong cùng file. Phải làm:

1. **Trang chủ:** gom các mẫu theo `group`, có ô tìm kiếm, mỗi mẫu một thẻ.
2. **Render form từ `fields`:** gom theo `section`; hiện huy hiệu tier; hiện `help`; với `default` hiện gợi ý "Nếu để trống: …"; ẩn/hiện field theo `condition`.
3. **Điền `{{key}}` (hàm cốt lõi)** với xử lý theo tier: có giá trị → in (định dạng theo type); `required` trống → cảnh báo đỏ; `default` trống → in `defaultText`; `optional` trống → "………"; (phòng hờ: `{{}}` không khớp field nào → coi như optional, để không vỡ văn bản).
4. **Lint:** đánh giá `min`/`max`/`note`, cảnh báo ngay khi nhập + tổng hợp ở bản xem trước.
5. **Tự đánh số "ĐIỀU":** bỏ số gốc trong dữ liệu, đếm lại theo các điều **đang hiển thị** (để điều khoản điều kiện ẩn/hiện không làm nhảy số). Remap số mục con (vd `3.1`) theo số điều mới.
6. **Đọc số tiền bằng chữ** (tự viết, ~30 dòng): nhóm 3 chữ số + nghìn/triệu/tỷ + quy tắc lẻ/mười-mươi/lăm/mốt.
7. **Định dạng văn bản:** header CHXHCNVN, tiêu đề, preamble, khối thông tin các bên, các điều, phần ký, **dòng disclaimer** ở cuối.
8. **Xuất bản (toàn bằng API trình duyệt):** In/PDF qua `window.print()` + CSS `@media print`; tải `.doc` bằng Blob có header MS Word HTML; tải `.html` standalone; **lưu nháp** bằng `localStorage`; **sửa tay** bằng `contentEditable`.
9. **Chuẩn hóa dữ liệu lúc nạp:** đổi ký tự `\n` dạng văn bản thành xuống dòng thật (bẫy thường gặp khi dữ liệu do AI sinh).
10. **3 màn hình = 3 div đổi class** (không cần framework/router).

---

## 6. HAI cách lấy mẫu hợp đồng (chốt với người dùng — xem mục 1)

### (A) AI tự tạo mẫu
- **Recipe:** với mỗi loại hợp đồng → 1 agent *nghiên cứu* (tra cứu luật hiện hành: điều khoản chủ yếu bắt buộc, quy định mặc định/bổ khuyết, giới hạn pháp lý) và soạn object đúng schema → **1 agent *kiểm định pháp lý độc lập*** tự tra cứu lại luật (KHÔNG tin agent soạn), sửa lỗi, bổ sung lint, đảm bảo nhất quán `{{}}`↔field.
- **Bắt buộc:** kiểm định độc lập + người chuyên môn rà cuối. AI có thể bịa số điều/sai văn bản.
- **Hợp khi:** TEST/demo, phủ nhanh nhiều loại.

### (B) Dùng mẫu sẵn có của người dùng
- **Recipe:** nhận mẫu hợp đồng đã được duyệt → *số hóa* thành schema: tách từng điều khoản vào `body`, xác định trường nào `required`/`default`/`optional`, đặt `{{placeholder}}`, mã hóa `lint` từ các giới hạn luật đã biết, viết `defaultText` cho điều khoản thường lệ.
- **Ưu:** nội dung đáng tin (do người chuyên môn duyệt), rủi ro pháp lý thấp. **Cần:** có sẵn mẫu tốt.
- **Hợp khi:** dùng thật.

### (C) Kết hợp (khuyến nghị cho dùng thật)
AI tạo nháp (A) → người chuyên môn sửa → thành mẫu chuẩn (B).

> Bản TEST gốc dùng cách (A): 10 mẫu, mỗi mẫu một verifier độc lập. Đủ cho demo; dùng thật nên chuyển (C).

---

## 7. Recipe điều phối (build bằng nhiều agent)

1. **Chốt schema (mục 3) trước.** Đây là giao kèo chung.
2. **Lấy mẫu** theo cách đã chốt (mục 6). Nếu (A): chạy *song song* `nghiên cứu → kiểm định` cho từng loại (pipeline/fan-out). Ép đầu ra đúng schema (structured output nếu có).
3. **Dựng engine single-thread** (đây là craft cần ngữ cảnh liền mạch — đừng chia nhỏ cho nhiều agent).
4. **Ráp:** nhét mảng dữ liệu vào engine (đơn giản: dán thẳng vào `<script>`; hoặc để mỗi mẫu một file rồi thay vào một chỗ đánh dấu kiểu `/*__TEMPLATES__*/`).
5. **Nghiệm thu bằng cách CHẠY THẬT** (mục 8).

Cảnh báo điều phối: chỉ fan-out phần *thực sự song song & độc lập* (nghiên cứu từng mẫu). Phần craft liền mạch (engine, văn phong tài liệu) để single-thread. Đừng spawn agent để "trông hoành tráng".

---

## 8. Luật liêm chính & kiểm định (đừng tin "đã xong")

- **Nghiên cứu trước, không dùng trí nhớ** cho nội dung luật; gắn nguồn cho mỗi khẳng định pháp lý.
- **Kiểm định độc lập với đầu vào không do người tạo soạn:** agent rà phải *tự đọc artifact/tự tra luật*, không nhận bản tóm tắt đã được "chọn lọc". (Tách ngữ cảnh thôi chưa đủ.)
- **Nghiệm thu engine bằng cách chạy thật** (mở trong trình duyệt/preview), với CẢ N mẫu, kiểm:
  - mọi `{{}}` đều resolve về một field (không sót placeholder),
  - số "ĐIỀU" liên tục (không nhảy),
  - thông tin các bên có xuất hiện trong văn bản,
  - không có lỗi console,
  - đọc số tiền bằng chữ đúng (test vài ca khó: lẻ/mười/mươi/lăm/mốt/tỷ).
- **Gắn độ tin cậy** (chắc / suy luận / cần người kiểm) và **để lại bằng chứng** (đã chạy gì, tra nguồn nào). Không có bằng chứng = coi như chưa làm.

---

## 9. Các cái bẫy đã gặp (đưa vào checklist)

1. **Escape HTML** mọi chuỗi người dùng trước khi nhét vào `innerHTML` (tránh vỡ layout/XSS).
2. **Bẫy `\n`:** dữ liệu AI sinh có thể chứa `\n` dạng văn bản → chuẩn hóa lúc nạp.
3. **Số "ĐIỀU" hardcode + điều khoản điều kiện = nhảy số** → tự đánh số lại theo điều đang hiển thị.
4. **Đừng âm thầm làm rơi dữ liệu:** trường người dùng nhập mà thân hợp đồng quên `{{}}` → vẫn render ra (lưới an toàn), nhất là thông tin các bên.
5. **Ô tiền tệ:** lưu số thô trong state, chỉ format khi hiển thị.
6. **Trích dẫn luật:** rất dễ sai số điều/nhầm văn bản (Nghị định ↔ Thông tư) → verifier phải tra lại từng cái.

---

## 10. Definition of Done

- [ ] **Một file HTML tự chứa**, mở offline (0 thư viện, 0 tài nguyên ngoài, 0 gọi mạng).
- [ ] N loại hợp đồng theo schema; mọi mẫu đã được **chạy thử thật** không lỗi.
- [ ] 3 tầng điều khoản hoạt động (đặc biệt: `default` để trống → in nội dung luật).
- [ ] Lint tuân thủ chạy đúng; số "ĐIỀU" liên tục; số tiền ra chữ đúng.
- [ ] In/PDF, tải .doc/.html, lưu nháp, sửa tay đều chạy.
- [ ] **Disclaimer** đúng định vị (nếu là bản TEST: "không có giá trị pháp lý, không chịu trách nhiệm").
- [ ] Nội dung pháp lý có **kiểm định độc lập** (+ ghi rõ phần nào cần người chuyên môn rà).

---

## 11. Cần xác nhận với người dùng — đừng tự giả định

Tài phán/luật · định vị & trách nhiệm · nguồn mẫu (A/B/C) · phạm vi loại hợp đồng · đối tượng dùng · ngôn ngữ · mức độ "thông minh" v1.

> Tinh thần: **một mẫu cứng chỉ ra một hợp đồng; một bộ công thức (engine + dữ liệu mô tả) ra cả một họ hợp đồng.** Giữ engine nhỏ và generic, đẩy mọi tri thức vào dữ liệu, và luôn kiểm định độc lập phần pháp lý.

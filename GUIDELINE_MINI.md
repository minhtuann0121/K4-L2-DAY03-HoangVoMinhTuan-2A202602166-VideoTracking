# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `T020 / Hoàng Võ Minh Tuấn`
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | **xe máy / mô tô** |
| xe tải, xe đầu kéo | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung của nhóm (nếu có): ``

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới ... frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | `Cùng một xe, chỉ mất tầm nhìn ngắn → nối lại đúng ID, tránh IDSW giả` |
| Xe bị che lâu hơn ngưỡng trên | `quá 2 giây thì mở track mới` | `Không còn bằng chứng thị giác để khẳng định là cùng một xe: sau 2 giây vật thể đã đi được quãng đáng kể, chỗ cũ có thể đã bị xe khác chiếm.` |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | `Rời khung là không còn quan sát nào để nối: không có bbox để so khớp, và khi quay lại thì góc nhìn/mặt xe/ánh sáng thường đã khác nên nối ID chỉ là phỏng đoán.` |
| Hai xe cắt nhau / chồng lên nhau | `Mỗi xe giữ ID riêng, không đổi ID cho nhau; vẫn vẽ một bbox cho mỗi xe, ôm phần nhìn thấy được; xe bị che hoàn toàn thì dùng Outside, không vẽ bbox vô hình; giữ ID nếu vắng < 25 frame; kiểm frame-by-frame quanh điểm cắt` | `Lúc cắt nhau hai bbox chồng gần hết nên IoU một mình không phân biệt được ai là ai — đây là chỗ dễ tráo ID nhất, mà ID là thuộc tính của vật thể, không phải của vùng ảnh. Tráo ID phá cả hai track cùng lúc: AssA/IDF1 mất toàn bộ phần còn lại của hai track, còn MOTA chỉ tốn 2 đơn vị IDSW nên gần như không phản ánh. Bbox chỉ ôm phần nhìn thấy vì luật cấm đoán phần bị che, và che hoàn toàn thì phải dùng Outside — vẽ bbox vô hình sẽ tính thành FP.` |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: **bbox rộng ≥ ~200 px (≥ 20% khung 960 px) và thấy được thân xe + bánh**; dưới ngưỡng đó coi như chưa xác định. Nhỏ nhất tôi thực sự bắt đầu một track: **208×31 px** (ID 6, frame 108) |
| Xe đang đỗ, không di chuyển | vẫn là `vehicle` và **track liên tục suốt thời gian trong khung, giữ một ID**; không bấm Outside chỉ vì xe không di chuyển (`GUIDE.md` dòng 42) |
| Keyframe đặt dày ở đâu | xe đi thẳng đều: cách **20–30 frame** (lướt kiểm tra trước — xe đi nhanh có thể chỉ còn 8–10 frame); dày **5–10 frame** ở chỗ xe **rẽ / phanh / bị che**, ở **frame đầu và cuối track**, và quanh chỗ bbox bị trôi (`GUIDE.md` dòng 81–89) |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01` — gold ID 6 **f101–116**, gold ID 5 **f79–89**, gold ID 8 **f136–138**; track của tôi tương ứng là ID 7 (bắt đầu f117), ID 5 (f90), ID 8 (f139)
- Tình huống: xe vào khung từ rìa, còn nhỏ/mờ hoặc bị che một phần, tôi chưa chắc là xe bốn bánh nên chờ tới lúc thấy rõ mới gán
- Quyết định: **chờ**, không gán ở frame đầu tiên xác định được
- Lý do: sợ gán nhầm sang vật thể khác. Đổi lại: **32/35 FN của tôi đến từ đây** (trễ 11–16 frame mỗi track). Luật đúng là "bắt từ frame đầu xác định được" → ngưỡng của tôi quá chặt

### Ca 2
- Clip / frame / ID: `clip_01`, **f101–117** — gold 6 ↔ ID 7 của tôi, gold 7 ↔ ID 6 của tôi (đánh số **ngược** nhau)
- Tình huống: hai xe vào khung lệch nhau vài frame rồi xếp gần/chồng nhau ở vùng rìa; tôi gán số theo thứ tự mình nhìn thấy chứ không theo thứ tự vào khung
- Quyết định: **giữ nguyên hai track, không đổi số**
- Lý do: tên ID không mang thông tin — evaluator giải bài toán gán cặp một-một trên toàn sequence nên **hoán vị nhất quán là miễn phí** (IDSW vẫn 0, IDF1 vẫn 0.957). Cái thực sự mất điểm là trễ 16 frame ở track đó


### Ca 3
- Clip / frame / ID: `clip_01` — ID 4 **f149–151** (ghost track, có trong diagnostics của `eval_vs_gold.json`), ID 3 **f44–45**, ID 5 **f139**, ID 7 **f157**, ID 8 **f169–170**; thêm ID 4 **f53** (khởi tạo sớm)
- Tình huống: xe ra khỏi khung, chỉ còn một phần/vệt ở rìa, tôi vẫn giữ bbox vì "vẫn còn thấy"
- Quyết định: **giữ thêm 1–3 frame** rồi mới bấm Outside
- Lý do: gold kết thúc track sớm hơn tôi, nên mỗi frame giữ thêm là 1 FP — **9/13 FP của tôi đến từ đây**. Luật đúng: Outside ngay ở frame cuối còn thấy xe

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- `Không có`

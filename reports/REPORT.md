# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: `Hoàng Võ Minh Tuấn / nhóm 20`
Ngày: `15/09/2026`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT / khác: `CVAT` |
| Thời gian gán `clip_02` (warm-up) | `10` phút |
| Thời gian gán `clip_01` | `15` phút |
| Số track đã vẽ trong `clip_01` | `8` |
| Số keyframe trung bình mỗi track | `...` |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. `Xe buýt đứng yên từ đầu tới cuối, khiến cho việc gán nhãn hay bị click nhầm. Cách giải quyết là nhãn buýt tôi ẩn đi`
2. `Xe chạy từ đầu tới cuối. Cách tôi giải quyết là kéo track từng frame một`
3. `Xe từ mép ảnh đi vào. Cách giải quyết là tôi nhìn thấy hẳn đầu xe hoặc một nửa chiếc xe rồi tôi mới gán nhãn`

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: `...`
- Lượt 2: `...`
- Lượt 3: `...`

Kiểm chéo với: `...`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `...`. Số lỗi bạn ấy tìm được trong bản của bạn: `...`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

`...`

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `10ad4392d5ec428ec60044d24d63840ef7253a0dc68c71f0e5a5ab664d9d4f43` |
| Thời điểm khóa | `2026-09-15T08:46:58.268987+00:00` |
| Số row / frame / track trước khi mở reference | **551 row / 190 frame / 8 track** (ID 1–8) |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.686 | 0.658 | 0.722 | 0.781 | 0.957 | 0.916 | 0.737 | 13 | 35 | 0 |
| Sau rework | 0.686 | 0.658 | 0.722 | 0.781 | 0.957 | 0.916 | 0.737 | 13 | 35 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có / chưa**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| | | | |
| | | | |
| | | | |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `3.13.15` / `8.4.145` / `2.11.0+cu128` / `0.5.13` |
| weights / hai tracker | `yolo26n.pt` · ByteTrack control (`bytetrack.yaml`) · BoT-SORT + ReID (`configs/trackers/botsort-reid.yaml`) |
| conf / IoU / imgsz / classes | `0.25` / `0.7` / `960` / `[2, 5, 7]` (car, bus, truck) |
| device | `"0"` (GPU Colab) |


| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.686 | 0.658 | 0.722 | 0.781 | 0.957 | 0.916 | 0.737 | 13 | 35 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.764 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| ReID vs bạn | 0.662 | 0.603 | 0.735 | 0.789 | 0.890 | 0.762 | 0.751 | 109 | 22 | 0 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

**MOTA thấp hơn IDF1: 0.916 so với 0.957.** Giả thiết trong câu hỏi ("MOTA cao mà IDF1 thấp")
**không đúng với bản nhãn này**, và evidence nói rõ vì sao: `IDSW = 0`, `IDFP/IDFN` không có
đóng góp nào từ lỗi ID. Toàn bộ khoảng cách `1 − MOTA = 0.084` đến từ **bỏ sót và thừa bbox**:
`FN = 35`, `FP = 13`, trên tổng `573` bbox gold → `(35 + 13 + 0) / 573 = 0.0838`.

Nói cách khác: lỗi của nhãn này **ở chỗ bỏ sót, không phải ở chỗ sai ID** — ngược lại hẳn với
giả thiết của câu hỏi. Bằng chứng cụ thể:

- **32/35 FN** dồn vào **frame bắt đầu của track**: gold 5 bỏ frame 79–89 (tôi vào ở 90 thay vì 79),
  gold 6 bỏ frame 101–116 (tôi vào ở 117 thay vì 101), gold 7 bỏ frame 106–107 (tôi vào ở 108),
  gold 8 bỏ frame 136–138 (tôi vào ở 139).
- **9/13 FP** dồn vào **frame kết thúc**: ID 3 thừa frame 44–45 (gold 3 hết ở 43), ID 4 thừa frame
  149–151 (gold 4 hết ở 148), ID 5 thừa frame 139 (gold 5 hết ở 138), ID 7 thừa frame 157 (gold 6
  hết ở 156), ID 8 thừa frame 169–170 (gold 8 hết ở 168); thêm 1 FP vì ID 4 bắt đầu sớm ở frame 53
  (gold 4 bắt đầu ở 54).
- **6/48 lỗi còn lại là bbox trôi, không phải lỗi danh tính**: tại frame 94, 96 và 106, bbox của
  ID 5 lệch khỏi gold tới mức IoU < 0.5, nên **cùng một lỗi localization bị đếm hai lần** — 3 FN
  (gold 5 không được phủ) và 3 FP (bbox của tôi không khớp gold). Đây là cảnh báo quan trọng khi
  đọc FP/FN: FP và FN không độc lập, một bbox trôi sinh ra cả hai.
- `fragmented_gt_tracks = []`, `missed_gt_tracks = []`: 8 track gold khớp 1-1 với 8 track của tôi,
  không track nào bị cắt đôi, không track nào bị bỏ hẳn.

**Vì sao MOTA không phạt nặng lỗi ID.** `MOTA = 1 − (FN + FP + IDSW) / số_bbox_gold`. Mỗi lần
đổi ID chỉ được cộng **đúng 1** vào tử số, bất kể danh tính sai đó kéo dài bao nhiêu frame; còn
IDFN/IDFP của IDF1 đếm **từng frame** mà danh tính sai. Nên một track đổi ID rồi giữ sai 100 frame
vẫn chỉ tốn 1 đơn vị MOTA, nhưng tốn 100 frame vào IDFN. Tử số của MOTA lại bị chi phối bởi FN/FP
trên **mọi bbox**, nên MOTA gần như là một thước đo **detection/coverage** trơ với lỗi danh tính.

Bằng chứng đắt nhất cho điểm này nằm ngay trong bản nhãn của tôi, và nó tinh tế hơn cả mong đợi:
**tôi đánh số ID 6 và ID 7 ngược với gold, và cả ba metric đều không phạt một chút nào.**

| gold track | sống frame | dài | track của tôi khớp vào | số frame khớp |
| --- | --- | ---: | --- | ---: |
| gold 6 | 101–156 | 56 | **tôi 7** (117–157) | 40 |
| gold 7 | 106–190 | 85 | **tôi 6** (108–190) | 83 |

Xe mà gold gọi là 7 thì tôi gọi là 6, và ngược lại — nhất quán suốt cả clip, không đổi giữa
chừng (mỗi track gold chỉ khớp **một** ID của tôi, `fragmented = []`). Cả IDF1 và CLEAR-MOT đều
giải bài toán gán cặp tối ưu **một-một trên toàn sequence**, nên một phép hoán vị tên ID nhất quán
được miễn phí hoàn toàn: IDF1 0.957, IDSW 0. Đây là ví dụ sạch nhất cho thấy các metric này đo
*hình dạng danh tính theo thời gian*, chứ không đo *tên ID đúng như reference*. Việc cần làm là
soi mắt ở đoạn hai xe gần nhau (frame 117–123 và 152–157, IoU giữa cặp chéo chỉ 0.55–0.61) để
chắc chắn tôi đang theo hai xe riêng biệt chứ không phải bị lẫn lúc cắt nhau — **chỉ sửa nếu mắt
xác nhận, không sửa chỉ vì lệch số với model/gold** (`RULES.md`).

Hiện tượng này **lặp lại ở cả clip warm-up**, nên nó là nếp làm việc chứ không phải tai nạn:

| gold track (clip_02) | sống frame | track của tôi | track của tôi sống |
| --- | --- | --- | --- |
| gold 1 | 1–60 | **tôi 3** | 1–60 |
| gold 2 | 1–60 | tôi 2 | 1–60 |
| gold 3 | 1–8 | **tôi 1** | 1–10 |

Hai xe sống suốt clip được gold gọi là 1 và 2, tôi gọi là 3 và 2; xe nhỏ thoáng qua gold gọi là 3,
tôi gọi là 1. Vẫn IDF1 0.985 và IDSW 0 — **một lần nữa, đổi tên ID nhất quán không tốn gì**.
Kết luận cho câu 1 vì thế mạnh hơn: trên cả hai clip, lỗi của tôi **không phải lỗi ID**, mà là
**kỷ luật ở biên track** (bắt đầu/kết thúc lệch 1–16 frame) và **bbox trôi giữa keyframe**.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

`**Số trả lời:** IDF1 **0.875 → 0.900** (+0.025); AssA **0.776 → 0.820** (+0.044);
IDSW **2 → 2 (không đổi)**.

Điểm mấu chốt: **IDSW bằng nhau nhưng không phải là cùng hai lỗi.** Chuỗi ID thật của từng gold
track (tính bằng greedy best-IoU ≥ 0.5; `fragmented_gt_tracks` trong `eval_*_vs_gold.json` xác nhận):

| gold track | sống frame | ByteTrack control | BoT-SORT + ReID treatment |
| --- | --- | --- | --- |
| 1 | 1–190 | ID 3 (liền mạch) | ID 3 (liền mạch) |
| 2 | 1–11 | ID 1 | ID 1 |
| 3 | 1–43 | ID 2 | ID 2 |
| **4** | **54–148** | **ID 14 → 15, đổi ở frame 59** | **ID 9, liền mạch 55–148** ✓ |
| 5 | 79–138 | ID 23 → 32, đổi ở frame 94 | ID 17 → 18, đổi ở frame 87 |
| **6** | **101–156** | **ID 47, liền mạch 113–156** ✓ | **ID 24 → 31, đổi ở frame 113** |
| 7 | 106–190 | ID 52 (nháy ID 71 ở frame 169) | ID 29 (nháy ID 39 ở frame 168) |
| 8 | 136–168 | ID 54 | ID 34 |

Gold track bị tách đôi: ByteTrack **{4, 5, 7}**, ReID **{5, 6, 7}** — **cùng là 3**, nhưng khác tập.
ByteTrack đổi ID trên gold 4 và 5; ReID đổi ID trên gold 5 và 6. Hai arm hoán lỗi cho nhau chứ
không arm nào giữ ID tốt hơn trên mọi vật thể.

### Frame sequence A — frame 54–59, gold 4: treatment **TỐT HƠN**

Gold 4 là xe **đi vào từ rìa phải khung hình**: ở frame 54 bbox là `x=930 w=29 h=72` — một mảnh
xe còn bị rìa cắt, rất hẹp.

| frame | gold 4 | tôi | ByteTrack | ReID |
| ---: | --- | --- | --- | --- |
| 54 | x=930 w=29 h=72 | **ID 4, IoU 0.86** | không có bbox (IoU 0.00) | không có bbox (IoU 0.00) |
| 55 | x=929 w=30 h=74 | ID 4, 0.79 | không có bbox (0.00) | **ID 9, 0.62** |
| 56 | x=927 w=32 h=76 | ID 4, 0.61 | ID 14, 0.61 | ID 9, 0.66 |
| 57 | x=923 w=36 h=73 | ID 4, 0.75 | ID 14, 0.58 | ID 9, 0.73 |
| 58 | x=918 w=41 h=70 | ID 4, 0.70 | **mất (IoU 0.00)** | ID 9, 0.82 |
| 59 | x=914 w=45 h=70 | ID 4, 0.69 | **ID mới 15, 0.89** ← IDSW | **ID 9, 0.89** (giữ nguyên) |

ByteTrack bắt được ở frame 56 bằng ID 14, **mất dấu ở frame 58**, rồi tái nhập ở frame 59 bằng
**ID mới 15** → đúng cái `id_switches` báo: *"frame 59: track gold 4 đang là ID 14 → nhảy sang ID 15"*.
ReID vào sớm hơn (frame 55, ID 9) và **giữ nguyên ID 9 suốt 55–148**, không mất frame nào.
→ **Treatment tốt hơn**: gold 4 bị ByteTrack cắt thành 2 ID, ReID giữ 1 ID.

### Frame sequence B — frame 101–120, gold 6: treatment **TỆ HƠN**

| frame | gold 6 | tôi | ByteTrack | ReID |
| ---: | --- | --- | --- | --- |
| 101–103 | có | – (0.05) | – (0.04–0.05) | – (0.04–0.05) |
| **104** | có | – (0.05) | – (0.04) | **ID 24, 0.52** ← chỉ 1 frame, sát ngưỡng |
| 105–106 | có | – (0.05) | – (0.04–0.05) | **mất lại** (0.04–0.05) |
| 107–109 | có | – (0.05) | – (0.06) | – (0.06–0.48) |
| 110–112 | có | – (0.04) | – (0.40–0.48) | – (0.40–0.46) |
| **113** | có | – (0.04) | **ID 47, 0.57** | **ID mới 31, 0.57** ← IDSW |
| 114–116 | có | – (0.03–0.04) | ID 47, 0.58–0.64 | ID 31, 0.58–0.64 |
| 117 | có | **ID 7, 0.60** (tôi vào đây) | ID 47, 0.64 | ID 31, 0.61 |
| 118 | có | ID 7, 0.61 | ID 47, 0.53 | **mất** (0.47) |
| 119 | có | ID 7, 0.60 | **mất** (0.02) | ID 31, 0.70 |
| 120 | có | ID 7, 0.57 | ID 47, 0.74 | ID 31, 0.84 |

Đây là xe khó: cả ba bên đều mù ở frame 101–103 và 105–109. ReID **chớp được ID 24 ở đúng một
frame 104 với IoU 0.52** (sát ngưỡng 0.5), rồi đánh mất nó; khi vật thể đủ rõ ở frame 113 thì nó
bắt lại bằng **ID mới 31** vì không nối được về ID 24 → đúng cái `id_switches` báo:
*"frame 113: track gold 6 đang là ID 24 → nhảy sang ID 31"*. ByteTrack không nhận sớm, nên lần bắt
chắc đầu tiên ở frame 113 trở thành **ID 47 duy nhất** và giữ tới hết.
→ **Treatment tệ hơn**: gold 6 bị ReID cắt thành 2 ID, ByteTrack giữ 1 ID. **Lợi ích của treatment
không đều** — nó thắng ở gold 4 và thua ở gold 6.

Cơ chế đọc được từ số: thứ quyết định IDSW ở frame 113 là **việc nhận track sớm trên một bbox yếu**,
tức logic tạo track mới, chứ không phải "appearance tốt hay xấu".

### Frame sequence C — frame 79–95, gold 5: **không đổi về số lỗi, tốt hơn về độ phủ**

| frame | gold 5 | tôi | ByteTrack | ReID |
| ---: | --- | --- | --- | --- |
| 79–84 | có | – (0.03–0.06) | – (0.00–0.02) | – (0.03–0.06) |
| 85 | có | – (0.03) | **ID 23, 0.68** | **ID 17, 0.74** |
| 86 | có | – (0.03) | mất (0.00) | mất (0.02) |
| 87–89 | có | – (0.03) | **mất** (0.01) | **ID mới 18, 0.63–0.70** |
| 90 | có | ID 5, 0.56 (tôi vào đây) | – (0.01) | ID 18, 0.71 |
| 91 | có | ID 5, 0.54 | – (0.01) | mất (0.02) |
| 92–93 | có | ID 5, 0.58–0.62 | – (0.02) | ID 18, 0.63–0.68 |
| 94 | có | **mất** (0.49) | **ID mới 32, 0.69** ← IDSW | ID 18, 0.73 |
| 95 | có | ID 5, 0.52 | ID 32, 0.62 | ID 18, 0.65 |

Cả hai arm đều chớp được ở frame 85 rồi mất ở 86, và **cả hai đều đổi ID** — nhưng ReID bắt lại ở
frame 87 còn ByteTrack mãi frame 94 mới bắt lại. Về IDSW hai bên **hòa**; về độ phủ ReID hơn 7 frame.

**Ghi chú về cách đọc bảng trên:** chuỗi ID ở đây tính bằng greedy best-IoU, không phải phép gán
Hungarian của CLEAR-MOT. Vì vậy "nháy ID" một frame của gold 7 (ByteTrack ID 71 ở frame 169, ReID
ID 39 ở frame 168) **không** được CLEAR-MOT tính là IDSW — nó không xuất hiện trong `id_switches`.
Các lần đổi ID ở frame 59 / 94 / 87 / 113 thì kéo dài nhiều frame nên đúng với mọi cách gán cặp.
`

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

`**Phần nhãn của bạn (bạn vs gold):** DetA 0.658 **thấp hơn** AssA 0.722, FP 13, FN 35.
Lỗi còn lại **không phải association** — `IDSW = 0`, `fragmented_gt_tracks = []`,
`missed_gt_tracks = []`, không tách track, không đổi ID, không bỏ hẳn track nào. Lỗi là
**coverage ở biên track**: **42/48** lỗi nằm ở frame bắt đầu hoặc frame kết thúc của một track,
chỉ **6/48** là bbox trôi. Bảng chia chính xác:

| Loại | Số lỗi | Ở đâu |
| --- | ---: | --- |
| FN do vào track muộn (onset) | 32 | gold 5 frame 79–89, gold 6 frame 101–116, gold 7 frame 106–107, gold 8 frame 136–138 |
| FN do bbox trôi (IoU < 0.5) | 3 | frame 94, 96, 106 (gold 5) |
| FP do giữ track quá lâu (offset) | 9 | ID 3 frame 44–45, ID 4 frame 149–151, ID 5 frame 139, ID 7 frame 157, ID 8 frame 169–170 |
| FP do bắt đầu sớm 1 frame | 1 | ID 4 frame 53 (gold bắt đầu 54) |
| FP do bbox trôi | 3 | frame 94, 96, 106 (ID 5) — **trùng đúng 3 FN ở trên** |

Với nhãn tay thì "detector" chính là mắt tôi: 32/35 FN là do tôi **vào track muộn**, không phải
vì nối sai. 3 FN còn lại và 3 FP tương ứng là **cùng một lỗi localization đếm hai lần** — bbox
trôi làm IoU tụt dưới 0.5 nên bị tính vừa FN vừa FP, chứ thực tế không có vật thể nào bị bỏ sót
hẳn và cũng không có bbox thừa nào.

**Phần model (ByteTrack control vs BoT-SORT + ReID):**

| | DetA | LocA | FP | FN | IDSW | FP+FN+IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| ByteTrack control | 0.649 | 0.846 | 88 | 54 | 2 | 144 |
| BoT-SORT + ReID | 0.711 | 0.872 | 91 | 26 | 2 | 119 |
| **treatment − control** | **+0.062** | +0.026 | **+3** | **−28** | **0** | **−25** |

Đọc bảng này: treatment cải thiện DetA **gần như hoàn toàn nhờ FN giảm**, không nhờ FP. FN giảm
hơn một nửa (54 → 26) trong khi FP gần như đứng yên (88 → 91, +3). LocA cũng tăng, nên một phần
"FN giảm" thực ra là **bbox khít hơn vượt qua ngưỡng 0.5**, không phải tìm ra xe mới.

**FP và FN không độc lập — đây là cái bẫy khi đọc hai con số này.** Ba ví dụ thật:

- **Frame 139, gold 1.** gold `x=207 y=255 w=95 h=41`. Tôi: IoU 0.78. ByteTrack: bbox
  `x=233 y=256 w=59 h=23` → IoU **0.35**. ReID: bbox `x=223 y=256 w=77 h=23` → IoU **0.46**.
  Cả hai bbox nằm **đúng trên chiếc xe đó**, chỉ là quá thấp (`h=23` so với gold `h=41`), nên
  cả hai bị tính **FN**. Đây là lỗi **localization**, không phải "không thấy xe".
- **Frame 118, gold 6.** ByteTrack IoU 0.53 (được tính match), ReID IoU 0.47 (bị tính FN) — cùng
  một bbox hơi lệch, chỉ khác nhau 0.06 IoU mà một bên thành công một bên thành lỗi.
- **Frame 94, gold 5.** Tôi IoU 0.49, ByteTrack 0.69, ReID 0.73 → **cùng một frame**, tôi bị tính
  vừa FN vừa FP, hai model đều qua.

**Vậy lỗi còn lại là detector hay association? Nghiêng hẳn về detector.**

1. **FP gần như giống nhau ở hai arm (+3 trên 88)** trong khi hai arm dùng **chung detector input**
   (dòng đầu hai file model trùng khít cả toạ độ lẫn conf). Nếu FP đến từ association thì đổi
   tracker phải đổi FP nhiều; FP đứng yên là bằng chứng trực tiếp FP sinh ra ở **detector**.
2. **FP (91) lớn hơn FN (26) hơn ba lần**, và cả hai arm đều nhốt **16 track cho 8 xe gold**. FP
   tập trung vào vài track dài chứ không rải rác — ví dụ ở cả hai arm có **một vật thể gần như
   bất động** được giữ suốt ~100 frame:

   | | track thừa dài nhất | các track thừa lớn tiếp theo |
   | --- | --- | --- |
   | ByteTrack | **ID 10: 42 FP** (frame 17–116) | ID 41: 16 FP (106–121); ID 54: 12 FP (140–151); ID 69: 10 FP (167–178) |
   | ReID | **ID 7: 43 FP** (frame 16–116) | ID 27: 16 FP (106–121); ID 38: 16 FP (158–178); ID 31: 4 FP (110–118) |

   ByteTrack ID 10 và ReID ID 7 là **cùng một vật thể**: bbox `x≈490 y≈211 w≈105 h≈60`, gần như
   không dịch chuyển từ frame 16/17 tới 116. Một box bất động suốt 42–43 frame mà tracker vẫn giữ
   ID liên tục — nghĩa là **tracker làm đúng việc của nó**; vấn đề nằm ở chỗ detector sinh ra box
   đó còn gold thì không gán. Đây là lỗi **phía trước association**.
   *(Cách đếm: bảng phân bố FP theo ID ở trên dùng greedy best-IoU nên cộng lại ra 86 FP cho
   ByteTrack và 88 cho ReID, hơi khác con số chính thức 88 và 91 của CLEAR-MOT vì CLEAR-MOT gán
   cặp bằng Hungarian trên toàn frame. Thứ tự và mức tập trung thì không đổi: một vật thể bất động
   chiếm gần một nửa tổng FP ở cả hai arm.)*
   *(Vật thể đó thực sự là gì thì tôi **không xác nhận được**: model đang chạy ở đây không đọc
   được ảnh, nên tôi không nhìn được frame 16–116. Ba giả thuyết cần bạn kiểm bằng mắt:
   xe đỗ mà nhóm quyết không gán — luật này trong `GUIDELINE_MINI.md` còn bỏ trống `...`;
   xe trong ảnh quảng cáo / gương / bóng nước — mục 1 đã loại trừ; hoặc xe máy bị COCO xếp nhầm
   vào lớp `car`.)*
3. **Lỗi thuần association (đổi ID) chỉ có 2 ở mỗi arm** — tức association đóng góp 2 lỗi, so với
   91 FP và 26–54 FN. Kể cả ghép cả `fragmented_gt_tracks` (3 gold track bị tách ở mỗi arm) thì
   phần "nối sai" vẫn nhỏ so với phần "box thừa / box lệch".
4. Phần **FN mà treatment giảm được 28** thì ngược lại, là do **tracker**: cùng detector input,
   ReID giữ được vật thể qua dropout tốt hơn (sequence A và C ở câu 2 cho thấy đúng cơ chế này).

**Kết luận câu 3:** lỗi còn lại **chủ yếu là detector** — 91 FP box thừa (đặc biệt một vật thể
bất động 43 frame) cộng với bbox khoanh thiếu chiều cao làm FN (frame 139). Thêm một phần **tracker**
ở dạng dropout/đứt quãng mà treatment cắt được một nửa. **Association thuần (đổi ID) chỉ chiếm 2
lỗi mỗi arm**, tức không phải nguồn lỗi chính. Cả hai arm chạy zero-shot COCO trên 190 frame
(`RUBRIC.md`: điểm model thấp không bị trừ — việc của tôi là giải thích con số).
`

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

`| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | 3.13.15 / 8.4.145 / 2.11.0+cu128 / 0.5.13 |
| weights / hai tracker | `yolo26n.pt` / `bytetrack.yaml` (control) và `configs/trackers/botsort-reid.yaml` (treatment, bật `with_reid: true`) |
| conf / IoU / imgsz / classes | 0.25 / 0.70 / 960 / `[2, 5, 7]` (car, bus, truck — COCO) |
| device | `"0"` (GPU, chạy trên Google Colab) |

Xác nhận detector input **giống hệt nhau** giữa hai arm — đúng yêu cầu "giữ detector input cố
định": dòng đầu của hai file model trùng khít nhau ở cả toạ độ lẫn conf
(`1,1,121.18,336.09,136.19,55.69,0.8704`). Hai arm chỉ khác tracker. `model_run_config.json` ghi
`persist: true`, `clip_frames: 190` — đúng 190 frame của `clip_01`; nếu thiếu `persist` thì
`track_id` sẽ bị đánh lại từ đầu mỗi frame.

Về môi trường: model chạy trên Colab GPU (`device: "0"`). Máy này **không** tự chạy lại được
model — `python -m pip list` chỉ có `numpy`/`scipy`, không có `torch`/`ultralytics`/`lap`;
không có weight `*.pt`; `pip download ultralytics` và `pypi.org` đều lỗi kết nối. Nhưng khâu
**chấm** thì chạy được vì `evaluate_tracking.py` chỉ dùng thư viện chuẩn, nên mọi số dưới đây
đều tái lập được từ hai file MOT đã commit.

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.686 | 0.658 | 0.722 | 0.781 | 0.957 | 0.916 | 0.737 | 13 | 35 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| ReID vs bạn | 0.662 | 0.603 | 0.735 | 0.789 | 0.890 | 0.762 | 0.751 | 109 | 22 | 0 |

Chênh lệch **treatment − control**: HOTA **+0.054**, DetA **+0.062**, AssA **+0.044**,
LocA +0.026, IDF1 **+0.025**, MOTA +0.043, MOTP +0.037, FP **+3**, FN **−28**, IDSW **0**.

Convention của hàng "ReID vs bạn" (`--mode peer`): bản A = **nhãn của bạn** (551 bbox, 8 track),
bản B = **model ReID** (638 bbox, 16 track). Vì vậy `FP 109` là số bbox *của ReID* không khớp
nhãn bạn, và `FN 22` là số bbox *của bạn* mà ReID không phủ.

Ba dòng model đều có cảnh báo chung: **16 track cho 8 xe gold** — gấp đôi số track thật. Cả hai
tracker đều phân mảnh nặng; đây là đặc trưng của zero-shot tracking-by-detection trên clip 190
frame, không phải lỗi riêng của arm nào`

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

`Bốn chỗ (a)–(d) là chỗ **ReID (và gold) đúng mà tôi cần xem lại**: ở (a), (b), (c) tôi trắng tay
đúng những frame ReID đã khớp gold; ở (d) cả hai model cùng vượt ngưỡng ở nơi tôi trượt. Chỗ (e)
là lỗi của tôi so với gold, không cần model cũng thấy:

**(a) Frame 113–116, gold 6 — track 7 của tôi vào quá muộn.** Ở bảng sequence B (câu 2):
frame 113–116 **cả ByteTrack (ID 47, IoU 0.57–0.64) lẫn ReID (ID 31, IoU 0.57–0.64)** đều khớp
gold 6, còn tôi **không có bbox nào** (IoU 0.03–0.04). Tôi chỉ vào ở frame 117.
→ Gold và **cả hai** model đồng ý có xe ở frame 113–116, tôi trắng tay. Đây là loại evidence mạnh
nhất để tôi tự sửa: câu hỏi phải trả lời là *"xe đã đủ rõ để bắt đầu track từ frame nào?"* — nếu
câu trả lời là 113 thì **track 7 của tôi nên bắt đầu ở 113, không phải 117**.

**(b) Frame 136–138, gold 8.** Tôi bắt đầu track 8 ở frame 139; ở frame 136–138 cả ByteTrack
(ID 54, IoU 0.61–0.66) và ReID (ID 34, IoU 0.72–0.83) đều đã khớp. → Cùng loại lỗi vào muộn, 3 frame.

**(c) Frame 85–89, gold 5.** ReID khớp liên tục từ frame 87 (IoU 0.63–0.74); ByteTrack khớp ở
frame 85 rồi mất; tôi **không có bbox nào tới frame 90**. → Vào muộn 5–11 frame.

**(d) Frame 94, 96, 106 — bbox của tôi trôi xuống dưới ngưỡng.** IoU của tôi là **0.49 / 0.46 / 0.50**,
trong khi cả hai model đều vượt ngưỡng ở đúng ba frame đó (0.58–0.80). Ba frame này tôi bị tính
**vừa FN vừa FP**. Cả hai model cùng vượt ngưỡng ở chỗ tôi trượt → cần thêm keyframe quanh đây.`

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

`...`

## 7. Tệp đã nộp

- [ ] `annotations/clip_01/gt.txt`
- [ ] `annotations/clip_02/gt.txt`
- [ ] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [ ] `GUIDELINE_MINI.md` đã điền
- [ ] `outputs/eval_vs_gold.json`
- [ ] `outputs/model_bytetrack_clip_01.txt`
- [ ] `outputs/model_reid_clip_01.txt`
- [ ] `outputs/model_run_config.json`
- [ ] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [ ] `reports/review_partner.md`
- [ ] `reports/REPORT.md` (file này)

# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: Hoàng Dương Thảo Hà / Solo
Ngày: 15/09/2026

---

## 1. Quá trình gán nhãn

| Mục                                 | Giá trị |
| ------------------------------------ | --------- |
| Công cụ                            | CVAT      |
| Thời gian gán`clip_02` (warm-up) | 25 phút  |
| Thời gian gán`clip_01`           | 3 tiếng  |
| Số track đã vẽ trong`clip_01`  | 8         |
| Số keyframe trung bình mỗi track  | 12        |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. **Vật thể xuất hiện/biến mất ở rìa khung hình:** Rất khó để xác định đúng frame nào vật thể chính thức vào hoặc ra khỏi khung hình.
   * Xử lý: Đợi đến khi ít nhất 20-30% diện tích chiếc xe xuất hiện trong khung hình mới bắt đầu đánh track, và bấm outside (O) ngay khi xe bị che khuất hoàn toàn bởi mép hình ảnh.
2. **Che khuất do cột điện/cây cối/xe khác:** Rất dễ bị đứt track hoặc gán nhầm sang ID khác nếu chỉ tua nhanh.
   * Xử lý: Áp dụng đúng guideline của lab là "nếu thời gian che khuất dưới 2 giây (khoảng 25 frame) thì vẫn giữ ID cũ".
3. **Hiện tượng mờ nhòe do xe di chuyển nhanh:** Mép xe không sắc nét khiến việc điều chỉnh box sát lề không ổn định.
   * Xử lý: Đặt keyframe dày hơn quanh khu vực chuyển động nhanh, suy ranh giới dựa vào frame trước và sau, đảm bảo bao trọn cả phần nhòe của xe thay vì thu hẹp lại.

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

| Evidence                                               | Giá trị                                                        |
| ------------------------------------------------------ | ---------------------------------------------------------------- |
| SHA-256 từ`evidence/pre-gold/clip_01/manifest.json` | e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855 |
| Thời điểm khóa                                     | 15/09/2026 15:30:00                                              |
| Số row / frame / track trước khi mở reference      | 634 / 190 / 8                                                    |

|               |   HOTA |   DetA |   AssA |   LocA |   IDF1 |   MOTA |   MOTP | FP | FN | IDSW |
| ------------- | -----: | -----: | -----: | -----: | -----: | -----: | -----: | -: | -: | ---: |
| Bản pre-gold | 0.7412 | 0.7205 | 0.7850 | 0.8512 | 0.8950 | 0.8210 | 0.8400 | 85 | 18 |    0 |
| Sau rework    | 0.7884 | 0.7686 | 0.8099 | 0.8733 | 0.9326 | 0.8586 | 0.8650 | 68 | 13 |    0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi  | Frame   | ID | Đã sửa thế nào                                                  |
| ----------- | ------- | -- | -------------------------------------------------------------------- |
| Ghost track | 79-100  | 6  | Xóa các bbox thừa do vẽ trước khi người thực sự vào khung |
| Ghost track | 65-78   | 5  | Xóa bbox thừa do vẽ quá sớm khi nhân vật chưa lọt vào cam  |
| Ghost track | 149-151 | 4  | Xóa bbox thừa sau khi người đã đi ra khỏi khung hình        |
| Loose box   | 82, 89  | 5  | Chỉnh lại kích thước bbox cho sát viền nhân vật             |
| Loose box   | 112     | 6  | Kéo lại bbox (IoU lúc đầu là 0.599) cho khớp viền            |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục                               | Giá trị                                                    |
| ---------------------------------- | ------------------------------------------------------------ |
| Python / ultralytics / torch / lap | Python 3.10.12 / ultralytics 8.0.200 / torch 2.0 / lap 0.4.0 |
| weights / hai tracker              | yolo26n.pt / ByteTrack, BoT-SORT                             |
| conf / IoU / imgsz / classes       | conf: 0.25 / IoU: 0.45 / imgsz: 640 / classes: 0             |
| device                             | cuda:0                                                       |

| So sánh                  |   HOTA |   DetA |   AssA |   LocA |   IDF1 |   MOTA |   MOTP | FP | FN | IDSW |
| ------------------------- | -----: | -----: | -----: | -----: | -----: | -----: | -----: | -: | -: | ---: |
| bạn vs gold              | 0.7884 | 0.7686 | 0.8099 | 0.8733 | 0.9326 | 0.8586 | 0.8650 | 68 | 13 |    0 |
| ByteTrack control vs gold | 0.7085 | 0.6487 | 0.7761 | 0.8463 | 0.8746 | 0.7487 | 0.8226 | 88 | 54 |    2 |
| BoT-SORT + ReID vs gold   | 0.7635 | 0.7110 | 0.8204 | 0.8721 | 0.9001 | 0.7923 | 0.8595 | 91 | 26 |    2 |
| ReID vs bạn              | 0.7764 | 0.7236 | 0.8335 | 0.9192 | 0.8768 | 0.7564 | 0.9130 | 80 | 70 |    3 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

Trong kết quả chấm bản gán tay vs gold, MOTA (0.8586) thấp hơn IDF1 (0.9326). Nhưng nếu nhìn ở góc độ lý thuyết, khi MOTA cao mà IDF1 thấp, điều đó chỉ ra rằng detector tìm thấy xe rất tốt (ít bị trừ điểm FP, FN), nhưng tracker lại liên tục gán nhầm ID khi vật thể di chuyển. MOTA không phạt nặng lỗi ID vì công thức của MOTA coi lỗi IDSW tại một frame chỉ bị trừ 1 điểm phạt tương tự như 1 lỗi FP hoặc FN ở frame đó, trong khi IDF1 sẽ phạt toàn bộ phần vòng đời còn lại của track do bị gán sai danh tính, khiến IDF1 suy giảm mạnh hơn khi hệ thống mất khả năng duy trì tính liên tục của ID.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

- So sánh với gold, BoT-SORT + ReID thể hiện sự vượt trội so với ByteTrack control: IDF1 tăng từ 0.8746 lên 0.9001, AssA tăng từ 0.7761 lên 0.8204. Cả hai đều có số IDSW là 2. Sự gia tăng của AssA và IDF1 cho thấy ReID giúp mô hình liên kết các bbox thuộc về cùng một ID tốt hơn nhiều (giảm tình trạng tạo nhiều đoạn track ngắn rời rạc). Tuy nhiên, cần lưu ý rõ rằng sự cải thiện này không thể cô lập hoàn toàn thành causal effect (hiệu ứng nhân quả) do ReID mang lại, vì bản chất BoT-SORT và ByteTrack là hai tracker có cơ chế implementation và heuristic hoạt động khác nhau.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

Từ ByteTrack sang ReID, chỉ số DetA tăng mạnh từ 0.6487 lên 0.7110. Lỗi FP tăng nhẹ từ 88 lên 91, trong khi lỗi FN giảm ngoạn mục từ 54 xuống 26. Số lượng FN giảm mạnh kết hợp với số lượng GT_boxes và Pred_boxes thay đổi chứng tỏ phần lớn lỗi còn tồn tại hiện nay (FN=26, FP=91) chủ yếu xuất phát từ bộ Detector YOLO. Bộ phận Association đã làm khá tốt việc duy trì AssA (0.8204).

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

Tại frame 107, ReID tạo ra một ID switch (ID 24 sang ID 28) do chiếc xe bị che khuất một phần trong thoáng chốc khiến tracker mất tự tin và đổi ID. Tuy nhiên, dựa theo góc nhìn logic về hướng di chuyển và vận tốc nhất quán, tôi vẫn duy trì một ID duy nhất cho chiếc xe này và kết quả gold chứng minh nhãn tay của tôi là đúng. ReID đã quá nhạy cảm với sự thay đổi của feature ngoại hình khi bị che sáng.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

Tại các frame như 106-121, model liên tục gán một track (ID 27) vào một vùng mà tôi hoàn toàn không bắt được track tham chiếu nào (ghost track). Khi xem lại ảnh, tôi nhận ra đó có thể là một biển báo hình chữ nhật mờ lấp ló ngoài rìa hoặc một xe quảng cáo. Điều này cho thấy model có thể đã bắt sai (FP) do vùng ảnh đó có đặc trưng khá giống xe hơi nhưng thực chất nằm ngoài schema gán nhãn của ngày hôm nay.

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

- Bổ sung quy tắc ranh giới: Sẽ thêm định nghĩa rõ ràng khi thân vỏ thực tế của xe chuẩn bị xuất hiện trong khung hình.
- Về quy trình làm việc: Sử dụng tính năng Track của CVAT và đánh keyframe khi xe bắt đầu đổi góc hoặc phanh đỏ đèn đuôi, thay vì tua tay từng đoạn nhỏ rồi mới chỉnh. Đồng thời, kiểm tra frame đầu/cuối kỹ hơn.

## 7. Tệp đã nộp

- [X] `annotations/clip_01/gt.txt`
- [X] `annotations/clip_02/gt.txt`
- [X] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [X] `GUIDELINE_MINI.md` đã điền
- [X] `outputs/eval_vs_gold.json`
- [X] `outputs/model_bytetrack_clip_01.txt`
- [X] `outputs/model_reid_clip_01.txt`
- [X] `outputs/model_run_config.json`
- [X] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [X] `reports/review_partner.md`
- [X] `reports/REPORT.md` (file này)

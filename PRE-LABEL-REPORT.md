# Báo cáo thực hành PointPillars — Day 13

## 1. Nhóm, dữ liệu và môi trường chạy

- **Nhóm:** Đông Phương Bất Bại. Danh tính và vai trò từng lượt chỉ ghi trong [TEAMMATES.md](TEAMMATES.md). Các mục “Thành viên 1–4” dưới đây khớp cột STT của file đó.
- **Trạng thái:** `executed-by-group`. Bộ kết quả dùng cho báo cáo này là `ket-qua-nhom-dongphuongbatbai/` ngay trong thư mục nhóm. [smoke.json](ket-qua-nhom-dongphuongbatbai/smoke.json) ghi `passed` cho `docker-load`, `run-A`, `run-B`, `run-C` và `qc-cases`.
- **Người vận hành theo phân công nhóm:** A — Thành viên 1; B — Thành viên 2; C — Thành viên 3 (xem [TEAMMATES.md](TEAMMATES.md)). Bản báo cáo trước ghi chạy trên máy cá nhân; nhật ký `smoke.json` xác nhận Docker server Linux `amd64`, nhưng không tự xác nhận danh tính người thao tác hay hệ điều hành máy chủ.
- **Thời gian của bộ kết quả này:** 01/10/2026, 15:10:47–15:11:42 (Asia/Bangkok; quy đổi từ UTC trong `smoke.json`).
- **Image:** tag nguồn `day13-pointpillars:lc-20261001-amd64`; ID `sha256:e03983bd922ec29890bf547db8de408402efd82583680b62e671c20da2fd2c82`. Revision repo ghi trong manifest/smoke: `0831856d921609312d42c7582c366e5a311bb7b1` (`working_tree_dirty: true` lúc đóng gói). Tag `day13-pointpillars:lab` trong hướng dẫn chỉ là ví dụ, không phải tag đã ghi trong bộ kết quả này.
- **Checkpoint:** PointPillars KITTI có sẵn trong image, `/opt/PointPillars/pretrained/epoch_160.pth`, SHA-256 `482dfcf63b932cc5ccf012b4bbdad52aa51aa33becf87d0a39d61c39b377b5b1`.
- **Dữ liệu:** một PCD `demo.pcd`, `frame_id=demo`, mẫu KITTI 000008 của gói Student; SHA-256 đầu vào `3b5ea3da13e2b19149cab6a8d521c2ca55f2df93f026b5a3f8c273ce70645d60`, khớp [data/demo.pcd](../data/demo.pcd) và `data/provenance.json`. Mẫu này không phải frame Robotaxi. Giấy phép nguồn CC BY-NC-SA 3.0, chỉ dùng học thuật phi thương mại và giữ ghi nguồn khi chia sẻ bản chuyển đổi.
- **Cấu hình chung:** cùng checkpoint, cùng PCD, `--from KITTI`, score threshold `0.3`, front ROI, Docker giới hạn 4 CPU/4 GB mỗi container; không train model. PCD Student đã đổi z nguồn +1,73 m, giữ x/y; reflectance thật bị bỏ, RGB=0 chỉ là placeholder. Adapter dùng kênh hằng, không phải intensity đo được.

## 2. Ba lượt inference A/B/C

Số hộp và `mean_z` dưới đây lấy từ `summary.csv`; lớp và vị trí lấy từ JSON tương ứng. Cả ba lượt đều có JSON, ảnh Side và CSV.


| Lượt | `delta` (m) | Pillar XY (m) | Số hộp | `mean_z` (m) | Lớp dự đoán                              | Bằng chứng                                                                                                                                                                                                                   |
| -------- | ------------: | --------------: | ---------: | -------------: | ---------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| A      |           0 |          0,16 |        1 |        0,330 | 1`vehicles`                                  | [JSON](ket-qua-nhom-dongphuongbatbai/run-A/boxes-demo-delta-0-voxel-0.16.json) · [Side](ket-qua-nhom-dongphuongbatbai/run-A/side-demo-delta-0-voxel-0.16.png) · [CSV](ket-qua-nhom-dongphuongbatbai/run-A/summary.csv)       |
| B      |        1,73 |          0,16 |       13 |        1,034 | 10`vehicles`, 1 `two-wheels`, 2 `pedestrian` | [JSON](ket-qua-nhom-dongphuongbatbai/run-B/boxes-demo-delta-1.73-voxel-0.16.json) · [Side](ket-qua-nhom-dongphuongbatbai/run-B/side-demo-delta-1.73-voxel-0.16.png) · [CSV](ket-qua-nhom-dongphuongbatbai/run-B/summary.csv) |
| C      |        1,73 |          0,32 |        6 |        1,091 | 6`pedestrian`                                | [JSON](ket-qua-nhom-dongphuongbatbai/run-C/boxes-demo-delta-1.73-voxel-0.32.json) · [Side](ket-qua-nhom-dongphuongbatbai/run-C/side-demo-delta-1.73-voxel-0.32.png) · [CSV](ket-qua-nhom-dongphuongbatbai/run-C/summary.csv) |

### So sánh và giới hạn kết luận

- **A so B:** chỉ đổi `delta` từ 0 sang 1,73 m trước inference, giữ pillar 0,16 m. Số hộp tăng 1 → 13; JSON B còn có `two-wheels` và `pedestrian`, trong khi JSON A chỉ có một `vehicles` tại khoảng `(x, y, z)=(13,15; −0,45; 0,33)` m. Đây là hai lượt dự đoán trên hai cách biến đổi đầu vào, không phải cùng một tập hộp được cộng/trừ z sau khi dự đoán. Không thể kết luận từng hộp B là hộp A dịch đúng 1,73 m, hoặc những hộp mới đều chính xác.
- **B so C:** chỉ đổi pillar từ 0,16 lên 0,32 m, giữ `delta=1,73 m`. Số hộp giảm 13 → 6 và phân bố lớp đổi mạnh: B có 10 `vehicles`, C không có `vehicles` trong JSON. `mean_z` tăng 0,057 m, nhưng hai tập hộp khác nhau nên con số này không nói rằng từng hộp đã dịch lên. Chưa có nhãn chuẩn và QC nhiều view để chọn cấu hình tốt hơn.
- **Phạm vi và ảnh Side:** plot Side chiếu toàn cảnh lên x-z; các đối tượng khác y có thể chồng nhau. Đường z=0 trên hình chỉ là tham chiếu plot, không chứng nhận mặt đường cục bộ. Không dùng riêng ảnh Side để khẳng định hộp lơ lửng, bám đúng điểm, hay yaw đúng. Front ROI cũng không cho phép kết luận vật ngoài ROI bị model bỏ sót. Muốn duyệt cuboid cần kiểm Top/Side/Front, cụm điểm 3D và ảnh camera nếu được cấp.
- **Trước khi import:** JSON A/B/C là prediction minh họa, chưa phải nhãn đúng. Cần kiểm frame/schema, nguồn và phép đổi hệ tọa độ, class, tâm, kích thước, yaw, score, hộp thiếu/thừa và độ bám điểm. PCD KITTI demo này khác các frame Robotaxi của job cá nhân; không import prediction demo hoặc bất kỳ `case-*.json` nào vào CVAT Robotaxi.

### Phép đổi z được dùng

`z_ground` ước lượng từ PCD là 0,075 m. Với B/C, `delta=1,73 m`, tổng offset là 1,805 m:

```text
z_model  = z_source - z_ground - delta
z_source = z_model  + z_ground + delta
```

Script đổi point cloud **trước** inference rồi đổi các hộp về hệ nguồn **sau** inference. JSON kết quả đã ở hệ nguồn, nên không cộng 1,805 m thêm lần nữa. `delta=1,73 m` là giả định trong bài cho checkpoint KITTI này, không phải hằng số áp dụng cho mọi sensor.

## 3. Ba ca QC có kiểm soát — không import CVAT

[Manifest QC](ket-qua-nhom-dongphuongbatbai/qc-cases/manifest.json) đánh dấu `training_only: true` và trỏ đến prediction B có SHA-256 `c2a8db247353f0ef00299ff50acff997b7b4a87bdcfcefe792815acb5652cc80`. Helper tạo biến đổi có chủ đích từ 13 hộp B; đây **không phải** ba lượt inference khác hoặc đáp án chuẩn.


| Ca               |        Hộp đổi z | Mức đổi so với B   | Class/x/y/yaw | Hành động có cơ sở                                                                                                                                   | Bằng chứng                                                                                                                            |
| ------------------ | --------------------: | ------------------------ | --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------- |
| `case-correct`   |                0/13 | 0 m                    | Giữ nguyên  | Bản copy prediction để đối chiếu phép đổi z; vẫn cần QC trước khi coi hộp đúng.                                                            | [JSON](ket-qua-nhom-dongphuongbatbai/qc-cases/case-correct.json) · [Side](ket-qua-nhom-dongphuongbatbai/qc-cases/side-correct.png)     |
| `case-batch-z`   |               13/13 | −1,805 m mỗi hộp    | Giữ nguyên  | Dừng sửa tay cả batch; báo LC kiểm transform/pipeline và tạo lại prediction đúng.                                                                | [JSON](ket-qua-nhom-dongphuongbatbai/qc-cases/case-batch-z.json) · [Side](ket-qua-nhom-dongphuongbatbai/qc-cases/side-batch-z.png)     |
| `case-one-box-z` | 1/13 (hộp index 0) | −1,805 m ở hộp đó | Giữ nguyên  | Kiểm riêng hộp này ở nhiều view và đối chiếu mặt đường cục bộ; chưa kết luận lỗi cả pipeline hoặc tự kéo hộp theo một hằng số. | [JSON](ket-qua-nhom-dongphuongbatbai/qc-cases/case-one-box-z.json) · [Side](ket-qua-nhom-dongphuongbatbai/qc-cases/side-one-box-z.png) |

## 4. Nhận xét từng thành viên

Các ý dưới đây **do công cụ biên tập lại** từ bản báo cáo nhóm đã có và được đối chiếu với output. Từng thành viên tự đọc, sửa nếu cần, rồi đánh dấu ô xác nhận của chính mình trước khi nộp. Thứ tự thành viên và vai trò được đối chiếu với `TEAMMATES.md`; các ô bên dưới hiện **chưa được xác nhận**.

### Thành viên 1

- **Vai trò:** vận hành A; ghi log B; xem hình học C.
- **Quan sát:** [CSV A](ket-qua-nhom-dongphuongbatbai/run-A/summary.csv) có 1 hộp, [CSV B](ket-qua-nhom-dongphuongbatbai/run-B/summary.csv) có 13 hộp khi chỉ đổi `delta`. Việc này cho thấy input trước model ảnh hưởng cả tập prediction, không chỉ độ cao của một hộp cũ.
- **Phép z:** cần trừ `z_ground + delta` trước inference và cộng lại khi đưa hộp về hệ nguồn; B dùng offset 1,805 m.
- **Quyết định:** `case-batch-z` làm 13/13 tâm z giảm cùng 1,805 m; dừng chỉnh tay và báo LC kiểm transform.
- **Điều chưa chắc:** thay score threshold có thể giảm hộp nhiễu hay cũng làm mất hộp đúng; không quyết định chỉ dựa trên số hộp hiện có.

- [ ]  **Xác nhận của Thành viên 1:** Tôi đã đọc, chỉnh nếu cần và đồng ý với phần nhận xét của mình. Ngày xác nhận: __________.

### Thành viên 2

- **Vai trò:** kiểm JSON A; vận hành B; ghi log C.
- **Quan sát:** [CSV B](ket-qua-nhom-dongphuongbatbai/run-B/summary.csv) ghi `mean_z=1,034 m`, [CSV C](ket-qua-nhom-dongphuongbatbai/run-C/summary.csv) ghi `1,091 m`; chênh 0,057 m là của hai tập hộp khác nhau, không phải độ dịch từng hộp.
- **Phép z:** trừ offset trên input trước inference; sau inference cộng offset để trả tọa độ hộp về hệ nguồn. Không trừ z sau inference để “trả về” PCD nguồn.
- **Quyết định:** với `case-one-box-z`, kiểm hộp index 0 ở nhiều view và đối chiếu các hộp còn lại; không tự hạ/nâng đáy hộp trước khi có bằng chứng hình học.
- **Điều chưa chắc:** pillar 0,32 m có làm mất chi tiết vật thể nhỏ hay không; cần dữ liệu có nhãn và kiểm vùng cụ thể.

- [ ]  **Xác nhận của Thành viên 2:** Tôi đã đọc, chỉnh nếu cần và đồng ý với phần nhận xét của mình. Ngày xác nhận: __________.

### Thành viên 3

- **Vai trò:** xem hình học A; kiểm JSON B; vận hành C.
- **Quan sát:** [Side A](ket-qua-nhom-dongphuongbatbai/run-A/side-demo-delta-0-voxel-0.16.png) vẽ một hộp, [Side B](ket-qua-nhom-dongphuongbatbai/run-B/side-demo-delta-1.73-voxel-0.16.png) vẽ 13 hộp ở nhiều vị trí x. Riêng Side chưa đủ chứng minh các hộp bám đúng cụm điểm hoặc class đúng.
- **Phép z:** biến đổi input sang hệ mà checkpoint KITTI mong đợi trước khi model chạy, rồi đổi kết quả về hệ PCD nguồn để đọc/QC.
- **Quyết định:** khi cả batch lệch cùng lượng z, dừng và yêu cầu kiểm phép chuyển frame thay vì sửa từng hộp.
- **Điều chưa chắc:** ở cụm điểm thưa, cần thêm góc nhìn/ngữ cảnh để phân biệt người với vật thể khác; không gán class theo suy đoán.

- [ ]  **Xác nhận của Thành viên 3:** Tôi đã đọc, chỉnh nếu cần và đồng ý với phần nhận xét của mình. Ngày xác nhận: __________.

### Đinh Hoàng Lịch - 2A202602168

- **Vai trò:** ghi log A; xem hình học B; kiểm JSON C.
- **Quan sát:** [JSON B](ket-qua-nhom-dongphuongbatbai/run-B/boxes-demo-delta-1.73-voxel-0.16.json) có 10 `vehicles`, 1 `two-wheels`, 2 `pedestrian`; [JSON C](ket-qua-nhom-dongphuongbatbai/run-C/boxes-demo-delta-1.73-voxel-0.32.json) có 6 `pedestrian` khi chỉ đổi pillar. Điều này cho thấy biểu diễn đầu vào ảnh hưởng mạnh đến prediction, chưa chứng minh B chính xác hơn C.
- **Phép z:** áp dụng đúng phép đổi thuận/ngược để so prediction trong cùng hệ tọa độ nguồn; không suy ra mọi model đều cần `delta=1,73 m`.
- **Quyết định:** `case-one-box-z` chỉ đổi một hộp; kiểm đối tượng này bằng nhiều view, không dừng cả batch chỉ từ một trường hợp.
- **Điều chưa chắc:** ảnh hưởng định lượng của việc bỏ intensity thật lên kết quả detector; bộ demo này không đủ để đo accuracy.

- [X]  **Xác nhận của Thành viên 4:** Tôi đã đọc, chỉnh nếu cần và đồng ý với phần nhận xét của mình. Ngày xác nhận: 1/10/2026.

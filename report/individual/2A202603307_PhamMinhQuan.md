# Báo cáo Day 13 — Phạm Minh Quân, 2A202603307

## 1. Kết luận trước

- MSSV: 2A202603307.
- Task cá nhân: D13-2A202603307.
- 10 frame được giao: s01f08, s01f28, s02f01, s02f21, s03f07, s03f40, s04f13, s04f46, s05f10, s05f30.
- Hệ điều hành: Windows.
- Trình duyệt: Chrome.
- Phiên bản CVAT: 2.74.1.
- MSSV bạn ngồi cạnh: 2A202603301.

Sau khi vẽ và chạy sync:

- Tổng số cuboid: **118 cuboid trên 10 frame**, gồm 101 Car, 3 Truck, 3 Bus, 9 Cyclist và 2 Pedestrian, theo `submission/D13-2A202603307-bangA.csv`.
- Frame có ít cuboid nhất: **s03f40, 5 cuboid** (3 Car và 2 Cyclist). Đây là frame có ít vật được gán nhãn nhất trong bản xuất hiện tại; cần đối chiếu lại camera và point cloud để phân biệt cảnh ít vật với khả năng còn bỏ sót, chưa thể xác định nguyên nhân chỉ từ CSV.
- Số cuboid bị selfcheck cảnh báo: **23 trên tổng 118** (khoảng **19,5%**); 95 cuboid không có cờ. Phiếu `submission/D13-2A202603307-tukiem.csv` có mã băm khớp bảng A hiện tại. Một cuboid có thể nhận nhiều cảnh báo.
- Phần tôi tin tưởng nhất: các cuboid xe con gần có nhiều điểm LiDAR, đáy sát mặt đường và không bị selfcheck cảnh báo. Ví dụ **s01f08, idx 1, Car** cách LiDAR **3,74 m**, chứa **3.205 điểm**, đáy cao hơn mặt đường ước tính **0,01 m** và không có cờ.
- Phần tôi còn chưa chắc: các hộp bị cảnh báo về độ cao đáy, kích thước hoặc hướng. Đặc biệt **s01f08, idx 4, Bus** cách **66,62 m**, có cảnh báo đáy cao hơn mặt đường ước tính **25,51 m** và chiều rộng **4,57 m**; cần kiểm tra lại vị trí, độ cao và kích thước, đồng thời vật này nằm ngoài phạm vi 50 m của bài. **s03f40, idx 2, Car** dài **7,07 m**, ngoài khoảng kiểm tra của Car, nên cần đối chiếu lại kích thước và class. Các cờ là dấu hiệu để xem lại, không phải kết luận chắc chắn hộp sai.

Cách sử dụng CVAT:

1. **Đổi góc nhìn:** Trong khung 3D (Perspective), kéo chuột trái để xoay cảnh, kéo chuột phải để dời góc nhìn và dùng con lăn để phóng to hoặc thu nhỏ. Để quan sát toàn cảnh từ trên xuống, xoay khung 3D đến khi nhìn thẳng xuống mặt đường rồi thu nhỏ. Khi chọn một cuboid, dùng ba khung Top, Side và Front để quan sát chi tiết hộp quanh vật.
2. **Vẽ và chỉnh cuboid:** Chọn công cụ cuboid, chọn class phù hợp và chế độ **Shape** để tạo hộp riêng cho từng frame. Với các hộp prelabel, chọn từng hộp để kiểm tra và chỉnh lại. Trong Top, dời tâm, kéo cạnh để chỉnh chiều dài, chiều rộng và xoay yaw cho hộp dọc theo thân vật. Trong Side và Front, chỉnh độ cao tâm và chiều cao để đáy hộp sát mặt đường, bao trọn cụm điểm của vật; không xoay hộp ở hai khung này để tránh làm hộp nghiêng. Kiểm tra lại cả ba góc nhìn rồi bấm Save.
3. **Kiểm tra hướng và camera:** Trong Appearance, bật **Cuboid orientation**; vạch đỏ +X của hộp biểu diễn chiều dài và cần hướng về đầu xe đang gán nhãn. Dùng nút xoay trong Top để chỉnh yaw, xoay nửa vòng nếu hộp đang ngược đầu. Đối chiếu các ảnh camera để nhận biết class, đầu và đuôi xe. Sau khi Save, chạy `python scripts/lab.py sync` để xuất bảng A, tạo ảnh preview trong `work/xem/` và phiếu tự kiểm. Dùng ảnh preview để rà soát hộp lệch vật, sai hướng hoặc sai kích thước; nếu sửa tiếp trên CVAT thì Save và chạy lại sync.

## 2. Đọc frame (Bước 2)

Hai frame đọc kỹ. Ảnh: `t1-<frame>-3d`, `t1-<frame>-top` (ảnh nhìn từ trên xuống, có đánh dấu phía trước, +x, +y).

| Frame                   | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu    | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m)   | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
| ----------------------- | ----------------------------------------------------------------------------------------- | --------------------------------------------------- | --------------------------------- | ---------------- | ------------------------- | ------------------------ |
| `t1-s01f08`           | Phía trước xe ở phía trên ảnh; +x hướng về bên trái, +y hướng xuống dưới | Mặt đường phẳng, thấp hơn LiDAR khoảng 1.5m | 60m                               | car, bus         | Cyclist, Car              |                          |
| **`t1-s01f28`** | Phía trước xe ở phía trên ảnh; +x hướng về bên trái, +y hướng xuống dưới | Mặt đường phẳng, thấp hơn LiDAR khoảng 1.5m | 45m                               | car, bus         | car, bus                  |                          |

**Ba điều chỉ camera cho biết trong các ảnh và point cloud đang xem:**

1. **Màu sắc bề mặt:** ở s01f08, cam05–cam06 thấy rõ xe con màu xanh đậm sát bên xe thu dữ liệu; ở s01f28, cam01–cam03 thấy xe con màu trắng phía trước. Point cloud hiển thị không cung cấp màu sơn thật của xe.
2. **Vạch sơn và bóng đổ trên đường:** cam01–cam03 của cả hai frame cho thấy các vạch phân làn màu trắng; cam07–cam08 của s01f28 thấy bóng cây trên mặt đường. Đây là thông tin hình ảnh bề mặt, không thể đọc tương đương từ các điểm LiDAR đang hiển thị.
3. **Chi tiết ngoại hình để phân biệt đầu, đuôi xe:** cam03 của s01f28 thấy cụm đèn hậu và kính sau của xe trắng phía trước, còn cam08–cam09 thấy đèn trước và lưới tản nhiệt của xe trắng phía sau. Các chi tiết này giúp kiểm tra hướng yaw khi hình học cụm điểm của xe gần đối xứng.

**Ba điều LiDAR cung cấp trực tiếp mà ảnh camera đơn lẻ không đo được theo mét:**

1. **Vị trí và khoảng cách trong hệ tọa độ VF:** bảng A của s01f28, idx 1 (Car) ghi tâm hộp tại x = 3,780 m, y = −12,133 m, tương ứng khoảng cách ngang 12,71 m; xe nằm phía trước và bên trái LiDAR. Ảnh camera thể hiện phối cảnh nhưng không trực tiếp cho các tọa độ này.
2. **Độ cao và kích thước không gian để căn đáy hộp:** với cùng cuboid s01f28, idx 1, chiều cao hộp là 1,455 m, z đáy là −1,791 m; phiếu tự kiểm ước tính đáy cao hơn mặt đường 0,03 m. Đây là số đo từ hộp đã gán trên point cloud và ước lượng mặt đường, dùng để kiểm tra đáy hộp thay vì chỉ nhìn hình chiếu trên ảnh.
3. **Mật độ điểm và phần bề mặt thực sự được LiDAR ghi nhận:** các ảnh 3D/Top cho thấy các cung quét và những vùng điểm thưa hoặc trống. Trong s01f28, cuboid Car idx 2 ở 6,73 m chứa 2.094 điểm, còn Car idx 1 ở 12,71 m chứa 849 điểm. Camera không cho biết số điểm phản hồi này; chênh lệch còn phụ thuộc kích thước, góc nhìn và che khuất, không chỉ khoảng cách. Vùng không có điểm cũng chưa đủ để kết luận không có vật.

Tám frame lướt:

| Frame  | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có)                                                                                                                                                                                                                                                                                                                  |
| ------ | ------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| s02f01 | Cạnh trên của ô BEV trong preview.                       | Xe buýt rất gần bên trái, chiếm phần lớn cam05–cam06 và che một phần cảnh phía sau. Trong BEV, vùng phía ngoài xe buýt có ít điểm hơn vùng đường trống; bóng che LiDAR là một giả thuyết cần kiểm tra khi xoay cảnh 3D.                                                                            |
| s02f21 | Cạnh trên của ô BEV trong preview.                       | Dải phân cách có hàng rào nằm sát bên trái; bên phải có người và xe hai bánh gần lề đường, rõ ở cam07–cam08. Cần tách cụm người/xe khỏi cây, cột và các xe đỗ gần nhau, tránh kéo hộp bao cả vật bên cạnh.                                                                                |
| s03f07 | Cạnh trên của ô BEV trong preview.                       | Nhiều xe nối sát nhau phía trước và phía sau; xe trắng rất gần bên trái, còn bên phải là hàng rào công trình. Trong BEV, các cụm xe gần có điểm rõ hơn các xe xa bị che một phần; cần đối chiếu nhiều camera để phân biệt từng xe và xác định đầu–đuôi.                            |
| s03f40 | Cạnh trên của ô BEV trong preview.                       | Cảnh thoáng hơn s03f07, có xe phía trước và xe hai bánh ở phía sau bên phải, thấy trên cam08–cam09. Các vật nhỏ ở xa có cụm điểm ít rõ hơn; cần đối chiếu camera để tránh bỏ sót hoặc nhầm với điểm của hàng rào và cây ven đường.                                                    |
| s04f13 | Cạnh trên của ô BEV trong preview.                       | Đường nhiều làn, có hàng rào phân cách bên trái và tường công trình bên phải. Xe trắng sát bên phải bị cắt bởi mép ảnh ở một số camera nhưng hiện rõ hơn ở cam07; cần ghép thông tin giữa các góc nhìn, không căn kích thước hộp chỉ theo phần xe trong một ảnh.                  |
| s04f46 | Cạnh trên của ô BEV trong preview.                       | Xe đỏ và xe trắng ở gần nhau bên phải, trong khi hàng rào vàng chạy sát bên trái. BEV có các dải điểm dài của hàng rào/tường và các cụm xe gần; cần phân biệt điểm của xe với công trình sát bên khi chỉnh chiều rộng hộp.                                                                |
| s05f10 | Cạnh trên của ô BEV trong preview.                       | Giao lộ rộng, nhiều xe có hướng khác nhau; cam05–cam06 thấy xe nằm ngang khung ảnh, cam09 có xe buýt phía sau. Không thể áp cùng một yaw cho tất cả các hộp; các vật nhỏ ở mép giao lộ cần đối chiếu camera vì cụm điểm xa khó nhận dạng.                                                      |
| s05f30 | Cạnh trên của ô BEV trong preview.                       | Hàng cây và bụi cây sát bên trái che một phần cảnh ở cam05–cam06; một số hình chiếu cuboid nằm trên vùng bụi cây. Preview không xét che khuất, nên cần xem point cloud và các camera khác để xác định hộp ứng với vật bị che hay hộp đặt sai; không kết luận chỉ từ hình chiếu này. |

## 3. Ba cuboid có hướng dẫn (Bước 3)

Ba cuboid được chọn từ bảng A và đối chiếu với ảnh preview sau sync. Thư mục ảnh đã có `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`. Ba mẫu được mô tả dưới đây lần lượt là s01f08, idx 1; s01f28, idx 3; s03f40, idx 4. Ảnh chụp hiện không kèm tên frame/ID, nên cần đối chiếu lại đúng vật khi nộp.

| Vật                                                                        | Frame, idx    | Class   | Mũi tên yaw đúng hướng?                                                                                                                                    | Đáy chạm đường?                                                                                                                                               | Đã kiểm Top/Side/Front + ảnh?                                                                                                        | Kích thước trong bảng 4.3?                                                                                                                                                                                                                                     |
| --------------------------------------------------------------------------- | ------------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| (a) Xe con màu xanh đậm gần bên trái, rõ ở cam05–cam06             | s01f08, idx 1 | Car     | Yaw 267,0°, gần hướng −y (270°), phù hợp đầu xe hướng về phía trước xe thu dữ liệu trên camera.                                               | z đáy −1,769 m; ground_gap +0,01 m, gần sát mặt đường ước tính; hộp chứa 3.205 điểm.                                                                | Đã đối chiếu BEV, camera và phiếu tự kiểm; chưa có ảnh cận Top/Side/Front để xác nhận đủ ba view.                     | Dài × rộng × cao = 4,578 × 1,984 × 1,488 m. Dài và cao trong khoảng sedan; rộng vượt mức 1,90 m của bảng 4.3 khoảng 0,084 m. Cần kiểm tra mép thân và phần gương trên point cloud; không tự thu hộp chỉ để khớp bảng tham chiếu. |
| (b) Xe tải thùng hở ở phía đối diện, rõ ở cam05–cam06            | s01f28, idx 3 | Truck   | Yaw 88,9°, gần +y (90°), phù hợp xe quay đầu ngược chiều xe thu dữ liệu trên camera.                                                                | z đáy −1,727 m; ground_gap +0,34 m, còn cao hơn mặt đường ước tính dù chưa bị selfcheck gắn cờ; cần xem lại Side/Front. Hộp chứa 2.285 điểm. | Đã đối chiếu BEV, camera và phiếu tự kiểm; cần kiểm tra cận Side/Front và chụp đủ ba view.                               | 6,070 × 2,505 × 2,039 m. Dài và rộng trong khoảng xe tải nhẹ; cao thấp hơn mức 2,40 m của bảng tham chiếu. Ảnh cho thấy xe thùng hở, nên cần căn theo điểm thực tế của cabin/thùng thay vì ép cao theo xe tải hộp.                  |
| (c) Xe hai bánh có người lái phía sau bên phải, rõ ở cam08–cam09 | s03f40, idx 4 | Cyclist | Yaw 268,6°, gần −y; cần kiểm tra cận đầu xe trên ảnh gốc và vạch orientation trong Top để xác nhận, vì người và xe khá nhỏ trong preview. | z đáy −1,546 m; ground_gap +0,27 m, cần xem lại đáy ở Side/Front dù không có cờ; hộp chứa 222 điểm.                                                 | Đã đối chiếu BEV, camera và phiếu tự kiểm; chưa có ảnh cận Top/Side/Front để xác nhận hộp bao đủ cả người và xe. | 1,982 × 0,721 × 1,695 m, đều nằm trong khoảng xe máy có người của bảng 4.3 (dài 1,8–2,3 m; rộng 0,70–0,95 m; cao 1,40–1,80 m).                                                                                                                    |

**Vòng hiệu chuẩn:** chưa có kết quả đối chiếu cuboid với bạn học hoặc ghi chép kết luận chung của lớp trong dữ liệu hiện có, nên chưa xác nhận được mức lệch hay nội dung cả lớp đã chốt. Quy tắc dùng để tự đối chiếu theo guideline: mỗi vật một Shape, yaw hướng về đầu xe, đáy sát mặt đường và hộp Cyclist gồm cả người đang lái cùng xe.

**Vật bỏ qua vì không đủ cơ sở:** chưa có danh sách ghi nhận cụ thể để xác nhận trong báo cáo. Trường hợp cần kiểm tra thêm là các xe hai bánh cạnh lề đường trong s01f28, cam07: người và xe ở sát nhau, một phần bị cây/bụi cây che; cần xác định người đang lái, đang dắt hay xe dựng không người (UNCERTAIN_CLASS / LOW_VISIBILITY). Đây là trường hợp cần rà soát, chưa phải khẳng định đã chủ động bỏ qua khi gán nhãn.

## 4. Bốn hiện tượng chất lượng (Bước 4)

Các ví dụ dưới đây thuộc task cá nhân D13-2A202603307, không mượn frame. Nguyên nhân là **giả thuyết**, chưa phải kết luận về cảm biến. Bốn ảnh minh chứng đã có trong thư mục ảnh; riêng `t4-4-diem-la.png` hiện chưa chỉ rõ nhóm điểm dưới mặt đường nêu trong bảng, cần chụp lại hoặc đánh dấu đúng vị trí.

| Hiện tượng                                 | Frame / tên ảnh / nguồn                                                 | Nguyên nhân (giả thuyết)                                                                                                         | Thấy gì trên point cloud                                                                                                                                                                                                                                                                                                                        | Hệ quả khi gán (theo guideline)                                                                                                                                                                                                              |
| --------------------------------------------- | -------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1. Bóng che                                  | s02f01; `t4-1-bong-che.png`; task cá nhân                              | Xe buýt rất gần bên trái có thể chắn tia LiDAR tới vùng phía ngoài xe.                                                   | Camera thấy xe buýt chiếm phần lớn cam05–cam06; BEV cho thấy vùng phía ngoài xe ít điểm hơn vùng đường trống. Cần xoay 3D để kiểm tra ranh giới bóng che.                                                                                                                                                                 | Không coi vùng trống điểm là chắc chắn không có vật. Với vật bị che một phần nhưng có đủ cơ sở nhận dạng, bật occluded và bao phần có thể suy luận; nếu không đủ cơ sở thì ghi LOW_VISIBILITY.             |
| 2. Thưa ở xa                                | s01f28, Car idx 11; `t4-2-thua-o-xa.png`; task cá nhân                 | Khoảng cách xa làm ít tia quét trúng vật; che khuất và góc nhìn cũng có thể góp phần.                                | Xe ở 43,45 m chỉ có 32 điểm trong hộp; Car idx 2 ở 6,73 m có 2.094 điểm. Hộp xa còn bị cờ cao 1,00 m và đáy cao hơn đường ước tính 0,81 m.                                                                                                                                                                                 | Không căn hộp chỉ theo vài điểm còn lại. Đối chiếu camera và quy tắc thấy khoảng 15–20% thể tích hoặc có đủ cơ sở suy luận; kiểm lại chiều cao và đáy, không tự kết luận hộp đúng vì đã có prelabel. |
| 3. Phản xạ thấp / ít điểm bất thường | s01f28, Car idx 4 so với idx 1; `t4-3-phan-xa-thap.png`; task cá nhân | Bề mặt tối màu, kính và góc tới có thể làm phản hồi yếu; chênh lệch chưa đủ để quy nguyên nhân cho màu sơn. | Car idx 4 ở 14,68 m có 463 điểm, intensity trung vị 0,00; Car idx 1 ở 12,71 m có 849 điểm, intensity trung vị 0,03. Đây là giá trị thô trong file LiDAR, không phải hệ số phản xạ đã hiệu chuẩn. Cần xem cận bề mặt xe trên 3D và camera để minh họa vùng ít phản hồi.                                      | Dùng cả hình dạng, camera và điểm lân cận để nhận dạng; không thu nhỏ hộp chỉ vì kính/thân xe thiếu điểm, cũng không nới hộp theo điểm của vật bên cạnh.                                                       |
| 4. Điểm lạ                                 | s04f46, phía trước bên phải; `t4-4-diem-la.png`; task cá nhân     | Có thể là phản xạ nhiều đường hoặc nhiễu đo; chưa đủ bằng chứng xác định cơ chế.                               | File LiDAR có 3 điểm gần x ≈ −9,9 m, y ≈ −34,3 m, cách ngang khoảng 35,7 m, ở z từ −16,64 đến −16,62 m. Các điểm mặt đường lân cận có z trung vị khoảng −1,80 m: nhóm điểm thấp bất thường gần 14,8 m. BEV không thể hiện độ sâu này, cần xoay nghiêng 3D hoặc nhìn cạnh để chụp bằng chứng. | Không gán nhóm điểm này thành vật và không kéo đáy cuboid xuống để bao chúng. Giữ hộp theo cụm điểm của vật thật và mặt đường; ghi nhận bất thường để kiểm tra.                                            |

## 5. Số từ cuboid của tôi (Bước 5)

Số liệu lấy từ `submission/D13-2A202603307-bangA.csv` và `submission/D13-2A202603307-tukiem.csv`; hai file có mã băm đối chiếu khớp nhau. Khoảng cách ngang dist_m = √(x²+y²); khoảng cách không gian r dưới đây tính từ tọa độ tâm đã làm tròn trong bảng A. Các độ dài, độ cao và khoảng cách đều tính bằng mét; yaw tính bằng độ. Độ sát đường được đánh giá theo mặt đường lân cận trong phiếu tự kiểm, không mặc định mọi nơi có cùng z.

| Cuboid (frame, idx, class) | z (m)   | h (m) | z đáy (m) | dist_m (m) | r (m)  | yaw_deg (°) | Nhận xét                                                                                                        |
| -------------------------- | ------- | ----- | ----------- | ---------- | ------ | ------------ | ----------------------------------------------------------------------------------------------------------------- |
| s01f08, 1, Car             | −1,025 | 1,488 | −1,769     | 3,74       | 3,878  | 267,0        | Ground gap +0,01 m: gần chạm đường; không có cờ.                                                          |
| s01f28, 3, Truck           | −0,707 | 2,039 | −1,727     | 11,83      | 11,852 | 88,9         | Ground gap +0,34 m: có dấu hiệu đáy cao, cần xem Side/Front dù chưa bị cờ.                              |
| s03f40, 4, Cyclist         | −0,699 | 1,695 | −1,546     | 13,96      | 13,975 | 268,6        | Ground gap +0,27 m: cần đối chiếu đáy và bánh xe; không có cờ không đồng nghĩa chắc chắn đúng. |

**Gần nhất và xa nhất:** trong toàn bộ bảng A, cuboid gần nhất là **s03f07, idx 3, Car**, dist_m = **3,43 m**, r ≈ **3,578 m**. Xa nhất là **s01f08, idx 4, Bus**, dist_m = **66,62 m**, r ≈ **66,622 m**; hộp này ngoài phạm vi 50 m của lab. Có **2 cuboid ngoài 50 m** trong bản xuất. Nếu chỉ xét phạm vi ≤ 50 m, xa nhất là **s04f46, idx 8, Car**, dist_m = **47,79 m**, r ≈ **47,803 m**.

**Đối chiếu kích thước:** Car s01f08, idx 1 có l × w × h = **4,578 × 1,984 × 1,488 m**. So với sedan trong bảng 4.3, dài và cao nằm trong khoảng, rộng vượt 1,90 m khoảng **0,084 m**; cần kiểm mép thân và gương, chưa thể kết luận nguyên nhân chỉ từ CSV. Cyclist s03f40, idx 4 có kích thước **1,982 × 0,721 × 1,695 m**, nằm trong cả ba khoảng của xe máy có người. Hộp phải bao cả người lái và xe.

**Đối chiếu hướng:** Car phía trước cùng chiều tại **s01f28, idx 1** có yaw **268,6°**, gần **270°** (−y). Truck đối diện tại **s01f28, idx 3** có yaw **88,9°**, gần **90°** (+y). Camera giúp phân biệt đầu–đuôi; cùng phương thân xe chưa đủ để phân biệt yaw với yaw + 180°.

Mật độ theo khoảng cách, chọn bốn Car trong cùng frame s01f28:

| Cuboid (frame, idx, class) | dist_m (m) | Số điểm (`points_in`) |
| -------------------------- | ---------- | -------------------------- |
| s01f28, 2, Car             | 6,73       | 2.094                      |
| s01f28, 1, Car             | 12,71      | 849                        |
| s01f28, 6, Car             | 21,03      | 359                        |
| s01f28, 10, Car            | 38,83      | 71                         |

Trong cặp đầu–cuối, khoảng cách tăng khoảng **5,77 lần** thì số điểm còn khoảng **3,39%** (giảm khoảng 29,5 lần). Các giá trị này cùng xu hướng giảm theo khoảng cách với slide: khoảng 1.968 điểm ở 0–10 m, 718 ở 10–20 m, 214 ở 20–30 m và 72 ở 30–40 m. Đây là các vật khác nhau nên chênh lệch còn phụ thuộc kích thước, góc tới, che khuất và cách đặt hộp; không xem bốn mẫu này là quy luật cố định cho mọi xe.

**Hai chỉnh sửa xác nhận được khi đối chiếu bản vừa nhập prelabel với bảng A hiện tại:**

1. **s04f13, idx 3, Car:** tâm z từ khoảng **−0,987 m** xuống **−1,105 m**, chiều cao từ **1,499 m** lên **1,627 m**, đáy từ khoảng **−1,737 m** xuống **−1,918 m**; tọa độ y cũng dịch khoảng **0,015 m**. Sau chỉnh, hộp có 1.042 điểm, ground gap +0,08 m và không có cờ.
2. **s05f30, idx 10, Car:** yaw từ khoảng **267,9°** thành **88,4°**, tức xoay gần **180°**; tâm và kích thước giữ gần như nguyên vẹn. Đây là sửa hướng đầu–đuôi; số điểm trong hộp là 32, phiếu không có cờ và chưa ước lượng được ground gap.

Hai thay đổi trên có bằng chứng trước/sau, nhưng chưa có nhật ký để xác nhận chúng được thực hiện sau lần chạy preview/selfcheck nào. Vì vậy, chưa dùng chúng để khẳng định đã hoàn thành yêu cầu sửa ít nhất hai hộp sau vòng tự kiểm.

**Cảnh báo còn lại:** bản hiện tại có **23/118 cuboid bị cờ**. Chưa có ghi chép xác nhận hộp nào đã được xem lại và chủ động giữ nguyên vì hợp lý. Những trường hợp cần ưu tiên gồm s01f08, idx 4 (Bus: rộng bất thường và ground gap ước tính rất lớn), s01f28, idx 11 (Car: đáy cao, hộp thấp) và s03f40, idx 2 (Car: dài 7,07 m, cần kiểm kích thước/class). Ground gap lớn có thể do hộp hoặc ước lượng mặt đường; phải xem lại dữ liệu trước khi quyết định sửa hay giữ. Không chỉnh hộp chỉ để hết cờ.

**Đổi kiểm:** đã ghi MSSV bạn ngồi cạnh là **2A202603301**, nhưng chưa có bằng chứng người này đã kiểm chéo hoặc kết quả đối chiếu. Hai số có thể dùng để kiểm là chiều cao **1,627 m** và yaw **270,1°** của s04f13, idx 3; cần bổ sung người thực sự kiểm, kết quả khớp/lệch và cách xử lý sau khi thực hiện.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc

**Ý nghĩa đối với model:** nhãn 3D dùng mét và hệ tọa độ LiDAR để model học vị trí, kích thước và hướng của vật trong không gian thực; kích thước pixel thay đổi theo phối cảnh nên không trực tiếp biểu diễn các đại lượng đó. Vật thưa điểm cần quy tắc riêng vì ít phản hồi không đồng nghĩa vật nhỏ hoặc không tồn tại; nhãn phải dựa trên phần quan sát được và cơ sở suy luận đủ rõ để tránh dạy model những hộp đoán tùy ý.

**Cách dùng AI:**

- **Công cụ:** ChatGPT/Codex 
- **Yêu cầu chính:** tạo và nhập prelabel cho s03f40, s04f13, s04f46, s05f10, s05f30, giữ nhãn đã làm; sau đó hỗ trợ xuất nhãn, tính số liệu và soạn báo cáo từ dữ liệu hiện có.
- **Cách dùng kết quả:** AI đã thêm 63 cuboid vào task sau khi bỏ qua hai dự đoán trùng và sao lưu nhãn cũ; tôi chỉnh sửa lại nhãn trên CVAT rồi chạy sync. Các nhận xét và số liệu trong báo cáo được AI hỗ trợ tổng hợp từ ảnh, bảng A và phiếu tự kiểm; prelabel và nội dung tổng hợp vẫn cần tôi kiểm tra, đặc biệt các hộp còn cờ và các quan sát chưa có ảnh minh chứng.

**Điều còn chưa chắc:** việc căn đáy tại các vùng mặt đường ít điểm; phân biệt loại xe và kích thước thật khi bị che; hướng đầu–đuôi của vật nhỏ ở xa; nguyên nhân phản hồi yếu và các điểm nằm sâu dưới mặt đường.

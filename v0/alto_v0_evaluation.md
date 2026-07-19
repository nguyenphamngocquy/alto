# Đánh giá baseline ALTO v0

## 1. Mục tiêu của phiên bản v0

Phiên bản ALTO v0 được xây dựng nhằm kiểm chứng khả năng sinh người theo ngữ cảnh trên ảnh nền có sẵn và khảo sát mức độ ảnh hưởng của latent seed đến chất lượng kết quả. Thay vì triển khai ngay cơ chế tối ưu latent thích nghi, v0 tập trung xây dựng một baseline chạy xuyên suốt từ khâu lựa chọn dữ liệu, xác định vùng sinh người, sinh nhiều candidate theo các seed khác nhau, chấm điểm tự động, xếp hạng và lựa chọn kết quả Best-of-N.

Ba nhóm ngữ cảnh được khảo sát gồm:

1. **Free placement**: sinh người trong vùng trống của ảnh.
2. **Object-relative placement**: sinh người theo quan hệ không gian với một vật thể tham chiếu, chẳng hạn đứng cạnh bàn hoặc ô tô.
3. **Physical interaction**: sinh người có tương tác vật lý với vật thể, chẳng hạn ngồi trên ghế hoặc sofa.

## 2. Thiết lập thí nghiệm

Baseline sử dụng tập validation của SceneParse150/ADE20K để chọn sáu ảnh nền, mỗi nhóm ngữ cảnh gồm hai sample. Với mỗi sample, mô hình sinh tám candidate từ tám latent seed khác nhau.

Cấu hình chính:

- Backbone: Stable Diffusion XL Inpainting.
- Độ phân giải: 1024 × 1024.
- Số bước suy diễn: 40.
- Guidance scale: 8.0.
- Inpainting strength: 0.99.
- Số seed trên mỗi sample: 8.
- Tổng số ảnh sinh: 48.

Vùng inpainting được xác định từ semantic annotation và các quy tắc hình học theo từng loại ngữ cảnh. Mask của người đứng và người ngồi có kích thước khác nhau nhằm phản ánh tư thế dự kiến.

## 3. Quy trình automatic scoring

V0 triển khai automatic scoring để baseline có thể chạy end-to-end mà không cần dừng lại giữa chừng cho manual scoring. Các thành phần đánh giá gồm:

- phát hiện sự xuất hiện của người;
- độ tin cậy của person detector;
- số lượng người;
- mức độ phù hợp giữa person bounding box và target region;
- tỷ lệ kích thước người;
- mức độ phù hợp với prompt bằng CLIP;
- quan hệ hình học với reference object;
- mức độ bảo toàn background;
- hình phạt khi xuất hiện nhiều người.

Ảnh không phát hiện được người được gán tổng điểm bằng 0. Các candidate còn lại được chấm theo trọng số riêng cho từng nhóm context. Kết quả được lưu thành các bảng điểm, ranking, Best-of-N và tổng hợp theo sample/context.

## 4. Kết quả chính

### 4.1. Khả năng sinh người

Trong tổng số 48 candidate, 43 ảnh được detector xác định có người, tương ứng tỷ lệ khoảng 89,6%. Kết quả này cho thấy vấn đề chính không còn là mô hình hoàn toàn không sinh được người, mà chuyển sang chất lượng, vị trí, tỷ lệ, quan hệ không gian và mức độ tự nhiên của người được sinh.

Tỷ lệ sinh người theo context:

| Context | Tỷ lệ phát hiện người |
|---|---:|
| Free placement | 93,75% |
| Object-relative | 87,50% |
| Physical interaction | 87,50% |

Free placement là nhóm ổn định nhất. Object-relative và physical interaction có độ khó cao hơn do cần duy trì quan hệ giữa người với các thành phần có sẵn trong ảnh.

### 4.2. Seed sensitivity

Kết quả cho thấy cùng một background, mask và prompt nhưng latent seed khác nhau có thể tạo ra khác biệt đáng kể về:

- sự xuất hiện của người;
- số lượng người;
- vị trí;
- tỷ lệ cơ thể;
- mức độ gần reference object;
- tư thế và khả năng tương tác;
- tổng điểm automatic scoring.

Điều này xác nhận giả thuyết ban đầu rằng latent initialization là một yếu tố có ảnh hưởng đáng kể đến kết quả context-aware human generation.

### 4.3. Sample sensitivity

Ngoài latent seed, kết quả còn phụ thuộc mạnh vào background sample được lựa chọn. Khi thay candidate hạng đầu bằng các candidate khác trong cùng top-k heap, tỷ lệ sinh được người và chất lượng kết quả có thể thay đổi đáng kể, mặc dù loại context, prompt và cấu hình mô hình được giữ nguyên.

Sự khác biệt này có thể xuất phát từ:

- kích thước và hình dạng vùng trống;
- phối cảnh và khoảng cách camera;
- độ rõ của mặt sàn;
- vị trí và kích thước reference object;
- mức độ che khuất;
- độ phức tạp của texture;
- số lượng chi tiết nằm trong vùng inpainting;
- mức độ phù hợp tự nhiên giữa scene và tư thế cần sinh.

Do đó, candidate rank cao theo semantic-geometric heuristic chưa chắc là background thuận lợi nhất cho generation. Kết quả v0 vì vậy phản ánh đồng thời ảnh hưởng của latent seed và sample selection.

Có thể phân biệt hai loại biến thiên:

- **Within-sample variation**: thay latent seed trong khi giữ nguyên background, mask và prompt.
- **Between-sample variation**: thay background sample trong cùng một context specification.

Best-of-N chỉ khai thác được biến thiên giữa các seed trong một sample cố định. Nếu bản thân sample không phù hợp với backbone hoặc vùng inpainting, việc tăng số seed có thể không tạo ra candidate tốt.

### 4.4. Hiệu quả của Best-of-N

Best-of-8 cho kết quả tốt hơn điểm trung bình của các candidate ở hầu hết sample. Mức cải thiện không đồng đều: một số context dễ có các seed cho kết quả tương đối ổn định, trong khi các context khó có chênh lệch lớn giữa candidate tốt nhất và candidate trung bình.

Quan sát này cho thấy việc sử dụng cùng một ngân sách tìm kiếm cho mọi sample có thể không tối ưu. Các sample dễ có thể dừng sớm, trong khi sample khó cần nhiều candidate hoặc cơ chế tinh chỉnh latent sâu hơn. Đây là cơ sở thực nghiệm ban đầu cho thành phần “Adaptive” trong ALTO.

### 4.5. Khác biệt theo context

**Free placement** có kết quả ổn định nhất vì mô hình chỉ cần sinh người trong vùng trống, không phải duy trì quan hệ với vật thể cụ thể.

**Object-relative placement** vẫn là nhóm khó nhất. Mô hình thường sinh được người nhưng vị trí tương đối với bàn hoặc ô tô chưa luôn hợp lý. Một số candidate có thể nằm trong target region nhưng chưa thực sự thể hiện đúng quan hệ “beside” hoặc “near”.

**Physical interaction** có độ biến thiên cao giữa các seed. Một số seed tạo được candidate có điểm cao, nhưng pose ngồi, tiếp xúc với ghế và tính tự nhiên của cơ thể vẫn chưa được đánh giá đầy đủ bằng scorer hiện tại.

## 5. Ưu điểm của v0

1. Pipeline đã chạy xuyên suốt từ dữ liệu đầu vào đến lựa chọn Best-of-N.
2. Tỷ lệ sinh người đủ cao để nghiên cứu latent selection thay vì bị giới hạn bởi lỗi no-human.
3. Automatic scoring giúp giảm đáng kể thời gian chấm thủ công.
4. Kết quả xác nhận seed sensitivity và tiềm năng của Best-of-N.
5. Hệ thống đã phân biệt các loại context và sử dụng trọng số đánh giá khác nhau.
6. Các file kết quả và manifest hỗ trợ tái lập thí nghiệm.
7. V0 tạo nền móng trực tiếp cho context-aware scoring và adaptive latent optimization ở các phiên bản sau.

## 6. Hạn chế

### 6.1. Chất lượng hình ảnh và tính chân thực

Người được sinh đôi khi còn mang cảm giác nhân tạo, thiếu contact shadow, ánh sáng chưa hòa hợp và chi tiết giải phẫu chưa tự nhiên. Person detector vẫn có thể phát hiện những ảnh như vậy với confidence rất cao, nên detection confidence không phải thước đo tốt cho chất lượng hình ảnh.

### 6.2. Automatic scorer chưa phản ánh đầy đủ chất lượng cảm nhận

Scorer hiện đánh giá tốt các thuộc tính hình học cơ bản như human presence, placement, scale và coarse relation. Tuy nhiên, nó chưa đánh giá đáng tin cậy:

- chất lượng khuôn mặt và cơ thể;
- tính hợp lý của pose;
- tiếp xúc vật lý thực sự với ghế;
- bóng và ánh sáng;
- vật thể tham chiếu có bị thay đổi hay không;
- mức độ photorealistic;
- lỗi tay, chân và anatomy.

### 6.3. Relation score mới là proxy hình học

Relation score chủ yếu dựa trên bounding box, khoảng cách và overlap. Một candidate có thể đạt relation score tương đối cao dù người chưa thực sự ngồi đúng tư thế hoặc có hiện tượng xuyên vào furniture.

### 6.4. Detection confidence bị bão hòa

Ở nhiều candidate có người, detection confidence gần 1.0. Thành phần này phù hợp để làm human-presence gate nhưng có khả năng phân hạng thấp. Việc dành trọng số lớn cho nó có thể làm automatic score kém nhạy với chất lượng thực tế.

### 6.5. Prompt alignment có độ phân giải hạn chế

CLIP score hữu ích để đánh giá semantic alignment tổng quát, nhưng không đủ để xác định chính xác các quan hệ như “ngồi trên”, “đứng cạnh”, chân chạm đất hoặc vật thể có bị biến dạng hay không.

### 6.6. Background score chưa đánh giá đúng vùng bị chỉnh sửa

Nếu ảnh gốc được composite lại ngoài mask, background score ngoài mask có thể gần như luôn đạt tối đa. Trong khi đó, những thay đổi đáng kể thường xảy ra bên trong mask, chẳng hạn ghế bị thay đổi, tranh hoặc họa tiết phía sau người bị tái tạo. Vì vậy background metric hiện tại chưa cung cấp đủ khả năng phân biệt giữa các candidate.

### 6.7. Quy mô thí nghiệm còn nhỏ

V0 chỉ dùng sáu sample và tám seed cho mỗi sample. Kết quả đủ cho proof of concept nhưng chưa đủ để đưa ra kết luận tổng quát về toàn bộ bài toán context-aware human generation.

### 6.8. Phụ thuộc vào target mask

Placement và scale được đánh giá tương đối với target mask. Nếu kích thước mask chưa phản ánh đúng scale tự nhiên của scene, scorer có thể ưu tiên candidate vừa mask nhưng chưa chắc hợp lý về phối cảnh.

### 6.9. Độ nhạy đối với sample đầu vào

V0 chỉ sử dụng một background được chọn cho mỗi sample specification. Thử nghiệm thay đổi candidate trong heap cho thấy tỷ lệ sinh người và chất lượng ảnh có thể tốt hơn hoặc kém hơn đáng kể tùy background cụ thể.

Điều này cho thấy candidate-selection score hiện tại mới chủ yếu phản ánh semantic geometry, chưa chắc dự đoán đúng mức độ thuận lợi của sample đối với generation. Vì vậy, tỷ lệ thành công và Best-of-N gain của v0 không nên được xem là đại diện cho toàn bộ mỗi context category.

Quy mô đánh giá hiện tại cũng chưa cho phép tách rõ ảnh hưởng của sample selection khỏi ảnh hưởng của latent seed.

## 7. Kết luận về v0

ALTO v0 đã hoàn thành vai trò của một baseline proof of concept. Trên tập sample được lựa chọn trong thí nghiệm, phiên bản này cho thấy:

- SDXL Inpainting có thể sinh người theo nhiều loại context với tỷ lệ thành công tương đối cao, nhưng hiệu năng phụ thuộc đáng kể vào background sample cụ thể;
- Latent seed ảnh hưởng rõ rệt đến sự xuất hiện, vị trí, tỷ lệ và quan hệ của người trong cùng một sample;
- Best-of-N có thể cải thiện đáng kể kết quả ở các context có độ bất định cao, nhưng không bảo đảm khắc phục được những sample không thuận lợi đối với backbone;
- Automatic scoring cho phép pipeline chạy end-to-end và là thành phần cần thiết cho quá trình tìm kiếm latent.

Tuy nhiên, scorer hiện tại mới đủ dùng cho ranking sơ bộ, chưa đủ mạnh để trở thành objective cuối cùng cho adaptive latent optimization. Vì vậy, v0 nên được xem là mốc xác nhận tính khả thi và xác định các failure mode chính.

## 8. Đề xuất cho v1

V1 nên tập trung xây dựng **Context-aware Best-of-N với evaluator đáng tin cậy hơn**, thay vì triển khai ngay toàn bộ adaptive latent refinement.

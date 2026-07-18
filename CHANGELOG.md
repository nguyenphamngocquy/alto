# Changelog

File này ghi lại các mốc phát triển chính của dự án ALTO theo từng phiên bản.

## [v0] - 2026-07

### Added

- Xây dựng baseline end-to-end cho bài toán context-aware human generation.
- Hỗ trợ ba nhóm ngữ cảnh chính:
  - free placement;
  - object-relative placement;
  - physical interaction.
- Tích hợp pipeline lựa chọn ảnh nền và vùng sinh người từ SceneParse150/ADE20K.
- Sử dụng Stable Diffusion XL Inpainting để sinh nhiều candidate từ các latent seed khác nhau.
- Xây dựng automatic scoring theo ngữ cảnh, gồm phát hiện người, vị trí, tỷ lệ, mức độ phù hợp với prompt, quan hệ với vật thể tham chiếu và bảo toàn background.
- Bổ sung cơ chế xếp hạng candidate và lựa chọn Best-of-N.
- Chuẩn hóa đầu ra thí nghiệm gồm manifest, score, ranking và summary.


## [v1] - 2026-07

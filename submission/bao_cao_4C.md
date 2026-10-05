### 4C — Val gốc và val lật gương (YOLO26n-pose, 40 epoch, imgsz 640)

| Model | Pose mAP50-95 — val gốc | Pose mAP50-95 — val lật gương | Thay đổi |
|---|---:|---:|---:|
| flip_idx giải phẫu | 0.457 | 0.439 | -4% |
| flip_idx đồng nhất | 0.417 | 0.298 | -29% |

**Metric nào đã che lỗi?** Pose mAP (và cả box mAP) trên **val gốc**. Val gốc có cùng phân bố với train: 53/53 con hổ quay phải.
Model `flip_idx` đồng nhất học sai quy ước ("chân phía camera luôn là right_*") nhưng không bị phạt vì đảo trái/phải trên bất kỳ ảnh val nào, nên trên val gốc
hai model chỉ chênh 0.040 (0.457 so với 0.417, tức 9%), mức mà nhìn riêng rất dễ bị coi là nhiễu giữa hai lần train. Box mAP hoàn toàn không nhìn tên keypoint. OKS/pose mAP so keypoint theo
chỉ số, nên chỉ lộ ra việc đảo trái ↔ phải khi trong val có hổ quay trái. Trên val lật gương (giả lập hổ quay trái lúc triển khai), model đồng nhất đổi
-29%, còn model giải phẫu đổi -4%. Lỗi chỉ hiện ra khi tập đánh giá chứa đúng tình huống mà augmentation đã tạo ra lúc train.

**Thiết kế tập val:** (1) phủ đủ các biến thể sẽ gặp khi triển khai: cả hai hướng quay (thêm bản lật gương với nhãn giải phẫu, tốt hơn là video thật hổ đi ngược
chiều), nhiều góc nhìn, che khuất, bị cắt mép; (2) tách val theo video/phiên quay khác train để tránh các frame gần trùng nhau; (3) báo cáo thêm metric theo từng
keypoint, từng bên và tỉ lệ ảnh bị đảo trái/phải (OKS khi đổi nhãn trái/phải cao hơn OKS gốc), không chỉ một con số mAP; (4) chấm OKS bằng σ riêng cho từng
keypoint (`kpt_oks_sigmas`) ước lượng từ gán nhãn lặp, thay cho 1/K.

# Real-Time Multi-Camera Safety Monitoring System

This repository contains the implementation of our *Real-Time Multi-Camera Safety Monitoring System* designed for construction and industrial sites.  
The system integrates *PPE detection, fall detection, fire hazard recognition, and danger-zone monitoring* into a unified pipeline.

---

## Features
- *Multi-camera support* with end-to-end synchronization
- *High FPS performance* (30–40 FPS across 4+ streams)
- PPE compliance monitoring (*helmet, vest, etc.*)
- Fire hazard detection in real-time
- Fall detection with pose estimation backbone
- Polygon-based *danger zone* definition and violation alerts
- Web-based interface for live monitoring and analytics

---

## System Architecture
- **Backend**: Python, PyTorch, Django
- **Frontend**: React (real-time visualization)
- **Models**: YOLOv4/YOLOv8, custom pose-based fall detection
- **Deployment**: Docker + RTSP camera streaming

---

## Dataset & Training
We trained and evaluated the models using a mix of:
- Publicly available PPE and safety datasets  
- Our own *multi-camera industrial scenario recordings*  
- Synthetic augmentation to improve robustness against *occlusion, motion blur, and varying lighting conditions*

---

## Results
- End-to-end throughput: *30–40 FPS across 4 cameras*  
- PPE detection: *mAP ≈ 95% at 60+ FPS (single model)*  
- Fire detection: *Accuracy ≈ 96%*  
- Fall detection: *≈ 94% accuracy, 40+ FPS*  
- Danger-zone violation detection: *96% accuracy, ~50 FPS*

---

## References
For academic background and comparison, please see the following works:

- Nath, N. D.; Behzadan, A. H.; and Paal, S. G. 2020. *Deep learning for site safety: Real-time detection of personal protective equipment*. Automation in Construction 112: 103085.  
- Gu, Y.; Wang, Y.; Shi, L.; Li, N.; Zhuang, L.; and Xu, S. 2021. *Automatic detection of safety helmet wearing based on head region location and YOLOv4*. Advances in Civil Engineering 2021: 6625562.  
- Chen, Y.; and Demachi, K. 2020. *Automatic detection of protective equipment using YOLOv3 at Fukushima Daiichi Nuclear Power Station*. Journal of Radiation Research 61(5): 758–765.  
- Zaidi, S. M.; Yang, L.; Abbas, H.; Hussain, R.; Lee, C.; and Park, Y. 2024. *Vision-based construction safety monitoring utilizing temporal analysis to reduce false alarms*. Buildings 14(6): 1878.  
- Safe-Construct. 2025. *Safe-Construct: 3D multi-view safety violation recognition in construction environments*. arXiv preprint arXiv:2504.10880.

---

## Citation
If you use this codebase in your research, please cite our paper:

```bibtex
@inproceedings{YourPaper2025,
  title={Real-Time Multi-Camera Safety Monitoring System},
  author={Your Name and Co-Authors},
  booktitle={Proceedings of the AAAI Conference on Artificial Intelligence},
  year={2025}
}

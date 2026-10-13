---
id: "11_AI_and_ML/Applied_ML/Computer_Vision/Input_Resolution_Trade-offs"
level: 4
type: "concept"
topic: "11_AI_and_ML/Applied_ML/Computer_Vision"
tools: []
---

# Input Resolution Trade-offs

**L4 Advanced** · [↑ Computer Vision](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Intermediate] [In OpenCV, why does cv2.imread load images in BGR order and why can this cause bugs with matplotlib?](<../../../PRACTICE/11_AI_and_ML/Applied_ML/Computer_Vision/Q-09783_In_OpenCV_why_does_cv2_imread_load_images_in_BGR_order_and_w.md>)
- [Intermediate] [Your training crashes with OOM on large images. Walk me through the levers (batch size, gradient accumulation, resolution, checkpointing) to fit memory.](<../../../PRACTICE/11_AI_and_ML/Applied_ML/Computer_Vision/Q-09724_Your_training_crashes_with_OOM_on_large_images_Walk_me_throu.md>)
- [Deep] [A CNN trained with cv2 (BGR) preprocessing is served behind an API that decodes images with PIL (RGB). How would this channel-order mismatch manifest in metrics and how would you catch it in CI?](<../../../PRACTICE/11_AI_and_ML/Applied_ML/Computer_Vision/Q-09638_A_CNN_trained_with_cv2_BGR_preprocessing_is_served_behind_an.md>)
- [Deep] [You need to serve segmentation masks at high resolution but memory is tight. How would you trade off tiling, downsampling and model choice to fit the GPU?](<../../../PRACTICE/11_AI_and_ML/Applied_ML/Computer_Vision/Q-09654_You_need_to_serve_segmentation_masks_at_high_resolution_but_.md>)
- [Deep] [Explain how you'd export a PyTorch vision model to ONNX/TensorRT for serving and what accuracy or op-support pitfalls to watch for.](<../../../PRACTICE/11_AI_and_ML/Applied_ML/Computer_Vision/Q-09666_Explain_how_you_d_export_a_PyTorch_vision_model_to_ONNX_Tens.md>)
- [Deep] [Describe how you'd set up a CI/CD pipeline that automatically evaluates a new CV model on a golden test set and blocks deploy on regression.](<../../../PRACTICE/11_AI_and_ML/Applied_ML/Computer_Vision/Q-09672_Describe_how_you_d_set_up_a_CI_CD_pipeline_that_automaticall.md>)
- [Deep] [You need to serve both a detection and a classification model in one request under a tight SLA. How would you architect the pipeline to minimize latency?](<../../../PRACTICE/11_AI_and_ML/Applied_ML/Computer_Vision/Q-09691_You_need_to_serve_both_a_detection_and_a_classification_mode.md>)
- [Deep] [Your detector produces flickering boxes across video frames. How would you add temporal smoothing or tracking to stabilize outputs?](<../../../PRACTICE/11_AI_and_ML/Applied_ML/Computer_Vision/Q-09694_Your_detector_produces_flickering_boxes_across_video_frames_.md>)

## Related concepts

- [Performance Regression Gates](<../../../10_DevOps_and_Platform/Delivery/CI_CD/Performance_Regression_Gates.md>) *(CI CD)*
- [Multi-cloud Strategy](<../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Managed_Services_vs_Self-hosting/Multi-cloud_Strategy.md>) *(Cloud Fundamentals)*
- [Drift-triggered Retraining](<../../../TOOLS/Python/Python_Language/Python_Tail_Latency/Drift-triggered_Retraining.md>) *(Python Language)*


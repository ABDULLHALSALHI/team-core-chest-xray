# Backend Latency Notes

## Measurement setup
- Endpoint: `POST /predict`
- Hardware: CPU (no GPU available on test machine)
- Model: ConvNeXt-Tiny, 320x320, BCE checkpoint
- Measured via the `inference_time_ms` field returned in each response (wall-clock time for the model forward pass, from image received to prediction returned)

## Results (5 test images)

| Run | inference_time_ms |
|---|---|
| 1 | 351.8 |
| 2 | 475.4 |
| 3 | 344.1 |
| 4 | 438.5 |
| 5 | 448.6 |

**Average: ~411.7 ms**
**Range: 344.1 ms – 475.4 ms**

## Notes
- All measurements are on CPU. Inference would be noticeably faster on a GPU-enabled machine; this number reflects the local development/demo environment, not a production deployment target.
- Grad-CAM generation (a separate `/gradcam` call) is not included in these numbers — it requires an additional forward + backward pass and will add latency on top of the `/predict` time above.
- Sample size is small (5 runs) and meant to give an approximate, honest order of magnitude for the report — not a rigorous benchmark.

## Upload behavior & invalid-input handling (for report reference)
- Uploaded images are processed in memory only and are not saved to disk or retained after the request completes.
- Invalid input handling:
  - Unsupported file type (not JPEG/PNG) → `400 Bad Request` with a clear error message.
  - Corrupt or unreadable image data → `400 Bad Request` with a clear error message.
  - Grad-CAM generation failure → the `/predict` response still returns successfully with predictions; only the Grad-CAM step reports the failure separately.

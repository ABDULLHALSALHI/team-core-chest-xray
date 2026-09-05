# Frontend ↔ Backend Handoff

**Owner of frontend:** Layan Allhidean  
**Backend owner:** Deema Omar Alquwaei  
**Model:** ConvNeXt-Tiny, 320 × 320, 14 outputs  
**Status:** Proposed integration contract — please confirm before final connection.

The goal of this document is to keep the frontend and backend aligned without duplicating model logic in the browser.

---

## 1. Prediction endpoint

### Proposed request

```http
POST /predict
Content-Type: multipart/form-data
```

Multipart field:

```text
file
```

The frontend sends one uploaded chest X-ray file.

### Proposed success response

```json
{
  "model_version": "convnext_tiny_320_bce_v1",
  "predictions": [
    {
      "name": "Atelectasis",
      "probability": 0.21,
      "threshold": 0.34,
      "positive": false
    },
    {
      "name": "Consolidation",
      "probability": 0.09,
      "threshold": 0.12,
      "positive": false
    }
  ]
}
```

`predictions` must contain **exactly 14 items** in the agreed model label order.

### Required fields per pathology

- `name` — exact pathology name
- `probability` — sigmoid probability between 0 and 1
- `threshold` — saved pathology-specific threshold
- `positive` — boolean result after comparing probability to threshold

### Model metadata

Recommended:

```json
"model_version": "convnext_tiny_320_bce_v1"
```

This helps the developer panel confirm which model artifact produced the response.

---

## 2. Exact label order

The frontend currently validates this exact order:

```text
Atelectasis
Consolidation
Infiltration
Pneumothorax
Edema
Emphysema
Fibrosis
Effusion
Pneumonia
Pleural_Thickening
Cardiomegaly
Nodule
Mass
Hernia
```

Please do not rename, reorder, or add labels in the backend response without coordinating the change.

`No_Finding` is **not** a model output. The frontend can derive a No Finding state when all 14 `positive` values are false.

---

## 3. Preprocessing ownership

The frontend does not preprocess the image for the model.

The backend should own the final inference preprocessing used by the frozen model, including:

- image decoding / RGB handling,
- resize to 320 × 320,
- tensor conversion,
- ImageNet normalization,
- ConvNeXt-Tiny inference,
- sigmoid conversion,
- pathology-specific thresholds.

---

## 4. Grad-CAM handoff

The frontend needs a **class-specific** Grad-CAM for a selected pathology.

Current frontend proposal:

```http
POST /gradcam
Content-Type: multipart/form-data
```

Multipart fields:

```text
file
pathology
```

Example:

```text
pathology=Cardiomegaly
```

### Preferred response

A directly renderable PNG or JPEG image response.

Alternative:

A JSON response containing a stable heatmap URL/path is also acceptable, but the final format should be agreed once and documented.

### Important

The frontend will display one selected pathology heatmap at a time. The highest flagged finding can be selected by default, and the user can switch between other flagged findings.

The UI should show the Grad-CAM heatmap itself. Bounding boxes used for IoBB evaluation are evaluation artifacts and do not need to appear in the normal user interface.

---

## 5. Error response

Preferred error body:

```json
{
  "detail": "Clear error message"
}
```

Please return an appropriate HTTP status code for invalid files or inference failures.

The frontend will show a short user-friendly message while technical details remain in developer diagnostics.

---

## 6. File types

The frontend currently accepts:

```text
PNG
JPG / JPEG
```

PDF uploads are intentionally disabled because the model pipeline is image-based.

---

## 7. CORS / serving strategy

If frontend and backend run on different local origins, FastAPI must allow the frontend origin during development.

Alternative for the final demo: FastAPI can serve the static frontend files so the full application runs from one origin.

Either approach is acceptable; the team should choose the simpler reliable demo setup.

---

## 8. Frontend assumptions that should not change silently

- 14 model outputs only
- exact label order above
- probabilities are values from 0 to 1
- pathology-specific thresholds are supplied by the inference pipeline
- `positive` is a boolean
- `No Finding` is derived, not predicted
- Grad-CAM is class-specific
- model language remains probabilistic / non-clinical

---

## 9. Integration checklist

Before switching `MOCK_MODE` to `false`:

- [ ] `/predict` request field name confirmed
- [ ] prediction JSON shape confirmed
- [ ] exact 14-label order confirmed
- [ ] error response confirmed
- [x] accepted file types confirmed: PNG and JPG/JPEG only
- [ ] Grad-CAM endpoint / response format confirmed
- [ ] CORS or single-origin serving confirmed
- [ ] one real X-ray tested end to end
- [ ] 2–3 fixed demo images tested

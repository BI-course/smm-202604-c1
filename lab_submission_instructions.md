# Serving ML Models: Student Details and Individual Member Contributions

**Name of the team on GitHub Classroom:** `C1`

---

## Member 1

| Details | Comment |
| ------- | ------- |
| **Student ID** | 170214 |
| **Name** | Grace Chelangat |
| **Branch** | [`feature/regression-endpoint`](https://github.com/GraceChelangat/BI-ServingMLModels-C1/tree/feature/regression-endpoint) |

### Contribution

**1. Retail Sales Regression API**

- Trained a Lasso regression pipeline (imputation, scaling, one-hot encoding, Lasso) in `regressionC1.ipynb` to predict a store's daily sales revenue in KES, and exported it to `models/retail_sales_regressor.joblib`.
- Added the `POST /predict/regression` endpoint to the shared `api.py`. It loads the saved model once at startup and never retrains it.
- It validates the request and returns a 400 for a missing or invalid JSON body, missing features, non-numeric values in numeric fields, and empty values in categorical fields.
- It accepts `null` for numeric fields because the pipeline imputes missing values.
- It converts the prediction back to KES with `np.expm1`, because the model was trained on `log1p(daily_sales_revenue_kes)`.
- Deployed an earlier version of the API to Render with Gunicorn and debugged deployment failures from the Render logs.

**Endpoint:** `POST /predict/regression`

**Example request**

```json
{
  "store_format": "Supermarket",
  "location_type": "Mall",
  "delivery_available": "Yes",
  "store_size_category": "Medium",
  "store_age_years": 5.0,
  "store_size_sqm": 450.0,
  "num_staff": 12,
  "promotions_active": 1,
  "footfall_count": 800.0,
  "loyalty_program_members": 150,
  "online_order_count": 40.0,
  "avg_basket_size_kes": 1200.0,
  "local_events_nearby": 0
}
```

**Example response**

```json
{
  "predicted_daily_sales_revenue_kes": 636875.52
}
```

### What I learned

- A saved model must be loaded under the same scikit-learn version it was saved with. My model was saved with 1.9.0 and the classification model with 1.6.1, so one `api.py` could not load both. I re-saved my model under 1.6.1 and pinned the version in `requirements.txt`.
- If the target was transformed during training (`log1p`), the API has to reverse it, otherwise predictions come back on the wrong scale.
- Deployment errors are best read from the server logs, because the API's own error messages are deliberately generic.
- A 405 Method Not Allowed means the route exists but was called with the wrong HTTP method.

---

## Member 2

| Details | Comment |
| ------- | ------- |
| **Student ID** | 168051 |
| **Name** | Joy Mbote |
| **Branch** | [`classification-api`](https://github.com/GraceChelangat/BI-ServingMLModels-C1/tree/classification-api) |

### Contribution

**2. Financial Distress Classification API**

- The API uses a saved machine learning pipeline, loaded from `models/distress_model_pipeline.joblib`, to classify a company's financial condition as Healthy, Watch or Distressed.
- I implemented the classification endpoint (`POST /predict/classification`) with Flask, added input validation and error handling, and wrote automated tests using pytest. All six tests passed.

### What I learned

I learned how to serve a trained machine learning model through a REST API, validate inputs, test endpoints, and manage code using Git.

---

## Member 3

| Details | Comment |
| ------- | ------- |
| **Student ID** | 166567 |
| **Name** | Nicole Wachira |
| **Branch** | [`feature/clustering-endpoint`](https://github.com/GraceChelangat/BI-ServingMLModels-C1/tree/feature/clustering-endpoint) |

### Contribution

**3. E-commerce Customer Clustering API**

- Used the K-Means model our group trained in the clustering lab (`ecommerce_customer_clustering.ipynb`). It was saved as a bundle (`models/ecommerce_clustering_bundle.joblib`) holding the median imputer, StandardScaler, K-Means model (5 clusters), the feature list and the segment names.
- Added the `POST /predict/clustering` endpoint to the shared `api.py`. It loads the saved bundle once at startup and never retrains it.
- It applies the same steps as training (impute, scale, predict) with the four features in training order, and returns the cluster number and its segment name.
- It validates the request and returns a 400 for a non-JSON body, missing features and non-numeric values.
- Tested with Postman and curl: an active customer returned cluster 1 and a dormant customer returned cluster 2, matching the notebook.

**Example request**

```json
{
  "recency_days": 5,
  "frequency_per_year": 28,
  "avg_order_value_kes": 9500,
  "num_categories_purchased": 6
}
```

**Example response**

```json
{
  "predicted_cluster": 1,
  "segment_name": "High-Frequency Big Spenders"
}
```

### What I learned

- New data must go through exactly the same preprocessing and feature order as the training data, otherwise the model still returns a cluster number, but a wrong one, with no error.
- A saved model must be loaded under the same scikit-learn version it was saved with.
- Models are loaded once at startup and never retrained inside the API.
- Clear error messages (400) and documentation matter, because the API is how other people use the model.

---

## Member 4

| Details | Comment |
| ------- | ------- |
| **Student ID** | 160819 |
| **Name** | Alfred Njaku |
| **Branch** | [`feature/association-rules-endpoint`](https://github.com/GraceChelangat/BI-ServingMLModels-C1/tree/feature/association-rules-endpoint) |

### Contribution

**4. Streaming Series Association Rules Recommender API**

- Used the 21 cleaned association rules our group saved in the Association Rule Mining lab (min support 0.02, min confidence 0.30, min lift 2.0) and the show lookup table.
- Wrote `scripts/save_association_bundle.py`, which loads those saved rules (without re-mining them) and saves them with joblib as `models/association_rules_bundle.joblib`.
- Added the `POST /recommend/association` endpoint to the shared `api.py`. It loads the saved bundle once at startup and never re-mines the rules.
- A rule matches when its whole antecedent is in the viewer's basket. Shows already watched are removed, there is one entry per recommended show, and results are ranked by confidence, then lift, then support.
- It returns an empty list with a clear message when no rule matches, and a 400 for bad input.
- Added `tests/test_association.py` (10 pytest tests).

**Example request**

```json
{
  "items": ["Galactic Wars", "Time Anomaly"]
}
```

**Example response (200)**

```json
{
  "input_items": ["Galactic Wars", "Time Anomaly"],
  "recommendations": [
    {
      "show_id": "S06",
      "show_name": "Space Frontier",
      "because_you_watched": ["Galactic Wars", "Time Anomaly"],
      "confidence": 0.7681,
      "lift": 4.844,
      "support": 0.0303
    }
  ]
}
```

### What I learned

- An association rule recommender has no "model" to predict with. The saved rules are the model, so serving it means loading the rules and matching them, not retraining.
- A rule only applies when its whole antecedent is in the basket, and shows the user has already watched should not be recommended back.
- "No recommendation" is a valid answer and returns 200 with a message, while bad input returns 400.
- Keeping the bundle to plain Python objects avoids pandas and scikit-learn version problems.

---

## Member 5

| Details | Comment |
| ------- | ------- |
| **Student ID** | 169302 |
| **Name** | Anastacia Muya |
| **Branch** | [`feature/docker-nginx`](https://github.com/GraceChelangat/BI-ServingMLModels-C1/tree/feature/docker-nginx) |

### Contribution

**5. Docker, Gunicorn and Nginx Deployment**

- Wrote the `Dockerfile` on a `python:3.11-slim` base. It installs `requirements.txt`, copies the project (including `models/`) into the image, and starts the API with Gunicorn: `gunicorn --bind 0.0.0.0:5000 --workers 2 --timeout 120 api:app`. The Flask development server is not used.
- Wrote `nginx/nginx.conf`, a reverse proxy that forwards all requests to the API container (`proxy_pass http://api:5000`).
- Wrote `docker-compose.yml` with two services. `api` is only exposed inside the Docker network, and `nginx` is published on port 8080, so the API can only be reached through Nginx.
- Added a `.dockerignore` that keeps `models/` inside the image.
- Did not change `api.py`. The same `api.py` runs locally, in Docker and on Render.
- Tested all four endpoints through port 8080. Every response carries the `Server: nginx` header.
- Added a simple HTML/CSS/JavaScript demo page (`frontend/index.html`) served by the same Nginx container at `http://localhost:8080/demo/`. It shows an error message when an input is missing.

**How to run**

```
docker compose up --build
docker compose ps
curl -i http://localhost:8080/
docker compose down
```

### What I learned

- Gunicorn is a production server, while `app.run()` is only for development.
- Nginx is a reverse proxy: clients talk to Nginx, and Nginx talks to the API. Inside Docker Compose, containers reach each other by service name.
- `expose` keeps a port private to the Docker network, while `ports` publishes it to the host.
- Saved model files must be inside the image, and a saved model is tied to the library versions it was created with.
- A running container is only proof if it answers real requests, so I tested through port 8080 and checked the `Server: nginx` header.

---

## Member 6

| Details | Comment |
| ------- | ------- |
| **Student ID** | 163604 |
| **Name** | Urvi Patel |
| **Branch** | [`render-deployment`](https://github.com/GraceChelangat/BI-ServingMLModels-C1/tree/render-deployment) |

### Contribution

**6. Render Deployment**

- I deployed the team's unmodified `api.py` to Render as a Web Service, using the Gunicorn start command `gunicorn api:app --bind 0.0.0.0:$PORT` and the build command `pip install -r requirements.txt`.
- Live URL: https://bi-servingmlmodels-c1.onrender.com
- The root link only shows a health message. The four endpoints are POST routes, so test them with curl or Postman, for example: `curl -X POST https://bi-servingmlmodels-c1.onrender.com/predict/clustering -H "Content-Type: application/json" -d '{"recency_days":5,"frequency_per_year":28,"avg_order_value_kes":9500,"num_categories_purchased":6}'`
- I tested all four endpoints from outside my computer with curl: `/predict/regression`, `/predict/classification`, `/predict/clustering` and `/recommend/association`. All four return correct predictions.
- While testing, I found that the clustering and regression code had been mixed up in a merge, and I reported it to the team so it could be fixed.
- I read the Render logs and found that the regression model file had been saved with scikit-learn 1.9.0, while Render uses 1.6.1. I reported it, and the model was saved again with the right version.
- I recorded the local, Docker + Nginx and Render demonstrations, and compiled them together with Noella's clip into one video, trimming it to fit the 10-minute limit.

### What I learned

- The start command must match the file name. `api:app` means file `api.py`, object `app`.
- A saved model only loads properly with the same library versions it was saved with.
- The 500 error message is generic, so the real cause is found in the Render logs.
- The free tier sleeps after about 15 minutes, so I woke it up before recording.
- How to record my screen, then trim and join several clips into one video with Clipchamp.

---

## Chosen Level of Difficulty

**Intermediate** (plus the optional HTML/CSS/JavaScript web page demo)

## Video Demonstration

**Link to the video:** https://drive.google.com/file/d/1h9I9Yt7Fj3i18cP_5lDt0IdRb1hoajH_/view?usp=sharing

Video order: 1) Local (Flask dev server, port 5000), 2) Docker + Nginx (port 8080), 3) Render (bi-servingmlmodels-c1.onrender.com), 4) Web page demo: HTML, CSS and vanilla JavaScript, served through Nginx (optional bonus).

## Public URLs

| Deployment target | URL |
| ----------------- | --- |
| Render (the full `api.py`, all endpoints) | https://bi-servingmlmodels-c1.onrender.com |
| Hugging Face Space (Gradio, one tab per model) | |
| Streamlit Community Cloud app (one tab per model) | |

Render endpoints (POST): `/predict/regression`, `/predict/classification`, `/predict/clustering`, `/recommend/association`. The root link only returns a health message. See the video, Render section.

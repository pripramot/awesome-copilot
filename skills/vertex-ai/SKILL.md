---
name: vertex-ai
description: 'Integrate Google Cloud Vertex AI for enterprise-grade machine learning including model deployment, Vertex AI Studio, PaLM API, custom model training, batch predictions, and MLOps pipelines. Covers both the Python SDK and REST API patterns.'
---

# Vertex AI Integration (ผสานรวม Google Cloud Vertex AI)

Deploy enterprise-grade AI solutions using Google Cloud's Vertex AI platform.

## Setup

```bash
# Install SDK
pip install google-cloud-aiplatform

# Authenticate
gcloud auth application-default login
gcloud config set project YOUR_PROJECT_ID
```

```python
import vertexai
from vertexai.generative_models import GenerativeModel, Part, SafetySetting

# Initialize
vertexai.init(project="your-project-id", location="us-central1")
```

## Generative AI with Gemini on Vertex AI

```python
from vertexai.generative_models import GenerativeModel

model = GenerativeModel("gemini-1.5-flash-001")

# Text generation
response = model.generate_content("อธิบาย Machine Learning ให้เข้าใจง่าย")
print(response.text)

# With safety settings
from vertexai.generative_models import SafetySetting, HarmCategory, HarmBlockThreshold

safety_settings = [
    SafetySetting(
        category=HarmCategory.HARM_CATEGORY_DANGEROUS_CONTENT,
        threshold=HarmBlockThreshold.BLOCK_MEDIUM_AND_ABOVE,
    ),
]

response = model.generate_content(
    "Generate content here",
    safety_settings=safety_settings,
    generation_config={"temperature": 0.7, "max_output_tokens": 1024}
)
```

## Multimodal with Vertex AI

```python
from vertexai.generative_models import GenerativeModel, Part
import base64

model = GenerativeModel("gemini-1.5-flash-001")

# Analyze image from GCS
image_part = Part.from_uri(
    uri="gs://your-bucket/image.jpg",
    mime_type="image/jpeg"
)

response = model.generate_content([
    "Describe this image in Thai",
    image_part
])
print(response.text)

# Analyze video from GCS
video_part = Part.from_uri(
    uri="gs://your-bucket/video.mp4",
    mime_type="video/mp4"
)

response = model.generate_content([
    "Summarize what happens in this video",
    video_part
])
```

## Model Garden - Fine-tuned Models

```python
from google.cloud import aiplatform

# Deploy a model endpoint
def deploy_model(model_name: str, project: str, location: str):
    aiplatform.init(project=project, location=location)

    model = aiplatform.Model(model_name=model_name)

    endpoint = model.deploy(
        deployed_model_display_name="my-deployed-model",
        machine_type="n1-standard-4",
        min_replica_count=1,
        max_replica_count=5,
        accelerator_type="NVIDIA_TESLA_T4",
        accelerator_count=1,
    )
    return endpoint
```

## Batch Predictions

```python
from google.cloud import aiplatform

def run_batch_prediction(
    model_name: str,
    input_gcs_path: str,
    output_gcs_path: str,
    project: str,
    location: str
):
    aiplatform.init(project=project, location=location)

    model = aiplatform.Model(model_name=model_name)

    batch_prediction_job = model.batch_predict(
        job_display_name="batch-prediction-job",
        gcs_source=input_gcs_path,
        gcs_destination_prefix=output_gcs_path,
        instances_format="jsonl",
        predictions_format="jsonl",
        machine_type="n1-standard-4",
        starting_replica_count=1,
        max_replica_count=5,
    )

    batch_prediction_job.wait()
    return batch_prediction_job
```

## Vertex AI Pipelines (MLOps)

```python
from kfp import dsl
from google.cloud import aiplatform
from google.cloud.aiplatform import pipeline_jobs

@dsl.component(
    base_image="python:3.11",
    packages_to_install=["scikit-learn", "pandas", "google-cloud-storage"]
)
def train_model(
    training_data_path: str,
    model_output_path: str,
    accuracy: dsl.Output[float]
):
    """Train a classification model."""
    import pandas as pd
    from sklearn.ensemble import RandomForestClassifier
    import joblib

    df = pd.read_csv(training_data_path)
    X, y = df.drop("label", axis=1), df["label"]

    model = RandomForestClassifier(n_estimators=100, random_state=42)
    model.fit(X, y)

    accuracy.set(model.score(X, y))
    joblib.dump(model, model_output_path)


@dsl.pipeline(
    name="ml-training-pipeline",
    description="Train and evaluate ML model"
)
def training_pipeline(training_data: str, output_dir: str):
    train_task = train_model(
        training_data_path=training_data,
        model_output_path=f"{output_dir}/model.joblib"
    )
```

## Vector Search (Embeddings)

```python
from vertexai.language_models import TextEmbeddingModel

def get_embeddings(texts: list[str]) -> list[list[float]]:
    """Get vector embeddings for semantic search."""
    model = TextEmbeddingModel.from_pretrained("text-embedding-004")
    embeddings = model.get_embeddings(texts)
    return [e.values for e in embeddings]

# Example: Find similar documents
from sklearn.metrics.pairwise import cosine_similarity
import numpy as np

def find_similar(query: str, documents: list[str]) -> list[tuple[str, float]]:
    all_texts = [query] + documents
    embeddings = get_embeddings(all_texts)

    query_emb = np.array([embeddings[0]])
    doc_embs = np.array(embeddings[1:])

    similarities = cosine_similarity(query_emb, doc_embs)[0]
    ranked = sorted(zip(documents, similarities), key=lambda x: x[1], reverse=True)
    return ranked
```

## Cost Optimization Tips

1. **Use Gemini Flash** for most tasks - 10x cheaper than Pro with minimal quality difference
2. **Enable caching** for repeated prompts with same context
3. **Batch requests** when processing large datasets to reduce overhead
4. **Set max_output_tokens** appropriately - don't over-provision
5. **Use regional endpoints** closest to your users to reduce latency
6. **Monitor with Cloud Monitoring** - Set billing alerts before costs escalate

## Environment Variables

```bash
export GOOGLE_CLOUD_PROJECT="your-project-id"
export GOOGLE_CLOUD_LOCATION="us-central1"
export GOOGLE_APPLICATION_CREDENTIALS="/path/to/service-account-key.json"
```

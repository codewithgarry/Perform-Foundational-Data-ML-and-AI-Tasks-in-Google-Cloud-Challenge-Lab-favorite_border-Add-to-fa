# Perform Foundational Data, ML, and AI Tasks in Google Cloud: Challenge Lab (GSP323)

![Google Cloud](https://img.shields.io/badge/Google_Cloud-4285F4?logo=googlecloud&logoColor=white)
![Lab](https://img.shields.io/badge/Lab-GSP323-34A853)
![ML APIs](https://img.shields.io/badge/Topic-ML_APIs-orange)

Reference solution for the Google Cloud challenge lab **Perform Foundational Data, ML, and AI Tasks in Google Cloud** (GSP323).

## 📖 Overview

The lab covers a broad set of data and AI services. This repo documents the **pre-trained ML API** tasks:

| API | Input | Output file |
|---|---|---|
| Cloud Natural Language | A sentence about Odin | `task4-cnl.result` |
| Speech-to-Text | `task4.flac` | `task4-gcs.result` |
| Video Intelligence | `chicago.mp4` | `task4-gvi.result` |

The Dataflow, Dataproc and Dataprep tasks are done in the console following the lab instructions.

## 🧰 Services used

Natural Language API · Speech-to-Text API · Video Intelligence API · Cloud Storage · IAM service accounts · API keys

## ✅ Prerequisites

```bash
export PROJECT_ID=$(gcloud config get-value project)
export MARKING_BUCKET="gs://$PROJECT_ID-marking"
export API_KEY="<api-key-created-in-the-console>"
```

Inspect the lab's input data:

```bash
gsutil cat gs://cloud-training/gsp323/lab.csv | head
gsutil cat gs://cloud-training/gsp323/lab.schema
```

## 🚀 Solution

### Cloud Natural Language: entity analysis

```bash
gcloud iam service-accounts create my-natlang-sa --display-name "Natural Language SA"
gcloud iam service-accounts keys create ~/key.json \
  --iam-account my-natlang-sa@$PROJECT_ID.iam.gserviceaccount.com
gcloud auth activate-service-account --key-file ~/key.json

gcloud ml language analyze-entities \
  --content="Old Norse texts portray Odin as one-eyed and long-bearded, frequently wielding a spear named Gungnir and wearing a cloak and a broad hat." \
  > result.json

gcloud auth login   # switch back to the student account to write to the marking bucket
gsutil cp result.json $MARKING_BUCKET/task4-cnl.result
```

### Speech-to-Text: transcribe a FLAC file

```bash
cat > request.json <<'EOF2'
{
  "config": { "encoding": "FLAC", "languageCode": "en-US" },
  "audio":  { "uri": "gs://cloud-training/gsp323/task4.flac" }
}
EOF2

curl -s -X POST -H "Content-Type: application/json" --data-binary @request.json \
  "https://speech.googleapis.com/v1/speech:recognize?key=$API_KEY" > result.json

gsutil cp result.json $MARKING_BUCKET/task4-gcs.result
```

### Video Intelligence: detect text in a video

```bash
gcloud iam service-accounts create quickstart
gcloud iam service-accounts keys create key.json \
  --iam-account quickstart@$PROJECT_ID.iam.gserviceaccount.com
gcloud auth activate-service-account --key-file key.json
export ACCESS_TOKEN=$(gcloud auth print-access-token)

cat > request.json <<'EOF2'
{
  "inputUri": "gs://spls/gsp154/video/chicago.mp4",
  "features": ["TEXT_DETECTION"]
}
EOF2

# Start the long-running annotation and capture the operation name
OPERATION=$(curl -s -H 'Content-Type: application/json' \
  -H "Authorization: Bearer $ACCESS_TOKEN" \
  'https://videointelligence.googleapis.com/v1/videos:annotate' \
  -d @request.json | python3 -c 'import json,sys; print(json.load(sys.stdin)["name"])')

# Wait a minute, then fetch the result
curl -s -H "Authorization: Bearer $ACCESS_TOKEN" \
  "https://videointelligence.googleapis.com/v1/$OPERATION" > result.json

gcloud auth login
gsutil cp result.json $MARKING_BUCKET/task4-gvi.result
```

---

## ⚠️ Notes

- Resource names, zones, regions and ports are generated per lab session. Always use the values shown in **your** lab instructions.
- These steps were written when the lab was live. Google revises labs over time, so the current version may differ.

## 📌 Disclaimer

This repository is a study reference. Attempt the lab on your own first, then compare approaches. It is not affiliated with or endorsed by Google.

## 👤 Author

**Girish Sharma** · Cloud & DevOps Engineer · [GitHub](https://github.com/codewithgarry) · [LinkedIn](https://www.linkedin.com/in/codewithgarry/)

# PlanktonSee (FastAPI)

A FastAPI web app that detects plankton in an uploaded microscope image with YOLOv8 models, writes a short taxonomy explanation of the detected classes with an LLM, and stores the result in Firebase so it can be opened again through a result link. The interface text is in Indonesian.

## How it works

1. The user uploads an image on `/predict_ui` and picks a detection model and an LLM.
2. `POST /predict_action` stores the original image in Firebase Storage, runs the selected YOLOv8 model, and uploads the annotated image and the LLM text.
3. The prediction (image URLs, detected classes, confidences) is saved in Firestore, and the API returns a `doc_id`.
4. `/result/{doc_id}` renders the annotated image, the classes with their confidence scores, and the taxonomy notes.

**Detection models** (weights are included in `model/`): `yolov8-detect` and `yolov8-acvit`.
**LLM options** for the taxonomy notes: Gemini (`gemini-1.5-flash-002`), Qwen2 and DeepSeek (the last two are called through public Hugging Face Spaces with `gradio_client`).

## Tech stack

- Python 3.12, FastAPI, Uvicorn, Jinja2 templates
- YOLOv8 through a bundled `ultralytics/` package, PyTorch, OpenCV
- Firebase Admin SDK: Cloud Storage and Firestore
- Google Generative AI SDK, `gradio_client`
- Docker, Railway

## Project structure

```
.
├── main.py                 # FastAPI app and routes
├── utils.py                # image reading, YOLO inference, LLM calls, Firebase uploads
├── model/                  # yolov8-detect.pt, yolov8-acvit.pt
├── templates/              # start, predict and result pages
├── all/                    # static assets (CSS, JS) and the temporary upload folder
├── ultralytics/            # bundled YOLO package
├── plankton_predict.ipynb  # notebook for exploring the models
├── sample_image.jpg        # example input
├── Dockerfile              # python:3.12-slim image, serves on port 8000
└── railway.json            # Railway deployment settings
```

## Running it locally

You need a Firebase project with a Storage bucket and a Firestore database, plus a Google AI API key.

```sh
python -m venv venv
venv\Scripts\activate          # Windows
source venv/bin/activate       # macOS / Linux
pip install -r requirements.txt
```

1. Put your Firebase service-account file at `credential/credentials.json`. The folder is git-ignored.
2. Create a `.env` file in the project root:

   ```
   GOOGLE_API_KEY=your_google_ai_api_key
   ```

3. `main.py` refers to the bucket `planktosee-temp-file` and the Firestore database `planktonsee-database`. Change both names to your own before running.
4. Start the server and open http://localhost:8000/

   ```sh
   uvicorn main:app --host 0.0.0.0 --port 8000
   ```

### With Docker

```sh
docker build -t planktonsee-fastapi .
docker run -p 8000:8000 --env-file .env -v ./credential:/app/credential planktonsee-fastapi
```

## Deployment

`railway.json` and the `Dockerfile` start the app with `uvicorn main:app` on port 8000. Provide the credential file and `GOOGLE_API_KEY` as environment or mounted files on the platform you use.

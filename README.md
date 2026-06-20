name: Build and Deploy to Google Cloud Run

on:
  push:
    branches:
      - main
  workflow_dispatch:

env:
  PROJECT_ID: ${{ secrets.GCP_PROJECT_ID }}
  REGION: us-central1
  SERVICE_NAME: grabit-api
  IMAGE_NAME: grabit-api

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: google-github-actions/setup-gcloud@v1
        with:
          project_id: ${{ secrets.GCP_PROJECT_ID }}
          service_account_key: ${{ secrets.GCP_SA_KEY }}
          export_default_credentials: true
      
      - run: gcloud auth configure-docker us-central1-docker.pkg.dev
      
      - run: |
          docker build -t us-central1-docker.pkg.dev/$PROJECT_ID/docker-repo/$IMAGE_NAME:latest .
      
      - run: |
          docker push us-central1-docker.pkg.dev/$PROJECT_ID/docker-repo/$IMAGE_NAME:latest
      
      - run: |
          gcloud run deploy $SERVICE_NAME \
            --image us-central1-docker.pkg.dev/$PROJECT_ID/docker-repo/$IMAGE_NAME:latest \
            --region $REGION \
            --platform managed \
            --allow-unauthenticated \
            --memory 512Mi \
            --cpu 1 \
            --timeout 120

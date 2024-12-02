### Deploy Places Service API
1. In the Google Cloud console, type in "Places API", enable the API, and store your API key somewhere safe to use later
2. Give your compute service account the following roles:
  * Artifact Registry Repository Administrator
  * Cloud Build Service Account
  * Logs Writer
  * Storage Object Viewer
3. Deploy the service to Cloud Run
    ```
    gcloud run deploy places-api-service --source . \
    --region us-central1 \
    --allow-unauthenticated \
    --set-env-vars GOOGLE_PLACES_API_KEY=<your_api_key>
    ```
### Tutorial
Follow the steps [here](agent-builder-for-all.ipynb) for the main tutorial.

### Credit
* Patrick Marlow for creating the Places service and the steps for the Agent Builder part of agent-builder-for-all.ipynb

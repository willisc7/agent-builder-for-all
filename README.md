## Tutorial
This tutorial will show you how Vertex AI supports no code, low code, and medium-high code Gen AI Agent use cases. It will be broken into three parts:
* Part 1: A no code way to create a data store tool that can be used to perform Retrieval Augmented Generation
* Part 2: A low code way to create a conversational agent using Agent Builder
* Part 3: A code-first way to create a workflow agent

### Pre-requisites
#### Deploy Places Service API
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
4. Test the service
    ```
    curl -X POST \
      -H "Content-Type: application/json" \
      -d '{"preferences": "Tesla Service Center","city": "San Diego"}' https://<CLOUD_RUN_URI>/tourist_attractions
    ```

### Part 1: No Code Data Store Tool
**Pros**
* Managed runtime
* UI based
* No code involved in making the tool

**Cons**
* Less control due to managed runtime (still fits most conversational use cases and some non-conversational use cases)

#### Steps
1. Create a Cloud Storage bucket and upload the [Cymbal Starlight PDF](cymbal-starlight-2024.pdf)
2. In the Google Cloud console, go to Agent Builder
3. Create a new app
4. Under `Search for your website` click on **Create**
5. Fill in `App name` and and `Company name` and click **CONTINUE**
6. Click **CREATE DATA STORE**
7. Under `Cloud Storage` click **SELECT**
8. Under `Select a folder or a file you want to import` click **BROWSE** and upload the PDF and click **CONTINUE**
9. Name your datastore `car_manuals` and click **CREATE**
10. Click **CREATE** again
11. Click on the **Activity** tab and wait until the import process completes
12. Click on **Preview** and search `How do I use cruise control?`
13. Click on **Integration** and the **API** tab
14. Under **Search query**, enter `How do I use cruise control?` and click **RUN IN CLOUD SHELL**. Notice how it gives you the same output.
15. Check out the other customizations you can use when calling the API [here](https://cloud.google.com/generative-ai-app-builder/docs/preview-search-results?authuser=3). For example, you can use `RESULT_MODE` to return chunks of text instead of documents. This could be useful to an agent that is trying to reason about how to handle the returned results. 

### Part 2: Low Code Conversational Agent
**Pros** 
* Managed runtime
* UI based

**Cons**
* Less control due to managed runtime (still fits most conversational use cases and some non-conversational use cases)
* You need to make the tool (although this might be inevitable depending on your use case)

#### Steps
1. In the Google Cloud console, go to Agent Builder
3. Create a new app
4. Under `Conversation agents > Conversational agent` click on **Create** (dont select the Chat option)
5. Click **Build Your Own**
6. Type in "Tesla Agent" and click **Create**
7. Click on **Toggle Simulator** in the top right, select Gemini Flash as the model, and ask "What do you know about the Tesla Model X?" It should respond with whatever knowledge the base model has on the Model X.
8. Lets improve the conversational agent a little. Configure the following under **Goal** in the lefthand pane:
    ```
    You are a friendly Tesla service center agent.
    Your job is to answer questions about Tesla products.
    ```
9. Under **Instructions** put the following:
    ```
    - Greet the user and answer their questions to the best of your knowledge
    - If the user asks about a Tesla service center location, do not provide information unless you are certain that you know the address.
    ```
10. Now chat with the bot again and say "Where can I test drive a Model X in San Diego?" an it will say it doesnt have the information.
11. We need a tool to be able to get Tesla Service Center locations. So click on **Tools** on the lefthand pane
12. Click **+ Create**
13. Under **Tool name** type in `places_search_tool`
14. Under **Description** type in
    ```
    A Places Search API that provides information on points of interest a city.
    ```
15. Under Schema, make sure its set to YAML and paste in the contents of [places_search_tool_spec.yaml](places_search_tool_spec.yaml) (make sure to change the url to point to your cloud run service)
16. Click **Save**
17. Go back to the agent playbook and change the instructions to:
    ```
    - Greet the user and answer their questions to the best of your knowledge
    - If the user asks about a Tesla service center location, use the ${TOOL: places_search} to answer their question.
    ```
18. Click **Save**, restart the conversation, and ask "Tell me about the new Model X" and then say "Where can I test drive a Model X in San Diego?"
19. Click on **Default Playbook** under Agent Invocations in the middle of the screen. It will select the full conversation. Click **Save example** and name it `example_1`
20. Change the format of the conversation to something you prefer. Make sure you copy over the response body for the places_search_tool and click **Create**
21. Go back to the conversation and reset it and ask "Where can I test drive a model x in San Diego?" and youll notice it will use the formatting you trained it to use.
22. Follow up with "What about Austin, TX?" and itll use the context of the conversation to supply a similar answer

### Part 3: Medium-High Code Agent
**Pros** 
* Full control as a developer

**Cons**
* Steep learning curve
* No managed runtime (you need to figure out how to deploy and host)

#### Steps
[This notebook](https://www.kaggle.com/code/markishere/day-3-building-an-agent-with-langgraph) provides a fantastic overview of how to build a conversational agent using LangGraph. In case this link becomes unavailable later, I've exported the notebook to [building-an-agent-with-langgraph.ipynb](building-an-agent-with-langgraph.ipynb)

### Credit
* Patrick Marlow for creating the Places service and the steps for the Agent Builder part of agent-builder-for-all.ipynb

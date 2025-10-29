**This is some rough instructions and code for a stateful BigQuery continuous queries (performing JOINs, aggregations, and windowing) demo with integration with Agent Developer Kit (ADK).**

**Demo video:** https://drive.google.com/file/d/1RQVHdBL_XndFGbqKT5JOq_5cCLixdCOW/view?usp=sharing

**NOTE:** BigQuery continuous queries doesn't yet support stateful operations outside of a **VERY LIMITED** set of allowlisted projects. Public preview for this functionality is targeted for Q4-25 or Q1-26. 

**Rough instructions:**
1. Create a BigQuery dataset named Cymbal_Cyber
2. Create a BigQuery table named `user_access_events` for where the raw network access logs will be streamed to. [DDL to create the table](https://paste.googleplex.com/5487041932558336)
3. Create a BigQuery table named `network_events` for where the network connection and firewall access logs will be streamed to. [DDL to create the table](https://paste.googleplex.com/4669948563685376)
4. Create a BigQuery table named `false_positives` for the logged false positives from ADK. [DDL](https://paste.googleplex.com/4790561718140928)
5. Create a BigQuery table named `escalations` for the logged escalations from ADK. [DDL](https://paste.googleplex.com/6624169516859392)
6. Create a BigQuery remote connection for the object tables named continuous-query-vertex-ai-connection
   - Add some example screenshots, a screenshot for each individual user in the benign and malicious notebooks. The file names are the user names.
7. Create a BigQuery objects table named screenshots_object_table. [DDL](https://paste.googleplex.com/4785895127121920)
8. Create a BigQuery view on top of the object table named user_screenshots_view. [DDL](https://paste.googleplex.com/6684533134721024)
   - Query this view to make sure it all works by supplying a user name
9. Create a Service Account which will be leveraged to basically orchestrate the full demo. You'll use this throughout the demo. I haven't verified permissions but they should include the following roles
   - BigQuery Connection User
   - BigQuery Data Editor
   - BigQuery Data Viewer
   - BigQuery User
   - Cloud Run Invoker
   - Logs Writer
   - Pub/Sub Publisher
   - Pub/Sub Viewer
   - Service Account Token Creator
   - Service Usage Consumer
   - Storage Object Admin
   - Vertex AI User
10. Create a BigQuery Colab notebook for the benign events generator. The code is [HERE](https://paste.googleplex.com/5865336914182144)    
11. Create a BigQuery Colab notebook for the malicious events generator. The code is [HERE](https://paste.googleplex.com/5930999984816128)
12. Run both notebooks independently and ensure both the user_access_events and firewall_events tables in BQ are receiving the traffic 
13. Create the Pub/Sub topic cymbal_cyber_dest_topic to receive the output from your BigQuery continuous query
14. Create and save a BigQuery continuous query. Code is [HERE](https://paste.googleplex.com/6423058818662400)
    - Supply the service account you created earlier
    - Run the continuous query and ensure it works by manually pulling events from the Pub/Sub topic cymbal_cyber_dest_topic
15. Create a few GCS buckets:
  -  cymbal_cyber_adk_staging_bucket for the Cloud Run staging files
  -  cymbal-cyber-adk-bucket for the output of the ADK
  -  cymbal-cyber-screenshots for the screenshot images used in the multi-modal piece of the demo
16. Enable the IAM Credentials API to generate signed URLs gcloud services enable iamcredentials.googleapis.com
17. Copy the full [agent_code](https://github.com/norlove/BigQuery-Continuous-Queries-ADK-Network-Security-Demo/tree/main/agent_code) directory to your developer environment (I used Google Cloud Shell)
18. Within your directory, run the following command to deploy a Cloud Run function:
   ```
   gcloud run deploy cymbal-cyber-trigger \
     --source . \
     --project="nickorlove-demos" \
     --region="us-central1" \
     --service-account="bigquery-sa@nickorlove-demos.iam.gserviceaccount.com" \
     --no-allow-unauthenticated \
     --no-cpu-throttling
   ```
19. Copy the resulting URL looking something like https://cymbal-cyber-trigger-<project_number>.us-central1.run.app
20. Under the Pub/Sub topic cymbal_cyber_dest_topic, create a push Pub/Sub subscription named alerts-to-adk-agent-push to write these events to a Cloud Run function (with the URL above)
21. Within your directory, run the command ```python deploy_agent_script.py``` to deploy your agentic framework into the Google Cloud Agent Engine ecosystem
22. Copy the resulting Agent ID that is printed from the above command. It will be something like "projects/271115234308/locations/us-central1/reasoningEngines/5196160011474042880"
23. Open your pubsub_trigger.py file and paste this into line 28 in that file by overwriting what is there already.
24. Redeploy your Cloud Run app with this new link to your Agent by again running:
   ```
   gcloud run deploy cymbal-cyber-trigger \
     --source . \
     --project="nickorlove-demos" \
     --region="us-central1" \
     --service-account="bigquery-sa@nickorlove-demos.iam.gserviceaccount.com" \
     --no-allow-unauthenticated \
     --no-cpu-throttling
   ```
25. Your agent should now be running and ready to start processing incoming messages.
26. In a separate CLI tab, run the following to start up streamlit:
   ```streamlit run streamlit_app.py --server.enableCORS=false```
27. You can run the run_local script by running the command ```./run_local.sh``` and it will process the malicious sample JSON.
28. Open the URL provided with the specific port
29. You should see the test event arrive in Streamlit
30. Start up the continuous query, start the benign notebook (it will run until cancelled or the runtime ends), when ready initiate the malicious generator






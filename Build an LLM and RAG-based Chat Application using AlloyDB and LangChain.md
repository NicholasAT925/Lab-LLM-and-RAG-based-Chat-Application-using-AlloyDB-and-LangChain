---
Category: AI Programming
Sub-Category: 
tags:
  - AI_Programming
  - Incomplete
  - Large_Language_Models_LLMs
  - Retrieval_Augmented_Generation_RAG
  - AlloyDB
  - LangChain
Created At: 2025-05-24
Last Updated: 2025-05-24T22:46
Source Links:
  - https://www.cloudskillsboost.google/focuses/93570?catalog_rank=%7B"rank"%3A1%2C"num_filters"%3A0%2C"has_search"%3Atrue%7D&parent=catalog&search_id=46095693
---
# <div class="divider">AI Programming</div>
# Lab Source
[Source](https://www.cloudskillsboost.google/focuses/93570?catalog_rank=%7B"rank"%3A1%2C"num_filters"%3A0%2C"has_search"%3Atrue%7D&parent=catalog&search_id=46095693)
# PDF
![[Build an LLM and RAG-based Chat Application using AlloyDB and LangChain _ Google Cloud Skills Boost.pdf]]

# Pic
![[IMG-20250524105549714.png]]![[IMG-20250524105603844.png]]



# What is AlloyDB
[[AlloyDB Content Page]]
# LangChain
[[LangChain]]
# PostgreSQL
[[PostgreSQL]]
# Quick Look
[[Quick Look]]
# 1. Install PostgreSQL
***Insatll***
```bash
sudo apt-get update
sudo apt-get install --yes postgresql-client
```
- `sudo`: Runs the command with **administrator (root)** privileges.
- `apt-get update`: Updates your system’s **package list** from all configured sources (like Ubuntu's software repositories).
- ✅ This ensures you're installing the **latest available version** of any package.
- `apt-get install`: Tells your system to **install a package**.
- `--yes`: Automatically answers "yes" to any prompts (so you don’t have to manually confirm).
- `postgresql-client`: The name of the package — this installs the **PostgreSQL client**, which gives you tools like:
    - `psql`: The PostgreSQL command-line tool
    - Useful for connecting to and managing remote PostgreSQL databases (e.g. AlloyDB or local PostgreSQL servers)
## Connect to Instance
*How to **connect to an AlloyDB instance** on Google Cloud from a **Compute Engine VM** using the `psql` command-line tool (PostgreSQL client).

You want to connect to your AlloyDB (PostgreSQL-compatible database) from a VM using `psql`.

### Step-by-step Breakdown
```shell
# Step 1
export PGPASSWORD=PG_PASSWORD
# Step 2
export PROJECT_ID=$(gcloud config get-value project)
export REGION=REGION
export ADBCLUSTER=CLUSTER
export INSTANCE_IP=$(gcloud alloydb instances describe $ADBCLUSTER-pr --cluster=$ADBCLUSTER --region=$REGION --format="value(ipAddress)")
# Step 3
psql "host=$INSTANCE_IP user=postgres sslmode=require" 
```
#### Step 1
- Sets your PostgreSQL password in an environment variable called `PGPASSWORD`
- `psql` will use this variable to authenticate when you connect.
- Replace `PG_PASSWORD` with your actual password (not literally that string).
#### Step 2
- Automatically gets the current Google Cloud project you're working in.
- You must replace `REGION` with the actual region where your AlloyDB cluster is deployed (e.g., `us-central1`).
- Replace `CLUSTER` with the name of your AlloyDB cluster.
- This command gets the **IP address** of your **primary AlloyDB instance**.
- It stores that IP into a variable called `INSTANCE_IP`.
#### Step 3
- Uses the IP address from above.
- Connects as user `postgres`.
- Uses `sslmode=require` for secure connection.
- Since you already set the password with `PGPASSWORD`, `psql` won't ask for it again.
# 2. Initialize the database
Use your client VM as a platform to populate the database with data and host your application. The first step is to create a database and populate it with data. (**Using PostgreSQ DB***)
## 2.1 Create Data Base
```shell
psql "host=$INSTANCE_IP user=postgres" -c "CREATE DATABASE assistantdemo"  
```
- `$INSTANCE_IP` taken and set in the [[#Step-by-step Breakdown|step above]] . It May differ or not apply in real world.
- `assistantdemo` is the DB name
## 2.2 Vector Extension
The `vector` extension allows PostgreSQL to store and query **vector embeddings**, enabling operations like:
- **Similarity search** (e.g. `cosine`, `dot-product`, `euclidean`)
- **KNN (k-nearest neighbors)** indexing
- **Efficient AI/ML integration** (useful in RAG systems, recommendation engines, etc.)
It’s used in tools like [pgvector](https://github.com/pgvector/pgvector), and often part of GenAI + RAG systems built on AlloyDB or PostgreSQL.

```shell
psql "host=$INSTANCE_IP user=postgres dbname=assistantdemo" -c "CREATE EXTENSION vector"   
```
- `host=$INSTANCE_IP`: 
	- Connects to the database server at the given IP address (stored in the `INSTANCE_IP` environment variable)
- `user=postgres`: 
	- Logs in as the `postgres` database user
- `dbname=assistantdemo`: 
	- Connects to the database named `assistantdemo`
- `-c`: 
	- Tells `psql` to **run a single SQL command**
- `"CREATE EXTENSION vector"`: 
	- Runs SQL to **install the `vector` extension** in the current database

# 3. Populate Database
In this lab we first clone GitHub repo presumably it has all the dm data we need.
```shell
git clone https://github.com/GoogleCloudPlatform/genai-databases-retrieval-app.git
```

## 3.1 Prepare configuration file
```shell
cd genai-databases-retrieval-app/retrieval_service
cp example-config.yml config.yml
sed -i s/127.0.0.1/$INSTANCE_IP/g config.yml
sed -i s/my-password/$PGPASSWORD/g config.yml
sed -i s/my_database/assistantdemo/g config.yml
sed -i s/my-user/postgres/g config.yml
cat config.yml
```

You're:
1. Entering the project folder
2. Copying a config template
3. Replacing default placeholders with **actual values**
4. Verifying the result

### Line 1

```shell
cd genai-databases-retrieval-app/retrieval_service
```
- Changes your working directory to the **retrieval service** subfolder inside your app project.
- This is where the `config.yml` lives and where the service code runs.

---
### Line 2
```shell
cp example-config.yml config.yml
```
- Copies the example config file to a new one called `config.yml`.
- This is the actual file your app will use at runtime.

---
### Line 3
```shell
sed -i s/127.0.0.1/$INSTANCE_IP/g config.yml
```
- Replaces all instances of `127.0.0.1` (localhost) in `config.yml` with the IP address of your AlloyDB instance.
- `$INSTANCE_IP` should already be set using `gcloud`.

---

### Line 4
```shell
sed -i s/my-password/$PGPASSWORD/g config.yml
```
- Replaces `my-password` with the value stored in `$PGPASSWORD` (your database password).
- Ensures the app can authenticate to the database.

---
### Line 5
```shell
sed -i s/my_database/assistantdemo/g config.yml
```
- Replaces `my_database` with your actual database name, in this case: `assistantdemo`.

---
### Line 6
```shell
sed -i s/my-user/postgres/g config.yml
```
- Replaces `my-user` with `postgres` (the database user you're connecting with).
---
### Line 7
```shell
cat config.yml
```
- Prints the final `config.yml` to the terminal so you can review and confirm all replacements were successful.

### The End Result
```shell
host: 0.0.0.0
# port: 8080
datastore:
  # Example for AlloyDB
  kind: "postgres"
  host: 10.65.0.2
  # port: 5432
  database: "assistantdemo"
  user: "postgres"
  password: "P9..."
```

## 3.2 Populate database with the sample dataset
In the `retrieval_service` directory. They install Python dependencies and run a script to populate your PostgreSQL-compatible database (e.g. AlloyDB) with sample data.

```shell
pip install -r requirements.txt
python run_database_init.py
```

It will look like this:
```shell
student@instance-1:~/genai-databases-retrieval-app/retrieval_service$ pip install -r requirements.txt
python run_database_init.py
Collecting asyncpg==0.28.0 (from -r requirements.txt (line 1))
  Obtaining dependency information for asyncpg==0.28.0 from https://files.pythonhosted.org/packages/77/a4/88069f7935b14c58534442a57be3299179eb46aace2d3c8716be199ff6a6/asyncpg-0.28.0-cp311-cp311-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata
  Downloading asyncpg-0.28.0-cp311-cp311-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (4.3 kB)
Collecting fastapi==0.101.1 (from -r requirements.txt (line 2))
...
database init done.
student@instance-1:~/genai-databases-retrieval-app/retrieval_service$
```
# 4. Deploy the Retrieval Service to Cloud Run by Creating Service Account
The retrieval service connects to the database and responds to AI application queries. Before deploying it to Cloud Run, you need to create a **dedicated service account** with the proper permissions.

In a new shell 
```shell
export PROJECT_ID=$(gcloud config get-value project)
gcloud iam service-accounts create retrieval-identity
gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:retrieval-identity@$PROJECT_ID.iam.gserviceaccount.com" \
  --role="roles/aiplatform.user"
```
- `gcloud iam service-accounts create retrieval-identity`
	- Creates a new identity for Cloud Run
- `gcloud projects add-iam-policy-binding`
	- Grants `aiplatform.user` permission to the service account
- `roles/aiplatform.user`:  
	- Grants access to AI Platform resources — this is necessary if your retrieval service is interacting with Vertex AI or other ML services.
- 
# 5. Deploy the Retrieval Service
```shell
cd ~/genai-databases-retrieval-app
gcloud alpha run deploy retrieval-service \
    --source=./retrieval_service/\
    --no-allow-unauthenticated \
    --service-account retrieval-identity \
    --region us-central1 \
    --network=default \
    --quiet
```
This command deploys the `retrieval-service` to **Cloud Run**, making it accessible through a private endpoint while securely connecting to your AlloyDB instance via the **`retrieval-identity`** service account.
## Step 1: Navigate to project root
```shell
cd ~/genai-databases-retrieval-app
```
- Changes directory to your app’s main folder. 

---

## Step 2: Deploy using `gcloud`

```shell
gcloud alpha run deploy retrieval-service \
    --source=./retrieval_service/\
    --no-allow-unauthenticated \
    --service-account retrieval-identity \
    --region us-central1 \
    --network=default \
    --quiet
```

|Flag|Purpose|
|---|---|
|`--source=./retrieval_service/`|Specifies the source code folder for the service|
|`--no-allow-unauthenticated`|Requires authenticated access (prevents public access)|
|`--service-account retrieval-identity`|Uses the service account with IAM permissions for AlloyDB access|
|`--region us-central1`|Specifies the deployment region (must match AlloyDB and other resources)|
|`--network=default`|Ensures the service runs on the default VPC network|
|`--quiet`|Suppresses confirmation prompts|
## 5.1 Verify The Service

Now we can check if the service runs correctly and the VM has access to the endpoint. We use gcloud utility to get the retrieval service endpoint. Alternatively you can check it in the cloud console and replace in the curl command the "$(gcloud run services list –filter="(retrieval-service)" by the value from there.
```shell
curl -H "Authorization: Bearer $(gcloud auth print-identity-token)" $(gcloud  run services list --filter="(retrieval-service)" --format="value(URL)")
```

***Output***
```shell
student@instance-1:~/genai-databases-retrieval-app$ curl -H "Authorization: Bearer $(gcloud auth print-identity-token)" $(gcloud  run services list --filter="(retrieval-service)" --format="value(URL)")
{"message":"Hello World"}student@instance-1:~/genai-databases-retrieval-app$
```
# 6. Deploy sample application
Once your retrieval service is deployed, the next step is to run a **sample AI application** that interacts with it. This application can run on the same **Compute Engine VM**, and it simulates the user-facing interface of your RAG pipeline.
## 6.1 Prep the Environment
### Navigate to App Directory
```shell
cd ~/genai-databases-retrieval-app/llm_demo
```
### Install Dependencies
```shell
pip install -r requirements.txt
```
This command installs all required Python packages listed in the `requirements.txt` file.
The `-r` flag in `pip install` stands for: **"read from requirements file"**

## 6.2 Run Assistant application
Before running the **LLM Assistant application**, you need to define an environment variable named `BASE_URL`. This tells the app where to find your deployed **retrieval service**.
```shell
export BASE_URL=$(gcloud run services list \
  --filter="(retrieval-service)" \
  --format="value(URL)")
```

| Part                             | Description                                                      |
| -------------------------------- | ---------------------------------------------------------------- |
| `gcloud run services list`       | Lists all services deployed to Cloud Run                         |
| `--filter="(retrieval-service)"` | Narrows down to just the retrieval service                       |
| `--format="value(URL)"`          | Extracts only the public (internal) URL of that service          |
| `export BASE_URL=...`            | Saves the resulting URL into the `BASE_URL` environment variable |
## 6.3 Prepare Client ID
Refer to Source Link if image breaks.

To use more advanced capabilities of the application like booking and changing flights we need to sign-in to the application using our Google account and for that purpose we need to provide CLIENT_ID environment variable using the OAuth client ID from the Prepare Client ID chapter:

To use booking functionality of the application we need to prepare OAuth 2.0 Client ID using Cloud Console. It will be when we sign into the application since booking is using clients credentials to record the booking data in the database.

1. In the Cloud Console go to the **APIs and Services** and click on "OAuth consent screen" and click **Get Started**.
2. Then follow on the next screen.

![OAuth consent screen app information next page](https://cdn.qwiklabs.com/EYH%2FEoMRZGlDtEsIyjBmpet1Ijhy7oXk8pXO1L7dxq0%3D)

3. You need to fill out required fields such as "App name" and "User support email". Select **Internal** for "Audience" and finally the "Contact information".

![OAuth consent screen app information](https://cdn.qwiklabs.com/uEAbA%2F7deFdY7%2BoHDIwCD9DruJk%2BudJkaXB3VFRfYA0%3D) ![OAuth consent screen app information](https://cdn.qwiklabs.com/8r7xKUjBQ2fzagQvn9T6cPTPnagOeypql6iZpgpoMKc%3D)

4. Agree to the user data policy. Click **Continue** and then click **Create** at the bottom of the page and it will lead you to the next page.
5. The next step is to create the **client ID**. On the left panel you click "Clients" which lead you to the credentials for OAuth2.

![OAuth consent screen app information](https://cdn.qwiklabs.com/X%2FEvaVli%2BnCAF7%2BBxtBdD2KW38SjugZ1cvcXg7gNF9A%3D)

6. Here you click "Create client" at the top. Then it will open another screen.
![OAuth consent screen app information](https://cdn.qwiklabs.com/I4VKdInLkiZqnmRXIUPX7G3thTz%2B4eV1U7EBOWxDIhE%3D)

7. Pick up "Web application" from the dropdown list for application type and put your application URI (and port - optionally) as the "Authorized JavaScript origins". And you need to add to the "Authorized redirect URIs" your application host with "/login/google" at the end to be able to use the authorization popup screen.
8. After pushing the "Create" button.

We will need the Client ID (and optionally Client secret) later to use with your application.

```shell
export CLIENT_ID=450....apps.googleusercontent.com
```
>[!note] 
>*Note:** Replace the CLIENT_ID value with your Client Id that you just created.

## 6.4 Now you can run the application
```shell
python run_app.py
```


# Biohackathon-24-Basic-RAG-Vercel

This repo covers all updates done after following tutorial in https://sdk.vercel.ai/docs/guides/rag-chatbot , including the debugging.

Here we use Ubuntu 20.04.6 LTS server.

1. First of all, install the starter code from the repository : https://github.com/vercel/ai-sdk-rag-starter

```
git clone https://github.com/vercel/ai-sdk-rag-starter
cd ai-sdk-rag-starter
pnpm install
```

2. Replace the lib and app folder with the ones in this repo, also upload the .env file inside ai-sdk-rag-starter folder. 

3. Setup the postgresql database (here we install the database on the local Ubuntu server)
```
sudo apt update
sudo apt install postgresql
sudo -u postgres psql
```

Within the psql :
```
ALTER USER postgres PASSWORD 'postgres'
CREATE EXTENSION IF NOT EXISTS vector;

\quit
```
The postgres installed is version 12.
4. Additionally, we also need some dev packages related to this version in order to install pgvector.

```
sudo apt install postgresql-server-dev-12
```

5. Next, we need to install pgvector add-in. Source : https://github.com/pgvector/pgvector

```
cd /tmp
git clone --branch v0.7.4 https://github.com/pgvector/pgvector.git
cd pgvector
make # may need sudo
make install # may need sudo
```

6. Then, we do the database migration
```
pnpm db:migrate
```
It will create 2 tables within postgres database : embeddings and resources.

(optional) Create a new database and change the database name in the .env file if we wish to change the database to store these tables.

(optional) If further changes to lib/db/schema/embeddings.ts done after this step, do this to push the changes to the database
```
pnpm db:push
```

7. Now, we can start the dev server by running this :

```
pnpm run dev
```
It will refer to open http://localhost:3000.


8. We can also check the entries of the information we give to the chatbot : (When we say something like "my favorite food is pizza", the chatbot will put that information automatically to the database)
```
pnpm db:studio
```
It will refer to open https://local.drizzle.studio (it will detect the local port respective to the database in our server, so it needs to be opened within the server)


Hack : We utilized MobaXterm to SSH to the server, also to open browser from within the server (firefox is preinstalled), so that we can open http://localhost:3000 and https://local.drizzle.studio and check things around. 

The Drizzle UI looks like this :
![image](https://github.com/user-attachments/assets/4a1d27ff-a565-46ae-9333-7703897636eb)

And the simple RAG UI looks like this :
![image](https://github.com/user-attachments/assets/2f2ac65f-8a26-4c98-8623-97e8e2fba393)

As a note, we didn't put the "favorite drink" information to the chatbot, therefore it said "sorry, I don't know", which is something we desire from a RAG : only answer based on information we provide.  


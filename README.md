# NO REAL BRAIN, WILL BE ARTIFICIAL

**Bot alias:** [@NRBWBA_bot](https://t.me/NRBWBA_bot)

NRBWBA team's bot is an a tool for working with a Agnia AI that allows to interact with different services inside a Telegram

Supporting services:
- 🗒️ [Notion](#example-of-interaction-with-notion)
- 🌐 [TeamFlame](https://teamflame.ru/)
- 🔥 [GitFlame](https://gitflame.ru/)

## Installation

### Install pip dependencies

```pip -m venv .venv```

```source .venv/bin/activate```

```pip install -r requirements.txt```

Then create `.env` file in the root directory following the format: 
```
TEAM_ID=<Token1> 
ACCESS_TOKEN=<Token2>
MILVUS_ENDPOINT=http://localhost:19530
```

Also u need to start docker container with milvus
```commandline
wget https://github.com/milvus-io/milvus/releases/download/v2.4.5/milvus-standalone-docker-compose.yml -O docker-compose.yml
sudo docker compose up -d
```

Create integration api token in Notion.
Follow the https://www.notion.so/profile/integrations
 .> New integration > Give name >Select Associated workspace > Type : 
Internal > next copy the api token.
Then you should authorize using swagger to save notion api token 
`http://127.0.0.1:8845/docs#/`
and then start socket listener 
`python /src/actions_socket_listener.py`

Finally, you are ready to work
## Quick start

For interact with a bot follow the steps:
1. Open telegram and find @NRBWBA_bot or follow the [link](https://t.me/NRBWBA_bot)
2. Click `Start` or run `/start` command for activation
3. To use bots services follow its individual guides: Notion, TeamFlame, GitFlame

## Example of interaction with [Notion](https://www.notion.so/product)

### 1. Create a note

User send a message to the bot in the format:

```
Create a note {note_name} {note_content}
```

If there is no title for the note, but there is content, the title will be generated based on the text of the note. Also the approximate content for the note will be generated based on it's title if content is absent.

![create](assets/create.png)

### 2. Delete a note

User send a message to the bot in the format:


```
Delete a note {key_words_of_the_name_of_note}
```

Further embedder finds the desired note by key words of it's name, deletes it and shows title of deleted note.

![delete](assets/delete.png)

### 3. Search for a note

User send a message to the bot in the format:


```
Find me a note {key_words_of_the_name_of_note}
```

Further embedder finds the desired note by key words of it's name. Then note's id and content are given to the user in bot.

![find](assets/find.png)

### 4. Show all notes

User send a message to the bot in the format:


```
Show me all notes
```

Than all notes outputs to the bot. 

![show](assets/show.png)

### 5. Find answer to question in notes
User send a message to the bot in the format:

```
{question}
```

RAG system analyse all notes and gives context (the most useful information for answering this question). Then context and user request goes to AI module and then to LLM which gives answer to the question if it can be found in context or answers "no information" otherwise.

### 6. Summarize the note
User send a message to the bot in the format:


```
Summarize the text of the {note_name}
```

Further embedder finds the desired note by key words of it's name. Then the content of this note goes into AI module and LLM that shortens the note and gives the short form to the user in bot. **(In progress)**

## Developers

This project is developed by the NRBWBA team:

- [**Remal Gareev**](https://gitlab.pg.innopolis.university/r.gareev)
- [**Victor Mazanov**](https://gitlab.pg.innopolis.university/v.mazanov)
- [**Artyom Grishin**](https://gitlab.pg.innopolis.university/ar.grishin)
- [**Arsen Galiev**](https://gitlab.pg.innopolis.university/a.galiev)
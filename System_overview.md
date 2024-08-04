# System Overview

The general internal workflow of the system while processing user requests is as follows:
1. The user sends a request to the Telegram bot.
2. The classifier in the system matches the request to the most similar predefined plan. This plan is then selected for further execution.
3. The plan, which is a graph of sequentially interconnected actions, will be decomposed into a sequence of actions aligned with the user request.
4. Each action takes input and output parameters and will be processed sequentially.


# Team data

When you started to work on repositopy please specify your `team_id` and `access_token` that you obtained from telegram bot in `settings.py`

```
class TeamSettings(BaseSettings):
    team_id: str = "<your_team_id>"
    access_token: str = "<your_access_token>"
```

# Actions

An Action is a function that takes input parameters, accesses the specified endpoint using those parameters, and returns output parameters. For example: sending an email, creating a task, summarizing text.

## How to Create Your Own Actions

You can write an action in two ways:

1. **Class with Action with Obligatory Execute Action**: This is convenient if you have common supplementary functions for several actions.

```python
class ExtractIssueTitle:
    def __init__(self) -> None:
        self.action_name: str = "extract_issue_title"
        self.stop: str = "</Answer>"
        self.max_tokens: int = 50
        self.llm: LLM = LLM()

    def get_prompt(self) -> str:
        return "Sample prompt"

    async def execute(self, input_data: ExtractInputParams) -> ExtractOutputParams:
        user_request: str = input_data.user_request
        prompt: str = self.get_prompt()
        prompt = prepare_prompt(prompt, user_request)
        answer: str = await self.llm.get_response(
            {
                "prompt": prompt,
                "stop": self.stop,
                "max_tokens": self.max_tokens
            }
        )
        return ExtractOutputParams(answer=answer)
```

2. **Function**:

```python
async def extract_title(authorization_data: dict, input_data: ExtractInputParams) -> ExtractOutputParams:
    llm: LLM = LLM()
    user_request: str = input_data.user_request
    prompt: str = "Sample prompt"
    prompt = prepare_prompt(prompt, user_request)
    answer: str = await llm.get_response(
        {
            "prompt": prompt,
            "stop": "</Answer>",
            "max_tokens": 50
        }
    )
    return ExtractOutputParams(answer=answer)
```

## Registration of Actions

To let the router know about the existence of actions, you need to register them. To register your action, you need to add the decorator `register_action` from `src/actions/registry.py` above the action. You can use the same decorator for both functions and classes.

```python
register_action(
    input_type: Type[BaseModel],
    output_type: Type[BaseModel],
    system_name: str = "General",
    action_name: str | None = None,
    result_message_func: Callable | None = None
)
```

### Input/Output Parameters

The decorator must specify a Python schema with input and output parameters for the action. 
- In the input parameters, you list the arguments that will be used to successfully complete the action.
- In the output parameters, you list the arguments that will be returned after the action is executed.

```python
class CreateIssueInputParams(BaseModel):
    owner: str
    repo: str
    title: str
    body: str

class CreateIssueOutputParams(BaseModel):
    title: str
    url: str
    number: int
    created_at: str
    repository: dict
    body: list
```

### System and Action Name

To let the router know about an action being executed, you need to specify the same system and action name that you used in the plan.

### Show Result of the Action

To return custom results of the action in Telegram, you can write a function and pass it to the decorator's `result_message_func` field. Pay attention to the function's output, which must return a tuple with two elements:
1. A dictionary with information used in the message.
2. A string that will be displayed in Telegram.

Example of a result function:

```python
def form_create_issue_result_message(action_result_data: dict) -> tuple[str, dict]:
    repo: str = action_result_data["repository"]["name"]
    issue_title: str = action_result_data["title"]
    issue_body: list = action_result_data["body"]

    message_dict: dict = {
        "Issue repository": repo,
        "Issue title": issue_title,
        "Issue body": issue_body,
    }

    # Will be shown in Telegram 
    message_str: str = (
        f"The following GitFlame Issue was created:\n"
        f"Repository: {repo}\n"
        f"Issue title: {issue_title}\n"
        f"Issue body:\n{issue_body}"
    )

    return message_str, message_dict
```

## Register Actions Using Endpoint

After writing your actions, you need to register them in our system. To add your actions, you should access the "/register-actions" endpoint with the list of all actions in JSON format and your access token.

```json
[
    {
        "system": "My system",
        "action_name": "My action",
        "description": "This is a sample description",
        "input_parameters": {
            "user_request": {
                "type": "string"
            }
        },
        "output_parameters": {
            "some_output": {
                "type": "string"
            }
        }
    },
    {
        "system": "MySystem",
        "action_name": "MyAnotherDumpAction",
        "description": "This is another description",
        "input_parameters": {
            "user_request": {
                "type": "integer"
            }
        },
        "output_parameters": {
            "some_output": {
                "type": "integer"
            }
        }
    }
]
```

# Authorizations

To add integration with new system, you would need to write authorization for it. There are
two examples of authorization provided: for GitFlame and for Todoist. The authorization in these systems can be done
through an authorization server provided.

### Running authorization server
To run authorization server, you need to install the requirements.

`pip install -r requirements.txt`

Then you need to run the file _authorizations/api_server/auth_server.py_, e.g.

`python -m src.authorizations.api_server.auth_server`

Then you can open swagger docs at http://127.0.0.1:8845/docs

P.S. you would need to setup your team_id and access_token (can be received through telegram bot) in _src/settings.py_ first, to avoid errors on server startup

### Authorizing in existing systems

To authorize in GitFlame, you would first need to register in this system (https://gitflame.ru/). Then you would need
to login at http://127.0.0.1:8845/authorize/git-flame with your login/password. The data will be saved ono our servers

To authorize in Todoist, you would need to :
- register first
- register an Oauth app [here](https://developer.todoist.com/appconsole.html), and add the received client id and client secret to settings (src/settings.py) 
client id and client secret to settings
- follow http://127.0.0.1:8845/authorize/git-flame, then copy url to the address bar, and authorize your app to access your todoist account
- the data will be saved on our servers

### Writing new authorization

To add authorization in new system, you need to:
1. **Add endpoint, through which you will authorize in your system**, in _src/api_server/auth_server.py_. Please follow the examples for GitFlame (login/password) and Todoist (Oauth), located in _src/api_server/auth_server.py_
2. **Write authorization logic inside the endpoint**, to get authorization tokens
3. Your endpoint function needs to **finish with**  
`save_authorization_data(authorization_data, "YourSystemName") `, where authorization_data is the tokens/other type of authorization data that you received. This ensures that your authorization data is saved to our system and can be used when running actions through tg bot.

**Important!** The authorization data will be saved to our servers, so do not login in accounts using profiles that contain sensitive data (better to create test accounts)


# Run Your Code

After registering all your actions, and when you want to run your code, you need to work on the `src.actions_socket_listener` file.

**Important!**  You need to explicitly use `importlib` and list all files where you used the decorator for actions registration:

```python
import importlib

importlib.import_module(".extract", package="src.actions.ai")
importlib.import_module(".gitflame_actions", package="src.actions.backend")
```

When you run `actions_socket_listener.py`, you start to listen to the socket that was opened for your team.

```bash
python -m src.actions_socket_listener
```

After that, you can write prompts to the telegram and execution of your action will come to your socket.

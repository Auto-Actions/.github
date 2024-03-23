# Auto startup workspace
<p>
    <a href="https://www.python.org/" target="blank_"><img alt="python" src="https://img.shields.io/badge/python-3.10.13-green" /></a>
    <a href="https://fastapi.tiangolo.com/" target="blank_"><img alt="FastAPI" src="https://img.shields.io/badge/MetaGPT-0.7.4-orange" /></a>
    <a href="https://reactjs.org/" target="blank_"><img alt="action" src="https://img.shields.io/badge/Github-Action-purple" /></a>
    <a href="https://opensource.org/licenses/MIT" target="blank_"><img alt="mit" src="https://img.shields.io/badge/License-MIT-blue.svg" /></a>
</p>
<br/>

# Overview
![Startup overview](./assets/overview.svg)

The workspace allows developer start up a project easly by creating an issue within your requirements.

# Usage

- When you come up with an idea then want to implement a software for that idea. Let's start by creating a new repository in our workspace.
![Create repository](./assets/create_repository.png)

- From homepage of new repository, switch to Action tab to manually setup action through `set up a workflow yourself →`

![Setup action](./assets/create_action_script.png)

- This is a first version of action script.

```yaml
name: Auto Initialization
on:
  issues:
    types:
      - opened
      - labeled
      - reopened

jobs:
  run:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        id: checkout
        uses: actions/checkout@v4
        
      - name: Generate Stage
        uses: fjogeleit/http-request-action@main
        id: genRequest
        with:
          url: 'http://mandoguru.com:8000/v1.0.0/gen_prog/'
          method: 'POST'
          customHeaders: '{"Content-Type": "application/json", "KEY": "MqQVfJ6Fq1umZnUI7ZuaycciCjxi3gM0"}'
          data: '{"idea": "${{ github.event.issue.body }}", "n_rounds": 3}'
          preventFailureOnNoResponse: 'true'
          timeout: 1000000
          
      - name: Show Generation Response
        run: echo ${{ fromJson(steps.genRequest.outputs.response).repo_name }}

      - name: Push Project
        uses: fjogeleit/http-request-action@main
        id: initRequest
        with:
          url: 'http://mandoguru.com:8000/v1.0.0/init_repo/'
          method: 'POST'
          customHeaders: '{"Content-Type": "application/json", "KEY": "MqQVfJ6Fq1umZnUI7ZuaycciCjxi3gM0"}'
          data: '{"id": "01", "name": "${{ fromJson(steps.genRequest.outputs.response).repo_name }}", "local": "${{ fromJson(steps.genRequest.outputs.response).repo_name }}", "remote_url": "${{ github.server_url }}/${{ github.repository }}", "remote_name": "origin", "branch": "initial", "commit_message": "Init commit"}'
          preventFailureOnNoResponse: 'true'
          timeout: 600000
```

- Commit the file as initial commit. For further actions, you have to update this file as a new commit.

![Setup action](./assets/commit_action.png)

  - There are some sensitive information in the action script such as API key. It can be hidden by setting `API_KEY` secret variable in `Settings -> Secretes and variables -> Actions New repository secret`

    ![Assign secret variable](./assets/assign_secret_variable.png)

  - Then replace the api key in the action script by:
  
    ```yaml
            customHeaders: '{"Content-Type": "application/json", "KEY": &{{ secret.API_KEY }}}'
    ```

- Next step is tell the action about your idea by creating a new issue.

![Create issue](./assets/create_issue.png)

- Then you can switch to Action tab to monitor the development process.

![Action running](./assets/monitor_action_running.png)

- When the action running finish, there initial software is already in branch `initial`.

![Checkout source code](./assets/checkout_source_code.png)


# TODO

## Service
- [ ] Tracking local repository on server for further update.
- [ ] Serve the feedback/review of developor from action.
- [ ] Agent for particular task such as summarize code, create readme files.
 
## Git Action
- [ ] Trigger action call reprogramming service when developer comment/create issue/merge request.
- [ ] Improve the customization of action. Avoid hardcode, allow developer modify request parameters by environment variables.
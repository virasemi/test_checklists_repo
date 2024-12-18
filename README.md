# Pipeline workflow description


## Note

Before setting up the pipeline, SSH public and private keys must be generated using the following command:
```
ssh-keygen -p -m PEM -f github-actions-key
```
- The public key must be added to the target repositories (in our case, uec-compliance) under Settings > Deploy keys
- The private key must be added to the initiating repository (in our case, test_checklists_repo) under Settings > Secrets and variables > Actions secrets


Only after this setup can the target repository be securely cloned within the initiating repository's Actions.

Please also note that some steps are deliberately done using Bash instead of GitHub Actions in order to make it easier to reproduce the process on other systems.


## Triggers

Only pull requests trigger the pipeline `.yaml` script.


## Jobs


### setup-environment

This job runs on the latest Ubuntu image and is responsible for 
setting up the environment before running tests. This commands Installs
Python3 and Pip3. 
```
  setup-environment:
    runs-on: ubuntu-latest
    if: github.event.pull_request.merged == false || github.event.pull_request.merged == true
    steps:
      - name: Install Python and Pip
        run: |
          sudo apt update
          sudo apt install -y python3 python3-pip
```
Python3 and Pip3 will be used in subsequent steps.


### run-tests

This job runs after the setup-environment job and performs the actual testing tasks.

#### Check out the code 

Uses the `actions/checkout@v3` action to check out the code from the repository.
This makes the repository code available to the workflow so it can be modified and tested.


#### Set up SSH Key for GitHub

Uses the `webfactory/ssh-agent@v0.5.3` action to set up an SSH key for authentication with GitHub. 
The SSH private key is passed using the secret `TEST_CHECKLIST_SECRET`. This is necessary for 
securely cloning private repositories or accessing remote resources via SSH. The SSH key is 
added to the SSH agent, enabling Git operations to be authenticated without password prompts.

It is essential to first add private key (secret) into initiating repository's Actions as described
in 'Note' above, and specify a the secret name in YAML:

```
      - name: Set up SSH Key for GitHub
        uses: webfactory/ssh-agent@v0.5.3
        with:
          ssh-private-key: ${{ secrets.TEST_CHECKLIST_SECRET }}
```


#### Git Clone

Clones the `uec-compliance` repository using SSH, then executes `git checkout feature-schema-tool` to switch 
to the specified branch in the `uec-compliance` repository. Also runs `pip3 install -r requirements.txt`
to install the dependencies defined in the `requirements.txt` file of the `uec-compliance` repository.

#### infile Option Check

Runs the `schema-tool/schema-tool.py` script with the `--infile` option, 
specifying an Excel file (`PHY-LL_Matrix_and_Checklist_v0.1.xlsx`).


#### checklist Option Check

Runs the `schema-tool/schema-tool.py` script again, this time with the `--checklist` option, 
pointing to a CSV file (`PHY-LL_Matrix_and_Checklist_v0.1_checklist.csv`).



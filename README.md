# Pipeline workflow description

## Note

Before pipeline setup, ssh public and private keys must be generated using this command.
```
ssh-keygen -p -m PEM -f github-actions-key
```
Public key must be added to target repos 
(in our case uec-compliance) Settings/Deploy keys.

Private key must be added to initiator repos
(in our case test_checklists_repo) Settings/Secrets and variables/Actions keys.

Only after this target repo can be securely cloned inside initiator repos Actions.

## Trigger

Pipeline .yaml script triggers only on pull requests.

## Jobs

### setup-environment

This job runs on the latest Ubuntu image and is responsible for 
setting up the environment before running tests. This commands Installs
Python3 and Pip3. 
```
sudo apt update
sudo apt install -y python3 python3-pip
```
Python3 and Pip3 must be used in subsequent steps.

### run-tests

This job runs after the setup-environment job and performs the actual testing tasks.

#### Check out the code 

Uses the actions/checkout@v3 action to check out the code from the repository.
This makes the repository code available to the workflow so it can be modified and tested.

#### Set up SSH Key for GitHub

Uses the webfactory/ssh-agent@v0.5.3 action to set up an SSH key for authentication with GitHub. 
The SSH private key is passed using the secret TEST_CHECKLIST_SECRET. This is necessary for 
securely cloning private repositories or accessing remote resources via SSH. The SSH key is 
added to the SSH agent, enabling Git operations to be authenticated without password prompts.

#### Git Clone

Clones the uec-compliance repository using SSH, after executes git checkout feature-schema-tool to switch 
to the specified branch in the uec-compliance repository. Also runs pip3 install -r requirements.txt
to install the dependencies defined in the requirements.txt file of the uec-compliance repository.

#### infile Option Check

Runs the schema-tool/schema-tool.py script with the --infile option, 
specifying an Excel file (PHY-LL_Matrix_and_Checklist_v0.1.xlsx).

#### checklist Option Check

Runs the schema-tool/schema-tool.py script again, this time with the --checklist option, 
pointing to a CSV file (PHY-LL_Matrix_and_Checklist_v0.1_checklist.csv).



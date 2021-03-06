# CI/CD with TEAMCITY (TC)


## Github Branches
Branch Name | Purpose
---|---
Master | Production ready, stable code
Staging | Accumulation of new software features from Develop branch to do integration testing before deploying on production system (Master branch)
Develop | Developers merge new software features to this branch to do testing via Pull Requests
Feature | Feature branches used by developers for each new software feature

## CI/CD Process:
This varies dependent on software development team needs and preferences.

**Feature 1 -> Develop -> Staging -> Master**


1) Developers checkout from `Master` and create `Feature` branch.  Developers can fork the original repo, or push to the same repo. 
2) Create Pull Request from `Feature` branch into `Develop` branch
3) New Pull Requests to `Develop` branch triggers TC to do software testing.  You can download use Teamcity.Github plugin to submit test results back to Github PR webpage.  If PR passes testing, merge PR into `Develop` branch using Github or manual merge if there are conflicts.
4) Commits in `Develop` branch triggers TC to do software testing, and upon passing TC will automatically commit to `Staging` branch.
5) Commits to `Staging` branch trigger TC to do integration testing.  You can squash commits using `git reset` or `git rebase` in `Staging` branch to clean up Git log history.
6) Create Pull Request from `Staging` into` Master`.  Project Owner/ Team Lead review/approve Pull Request into `Master` branch.
7) CD setup should be using code from `Master` branch using `git pull` or download zip artefact.
8) You can have a `Hotfix` branch that is checked out from `Master` and merged back to `Master`.



## 1) Teamcity Setup server

You can use docker image, and it can be behind nginx if you are hosting many web apps on the same VM host.

Teamcity requires default port `8111` to be open inbound unless you change it to port 80.

## 2) Setup Teamcity Build Agent


Build agent requires port `9090` to be open.

```sh
# download the agent distribution from your team city server
wget http://X.X.X.X/update/buildAgent.zip 
unzip buildAgent.zip

# install java8 if you do not already have it
sudo apt-get update
sudo apt-get install default-jre
sudo apt-get install default-jdk
sudo update-alternatives --config java

# make a copy of the build agent config file
cp conf/buildAgent.dist.properties conf/buildAgent.properties

```

Open `conf/buildAgent.properties` and change the value of serverUrl to match your TC server

```js
serverUrl=https://localhost:8111
or 
serverUrl=https://buildserver.mydomain.com:8111
```

Now start the build agent:

```sh
#start build agent
./bin/agent.sh start

# watch build agent logs
tail -f logs/teamcity-agent.log

# stop build agent
./bin/agent.sh stop
```


Now go to Teamcity web portal, and click agents.  Under the `unauthorized` tab, you will see the new build agent. Click `authorize` to start using the new build agent.


## 3) Configure Teamcity Server for your repo/project

1) Create **Project**.  


2) Add a **VCS root** to the project.  

Enter the fetch url (looks like this git@github.com/user-id/repo-name)  

Enter the default branch `refs/heads/develop`

Enter the branch specification `+:refs/pull/(*/merge)` and `+:refs/heads/staging`

Select Authentiation Method `Uploaded Key` and upload the private key for the public Deploy Key that has been added to the Github repo.

Click `Test Connection`

Click `Save`


3) Create **build configuration**

Enter the name.  eg.  Test Develop Build

Take note of the Build Configuration ID.  You will need this for other purposes such as displaying the build status icon

Select `enable status widget` to display the buld status icon

Click `Save`


3.1) Add **Build Step**

Runner type: Command Line

Enter step name: eg. Run Docker Container and Run Test Suite

Execute step: if all previous steps have finished successfully

Run: custom script

Custom script:  enter bash code to spin up docker container and execute test suite

Click `Save`

3.2) Add  **Triggers**

VCS Trigger will ad a build to the queue when a VCS check-in is detected

Click `Save`

3.3) Add **Build Features**

Automatic merge

Watch builds in branches: `refs/heads/develop`

Merge into branch: `refs/heads/staging`

Merge Policy: `merge commit` or `fast-forward`

Click `Save`


## Optional Nice-to-have:
- You may wish to configure Github with **webhooks** so that Github will notify Teamcity when a pull request or commit has been made, and to immediately start the build/test.  This can mitigate the 60 seconds waiting time for teamcity to poll Github for updates.
- You may wish to configure Github with **protected branches** to ensure that build checks are done on all Pull Requests before a merge can be performed. For example, all Feature branch pull requests must be tested before merge on Develop branch.





## 4) Build / Testing Icon
Add this sniplet of code to the Github repo README.md file to display a image representing the the pass (green) or fail (red) of building and/or testing.  You may wish to remove the anchor if you do not want a hyper link back to the build status webpage.

```html
<a href="http://YOURTEAMCITYURL/viewType.html?buildTypeId=YOURBUILDID&guest=1">
<img src="http://YOURTEAMCITYURL/app/rest/builds/buildType:(id:YOURBUILDID)/statusIcon"/>
</a>
```

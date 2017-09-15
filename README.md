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

## 3) Build / Testing Icon
Add this sniplet of code to the Github repo README.md file to display a image representing the the pass (green) or fail (red) of building and/or testing.  You may wish to remove the anchor if you do not want a hyper link back to the build status webpage.

```html
<a href="http://YOURTEAMCITYURL/viewType.html?buildTypeId=YOURBUILDID&guest=1">
<img src="http://YOURTEAMCITYURL/app/rest/builds/buildType:(id:YOURBUILDID)/statusIcon"/>
</a>
```

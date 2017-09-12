# TEAMCITY

## 1) Setup server

You can use docker image, and it can be behind nginx if you are hosting many web apps on the same VM host.

Teamcity requires default port `8111` to be open inbound unless you change it to port 80.

## 2) Setup Build Agent


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


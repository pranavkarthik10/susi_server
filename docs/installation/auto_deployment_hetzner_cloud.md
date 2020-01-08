# Install susi_server on Hetzner Cloud

## Setup
You can create an account to Hetzner Cloud on their [website](https://www.hetzner.com). After logging in you may need to perform verification in the form of PayPal or Government ID.

### Create a new Project and Connect
In the [dashboard](https://console.hetzner.cloud/) select 'Add a new project'. Fill all the required fields and proceed to the Project page.

Now select the red 'Add Server' button in the top right. You can choose your desired specifications and location.

After creation, you should have a root password sent to the email address provided. Using this we can connect to our server.

Open up a terminal window (Mac, Linux) or use SSH software (Windows) to connect to the server.

On Linux or Mac systems it is very simple. Substitute ip address for the ip address of your server.
```
ssh <ip address> -l root
```
Now enter your root password. After connection it will ask you to change the password. You can change it to whatever you desire.

## Installing prerequisites
Now to install and setup Yaydoc.
We need to install
- git
- jre
- gradle

Install these through the following commands:
```
sudo apt-get update
sudo apt-get install git
sudo apt-get install default-jre
sudo apt-get install openjdk-8-jdk-headless
```

After installing these let's continue. It is preferred to not use the root folder as your working directory.
```
cd /home
git clone https://github.com/fossasia/susi_server
```

## Deploy the application
After cloning lets navigate to the folder.
```
cd /home/susi_server
```
Let's setup git submodule:
```
git submodule update --recursive --remote
git submodule update --init --recursive
```

Now we can build the application using this command:
```
./gradlew build
```
Now deploy the application using the following command. 
```
bin/start.sh
```
After running this you can view it at `localhost:4000` or in this case `<server-ip>:4000`.

# Setup continuous integration.
What if we wanted to update the deployment everytime there is new code pushed to the repository?

We can do that with the help of a simple [script](https://github.com/logsol/Github-Auto-Deploy).

Let's set it up.
```
cd /home
git clone https://github.com/logsol/Github-Auto-Deploy.git
```
Now 
```
cd Github-Auto-Deploy
```
Copy 	GitAutoDeploy.conf.json.example and change it accordingly.
```
cp GitAutoDeploy.conf.json.example GitAutoDeploy.conf.json
nano GitAutoDeploy.conf.json
```
Change the file to the following
```
{
	"port": 3001, 
	"repositories": 
	[{
		"url": "https://github.com/fossasia/susi_server", 
		"path": "/home/susi_server/",
		"deploy": "bin/stop.sh ; ./gradlew build ; bin/start.sh"
	}]
}
```
Make your changes and use `CTRL + X` to save and return.

This uses port 3001 to listen to any changes in the GitHub repository and stops, builds, and deploys the application.

Make sure to add a webhook pointing to your server ip at port 3001 in the GitHub repository.

We need to run this script continuously on this server so it is always ready for changes. We will be using a tool known as `cron` to do this.

If it's not already installed, do so by typing the following commands :
```
sudo apt-get update
sudo apt-get install cron
```

After installing cron we can type the following command to open crontab to schedule a command.
```
crontab -e
```

Now change the last line to
```
* * * * * sudo python /home/Github-Auto-Deploy/GitAutoDeploy.py --daemon-mode
```
Now it should be running successfully. Try going to `<server-ip>:3001` and you will be left with a GET error because it only accepts POST requests.

After the next commit/merge to the repository, the app should successfully be running on `<server-ip>:4000`

## Deploying on a custom domain
Deploying the application to a custom domain is simple. First we must add an A record from the domain hosting service to the server ip. This is different for different service providers so please learn how to by searching `How to add an A record to <hosting provider>`. Then we need to forward the base port 80 to port 8000 where our app is deployed. We can use `iptables` a preinstalled tool to do this.

We can add this command in the crontab to make this run continuously as well.

```
crontab -e
```
to open up your crontabs and add the following line to the end of the file:
```
sudo iptables -A PREROUTING -t nat -i eth0 -p tcp --dport 80 -j REDIRECT --to-port 4000
```
Change 4000 to whichever port you have hosted the app on.

Now after a few minutes you should see the deployment live on your custom domain!
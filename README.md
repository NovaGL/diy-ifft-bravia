# DIY IFTTT for Harmony
IFTTT Bravia commands for Google Assistant and Alexa

This is a DIY Bravia project to allow Alexa or Googgle Assistant to control any TV button press on your Bravia TV.
This has been tested on Sony Android TV's

An example would be "Ok Google, TV On" or "Alexa, trigger, TV Input 1"

This process has several preequistes that might not make it possible or advisable to use for some users.
There are offical methods that maybe better for some users, but this method gives me greater flexibility.
If you have any suggestions please create an issue.

- A device capable of running Node.JS (Raspberry PI, Windows, OSX, Linux)
- [IFTTT Account](http://ifttt.com)
- [BST](https://github.com/bespoken/bst)  (this will receive the actions from IFTTT)
- [PM2](https://www.npmjs.com/package/pm2) (used for keeping BST in memory)
- [Node-Red](http://nodered.org) (this will peform the actions that IFTTT sends out)


### Node-Red Installation

Get Node-Red using the following command 

```bash <(curl -sL https://raw.githubusercontent.com/node-red/raspbian-deb-package/master/resources/update-nodejs-and-nodered)```

This should install Node.JS which is used for BST and PM2

## Bravia TV Setup
Make sure your TV has a fixed IP otherwise commands won't work if IP is changed.

1. Turn on your TV
2. On the TV go to Settings > Network > Home network setup > Remote device/Renderer > On
3. On the TV go to Settings > Network > Home network setup > IP Control > Authentication > Normal and Pre-Shared Key
4. On the TV go to Settings > Network > Home network setup > Remote device/Renderer > Enter Pre-Shared Key > 0000 (or whatever you want your PSK Key to be)
5. On the TV go to Settings > Network > Home network setup > Remote device/Renderer > Simple IP Control > On

Keep note of the PIN and IP address of the TV this will be used later.

## BST Proxy setup
**Make sure you are using at least v1.0.7 for better security with a key in the url**    
__If v1.0.7 is not out yet add @1.0.7 to end of below command__

Installed BST using `npm install bespoken-tools -g`
To test BST Proxy we use the following command after BST is installed.
```bst proxy http 1880```

It will then say something similar to.
```Your public URL for accessing your local service:```
```https://myskill.bespoken.link?bespoken-key=XXXXXXXX-XXXX-XXXX-XXXX-XXXX-XXXXXXXX```

This URL will be used for our webhook commands in the IFTTT section.

## PM2 Setup

To make this run in background we use PM2.
```npm install pm2 -g```

Create a directory to store the BST command in.

Eg make a directory called "bst"   
Store a file in there called proxy.sh  
In that file save the same command as above ```bst proxy http 80 --secure```

Now to save that command to BST, we do the following in the same directory as the script.

```pm2 start proxy.sh --name="bst-proxy"```
```pm2 startup```

This will save the script under the name "bst-proxy" and make sure it starts on boot.


## IFTTT

What we want to do with ifttt is to create a trigger using either Alexa or Google Assistant depending on your platform.
Choose what you want to say. (If you have multiple rooms, it's best to include the room name in the command) 

It really doesn't matter what you choose for the action, choose something that is easy for you to remember.
For a list of commands refer to the JSON file in this repo.
#is for number $ is for text

### "Say a phrase with a text ingredient" (Selecting channel by name)

Title| Action
------------ | -------------
What do you want to say? | TV $
What do you want the Assistant to say in response?| Pressing TV button $


### Make a web request
Next we want to choose the "Webhook" Action 
The URL is the one we got from BST and add bravia to the end.


Title| Action
------------ | -------------
URL | https://[myskill].bespoken.link/bravia?bespoken-key=XXXXXXXX-XXXX-XXXX-XXXX-XXXX-XXXXXXXX/
Method| POST
Content Type| application/json
Body | {"command":"{{TextField}}" ,"type": "TVCommand", "pin": "**[PIN]**", "tv_ip": "**[TV_IP]**"}

#### Body Values
The body of the request is broken down into several values to be used to determine the action you want to perform

Key| Value
------------ | -------------
command | TextField (this will be the command we send to the TV eg Power On)
type | TVCommand
pin | The Pin you setup on your TV
tv_ip | The static ip of your TV

On the next step give it a descriptive name and save.

## Node-Red Flow
Once you have Node-Red up and running. Copy harmony-ifft.json to clipboard.

**Menu > Clipboard > Import > paste the example and press deploy.**

This should create a HTTP endpoint which will accept HTTP POST requests at /bravia

Now assuming that everything is setup correctly, you should be able to say "Ok Google, TV On" and the TV will turn on.

Feel free to modify the list to suit your needs.

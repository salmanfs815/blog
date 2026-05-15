---
title: Developing a Flask App & Deploying on AWS
metaTitle: Developing a Flask App & Deploying on AWS
metaDesc: Tutorial on how to create a Python Flask application and deploy to AWS
  Elastic Beanstalk and AWS Lambda
date: 2021-09-12T06:13:58.404Z
tags:
  - tutorial
  - python
  - flask
  - aws
---
[Flask](https://flask.palletsprojects.com/en/2.0.x/) is a lightweight Python web framework.
It's lightweight nature makes it a great candidate for creating APIs.
In this tutorial, I'm going to cover how to make a simple RESTful API service with Flask and host it on AWS.

First, we'll make a basic Flask app and use [AWS Elastic Beanstalk](https://docs.aws.amazon.com/elastic-beanstalk/index.html) to deploy it.
Then, we'll use [Zappa](https://github.com/zappa/Zappa) to deploy a serverless app to [AWS Lambda](https://docs.aws.amazon.com/lambda/index.html). For more information on serverless architectures and how Zappa works, check out the [About](https://github.com/zappa/Zappa#about) section on their GitHub page.

## Python Version

Since the goal is to make an AWS-hosted API, we'll need to make sure the version of Python we use is supported by AWS and the other tools we'll be using.

[Supported Python Runtimes on AWS Elastic Beanstalk](https://docs.aws.amazon.com/elasticbeanstalk/latest/platforms/platforms-supported.html#platforms-supported.python)\
[Supported Python Runtimes on AWS Lambda](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html)\
[Supported Python Runtimes for Zappa](https://github.com/zappa/Zappa#installation-and-configuration)

Currently, the latest version of Python supported by all 3 of the above is 3.8. So please make sure [Python 3.8](https://www.python.org/downloads/release/python-3810/) is installed before proceeding.

## Setup

As best practice, we'll be using a virtual environment for managing dependencies. This will also be required by Zappa later on.

Start by creating a folder for the project. I'm going to be calling it `simple-flask-app`. Open a terminal console in the folder.

If you only have Python 3.8 installed, you can create a virtual environment with the following command:

```bash
virtualenv env
```

If you have multiple Python versions installed, you'll need to specify which version to use:

```bash
virtualenv --python 3.8 env
```

`env` is the name of the environment. You can name it whatever you like. Just make sure to add the folder with this name to your `.gitignore` if you're using Git.

To activate the virtual environment, enter the following command:

```bash
source env/Scripts/activate
```

Now that we've got our virtual environment activated, we can install our dependencies.

```bash
pip install flask
```

It's also a good idea to maintain an up-to-date list of project dependencies.

```bash
pip freeze > requirements.txt
```

This can be used by someone to setup their environment when trying to develop/run the application.

## Flask App

The main file will be called `application.py` and the Flask callable object will be named `application` since that is what AWS Elastic Beanstalk will look for when it runs the app.

```python
from flask import Flask

application = Flask(__name__)

@application.route("/")
def hello():
    return "Hello, World!"

@application.route("/send", methods=["POST"])
def send():
    return "Success"
```

We'll need to set some environment variables that Flask will use.

```bash
export FLASK_ENV=development
export FLASK_APP=application
```

`FLASK_ENV` tells Flask to run in [development mode](https://flask.palletsprojects.com/en/2.0.x/server/). This allows debug output when we inevitably encounter errors and a few other neat features.
`FLASK_APP` tells Flask the name of the main file (the `.py` extension is implicit).

We can now run the application.

```bash
flask run
```

The application should now be running on [localhost:5000](http://localhost:5000).

Let's test if it's working. In a new terminal window (so as to not kill the running Flask app), send a request to the app with `curl`.

```bash
curl localhost:5000/
```

You should see a "Hello, World!" response. This is our response to the `/` route.

Now, let's test the `/send` route.

```bash
curl localhost:5000/send
```

You should have received an error response. Since the only method allowed for the `/send` route is POST and `curl` is sending a GET request, Flask creates an HTML page to display the error response and returns that.

```html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
<title>405 Method Not Allowed</title>
<h1>Method Not Allowed</h1>
<p>The method is not allowed for the requested URL.</p>
```

So, let's try again. But this time, we'll tell `curl` to use the POST method.

```bash
curl -X POST localhost:5000/send
```

Now we should see our success message.

We can even send data to the server. Let's make a slight modification to our `send()` function.

```python
@application.route("/send", methods=["POST"])
def send():
    return "Success. Message: " + request.form['message']
```

Saving the file should automatically refresh the Flask app (another perk of development mode).

Now we can send some data to the app.

```bash
curl -X POST localhost:5000/send -d "message=testing"
```

We should see our new success message that parrots back the data we sent.

Albeit a basic one, this is the Flask app we will now be deploying to AWS.

## AWS Elastic Beanstalk

We're going to use Elastic Beanstalk (EB) to deploy our app. EB allows for quick and easy deployment while letting AWS configure and manage the underlying infrastructure required to run the application.
For this section, make sure you have the [EB CLI installed](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3-install.html).\
We can then run the following commands to deploy to AWS EB (on Windows, these will need to be run from Powershell).

```bash
eb init -i
eb create simple-flask-app
```

The first command configures the application. For the most part, you can go ahead with the defaults. I generally opt-out of using CodeCommit as I prefer to use GitHub. It's a good idea to use SSH.
The second command creates the environment in EB and deploys the contents of the working directory. `simple-flask-app` is the name of the environment. You can name it whatever you like as long as there is no other environment in your account with the same name.

You can view the running instance with the following command.

```bash
eb open
```

Copy the URL from the browser window that just opened. Let's use `curl` to send requests to the deployed instance to see if everything is working correctly.

```bash
curl <EB_URL>
curl -X POST <EB_URL>/send -d "message=testing"
```

You should receive the same success messages that were received when running locally.

If changes are made to the application, the deployment can be updated with the following command.

```bash
eb deploy
```

Once we're done using the EB deployment, we can destroy the instance with the following command.

```bash
eb terminate simple-flask-app
```

## AWS Lambda

Deploying to AWS Lambda is even easier, thanks to Zappa!

```bash
pip install zappa
```

As of this writing, there is an [open issue](https://github.com/zappa/Zappa/issues/984) on Zappa's GitHub about the error you're going to encounter: Zappa and Flask require conflicting versions of a common dependency, Werkzeug.
There's also another issue that I ran into. One of Zappa's dependencies, troposphere, made a breaking change. The issue was recently [closed](https://github.com/zappa/Zappa/issues/998) on GitHub but hasn't made it to the latest release yet.

To fix both these issues, run the following commands.

```bash
pip install Werkzeug --upgrade
pip install troposphere==2.7.1
```

After this, we should update our `requirements.txt` file to keep our dependency list up-to-date.

```bash
pip freeze > requirements.txt
```

Now we're ready to deploy to AWS Lambda.

```bash
zappa init
zappa deploy
```

Zappa will take care of all the AWS configuration and setup for us. At the end, you should see a URL given where you can access the Lambda functions.

Let's try it out in `curl`.

```bash
curl <LAMBDA_URL>
curl -X POST <LAMBDA_URL>/send -d "message=testing"
```

And again, you should receive the same success messages that were received when running locally and on EB.

Once we're done using the Lambda functions, we can undeploy the app with the following command.

```bash
zappa undeploy
```

## Conclusion

The full code is accessible on [GitHub](https://github.com/salmanfs815/simple-flask-app).

To sum up, we were able to create a Flask application and deploy to AWS Elastic Beanstalk and to AWS Lambda.

You may want to use EB when you need a traditional web service model. Perhaps your workloads are more or less constant, you need advanced monitoring, or have long-running functions.

The serverless model with Lambda may be ideal if you need greater scalability, minimal maintenance, and cheaper hosting expenses.

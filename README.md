# Prerequisites
I am using an Ubuntu image because its a good middle ground than to write a MacOS and a Windows version. Once you see how this is done you can apply this to the rest of the integrations that are out for SecureX and Cisco Threat Response.

- Install Ubuntu 20.04 Desktop version on your VMWare environment.
- Get a SecureX account and an AWS account.
- Make sure you can login to you AWS account and access your API keys.
- Purchase Have I been pwned API key for $3.50 USD for 1 month. Make sure you have the API key they provide handy.

# Install Ubuntu Prerequisites
```
sudo apt update
sudo apt install git python3-pip python3-venv httpie
sudo pip3 install zappa
echo 'PATH=$PATH:~/.local/bin:.' >> ~/.bashrc
source ~/.bashrc
mkdir Documents/Development && cd Documents/Development
sudo apt install awscli
```
# Follow the following in setting up IAM in AWS
Do this one time and you can stampout the integrations using this user

https://github.com/CiscoSecurity/tr-05-serverless-have-i-been-pwned/blob/develop/aws/HOWTO.md


# Configure AWS CLI on Ubuntu
- Login in to your AWS console and under IAM retrieve your AWS keys
- Now in your Ubuntu 20.04 vm
```
aws configure
AWS Access Key ID [*******************U]: 
AWS Secret Access Key [*******************h]: 
Default region name [us-east-1]: 
Default output format [None]: 
```

# Click to play the recording
[![asciicast](https://asciinema.org/a/TtkxwEWWducHFJmfklPS6vxqb.svg)](https://asciinema.org/a/TtkxwEWWducHFJmfklPS6vxqb)

![Image AWS Creds](https://github.com/bluecough/SecureX-HaveIBeenPwned/blob/master/img/aws_credentials.png)


# Cloning the SecureX HaveIBeenPwned Repo
Clone the repo then going into that directory and creating a virtual environment for your python packages that you need for this repo.
```
git clone https://github.com/CiscoSecurity/tr-05-serverless-have-i-been-pwned.git
cd tr-05-serverless-have-i-been-pwned
python3 -m venv venv # Note if you use virtualenv to create your virtual environment it just doesnt seem to work... it deploys ok but just doesnt work.
source venv/bin/activate (To leave venv type 'deactivate' at the command prompt)
pip install -U -r requirements.txt
```

Now randomize the last couple of characters in the zappa_settings.json this is so the S3 bucket can be unique.
```
sed -i 's/zappa-tr-have-i-been-pwned-relay/zappa-tr-have-i-been-pwned-relay-<SOMETHING RANDOM>/g' zappa_settings.json
zappa deploy dev
```
Output of running zappa deploy dev
```
Zappa is a system for running server-less Python web applications on AWS Lambda and AWS API Gateway.
This `init` command will help you create and configure your new Zappa deployment.
Let's get started!


Your Zappa configuration can support multiple production stages, like 'dev', 'staging', and 'production'.
What do you want to call this environment (default 'dev'): 
What do you want to call your bucket? (default ‘zappa-tr-have-i-been-pwned-relay-sjdfhsdkhfsdkhf’): <--- Type your own randomness.
It looks like this is a Flask application.
What's the modular path to your app's function?
This will likely be something like 'your_module.app'.
We discovered: app.app
Where is your app's function? (default 'app.app'): <--- leave it as app.app

You can optionally deploy to all available regions in order to provide fast global service.
If you are using Zappa for the first time, you probably don't want to do this!
Would you like to deploy this application globally? (default 'n') [y/n/(p)rimary]: n

Okay, here's your zappa_settings.json:

{
    "dev": {
        "app_function": "app.app",
        "aws_region": "us-east-1",
        "exclude": [".*", "*.json", "*.md", "*.txt"],
        "keep_warm": false,
        "log_level": "INFO",
        "manage_roles": false,
        "profile_name": "serverless",
        "project_name": "tr-have-i-been-pwned-relay",
        "role_name": "tr-serverless-relay-ZappaLambdaExecutionRole",
        "runtime": "python3.7",
        "s3_bucket": "zappa-tr-have-i-been-pwned-relay-sjdfhsdkhfsdkhf"
    }
}

Does this look okay? (default 'y') [y/n]: y
```
# Now create the JWT token

In order to create the JWT token we need to go back to our Documents/Development directory and clone another repo.

```
cd ~/Documents/Development
git clone https://github.com/CiscoSecurity/tr-05-jwt-generator.git
cp tr-05-jwt-generator/jwt_generator.py ./tr-05-serverless-have-i-been-pwned/.
cd tr-05-serverless-have-i-been-pwned/.
python3 jwt-generator.py dev
```

- The SECRET_KEY goes into the AWS console Lambda environment variable while the JWT is applied in CTR or SecureX
- NOTE: If you want to fix the Werkzeug error you can do so by echo 'Werkzeug==0.16.1' >> requirements.txt

[![asciicast](https://asciinema.org/a/Sg4QwE4Y0TMuXCW5les2FHjoE.svg)](https://asciinema.org/a/Sg4QwE4Y0TMuXCW5les2FHjoE)

Enter your API key that you obtained from HaveIBeenPwned (I am using a dummy key as you can see)
![Image of jwt](https://github.com/bluecough/SecureX-HaveIBeenPwned/blob/master/img/jwtout.png)

# Load all of these into AWS and SecureX/CTR
This will print out your Lambda URL location which you will need for CTR or SecureX when you add the integration.
```
zappa status
```
![Image Lamdba Enviroment Variable](https://github.com/bluecough/SecureX-HaveIBeenPwned/blob/master/img/SECRET_KEY.png)

# Commandline testing to see if this all works.
First install httpie so you can do a HTTP POST to the URL gotten from zappa status
```
sudo apt install httpie
```
Now perform the following and it should give an output similar to the below.
```
http POST <LAMBDA URL>/health 'Authorization: Bearer <JWT>'
```
https://security.cisco.com/

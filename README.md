# Prerequisites
I am using an Ubuntu image because its a good middle ground than to write a MacOS and a Windows version. Once you see how this is done you can apply this to the rest of the integrations that are out for SecureX and Cisco Threat Response.

- Install Ubuntu 20.04 Desktop version on your VMWare environment.
- Get a SecureX account and an AWS account.
- Make sure you can login to you AWS account and access your API keys.
- Purchase Have I been pwned API key for $3.50 USD for 1 month. Make sure you have the API key they provide handy.

# Install Ubuntu Prerequisites
```
sudo apt update
sudo apt install git python3-pip make gcc zlib1g-dev libffi-dev
sudo pip3 install zappa venv
echo 'PATH=$PATH:~/.local/bin:.' >> ~/.bashrc
source ~/.bashrc
mkdir Documents/Development && cd Documents/Development
sudo apt install awscli
```

# Optional: Downgrade Python to 3.7.2
I saw Python 3.8 wasn't supported for Zappa so I rolled it back https://github.com/Miserlou/Zappa/issues/1968
```
wget https://www.python.org/ftp/python/3.7.5/Python-3.7.5.tgz
tar zxvf Python-3.7.2.tgz
cd Python-3.7.2
./configure
make
sudo make install
python3 --version
```

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
[![asciicast](https://asciinema.org/a/cy7jp08ArR3Fno84tXHH4rh42.svg)](https://asciinema.org/a/cy7jp08ArR3Fno84tXHH4rh42)

# Cloning the SecureX HaveYouBeenPwned Repo
Clone the repo then going into that directory and creating a virtual environment for your python packages that you need for this repo.
```
git clone https://github.com/CiscoSecurity/tr-05-serverless-have-i-been-pwned.git
cd tr-05-serverless-have-i-been-pwned
virtualenv venv
source venv/bin/activate (To leave venv type 'deactivate' at the command prompt)
pip install -U -r requirements.txt
```
Now run zappa by initing it. I move the original config to .old, there is an explanation but mainly revolves around unique S3 buckets for the region.
```
mv zappa_settings.json zappa_settings.json.old
zappa init
```
Output of running zappa init
```
Zappa is a system for running server-less Python web applications on AWS Lambda and AWS API Gateway.
This `init` command will help you create and configure your new Zappa deployment.
Let's get started!


Your Zappa configuration can support multiple production stages, like 'dev', 'staging', and 'production'.
What do you want to call this environment (default 'dev'): pwned
What do you want to call your bucket? (default ‘zappa-*******’): <--- accept the random name or enter your own.
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
    "pwned": {
        "app_function": "app.app",
        "aws_region": "us-east-1",
        "profile_name": "default",
        "project_name": "tr-05-serverles",
        "runtime": "python3.7",
        "s3_bucket": "zappa-********"
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
python3 jwt-generator.py pwned
```

- The SECRET_KEY goes into the AWS console Lambda environment variable while the JWT is applied in CTR or SecureX
- NOTE: If you see any issues with Zappa or it doesnt upload to AWS Lambda you may need to fix the requirements.txt to have Werkzeug==0.16.1 : Then of course rerun the pip install -U -r requirements.txt

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

https://security.cisco.com/

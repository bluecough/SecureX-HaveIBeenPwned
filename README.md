# Pre-requisites
I am using an Ubuntu image because its too hard to write a MacOS and a Windows version

- Install Ubuntu 20.04 
- Get a SecureX account and AWS account.
- Make sure you can login to you AWS account as the root user
- Purchase Have I been pwned API key for $3.50 USD for 1 month

# Ubuntu Pre-requisites to install
<pre><code>
$ sudo apt update
$ sudo apt install git
$ sudo apt install python3-pip
$ sudo apt install virtualenv
$ sudo pip3 install zappa
$ echo “PATH=$PATH:~/.local/bin:.” >> ~/.bashrc
$ mkdir Documents/Development && cd Documents/Development
$ sudo apt install awscli
</code></pre>
# Optional so you can cut and paste for the time being
<pre><code>
$ sudo apt install open-vm-tools-desktop
</code></pre>

Configure AWS CLI on Ubuntu
# Login in to your AWS console and under IAM retrieve your AWS keys

# Now in your UBUNTU VM
<pre><code>
$ aws configure
AWS Access Key ID [*******************U]: 
AWS Secret Access Key [*******************h]: 
Default region name [us-east-1]: 
Default output format [None]: 
</code></pre>

https://asciinema.org/a/fqDK9KILyF4YtLmshDsFdGC3g

# Cloning the HaveYouBeenPwned Repo
<pre><code>
$ git clone https://github.com/CiscoSecurity/tr-05-serverless-have-i-been-pwned.git

$ cd tr-05-serverless-have-i-been-pwned.git
$ virtualenv venv
$ source venv/bin/activate

# (To leave venv do a $ deactivate)
<pre><code>
$ mv zappa_settings.json zappa_settings.json.old
$ zappa init
Zappa is a system for running server-less Python web applications on AWS Lambda and AWS API Gateway.
This `init` command will help you create and configure your new Zappa deployment.
Let's get started!


Your Zappa configuration can support multiple production stages, like 'dev', 'staging', and 'production'.
What do you want to call this environment (default 'dev'): pwned
What do you want to call your bucket? (default ‘zappa-*******’): <accept the random name>
It looks like this is a Flask application.
What's the modular path to your app's function?
This will likely be something like 'your_module.app'.
We discovered: app.app
Where is your app's function? (default 'app.app'): 

You can optionally deploy to all available regions in order to provide fast global service.
If you are using Zappa for the first time, you probably don't want to do this!
Would you like to deploy this application globally? (default 'n') [y/n/(p)rimary]: n

Okay, here's your zappa_settings.json:

{
    "pwned2": {
        "app_function": "app.app",
        "aws_region": "us-east-1",
        "profile_name": "default",
        "project_name": "tr-05-serverles",
        "runtime": "python3.8",
        "s3_bucket": "zappa-********"
    }
}

Does this look okay? (default 'y') [y/n]: y

</code></pre>
</code></pre>
# Now create the JWT token

In order to create the JWT token we need to go back to our Documents/Development directory and clone another repo.

<pre><code>
$ cd ~/Documents/Development
$ git clone https://github.com/CiscoSecurity/tr-05-jwt-generator.git
$ cp tr-05-jwt-generator/jwt_generator.py ./tr-05-serverless-have-i-been-pwned/.
$ cd tr-05-serverless-have-i-been-pwned/.
$ python3 jwt-generator.py pwned

- The SECRET_KEY goes into the AWS console Lambda environment variable while the JWT is applied in CTR or SecureX
</code></pre>

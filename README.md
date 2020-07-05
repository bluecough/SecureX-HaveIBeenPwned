# Pre-requisites
- Install Ubuntu 20.04
- Get a SecureX account and AWS account.
- Make sure you can login to you AWS account as the root user

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
$ aws configure
AWS Access Key ID [*******************U]: 
AWS Secret Access Key [*******************h]: 
Default region name [us-east-1]: 
Default output format [None]: 
</code></pre>



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
What do you want to call your bucket? (default ‘zappa-sdhjfsdfhks’): <accept the random name>
</code></pre>

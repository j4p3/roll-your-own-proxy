When I'm in a place that censors internet traffic, like China or a corporate office, I typically run all my browser traffic through a remote machine. It's really, really easy, and makes me wonder why I used to pay for services like this. This is a brief instruction manual for how to do it yourself.

Here's what you need:

1. An Amazon account
2. A UNIX command line (open Terminal in Mac or *nix. Windows users, you're out of luck. Go buy a proxy.)

##Set up your remote machine

* Go to [http://aws.amazon.com/](http://aws.amazon.com/). Sign up or sign in with your existing Amazon account.
* Navigate to the "IAM" service. Create a new User called "proxy", and add a Policy to its Permissions with the option `AmazonEC2FullAccess`.
* Get the [AWS command-line interface](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-set-up.html)
* [Add a named profile](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html#cli-multiple-profiles) to your `~/.aws/credentials` file called "proxy".
* Use your new access to AWS to [create an EC2 instance](http://docs.aws.amazon.com/cli/latest/userguide/tutorial-ec2-ubuntu.html), and don't forget to download the key file.
* Write down your new instance's `instance-id` value.

##Set up controls for your machine

* Open or create your `~/.bash_profile` file.
* Enter the following code, with your instance id (e.g. 'i-7215db8f') in place of <YOUR_INSTANCE_HERE>:

```
alias proxy_start="ec2-start-instances <YOUR_INSTANCE_HERE> -v"

alias proxy_stop="ec2-stop-instances <YOUR_INSTANCE_HERE> -v"

alias proxy_run="ssh -v -i ~/.ssh/proxy.pem -N -D 3128 ubuntu@$(aws ec2 describe-instances --instance-ids <YOUR_INSTANCE_HERE> --output text --query "Reservations[*].Instances[*].PublicIpAddress" --profile proxy)"
```

* Save the file.

##Tell your browser about your proxy

* Open your browser preferences and edit its connection settings (in Firefox, this is under Advanced > Network > Connection > Settings 
* Manually configure a proxy with a SOCKS v5 host to `localhost` on the port `3128`.

##Browse through your proxy

* In the terminal, enter `proxy_start`. AWS will take a minute or two to boot up your instance.
* Enter `proxy_run`. You may be asked "Are you sure you want to continue connecting (yes/no)?" Type `yes`.
* Browse at will.
* When you're done, enter `proxy_stop` in the terminal. This will prevent you from getting billed by Amazon (though a nano-sized server will only cost you $5/mo to run nonstop).

This method is not anonymizing - every site you visit will register the IP address of an AWS server attached to your account - but to any observers, the traffic to and from your machine will appear to be limited to Amazon.

Good luck, have fun, let me know if you use this!

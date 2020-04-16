
![ovc-install.sh](images/ovc-install.png?raw=true "ovc-install.sh")

# ovc-install

`ovc-install.sh` is a shell script that automates the [step-by-step instructions](http://docs.ovnconference.org/2.2/install.html) for setting up a OvnConference 2.2 server.

With only a few parameters, `ovc-install.sh` can have your OvnConference server setup and ready for use in 30 minutes (depending on your server's internet speed to download and install packages).

For example, given an Ubuntu 16.04 64-bit server with a public IP address, to install/update to the latest build of OvnConference 2.2 first SSH into the server as root and run the following command:

~~~
wget -qO- https://ubuntu.ovnconference.org/ovc-install.sh | bash -s -- -v xenial-220 -a
~~~

The command will pull down the latest version of `ovc-install.sh`, send it to the BASH shell interpreter, and pass the parameters `-v xenial-220` which specifies you want to install the latest build OvnConference 2.2 (i.e. 2.2.2) and `-a` which specifies want to install the API demos (this makes it easy to do a few quick tests on the server).

Note: If your server is behind a firewall -- such as behind a corporate firewall or behind an AWS Security Group -- you will need to manually configure the firewall to forward [specific internet connections](#configuring-the-firewall) to the OvnConference server before you can launch the client.

When `ovc-install.sh` finishes, you'll see a message that gives you a test URL to launch the OvnConference client and join a meeting called 'Demo Meeting'.  

~~~
# Warning: The API demos are installed and accessible from:
#
#    http://xxx.xxx.xxx.xxx
#
# and
#
#    http://xxx.xxx.xxx.xxx/demo/demo1.jsp  
#
# These API demos allow anyone to access your server without authentication
# to create/manage meetings and recordings. They are for testing purposes only.
# If you are running a production system, remove them by running:
#
#    sudo apt-get purge ovc-demo  
~~~

When you open the URL, you should see a login to join the meeting `Demo Meeting`.

![ovc-install.sh](images/html5-join.png?raw=true "HTML5 Page")

Enter your name and click Join.  The OvnConference client should then load in your browser and prompt you to join the audio.  

![ovc-install.sh](images/html5.png?raw=true "HTML5 Client")

Click the '[x]' to skip joining the audio.  Why?  With the above command, the OvnConference server is configured to only use an IP address (no security) and, as such, the browser will block access to the webcam and microphone.

For a production setup of OvnConference, the server needs to serve web pages using transport level security (TLS).  In other words, you can only access this server via `HTTP` (unencrypted) and not `HTTPS` (encrypted), as the server currently lacks a secure socket level (SSL) certificate configured for the server's hostname.

Without TLS/SSL support, the browser will not allow access to the user's webam or microphone via the builtin real-time communications (WebRTC) libraries.

`ovc-install.sh` can automatically request a TLS/SSL certificate (from Let's Encrypt) and configure the OvnConference server to use that certificate.  The following sections show you how.


## Getting ready
Before running `ovc-install.sh`, we _strongly_ recommend that you:

  * Read through all the documentation in this page
  * Ensure that your server meets the [minimal server requirements](http://docs.ovnconference.org/install/install.html#minimum-server-requirements)
  * Configure a fully qualified domain name (FQDN), such as `ovc.example.com`, that resolves to the external IP address of the server.

To setup a FQDN, you need to purchase a domain name from a domain name system (DNS) provider, such as [GoDaddy](https://godaddy.com) or [Network Solutions](https://networksolutions.com).  Once purchased, you'll use the web tools provided by the DNS provider to create an `A Record` that resolves to the public IP address of your OvnConference server.  (Check the DNS provider's documentation for details on how to setup the `A Record`.)

With a FQDN domain name place, you can then pass a few additional parameters to `ovc-install.sh` to have it:

  * Request and install a 4096 bit TLS/SSL certificate from Let's Encrypt (we love Let's Encrypt), and (optionally)
  * Install and configure [Greenlight](http://docs.ovnconference.org/greenlight/gl-overview.html) to provide a simple front-end for users to enable them to setup rooms, hold online sessions, and manage recordings.  (Greenlight also lets you, the administrator, manage user accounts within Greenlight).

Once the OvnConference server is configured with an TLS/SSL certificate, your users can use FireFox and Chrome (recommended browsers) to access and share their audio, video, and screen in a OvnConference session via WebRTC.

The full source code for `ovc-install.sh` is [here](https://github.com/ovnconference/ovc-install).  To make it easy for anyone to run the script with a single command, we host the latest version of the script at [https://ubuntu.ovnconference.org/ovc-install.sh](https://ubuntu.ovnconference.org/ovc-install.sh).


### Server choices

There are many hosting companies that can provide you virtual and dedicated servers to run OvnConference.  We list a few popular choices below.  Note: We are not making any recommendations here, just listing some of the more popular choices.

For quick setup, [Digital Ocean](https://www.digitalocean.com/) offers both virtual servers with Ubuntu 16.04 64-bit and a single public IP address (no firewall).  [Hetzner](https://hetzner.cloud/) offers dedicated servers with single IP address.

Other popular choices, such as [ScaleWay](https://www.scaleway.com/) (choose either Bare Metal or Pro servers) and [Google Compute Engine](https://cloud.google.com/compute/), offer servers that are setup behind network address translation (NAT).  That is, they have both an internal and external IP address.  When installing on these servers, the `ovc-install.sh` will detect the internal/external addresses and configure OvnConference accordingly.  

Another popular choice is [Amazon Elastic Compute Cloud](https://aws.amazon.com/ec2).  We recommend a `c5.xlarge` (or larger) instance.  All EC2 servers are, by default, is behind a firewall (which Amazon calls a `security group`).  You will need to manually configure the security group before installing OvnConference on EC2 and, in a similar manner, on Azure and Google Compute Engine (GCE).  (See screen shots in next section.)

Finally, if `ovc-install.sh` is unable to configure your server behind NAT, we recommend going through the [step-by-step](http://docs.ovnconference.org/2.2/install.html) for installing OvnConference.  (Going through the steps is also a good way to understand more about how OvnConference works).


### Configuring the firewall

If you want to install OvnConference on a server behind a firewall, such an Amazon's EC2 security group, you first need to configure the firewall to forward incoming traffic on the following ports:

  * TCP/IP port 22 (for SSH)
  * TCP/IP ports 80/443 (for HTTP/HTTPS)
  * UDP ports in the range 16384 - 32768 (for FreeSWITCH/HTML5 client RTP streams)

If you are using EC2, you should also assign your server an [Elastic IP address](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html) to prevent it from getting a new IP address on reboot.

On Microsot Azure, when you create an instance you need to add the following inbound port rules to enable incoming connections on ports 80, 443, and UDP port range 16384-32768:

![Azure Cloud ](images/azure-firewall.png?raw=true "Azure 80, 443, and UDP 16384-32768")

On Google Compute Engine, when you create an instance you need to enable traffic on port 80 and 443.

![Google Compute Engine 80-443](images/gce-80-443.png?raw=true "GCE 80 and 443")

After the instance is created, you need to add a firewall rule to allow incoming UDP traffic on the port range 16384-32768.

![Google Compute Engine Firewall](images/gce-firewall.png?raw=true "GCE Firewall")

### Installation Videos

Using Digital Ocean as an example, we put together this video to get you going quickly: [Using ovc-install.sh to setup OvnConference on Digital Ocean](https://youtu.be/D1iYEwxzk0M).

Using Amazon EC2, see [Install using ovc-install.sh on EC2](https://youtu.be/-E9WIrH_yTs).

# Command options

You can get help by passing the `-h` option.

~~~
$ wget -qO- https://ubuntu.ovnconference.org/ovc-install.sh | bash -s -- -h
Installer script for setting up a OvnConference 2.2 server.

This script also supports installation of a separate coturn (TURN) server on a separate server.

USAGE:
    ovc-install.sh [OPTIONS]

OPTIONS (install OvnConference):

  -v <version>           Install given version of OvnConference (e.g. 'xenial-220') (required)

  -s <hostname>          Configure server with <hostname>
  -e <email>             Email for Let's Encrypt certbot
  -x                     Use Let's Encrypt certbot with manual dns challenges
  -a                     Install OVC API demos
  -g                     Install GreenLight

  -c <hostname>:<secret> Configure with coturn server at <hostname> using <secret>

  -p <host>              Use apt-get proxy at <host>

  -r <host>              Use alternative apt repository (such as packages-eu.ovnconference.org)
  -d                     Skip SSL certificates request (use provided certificates from mounted volume)

  -h                     Print help

OPTIONS (install coturn):

  -c <hostname>:<secret> Configure coturn with <hostname> and <secret> (required)
  -e <email>             Email for Let's Encrypt certbot (required)


EXAMPLES

Setup a OvnConference server

    ./ovc-install.sh -v xenial-220
    ./ovc-install.sh -v xenial-220 -s ovc.example.com -e info@example.com
    ./ovc-install.sh -v xenial-220 -s ovc.example.com -e info@example.com -g
    ./ovc-install.sh -v xenial-220 -s ovc.example.com -e info@example.com -g -c turn.example.com:1234324

Setup a coturn server

    ./ovc-install.sh -c turn.example.com:1234324 -e info@example.com

SUPPORT:
     Source: https://github.com/ovnconference/ovc-install
   Community: https://ovnconference.org/support

~~~

## Install and configure with an IP address only

To install OvnConference 2.2 (no hostname or TLS/SSL certificate):

~~~
wget -qO- https://ubuntu.ovnconference.org/ovc-install.sh | bash -s -- -v xenial-220
~~~

That's it.  The installation should finish in about 15 minutes (depending on the server's internet connection) with the following message:

~~~
** Potential problems described below **

......
# Warning: The API demos are installed and accessible from:
#
#    http://xxx.xxx.xxx.xxx/demo/demo1.jsp
#
# These API demos allow anyone to access your server without authentication
# to create/manage meetings and recordings. They are for testing purposes only.
# If you are running a production system, remove them by running:
#
#    sudo apt-get purge ovc-demo
~~~

The script also installs the `ovc-demo` package so you can immediately test out the install.  If you want to remove the API demos, use the command

~~~
sudo apt-get purge ovc-demo
~~~

If you want to use this server with a third-party integration, such as Moodle, you can get the OvnConference server's hostname and shared secret with the command `sudo ovc-conf --secret`.

~~~
# ovc-conf --secret

       URL: http://xxx.xxx.xxx.xxx/ovnconference/
    Secret: yyy

      Link to the API-Mate:
      http://mconf.github.io/api-mate/#server=http://xxx.xxx.xxx.xxx/ovnconference/&sharedSecret=yyy
~~~

Since this default use of `ovc-install.sh` does not configure a SSL/TLS certificate, while you can login to the server, you won't be able to share audio/video as WebRTC requires a SSL/TLS certificate.

## Install with SSL/TLS

Before `ovc-install.sh` can install a SSL/TLS certificate, you will need to provide two pieces of information:
   * A fully qualified domain name (FQDN), such as `ovc.example.com`, that resolves to the public IP address of your server
   * An e-mail address

When you have setup the FQDN, check that it correctly resolves to the external IP address of the server using the `dig` command.

~~~
dig ovc.example.com @8.8.8.8
~~~

Note: we're using `ovc.example.com` as an example hostname. You would substitute your real hostname for the check (and for the commands below).

With just these two pieces of information -- FQDN and e-mail address -- you can use `ovc-install.sh` to automate the configuration of the OvnConference server with a TLS/SSL certificate.  For example, to install OvnConference 2.2 with a TLS/SSL certificate from Let's Encrypt using `ovc.example.com` and `info@example.com`, enter the command

~~~
wget -qO- https://ubuntu.ovnconference.org/ovc-install.sh | bash -s -- -v xenial-220 -s ovc.example.com -e info@example.com
~~~

(again, you would substitute `ovc.example.com` and `info@example.com` with your server's FQDN and your e-mail address).

The `ovc-install.sh` script will also install a cron job that automatically renews the Let's Encrypt certificate so it doesn't expire.  Cool.

### Installing in a private network

The default installation is meant to be for servers that are publicly available. This is because Let's Encrypt requires to access nginx in order to automatically validate the FQDN provided.

When installing OvnConference in a private network, it is possible to validate the FQDN manually, by adding the option `-x` to the command line. As in:

~~~
wget -qO- https://ubuntu.ovnconference.org/ovc-install.sh | bash -s -- -v xenial-220 -s ovc.example.com -e info@example.com -x
~~~

Confirm the use of the email account.

```
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Would you be willing to share your email address with the Electronic Frontier
Foundation, a founding partner of the Let's Encrypt project and the non-profit
organization that develops Certbot? We'd like to send you email about our work
encrypting the web, EFF news, campaigns, and ways to support digital freedom.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o:
```

Confirm the use of the IP address
```
Are you OK with your IP being logged?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o:
```

A challenge will be generated and shown in the console.

```
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please deploy a DNS TXT record under the name
_acme-challenge.ovc.example.com with the following value:

0bIA-3-RqbRo2EfbYTkuKk7xq2mzszUgVlr6l1OWjW8

Before continuing, verify the record is deployed.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Press Enter to Continue
```

Before hitting Eneter, create a TXT record in the DNS with the challenge that was generated.

```
_acme-challenge.ovc.example.com.  TXT   "0bIA-3-RqbRo2EfbYTkuKk7xq2mzszUgVlr6l1OWjW8"   60
```

The downside of this is that because Let's Encrypt SSL certificates expire after 90 days, it will be necessary to manually update the certificates. In that case an email is sent a few days before the expiration and the next command has to be executed through the console.

```
certbot --email info@example.com --agree-tos -d ovc.example.com --deploy-hook 'systemctl restart nginx' --no-bootstrap --manual-public-ip-logging-ok --manual --preferred-challenges dns --server https://acme-v02.api.letsencrypt.org/directory certonly
```


## Install API Demos

You can install the API demos by adding the `-a` option.

~~~
wget -qO- https://ubuntu.ovnconference.org/ovc-install.sh | bash -s -- -v xenial-220 -s ovc.example.com -e info@example.com -a
~~~

Warning: These API demos allow anyone to access your server without authentication to create/manage meetings and recordings. They are for testing purposes only.  Once you are finished testing, you can remove the API demos with `sudo apt-get purge ovc-demos`.


## Install Greenlight

[Greenlight](https://docs.ovnconference.org/greenlight/gl-overview.html) is a simple front-end for OvnConference written in Ruby on Rails.  It lets users create accounts, have permanent rooms, and manage their recordings.  It also lets you, as the administrator, manage the user accounts (such as approve or deny users).

You can install [Greenlight](http://docs.ovnconference.org/install/green-light.html) by adding the `-g` option.

~~~
wget -qO- https://ubuntu.ovnconference.org/ovc-install.sh | bash -s -- -v xenial-220 -s ovc.example.com -e info@example.com -g
~~~

Once Greenlight is installed, it redirects the default home page to Greenlight.  You can also configure GreenLight to use [OAuth2 authentication](http://docs.ovnconference.org/greenlight/gl-customize.html).

To launch Greenlight, simply open the URL of your server, such as `https://ovc.example.com/`.  You should see the Greenlight landing page.

![ovc-install.sh](images/greenlight.png?raw=true "Greenlight")

To setup an administrator account for Greenlight (so you can approve/deny signups), enter the following commands

~~~
cd greenlight/
docker exec greenlight-v2 bundle exec rake admin:create
~~~

This command will create an admin account and set a default password.  After running this command, login using the given username/password and change the default password. Next, select 'Administrator' and choose 'Organization'.

![ovc-install.sh](images/gl-admin.png?raw=true "Organization")

You can then select 'Site Settings' on the left-hand side and change the Registration Method to 'Approve/Decline'.

![ovc-install.sh](images/gl-approve.png?raw=true "Approve/Decline")

You can now contol who creates accounts on your OvnConference server.  For more information see [Greenlight administration](http://docs.ovnconference.org/greenlight/gl-admin.html).

## Linking /var/ovnconference to another directory

The install script allows you to pass a path which will be used to create a symbolic link with /var/ovnconference

~~~
wget -qO- https://ubuntu.ovnconference.org/ovc-install.sh | bash -s -- -v xenial-220 -m /mnt/test
~~~

This allows users to store the contents of /var/ovnconference, which can get quite large in a seperate volume

## Doing everything with a single command

If you want to setup OvnConference 2.2 with a TLS/SSL certificate and GreenLight, you can do this all with a single command:

~~~
wget -qO- https://ubuntu.ovnconference.org/ovc-install.sh | bash -s -- -v xenial-220 -s ovc.example.com -e info@example.com -g
~~~

Furthermore, you can re-run the same command later to update your server to the latest version of OvnConference 2.2.  We announce updates to OvnConference to the [ovnconference-dev](https://groups.google.com/forum/#!forum/ovnconference-dev) mailing list.


# Install a TURN server

You can use `ovc-install.sh` to automate the steps to [setup a TURN server for OvnConference](http://docs.ovnconference.org/install/install.html#setup-a-turn-server).  
Note: This step is optional, but recommended if your OvnConference server is publically available on the internet and will be accessed by users who may be behind restrictive firewalls.

OvnConference normally requires a wide range of UDP ports to be available for WebRTC communication. In some network restricted sites or development environments, such as those behind NAT or a firewall that restricts outgoing UDP connections, users may be unable to make outgoing UDP connections to your OvnConference server.  

The TURN protocol is designed to allow UDP-based communication flows like WebRTC to bypass NAT or firewalls by having the client connect to the TURN server, and then have the TURN server connect to the destination on their behalf.

You need a separate server (not the OvnConference server) to setup as a TURN server. Specifically you need:

  * An Ubuntu 18.04 server with a public IP address

On the TURN server, you need to have the following ports (in additon port 22) availalbe for OvnConference to connect (port 3478 and 443) and for the coturn to connect to your OvnConference server (49152 - 65535).

| Ports         | Protocol      | Description |
| ------------- | ------------- | ----------- |
| 3478          | TCP/UDP       | coturn listening port |
| 443           | TCP/UDP       | TLS listening port |
| 49152-65535   | UDP           | relay ports range |


We recommend Ubuntu 18.04 as it has a later version of [coturn](https://github.com/coturn/coturn) than Ubuntu 16.04.  Also, this TURN server does not need to be very powerful as it will only relay communications from the OvnConference client to the OvnConference server when necessary.  A dual core server on Digital Ocean, for example, which offers servers with public IP addresses, is a good choice.

Next, to configure the TURN server you need:

  * A fully qualified domain name (FQDN) with a DNS entry that resolves to the external public IP address of the TURN server
  * An e-mail address for Let's Encrypt
  * A secret key (it can be an 8 to 16 character random string that you create).

With the above information, you can setup a TURN server for OvnConference using `ovc-install.sh` as follows

~~~
wget -qO- https://ubuntu.ovnconference.org/ovc-install.sh | bash -s -- -c <FQDN>:<SECRET> -e <EMAIL>
~~~

Note, we've omitted the `-v` option, which causes `ovc-install.sh` to just install and configure coturn.  For example, using `turn.example.com` as the FQDN, `1234abcd` as the shared secret, and `info@example.com` as the email address, you can setup a TURN server for OvnConference using the command

~~~
wget -qO- https://ubuntu.ovnconference.org/ovc-install.sh | bash -s -- -c turn.example.com:1234abcd -e info@example.com
~~~

`ovc-install.sh` uses Let's Encrypt to configure coturn to use a SSL certificate.  With a SSL certificate in place, coturn can relay access to your OvnConference server via TCP/IP on port 443.  This means if a user is behind a restrictive firewall that blocks all outgoing UDP connections, the TURN server can accept connections from the user via TCP/IP on port 443 and relay the data to your OvnConference server via UDP.

With the TURN server in place, you can configure your OvnConference server to use the TURN server by running the `ovc-install.sh` command again and adding the same `-c <FQDN>:<SECRET>`.  For example,

~~~
wget -qO- https://ubuntu.ovnconference.org/ovc-install.sh | bash -s -- -v xenial-220 -s ovc.example.com -e info@example.com -g -c turn.example.com:1234abcd
~~~

You can re-use a single TURN server for multiple OvnConference installations.

# Next Steps

If you intend to use this server for production you should uninstall the API demos using the command

~~~
apt-get purge ovc-demo
~~~

You can also do a number of [customizations](http://docs.ovnconference.org/2.2/customize.html) to your server as well.

## Troubleshooting

### Packaging server is blocked

We are currently hosting the packaging on a Digital Ocean servlet, but recently the IP range for some Digital Ocean servers has been [blocked in some countries](https://www.digitalocean.com/community/questions/unable-to-reach-digitalocean-server-from-russia).

If you're having troubles installing, try running the `ovc-install.sh` command but change the value

~~~
https://ubuntu.ovnconference.org/ovc-install.sh
~~~

to

~~~
https://packages-eu.ovnconference.org/ovc-install.sh
~~~


### Greenlight not running

If on first install Greenlight gives you a `500 error` when accessing it, you can [restart Greenlight](http://docs.ovnconference.org/install/greenlight-v2.html#if-you-ran-greenlight-using-docker-run).

### Tomcat7 not running

If on the initial install you see

~~~
# Not running:  tomcat7 or grails LibreOffice
~~~

just run `sudo ovc-conf --check` again.  Tomcat7 may take a bit longer to start up and isn't running the first time you run `sudo ovc-conf --check`.

### Getting Help

If you have feedback on the script, or need help using it, please post to the [OvnConference Setup](https://ovnconference.org/support/community/) mailing list with details of the issue (and include related information such as steps to reproduce the error).

If you encounter an error with the script (such as it not completing or throwing an error), please open [GitHub issue](https://github.com/ovnconference/ovc-install/issues) and provide steps to reproduce the issue.


# Limitations

If you are running your OvnConference behind a firewall, such as on EC2, this script will not configure your firewall.  You'll need to [configure the firewall](#configuring-the-firewall) manually.

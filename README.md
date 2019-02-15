# Cisco Meraki CMX API demo app

This application show you how to get started using the [Cisco Meraki](https://meraki.cisco.com/) CMX API. CMX (Connected Mobile Experiences) is Cisco’s location analytics and engagement platform, and it’s integrated into Cisco Meraki wireless products. To learn more about what you can do with CMX, see the [Cisco Meraki CMX site](https://meraki.cisco.com/solutions/cmx).

## Table of contents

- [Installation and requirements](#installation-and-requirements)
- [Running the app](#running-the-app)
- [Sample output code](#sample-output-code)
- [Copyright and license](#copyright-and-license)

## DISCLAIMERS:

1. This code is for sample purposes only. Before running in production,
   you should probably add SSL/TLS support by running this server behind a
   TLS-capable reverse proxy like nginx.

2. You should also test that your server is capable of handling the rate
   of events that will be generated by your networks. A good rule of thumb is
   that your server should be able to process all your network's nodes once per
   minute. So if you have 100 nodes, your server should respond to each request
   within 600 ms. For more than 100 nodes, you will probably need a multithreaded
   web app.

## Installation and requirements


### Installation

- Create a new directory for this project.
- Clone the repo into this directory by running `git clone git@github.com:meraki/cmx-api-app.git` (this will clone the project into the subdirectory `cmx-api-app`).
- Alternatively, [download the ZIP file](https://github.com/meraki/cmx-api-app/archive/master.zip) and unzip it into your project directory.

### Software requirements:

- Ensure you have Ruby 1.9 installed. If you don’t, consider using [RVM](https://rvm.io) to install and manage your Ruby versions.

#### Gems:

- sinatra (if you don’t have it, run `gem install sinatra` via the command line in your project directory.)
- data_mapper
- dm-sqlite-adapter

### Network infrastructure requirements:

- The app requires using one or more [Cisco Meraki MR wireless access points](https://meraki.cisco.com/products/wireless) (APs).
- A valid Enterprise license is required for each Meraki AP.
- Note: this app does not work with other Cisco APs or non-Cisco APs.

## Elastic Beanstalk deployment

### AWS SDK Installation
You need to have the latest/later Ruby version than 1.9 to use, apparently, the aws-sdk, specifically with 1.9.3 the ```gem install aws-sdk``` command was just hanging... 
1. ```rvm install "ruby-2.6.1"``` if you don't already have one
2. ```rvm use 2.6```
3. ```gem install aws-sdk```
4. (Don't forget to change back to 1.9 for running the server locally.)

### Deployment

1. In the aws console -> ElasticBeanstalk create a new application (APP_NAME) 
1. ```awsfed``` (or other AWS SDK credential)
2. ```eb init --profile=federate```
    1. select APP_NAME app
    2. make sure you select Ruby 1.9.3
3. ```eb create APP_NAME-dev --profile=federate --vpc.id VPC --vpc.securitygroups SECURITY_GROUPS --vpc.elbsubnets LOAD_BALANCER_SUBNETS --vpc.ec2subnets EC2_SUBNETS --vpc.elbpublic --envvars SECRET=thesecret,VALIDATOR=thevalidatorcode,PORT=80```

## Running the app
Let’s say you plan to run this app on a server you control called pushapi.myserver.com.

1. Go to the Cisco Meraki dashboard and configure the CMX Location Push API (find it under Organization > Settings) with the url `http://pushapi.myserver.com:4567/events`
2. Choose a secret and enter it into the dashboard.
3. Make note of the validation code that dashboard provides.
4. Pass the secret and validation code to this server when you start it:

	```sample_location_server.rb <secret> <validator>```
5. You can change the bind interface (default: 0.0.0.0) and port (default: 4567)
using Sinatra's -o and -p option flags:

	```sample_location_server.rb -o <interface> -p <port> <secret> <validator>```
6. Click the “Validate server” button in CMX Location Push API configuration in
the dashboard. Meraki cloud servers will perform a GET to your server, and you will
see a log message like this:

	```[26/Mar/2014 11:52:09] "GET /events HTTP/1.1" 200 6 0.0024```

	If you do not see such a log message, check your firewall and make sure
you’re allowing connections to port 4567. You can confirm that the server
is receiving connections on the port using

  ```telnet pushapi.myserver.com 4567```

7. Once the Meraki cloud has confirmed that the URL you provided returns the expected
validation code, it will begin posting events to your URL. For example, when
a client probes one of your access points, you’ll see a log message like
this:

  ```[2014-03-26T11:51:57.920806 #25266]  INFO -- : AP 11:22:33:44:55:66 on ["5th Floor"]: {"ipv4"=>"123.45.67.89", "location"=>{"lat"=>37.77050089978862, "lng"=>-122.38686903158863,"unc"=>11.39537928078731}, "seenTime"=>"2014-05-15T15:48:14Z", "ssid"=>"Cisco WiFi","os"=>"Linux", "clientMac"=>"aa:bb:cc:dd:ee:ff","seenEpoch"=>1400168894, "rssi"=>16, "ipv6"=>nil, "manufacturer"=>"Meraki"}```

8. After your first client pushes start arriving (this may take a minute or two),
you can get a JSON blob describing the last client probe (where {mac} is the client mac address): `pushapi.myserver.com:4567/clients/{mac}`

9. You can also view the sample frontend at: `http://pushapi.myserver.com:4567/`. Try connecting your mobile device to your network, and entering your mobile device‘s WiFi MAC in the frontend.

## Sample output code

The JSON blob sent by Meraki servers to your app is formatted as follows:

```
{
  "apMac":"00:18:0a:79:08:60",
  "apFloors":["500 TF 4th"],
  "observations":[{
    "clientMac":"00:11:22:33:44:55:66",
    "probeEpoch":1388577600,
    "probeTime":"2014-01-01T12:00:00Z",
    "rssi":23,
    "ssid":"SSID 1",
    "manufacturer":"Meraki",
    "os":"Linux",
    "location":{
      "lat":37.77057805947924,
      "lng":-122.38765965945927,
      "unc":15.13174349529074
    }
  }]
}
```

A specific client device’s details can be retrieved, for example:

`http://pushapi.myserver.com:4567/clients/34:23:ba:a6:75:70` 

may return

```
{
  "id":65,
  "mac":"34:23:ba:a6:75:70",
  "seenAt":"Fri Apr 18 00:01:41.479 UTC 2014",
  "lat":37.77059042088197,"lng":-122.38703445525945
}
```



## Copyright and license

Code and documentation copyright 2013-2014 Cisco Systems, Inc. Code released under [the MIT license](LICENSE). Documentation released under [Creative Commons](DOCS-LICENSE).

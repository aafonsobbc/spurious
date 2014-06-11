# Spurious

Spurious is a toolset for testing your AWS applications locally. 

It gives you a number of local amazon services including:

* S3
* Dyanmodb
* SQS
* ElastiCache

that are run as docker containers on a [core-os](https://coreos.com/) VM.

## Usage

The following applications are required:

* [Vagrant](http://www.vagrantup.com/)
* [VirtualBox](https://www.virtualbox.org/) or [VMWare](http://www.vmware.com/uk/)

You'll also need to docker cli tools

### OSX

```
brew install docker
export DOCKER_HOST='tcp://0.0.0.0:2375
```

### Installation

```bash
git clone git@github.com:stevenjack/spurious.git
cd spurious
vagrant up
```

Once the box is up you'll have the following endpoints available on your local machine:

* s3 - localhost:4569
* sqs - localhost:4568
* dynamodb - localhost:4570
* elasticache - localhost:11212

## Example

To use the endpoints in your local environment you need to change the endpoints that you're using to communicate with AWS.

### Ruby

Using the ruby SDK you'd set the following before using any of the services:

```ruby
  AWS.config(
    :region              => 'eu-west-1',
    :use_ssl             => false,
    :dynamo_db_endpoint  => "localhost",
    :dynamo_db_port      => 4570,
    :access_key_id       => "access",
    :secret_access_key   => "secret",
    :sqs_endpoint        => 'localhost',
    :sqs_port            => 4568,
    :s3_endpoint         => 'localhost',
    :s3_port             => 4569,
    :s3_force_path_style => true
  )

```

Then you can use the services as normal through the SDK:

```ruby

sqs = AWS::SQS.new
queue = sqs.queues.create('my-queue')

queue.send_message({"test" : "message"})

```

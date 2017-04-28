<a name="top" />

This page refers to instructions on testing the Android Tracker locally and debugging any issues that might crop up.
These instuctions are aimed at version 0.5.0 of the Android Tracker.

For instructions of previous versions:

* [0.4.0][0.4.0]

## Contents

- 1. [Testing Locally](#testing-locally)
- 2. [Common Issues](#issues)

<a name="testing-locally" />

## 1. Testing Locally

To test the Android Tracker locally we use a combination of two softwares: [Mountebank][mountebank] and [Ngrok][ngrok].

Mountebank provides a local webserver which we can apply `imposters` to run on different ports.  An `imposter` in this context is just a mock which, for our purposes, does some light validation to ensure that the event it receives is indeed a valid Snowplow Event.  This is the [imposter][imposter-link] that we use for validating our events.

We then get Ngrok to listen on the same port as the `imposter` port.  This provides two things:

- A clean easy to use Web Interface with details about each event.
  - Found at `http://localhost:4040/`
- An endpoint that you can send to from anywhere.

When you visit the Ngrok Web Interface you will see the tunnel URL that you can use to send events to.

To actually set this up you will need to follow these steps:

```bash
 host$ git clone https://github.com/snowplow/snowplow-android-tracker.git
 host$ cd snowplow-android-tracker
 host$ vagrant up && vagrant ssh
guest$ cd /vagrant
guest$ chmod +x ./testing/setup.bash
guest$ ./testing/setup.bash
```

The `setup.bash` script starts Mountebank and starts the imposter we want and then starts Ngrok listening on the imposter port.

Once you have it running you can supply the tunnel URL for all logging of events until you are ready to switch to a collector!

[Back to top](#top)

<a name="issues" />

## 2. Common Issues

This section will detail how to handle common problems with running the local testing setup.

[Back to top](#top)

<a name="port-conflicts" />

### 2.1 Port Conflicts

If you are already using ports `2525` or `4040` your vagrant up will fail.  Easiest way to resolve this is to stop any services running on these ports and then attempt to vagrant up again.

However if this is not an option you will need to edit the `Vagrantfile` in the root of the project:

```
config.vm.network "forwarded_port", guest: 4040, host: 4040 ## Change the host to something free
config.vm.network "forwarded_port", guest: 2525, host: 2525 ## Change the host to something free
```

[Back to top](#top)

<a name="events-failing" />

### 2.2 Sending failures

If you have successfully started Ngrok and you have an endpoint but your events are not getting there chances are that Mountebank as failed to start properly.

To resolve:

```bash
guest$ curl -X DELETE http://localhost:2525/imposters/4545
guest$ curl -i -X POST -H 'Content-Type: application/json' -d@/vagrant/testing/imposter.json http://localhost:2525/imposters
```

This will delete the imposter and create a new one in its place.

If you you cannot get to the Ngrok Web Interface at all the best way to resolve is the following:

```bash
host$ cd snowplow-android-tracker
host$ vagrant halt
host$ vagrant up && vagrant ssh
guest$ cd /vagrant
guest$ ./testing/setup.bash
```

This simply stops the VM and restarts it.

[Back to top](#top)

[imposter-link]: https://raw.githubusercontent.com/snowplow/snowplow-android-tracker/0.5.0/testing/imposter.json
[mountebank]: http://www.mbtest.org/
[ngrok]: https://ngrok.com/
[0.4.0]: https://github.com/snowplow/snowplow/wiki/Android-Testing-locally-and-Debugging-0.4.0

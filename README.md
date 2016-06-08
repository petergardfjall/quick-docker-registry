## quick-docker-registry
`quick-docker-registry` is a simple-to-setup Docker registry intended for 
development purposes. For example, to run a registry that is local to your
development environment.

The following instructions describes how to set up and use it.


## Start
Optionally specify the IP address that the hosting machine can be
reached on (by docker engine clients) and run `start`:

    export HOST_IP=<IP address reachable by clients>
    ./start
	
This starts a locally running docker registry on port `5000` with:

- a data directory under `./data`, storing all pushed images
- a self-signed certificate under `./certs`. The certificate will be
  issued for `${HOST_IP}.xip.io` (read about [xip.io](http://xip.io/) for 
  details), where `${HOST_IP}` can either be set explicitly before calling 
  `start` or left blank, in which case, the IP address of a local 
  network interface will be used. The hostname is output by the `start`
  script.
- a single authorized user (`${USER}` with password `secret`) which is
  controlled in the `./auth/htpasswd` file.
  
   The set of authorized users can be modified by editing the `htpasswd` file.
   To *append* a user `testuser` with password `testpassword` run:
   
        docker run --entrypoint htpasswd registry:2 -Bbn testuser testpassword >> auth/htpasswd
 
   Note that after updating the `htpasswd` file, the registry needs to be
   restarted via 
   
        ./stop && ./start

*Note: if either of the directories `data`, `certs` or `auth` already 
exists, `start` will use it as-is*.



## Use
In case the registry was started with a self-signed certificate under `certs`,
you must set up any clients' docker engines to trust that certificate
as explained [here](https://docs.docker.com/registry/insecure/#using-self-signed-certificates). On Ubuntu, this typically entails the following steps:

1. Add the registry's certificate to your list of trusted certificates:

        sudo cp certs/domain.crt /usr/local/share/ca-certificates/dummy-docker-registry.crt
        sudo update-ca-certificates
		
2. Restart the docker engine daemon:

        sudo service docker restart
		# ... or 
		#sudo systemctl restart docker

After this you will need to log in (*note the `xip.io` ending*):

     docker login https://${HOST_IP}.xip.io:5000
     # ... enter credentials when prompted

This will save your credentials under `~/.docker/config`.

Now you should be able to push a local image to the repository:

    docker pull ubuntu:wily
    docker tag ubuntu:wily ${HOST_IP}.xip.io:5000/my/ubuntu:1.0.0
    docker push ${HOST_IP}.xip.io:5000/my/ubuntu:1.0.0

And pull an image from the repository:

    docker pull ${HOST_IP}.xip.io:5000/my/ubuntu:1.0.0



## Stop
Whenever you feel like stopping the registry, just run

    ./stop



## Purge
To start over fresh (clear images, authentication and certificate), run:

    rm -rf ./data ./auth ./certs



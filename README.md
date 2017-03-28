
Docker Architecture
===================
  * The docker packages are managed in a layered approach hence reducing the footprints of the images with shareable applications.
  Eg: Sharing HTTP server in two webapps.

  * Traditionally Ubuntu was being used as the defacto OS
  Docker suggested OS: Alpine Linux: https://hub.docker.com/_/alpine/ (small 5MB footprint)  //docker is being used beyond enterprises in IoT devices etc.

  * Containers are thin writeable layers (provide by Docker Engine) with an image running inside. When containers are launched it mounts the FS+layers defined in the image.

  * Images are always RO (write locked).

  *  /var/lib/docker <-- is where the containers are persisted.

Create Docker Images
====================
    Use of Docker Files -- text file/plain text
      format:
        FROM -- base image (generally the OS)
        RUN -- build time instruction (not run time)

    Build:
      docker build -t "image-name:v1" .   ("image:version" Dockerfile_location)

    Build Cache:
      -- Used when additional instruction are being added to a Dockerfile.
      -- Build cache is broken when we EDIT existing instructions in a Dockerfile.

      Build context = directory with the Dockerfile


Docker Run
===========
    Containers:
      sudo docker ps
      sudo docker ps -a (also list not running containers)

    Images:
      docker images
      docker pull Ubuntu
      docker pull Ubuntu:14.04 (versioned)
      docker history ubuntu  (view history of images)
      eg: docker history ubuntu
          IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
          0ef2e08ed3fa        2 days ago          /bin/sh -c #(nop)  CMD ["/bin/bash"]            0 B                 
          <missing>           2 days ago          /bin/sh -c mkdir -p /run/systemd && echo '...   7 B                 
          <missing>           2 days ago          /bin/sh -c sed -i 's/^#\s*\(deb.*universe\...   1.9 kB              
          <missing>           2 days ago          /bin/sh -c rm -rf /var/lib/apt/lists/*          0 B                 
          <missing>           2 days ago          /bin/sh -c set -xe   && echo '#!/bin/sh' >...   745 B               
          <missing>           2 days ago          /bin/sh -c #(nop) ADD file:efb254bc677d66d...   130 MB  

    Lifecycle -->
      Run = Create + Start
        Running
          <stop><kill>
            Exited
              <rm>
                Removed

    Docker Run:
      D/C need to be run as daemons -- with '-d' option
        docker run -it ubuntu /bin/bash (lands in bash shell while running in i=interactive)
        docker run -d fedora sleep 1000 (running as a daemon)

    Container runs as long as the application is running.
    Containers host name is same as the Container ID (default)

      docker logs <container-id> or <name>
      docker kill <container-id>

    Even after a container is killed, we can still access the logs. Logs are stored in the host m/c and not inside the container.   

      docker run -d --name jagjit-singh-1 ubuntu echo JAGJIT_SINGH_LIVES_ON
          bb0f7e9f91b72a653d170a387ffba2505a3facacc9c305675085407617cff711
      docker ps -a
          CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS                          PORTS               NAMES
          bb0f7e9f91b7        ubuntu              "echo JAGJIT_SINGH..."   3 seconds ago        Exited (0) 2 seconds ago                            jagjit-singh-1
          42cc59e885a7        ubuntu              "/bin/bash"              About a minute ago   Exited (0) About a minute ago                       jagjit-singh
      docker logs jagjit-singh-1
          JAGJIT_SINGH_LIVES_ON

    Docker image can be run without a command
      docker run -d httpd (no command here-- the author needs to define a implicit command)

    *Case Study*
      Start httpd as a background service

      Dockerfile:   
        RUN apt-get install apache2 -y
        <DO not run the httpd service using instruction in the Dockerfile>

      Run:
        docker run -d apache2:v1 apache2ctl -D FOREGROUND
        9c1a8d702692d057260a912285271216902048fe50226634dc4d90d707b5c012

      Check:
        docker ps
          CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS               NAMES
          9c1a8d702692        apache2:v1          "apache2ctl -D FOR..."   About a minute ago   Up About a minute                       lucid_bell

     Port Configurations: //expose a service to outside the container
       Right side port: Container's
       Left side port: Host's

     * Uses NAT-ing and docker engine is responsible to add IP routes on the host.

     Dockerfile -- right side port defn.
      RUN apt-get update
      RUN apt-get install apache2 -y
      EXPOSE 80/tcp

    Run --  docker run -d -p 8080:80 apache2:v2 apache2ctl -D FOREGROUND
    Check -- netstat -nltp      
      Active Internet connections (only servers)
      Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
      tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      996/sshd
      tcp6       0      0 :::8080                 :::*                    LISTEN      4901/docker-proxy
      tcp6       0      0 :::80                   :::*                    LISTEN      3237/apache2
      tcp6       0      0 :::22                   :::*                    LISTEN      996/ssh

    Add CMD instruction to Dockerfile to avoid entering the command in the 'docker run ..' operation
      Dockerfile:
        CMD apache2ctl -D FOREGROUND

    Dockerfile COPY instruction:
          COPY <source> <destination>


    Docker instruction
      ENV - set environment variables as part of the Dockerfile on the Container
      Another way to inject an environment variable is the option "-e"
        docker run -d -e MAGIC=CISCO

      ENTRYPOINT - Defines a command. Can't be overridden at the run time //docker run <..> command
                  //In case defined in docker run, the command is passed as an arguement to ENTRYPOINT.

    CMD instruction styles:
      CMD apache2ctl -D FOREGROUND  // freeflowing command style    -- runs process child of /bin/sh
      CMD ["apache2ctl", "-D", "FOREGROUND"]   // json array style  -- runs independently



Administration
==============
    docker stats

    docker ps
      CONTAINER ID        IMAGE               COMMAND              CREATED              STATUS              PORTS               NAMES
      c629626e483a        httpd               "httpd-foreground"   About a minute ago   Up About a minute   80/tcp              vigilant_mclean

    docker stats
      CONTAINER           CPU %               MEM USAGE / LIMIT       MEM %               NET I/O             BLOCK I/O           PIDS
      c629626e483a        0.28%               5.973 MiB / 1.952 GiB   0.30%               782 B / 788 B       0 B / 0 B           82

    docker top <container-id>  //list all the processes within  

    PID=1 in Linux is the Parent of boot process. In a container it the service's PID.

    docker attach //attaches to PID 1 inside the container. In real world this may not be a PID of container's shell

    Login into a Container shell:
      docker exec
        docker exec -it 9c1a8d702692 /bin/bash
        root@9c1a8d702692:/# ps -waux
        USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
        root         1  0.0  0.0   4440   700 ?        Ss   05:31   0:00 /bin/sh /usr/sbin/apache2ctl -D FOREGROUND
        root        13  0.0  0.3  71300  3300 ?        S    05:31   0:00 /usr/sbin/apache2 -D FOREGROUND
        www-data    14  0.0  0.4 360464  4260 ?        Sl   05:31   0:00 /usr/sbin/apache2 -D FOREGROUND
        www-data    15  0.0  0.4 360464  4260 ?        Sl   05:31   0:00 /usr/sbin/apache2 -D FOREGROUND
        root        70  0.0  0.1  18168  1844 ?        Ss   05:33   0:00 /bin/bash
        root        83  0.0  0.1  15560  1132 ?        R+   05:34   0:00 ps -waux


      Inject a command in a running container:
        docker exec -d 9c1a8d702692 sleep 1000
        root@9c1a8d702692:/# ps -waux
        USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
        root         1  0.0  0.0   4440   700 ?        Ss   05:31   0:00 /bin/sh /usr/sbin/apache2ctl -D FOREGROUND
        root        13  0.0  0.3  71300  3300 ?        S    05:31   0:00 /usr/sbin/apache2 -D FOREGROUND
        www-data    14  0.0  0.4 360464  4260 ?        Sl   05:31   0:00 /usr/sbin/apache2 -D FOREGROUND
        www-data    15  0.0  0.4 360464  4260 ?        Sl   05:31   0:00 /usr/sbin/apache2 -D FOREGROUND
        root       101  0.0  0.0   4340   360 ?        Ss   05:40   0:00 sleep 1000
        root       105  0.0  0.1  18168  1856 ?        Ss   05:40   0:00 /bin/bash
        root       118  0.0  0.1  15560  1132 ?        R+   05:40   0:00 ps -waux

    Docker Registries
      Public: hub.docker.com

    Docker stop
      To gracefully kill containers



Networking
==========
    Container ports can be assigned explicitly or we may let Docker randomly assign one to the process.
      P (uppercase) - random
      p (lowercase) - explicit

    List the network bridges
      host$ brctl show
        bridge name	bridge id		STP enabled	interfaces
        docker0		8000.024209fa8aab	no		

    Network Bridge is installed on the host machine. The docker instance will share the network resources using the bridge.    

    ubuntu@ip-172-31-35-84:~$ sudo docker run --name -p 80  myweb -d  httpd
    ubuntu@ip-172-31-35-84:/$ sudo docker ps
      CONTAINER ID        IMAGE               COMMAND              CREATED             STATUS              PORTS                   NAMES
      efcfd81328e6        httpd               "httpd-foreground"   15 seconds ago      Up 15 seconds       0.0.0.0:32768->80/tcp   myweb

      ubuntu@ip-172-31-35-84:/$ sudo docker port efcfd81328e6
      80/tcp -> 0.0.0.0:32768


    Linking two containers
    ======================
      Start container 1:
        ubuntu@ip-172-31-35-84:~$ sudo docker run --name myweb -p 80  -d  httpd

      Start Container 2 while linked to Container 1:
      docker run --name myweb1 -p 80 --link myweb -d httpd  
      //this will create a entry in the etc/host with the name of the instance
      //in docker automation scripts we should use these hostnames

      ubuntu@ip-172-31-35-84:~$ sudo docker run --name myweb1 -p 80 --link myweb -it httpd /bin/bash
      root@75f46da88f58:/usr/local/apache2# ping myweb
      PING myweb (172.17.0.3): 56 data bytes
      64 bytes from 172.17.0.3: icmp_seq=0 ttl=64 time=0.090 ms
      64 bytes from 172.17.0.3: icmp_seq=1 ttl=64 time=0.069 ms
      64 bytes from 172.17.0.3: icmp_seq=2 ttl=64 time=0.071 ms

      root@75f46da88f58:/usr/local/apache2# cat /etc/hosts
        127.0.0.1	localhost
        ::1	localhost ip6-localhost ip6-loopback
        fe00::0	ip6-localnet
        ff00::0	ip6-mcastprefix
        ff02::1	ip6-allnodes
        ff02::2	ip6-allrouters
        172.17.0.3	myweb efcfd81328e6
        172.17.0.2	75f46da88f58
        root@75f46da88f58:/usr/local/apache2#


      Network disabled for a container.
        docker run -d -P --net=none httpd

      Using the host network's namespace //not the docker bridge. Reuses hosts namespace
        docker run -d -P --net=host httpd

      A container may use other container's network namespace.  
        docker run -d -P --net=container:<container-id> httpd  


      Docker Network
      ==============
        * connect container to multiple n/ws (virtual nics)
        * create your own networks

        ubuntu@ip-172-31-35-84:~$ sudo docker network ls
          NETWORK ID          NAME                DRIVER              SCOPE
          e5a1e4c78565        bridge              bridge              local               
          99dc4edf390d        host                host                local               
          054aed04d015        none                null                local               

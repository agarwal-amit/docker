
Docker Architecture
===================
  * The docker packages are managed in a layered approach hence reducing the footprints of the images with shareable applications.
  Eg: Sharing HTTP server in two webapps.

  * Traditionally Ubuntu was being used as the defacto OS
  Docker suggested OS: Alpine Linux: https://hub.docker.com/_/alpine/ (small 5MB footprint)  //docker is being used beyond enterprises in IoT devices etc.

  * Containers are thin writeable layers (provide by Docker Engine) with an image running inside. When containers are launched it mounts the FS+layers defined in the image.

  * Images are always RO (write locked).

  *  /var/lib/docker <-- is where the containers are persisted.

Docker Run
===========
  * Containers:
      sudo docker ps
      sudo docker ps -a (also list not running containers)

  * Images:
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

  * Docker Run:
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


Docker Orchestration
====================

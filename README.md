
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
    D/C need to be run as daemons.


Docker orchestration
====================

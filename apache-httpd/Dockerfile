FROM ubuntu:14.04                               #base image
MAINTAINER Amit Agarwal <myemail@host.dom>      #author
RUN apt-get update                              #must do as first instruction on Ubuntu based containers
RUN apt-get install apache2 -y                  #Pull apache2 process
EXPOSE 80                                       #Define the right-side port - container's
CMD apache2ctl -D FOREGROUND                    #the default command that will be executed when the container is instantiated. Alternate of -- docker run image <command>
COPY                                            #emulates local 'cp' command: copy <src> <dest>

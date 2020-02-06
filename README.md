# sansnom
This repo contains deb packages for configurations and various binaries

## build public deb fonts

$ fpm -s dir -t deb -C /home/martin/Develop/FPM/public-sans/opentype --name public-sans-sansnom --version 0.0.2 --iteration 2 -m martin@sansnom.uk   --description "public sans font" --prefix /usr/share/fonts/opentype/public --license 'open' --url "www.sansnom.io" --deb-no-default-config-files



## backports for debian 10

### first update the sources for the backports

vi the  /etc/apt/sources.list and add the following entry

deb http://ftp.debian.org/debian buster-backports main
deb-src http://ftp.debian.org/debian buster-backports main

### Next complete an apt update and then an upgrade
  
  ```sudo apt update
  sudo apt -t buster-backports upgrade
  sudo apt autoremove
  ```

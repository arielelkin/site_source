---
layout: page
title: "easily sync and stream your entire music collection using docker"
date: 2016-03-28 19:44
comments: true
sharing: true
footer: true
---

# Easily sync and stream your entire music collection using Docker, ownCloud, and Ampache

You like owning your music, but you own so many gigabytes of it that it's hard, if not impossible, to carry around. 

Luckily, you own a server too. 

> _What? So you're saying I should upload my eight terabytes of music up on my server? But dude, I add five gigabytes of music every day. And on a bad day I delete six gigabytes of music. And on a very bad day, I will obsessively edit the cover art, genre, and year recorded of 600 megabytes' worth of files, and the server is too stupid to know the difference._ 

Your life evolves, and your music collection evolves with you. But before you give up and start renting your music from Spotify, consider elevating your obsession for music collection by building your own music synchronisation and streaming service.

## Goal

1. Ensure that our music database at home and the music database on the server are fully and automatically synchronized. (So if a file is added at home, I shouldn't have to worry about uploading it to the server.)

1. Use separate and independent systems for synchronising the music collection and streaming the music to devices. (I want to be able to offer different streaming services and front-ends for the music streaming.)

## Architecture

We will create a directory _on the server_ for storing the music.

We will use ownCloud, an open-source file hosting app to handle the uploading and synchronization of our music collection from our personal computer to our server. We'll run the ownCloud instance inside a Docker container and mount the aforementioned music directory onto it. 

We will use Ampache, an open-source music database browser and streamer as a user-facing front-end to our music database, and for streaming the music. We'll also run the Ampache instance inside a Docker container and mount the music directory into it.

Both containers will thus share the music data volume.



## Requirements

You'll need:

 * Your server
 * Docker


## Steps

1. Create a directory on your server where the music collection will be stored. This directory will be managed by ownCloud's and so will include ownCloud's own configuration files:

```
$ mkdir owncloud_music
```

We need to note down the absolute path of this directory, so `cd` into it and print the working directory:

```
$ cd owncloud_music
$ pwd
/home/your_username/owncloud_music
```

The directory doesn't need to be in your home folder, but it should be somewhere where you're not likely to move it around. For the rest of this tutorial, we'll assume that `/home/your_username/owncloud_music` is the absolute path of your music directory. 

1. Install an ownCloud instance on your server, mount the above directory onto it as a data volume:

```
$ docker run --name audiosync -v /home/your_username/owncloud_music:/var/www/html -p 1996:80 -d owncloud:9
```

Let's break down the above command. 

`docker run` tells docker to pull the ownCloud image and its dependencies, build it, and start the container from that image with a particular set of options that I'll get into more detail below:

`--name audiosync` allows us to name our container in order to be able to refer to it more easily. We could have several owncloud instances running, but this one's dedicated to synchronizing our audio, so let's call it `audiosync`. 

`-v /home/your_username/owncloud_music:/var/www/html` is the money shot. This mounts the music directory we created before into the ownCloud container at `/var/www/html`, which is the directory _within the ownCloud container_ where ownCloud stores its data and local configuration. ownCloud is under the impression of running in its own lonely operating system, yet little does it know, it's writing directly onto our server filesystem. Make sure you pass in the absolute path to the music directory. 

`-p 1996:80` ownCloud listens to incoming web traffic on port 80. But so does Ampache, and so might other things running on your server. (Port 80 is the default port that your server listens to, it's what you see when you access `http://yourdomain.com/`). So our server will listen to traffic on port 1996 and route it to ownCloud's port 80. I recommend using port 1996 because that's the year that the album _Below the Bassline_ by Ernest Ranglin was released. 

`-d` starts the container in detached mode. This runs the container in the background rather than in the foreground. 

`owncloud:9` is the particular version of owncloud we want. This pulls the official ownCloud image from Docker Hub.

Depending on whether or not you've already pulled owncloud images, running this command should give you an output similar to this:

```
Unable to find image 'owncloud:9' locally
9: Pulling from library/owncloud

Digest: sha256:55323ed9980e09e5eb5d21d605145e989c8e92d5fe3db31cfdd047efaf1832a0
Status: Downloaded newer image for owncloud:9
77e25848399e8b3b56f4cb23d5490155a16482f382ac690ae98fb999f8ab4f7e
...
```

When Docker is done building and running, you can check the ownCloud container is correctly running by asking Docker to list the last container you created:

```
$ docker ps -l
```

Next, open your web browser, and go to `http://yourdomain.com:1996`

![](http://i.imgur.com/nza2Lar.png)

Create a username and password for your ownCloud account, and you should be in!

If you get the "You are accessing the server from an untrusted domain." error:

`cd` into the `owncloud_music/config/` directory on your server. Then `sudo` edit the config.php file so that your domain in the `trusted_domains` array does not specify a port:

So change this part of the file:

```
'trusted_domains' => 
  array (
    0 => 'yourdomain.com:1996',
  ),
```

to this:

```
'trusted_domains' => 
  array (
    0 => 'yourdomain.com',
  ),
```

Now, [install the ownCloud desktop client on your personal computer](https://owncloud.org/install/). Add an account:
![](http://i.imgur.com/rUD11j2.png)

Now tell the ownCloud client where your music folder is, make sure you select "Keep local data", and click **Connect...**:
![](http://i.imgur.com/fMJrrxj.png)

"Everything set up", as far as ownCloud is concerned, you can click "Finish".

![](http://i.imgur.com/OgXfJ5O.png)


ownCloud should have now started uploading your music to your server:
![](http://i.imgur.com/NIte8sz.png)

The `Documents` and `Photos` folders are created by default by ownCloud, you can delete them. 

Onto the streamer. But before we can build and start it, we need to find out exactly where our music is inside `my_music`.

You now need root privileges to `cd` into the `owncloud_music` directory. Run `sudo -i` and `cd` into the `owncloud_music` directory. You should be able to find your files here: `owncloud_music/data/your_owncloud_username/files`

Copy that path.

OK, now let's setup the streamer. Run this command:
```
$ docker run --name=ampache -v /owncloud_music/data/your_owncloud_username/files:/media:ro -p 2008:80 -d ampache/ampache
```

And let's break that down:

`docker run --name=ampache` Here we're pulling, building, and running a container with the Ampache, and we're naming it `ampache`. 

`-v /owncloud_music/data/your_owncloud_username/files:/media:ro` This mounts the server's music directory we created before into the ampache container where ampache looks for music files. We add `:ro` option to specify that the mount should be read-only. Remember that the path to the music folder needs to be absolute. 

`-p 2008:80` as above, this routes traffic coming and going to our server's port 2008, into Ampache's port 80. Again, I recommend port 2008 for technical reasons as it's the year that Will Bernard's _Blue Plate Special_ was released, with Stanton Moore on drums, you should listen to it. 

`ampache/ampache` tells Docker to pull and build the official Ampache image. 

Once Ampache has been pulled and built, open your web browser and visit `yourdomain.com:2008`

You'll be greeted with Ampache's installation wizard:

![](http://i.imgur.com/dGEfcZz.png) 

Click "Start Configuration". 

The next page tells you about if your environment has satisfies Ampache's technical prerequisites. Click "Continue".

The next page creates the Ampache database. All you have to do here is:

* Check "Create Database User"
* Set an "Ampache Database User Password" and note it down.

Leave the other default values, and click "Insert Database".

In this next page, make sure that 
MySQL Username and MySQL Password match the ones in the previous step (default username is "ampache", the password should be the one you inputted just before). 

Now click on the File Insight menu, and click on the "Write" button

 insert the password in the MySQL Password field. You can leave the other default options. Finally, click "Create config" 

In the next page, create your Ampache (admin) user account. Pick a username and a password, click "Create Account". Now you can login into Ampache!

![](http://i.imgur.com/mWk5MgC.png)

Ampache now needs you to add a Catalog for it to start building your music collection. Click on the second button from the left, the one next to the door:

![](http://i.imgur.com/TRIZYrb.png)

Now select "Add a Catalog" from the menu below. 

Select "local" as the Catalog Type, and fill in `/media/` as the path to the music. You can then "Add Catalog". Wait for the Catalog to finish building, and your music is ready for streaming! Click on the headphones icon on the top right of the left column, and you can Browse Music. If ownCloud has already uploaded music to your server, they will be there:

![](http://i.imgur.com/pAnbhAr.png)

And... **Happy sync'n'streaming!**

## A note on ownCloud folder structure

For the sake of simplicity, we instructed ownCloud to store both its own configuration files _and_ our music files into the same directory: `owncloud_music`becomes ownCloud's default volume at `/var/www/html`. You could have instead instructed Docker to mount an additional volume into the ownCloud container, where ownCloud could have save its configuration files:

```
$ docker run --name audiosync -v /home/your_username/owncloud_music_files:/var/www/html/data -v /home/your_username/owncloud_config_files:/var/www/html/config -p 1996:80 -d owncloud:9
```

## Troubleshooting

If you think you missed a step and your ownCloud or Ampache container doesn't look like it's been properly configured, the fastest way to fix it is to kill and delete the container, and build it again from scratch. One or both of these commands will do that: 

```
$ docker rm -f ampache
$ docker rm -f audiosync
```

## Feedback

... is always welcome. Has this tutorial been helpful? Is something not clear? Do you know of a more efficient way to setup ownCloud and Ampache with Docker?

 [Drop me a line](mailto:arielelkin@gmail.com) or send a tweet: [@AriVocals](https://twitter.com/arivocals)

## Useful sites

* [Ampache Documentation](https://github.com/ampache/ampache/wiki)
* [Ampache Clients](https://github.com/ampache/ampache/wiki/Clients) (mobile apps, other web interfaces, etc.)
* [ownCloud Documentation](https://doc.owncloud.org/)
* [Docker Documentation: Volumes](https://docs.docker.com/engine/userguide/containers/dockervolumes/)

## Acknowledgements 

Thanks to [Jérôme Petazzoni](http://jpetazzo.github.io/) and [Benjamin Nothdurft](https://twitter.com/dataduke) for helping me understand Docker and helping me get the above Docker + Ampache + ownCloud equation to work! I'm also grateful to [Sam Tuke](https://twitter.com/samtuke) for reviewing the draft. 



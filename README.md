## Supported tags and respective `Dockerfile` links

* [`latest` _(Dockerfile)_](https://github.com/lowfive/nginx-rtmp-docker/blob/master/Dockerfile)

# nginx-rtmp

[**Docker**](https://www.docker.com/) image with [**Nginx**](http://nginx.org/en/) using the [**nginx-rtmp-module**](https://github.com/arut/nginx-rtmp-module) and [**secure_link**](nginx.org/en/docs/http/ngx_http_secure_link_module.html) modules for live multimedia (video) streaming with stream keys.

## Description

This [**Docker**](https://www.docker.com/) image can be used to create an RTMP server for multimedia / video streaming using [**Nginx**](http://nginx.org/en/) and [**nginx-rtmp-module**](https://github.com/arut/nginx-rtmp-module), built from the current latest sources (Nginx 1.11.3 and nginx-rtmp-module 1.1.9).

This was inspired by other similar previous images from [dvdgiessen](https://hub.docker.com/r/dvdgiessen/nginx-rtmp-docker/), [jasonrivers](https://hub.docker.com/r/jasonrivers/nginx-rtmp/), [aevumdecessus](https://hub.docker.com/r/aevumdecessus/docker-nginx-rtmp/) and by an [OBS Studio post](https://obsproject.com/forum/resources/how-to-set-up-your-own-private-rtmp-server-using-nginx.50/).

The main purpose (and test case) to build it was to allow streaming from [**OBS Studio**](https://obsproject.com/) to different clients at the same time.

**GitHub repo**: <https://github.com/lowfive/nginx-rtmp-docker>

**Docker Hub image**: <https://hub.docker.com/r/tiangolo/nginx-rtmp/>

## Details


## How to use

* For the simplest case, just run a container with this image:

```bash
docker run -d -p 1935:1935 --name nginx-rtmp lowfive/nginx-rtmp
```

### Generate stream keys
```bash
echo -n "stream/namehere" | openssl dgst -md5 -binary | openssl enc -base64 | tr '+/' '-_' | tr -d '='
```
Substitute the desired stream name for ```namehere```. Make sure you don't use spaces.

## How to test with OBS Studio and VLC


* Run a container with the command above


* Open [OBS Studio](https://obsproject.com/)
* Click the "Settings" button
* Go to the "Stream" section
* In "Stream Type" select "Custom Streaming Server"
* In the "URL" enter the `rtmp://<ip_of_host>/live` replacing `<ip_of_host>` with the IP of the host in which the container is running. For example: `rtmp://192.168.0.30/stream`
* In the "Stream key" use the desiared stream name along with the "key" value that was generated during the "Generate stream keys" section. For example: `namehere?key=XmVLx2O3wREvzKTH1i53yg`
* Click the "OK" button
* In the section "Sources" click de "Add" button (`+`) and select a source (for example "Display Capture") and configure it as you need
* Click the "Start Streaming" button


* Open a [VLC](http://www.videolan.org/vlc/index.html) player (it also works in Raspberry Pi using `omxplayer`)
* Click in the "Media" menu
* Click in "Open Network Stream"
* Enter the URL from above as `rtmp://<ip_of_host>/stream/<streamname>` replacing `<ip_of_host>` with the IP of the host in which the container is running and the stream name you chose. For example: `rtmp://192.168.0.30/stream/namehere`
* Click "Play"
* Now VLC should start playing whatever you are transmitting from OBS Studio

## Debugging

If something is not working you can check the logs of the container with:

```bash
docker logs nginx-rtmp
```

## Extending

If you need to modify the configurations you can create a file `nginx.conf` and replace the one in this image using a `Dockerfile` that is based on the image, for example:

```Dockerfile
FROM lowfive/nginx-rtmp

COPY nginx.conf /etc/nginx/nginx.conf
```

The current `nginx.conf` contains:

```Nginx
worker_processes auto;
rtmp_auto_push on;
events {}
http {
    server {
        listen 8080;
        server_name localhost;

        location /on_publish {
            secure_link $arg_key;
            secure_link_md5 $arg_app/$arg_name;

            if ($secure_link = "") {
                return 501;
            }

            if ($secure_link = "0") {
                return 502;
            }
            return 200;
        }
    }
}
rtmp {
    server {
        listen 1935;
        listen [::]:1935 ipv6only=on;

        notify_method get;

        application stream {
            live on;
            record off;
            on_publish http://localhost:8080/on_publish;
        }
    }
}
```

You can start from it and modify it as you need. Here's the [documentation related to `nginx-rtmp-module`](https://github.com/arut/nginx-rtmp-module/wiki/Directives).

## Technical details

* This image is built from the same base official images that most of the other official images, as Python, Node, Postgres, Nginx itself, etc. Specifically, [buildpack-deps](https://hub.docker.com/_/buildpack-deps/) which is in turn based on [debian](https://hub.docker.com/_/debian/). So, if you have any other image locally you probably have the base image layers already downloaded.

* It is built from the official sources of **Nginx** and **nginx-rtmp-module** without adding anything else. (Surprisingly, most of the available images that include **nginx-rtmp-module** are made from different sources, old versions or add several other components).

* It has a simple default configuration that should allow you to send one or more streams to it and have several clients receiving multiple copies of those streams simultaneously. (It includes `rtmp_auto_push` and an automatic number of worker processes).

## License

This project is licensed under the terms of the MIT License.

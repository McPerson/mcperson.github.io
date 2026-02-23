---
layout: post
title:  "Self-Host Your Kobo Library with Calibre-Web and Docker"
date:   2026-02-23 00:00:00 -0700
categories: self-hosted media
tags: how-to self-hosted docker media
description: "A step-by-step guide to deploying Calibre-Web with Docker and wirelessly syncing your eBook library to your Kobo eReader."
---

Managing an eBook library is hard. You have to deal with horrible metadata, duplicate files, and worst of all, proprietary file types. Well, I have the solution for you.

With Calibre-Web, you can manage your eBook library, convert files automatically, and even sync your library to your Kobo eReader with the touch of a button. (F to pay respects to all Kindle users 🙏)

Calibre-Web is a web-based interface for managing a Calibre library, making your collection searchable, organized, and accessible from your browser.

This guide will walk you through:
1. Deploying Calibre-Web using Docker Compose
2. Initializing Calibre-Web
3. Syncing your library to Kobo

---

## Pros and Cons
First off lets start off with some pros, cons, and stuff I won't be showing you.

### Pros:
- It’s actually pretty easy.
- You keep the stock Kobo OS.
- Automatic conversion to kepub format.
- No more plugging your Kobo in to transfer files.

### Cons:
- You may temporarily lose access to the Kobo Store unless you enable the proper configuration option.
- This guide does not cover advanced networking or security setups.


### Things not covered by this guide.
- This is going to be a very basic setup, we assume both your server and Kobo are on the same network.
- We aren't going to use a reverse-proxy or even setup HTTPS.
- We aren't going to be exposing this server to the public web. Exposing services to the public internet without HTTPS, authentication hardening, and a reverse proxy is not recommended.

---

## Deploying Calibre-Web using Docker Compose

### Example File Structure
Here is the basic file structure I setup for Calibre-Web. Feel free to change this as you require if you have an existing eBook Library.
```
calibre-web/
├─ docker-compose.yml
├─ data/
│  ├─ metadata.db
├─ books/
│  ├─ William Shakespeare/
│  │  ├─ Hamlet/
│  │  │  ├─ cover.jpg
│  │  │  ├─ Hamlet - William Shakespeare.epub
```

To setup Calibre-Web you'll need a Calibre metadata.db file.
- If you already use Calibre, you can find one in your Calibre library folder.
- If you don't have one you can download an empty one from the [Calibre-Web Github repo here][calibre-web-db].

### Create docker-compose.yaml
Next up is the creation of a docker-compose.yaml file. Just copy and paste this. Change the file volumes on the left if you changed your file structure.

```yaml
name: "calibre-web"
services:
  calibre-web:
    image: lscr.io/linuxserver/calibre-web:latest
    container_name: calibre-web
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC # Set your timezone
      - DOCKER_MODS=linuxserver/mods:universal-calibre #Optional, we want this for conversion.
    volumes:
      - ./data:/calibre-web/config
      - ./books:/books
    ports:
      - "8083:8083"
    restart: unless-stopped
```

You can find more information about the Calibre-Web docker image [here][calibre-web-docker].

Once we have the compose file setup we can start the container.
```
docker compose up -d --force-recreate
```

---

## Initializing Calibre-Web
Once started open your browser and go to: `http://server-ip:8083/`

Use the default admin login
Username: admin
Password: admin123

After your first login you will be asked for the location of your Calibre Database.
Set that to the location of your Calibre database, `metadata.db`.
Select to "Seperate Book Files from Library" and set the location to the `/books` directory.

Your Calibre library is now setup!

⚠️ Remember to change your admin account password.

### Bonus section, container maintenance

To stop the container we just do this:
```
docker compose down
```

To update the application to the latest image available we simply pull the latest image and recreate the container.

```
docker compose pull
docker compose up -d --force-recreate
```

---

## Syncing your library to Kobo
Now for the fun part!
Go to the admin panel and click "Edit Basic Configuration".

Click the "Feature Configuration" option and "Enable Kobo sync".

If you plan on having multiple users, now is a good time to create separate accounts.

Go to your user profile and click "Create/View" under Kobo Sync Token to generate a token.
It will look like this: `api_endpoint=http://server-ip:8083/kobo/rZhyBjF8q50bPDQpacHtJtKt7Qp8fUpH`

Plug your Kobo into your computer and navigate to `.kobo/Kobo/Kobo eReader.conf`
Search for `api_endpoint` and replace it with your

Safely eject your Kobo.

Finally upload a book and add it to a bookshelf. Your bookshelves should appear on your Kobo when you sync.

Thanks for reading this little guide, hopefully this helps you manage your eBook library.

Just a final thanks to the [Calibre-Web][calibre-web] and [Linuxserver.io][calibre-web-docker] maintainers for all of the hard work.
As well as this [Reddit post][reddit-kobo] for the Kobo info.

[calibre-web]: https://github.com/janeczku/calibre-web
[calibre-web-db]: https://github.com/janeczku/calibre-web/tree/master/library
[calibre-web-docker]: https://hub.docker.com/r/linuxserver/calibre-web
[reddit-kobo]: https://www.reddit.com/r/kobo/comments/1nahk6f/got_calibreweb_working_on_my_kobo_in_2025_heres/
